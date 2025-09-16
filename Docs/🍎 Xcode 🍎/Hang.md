>[!question]
>GQ1. 행이란 무엇인가?
>GQ2. 행은 왜 발생하는 것이고, 어떻게 행이 발생했다는 것을 확인할 수 있을까?
>GQ3. 행이 발생하지 않도록 개발하려면 어떻게 해야하지?

## 1. 행 (Hang)이란 무엇인가? 

행이라는 것은 응답이 없는 순간을 의미합니다.  
사진이나 데이터를 화면에 표시하기 위해 서버로부터 불러와 나타날 때 **뷰에서 메인 스레드가 차단되어 사용자의 인터랙션을 받을수 없고, 화면이 멈추는 것처럼 어떤한 이벤트도 실행할수 없을 때** 우리는 행에 걸렸다고 합니다.  

애플에서는 **일정 주기동안 응답이 없는 것을 행**이라고 부릅니다. 
사용자 입장에서는 행이 오래 걸리면 앱이 멈추거나 정상적으로 동작하지 않는다고 느껴지기 때문에 이를 개선하는 것이 중요합니다. 

해결 순서는 찾고(Find) -> 분석(Analyze) -> 해결(Fix) 방식입니다.

>[!행의 임계값 = Acceptable delays are context dependent! (어떤 종류의 지연이 용인가능한지는 상황에 따라 다르다!)]
>
>100ms 미만의 딜레이가 인발적으로 즉각적으로 반응한다고 느껴집니다. 
>**250ms까지도 상황에 다르지만, 문제가 없다고** 느낄수 있습니다. 
>
>이보다 길면 무의식적으로 끊긴다고 느낌이 들지만 대부분 툴에선 기본적으로 **250ms부터 비정상적인 행으로 보아 행을 기록하지만 사실 무시하기 쉽기 때문에 마이크로 행** (Microhang)이라고 부릅니다. 
>**500ms 부턴 확실히 즉각적으로 반응한다고 느껴지지 않기에 정식 행 (Hang)**으로 부릅니다. 
>
>만약 (인터페이스의 UI 요소) 즉각적인 느낌을 원하면 1**00ms 잡는것이 가장 이상적**이라고 합니다. 
>다만 서버의 통신 같은 요청-응답 형태의 상호작용 (Request-Response)인 경우 500ms 동안 추가 피드백이 없어도 가능합니다.


![[행 임계.png]]


## 2. 행을 일으키는 원인을 이해하기 위해 알아야하는 것 : Event handling, rendering loop

행의 발생의 방식을 이해하기 위해 이벤트 처리 및 렌더링 루프에 대해서 알아봐야 합니다. 
UI 엘리먼트가 즉각적으로 반응할수 있도록 하려면 메인 스레드를 UI 이외의 작업으로부터 자유롭게 해주는 것이 중요합니다. 

사용자의 인터랙션에 의해 하드웨어가 운영체제에 전달되고, 운영체제는 프로세스에서 이벤트를 처리하도록 전달합니다. 여기서 이벤트를 처리하는 것은 앱의 메인 스레드의 역할 즉 UI 업데이트를 결정합니다. 

이 UI 업데이트는 개별 UI 레이어를 합성해 다음 프레임을 렌더링하는 별도의 프로세스인 렌더 서버로 전송됩니다. 마지막으로 디스플레이 드라이버는 렌더 서버에서 준비한 비트맵을 선택하고 이에 따라서 화면의 픽셀을 업데이트합니다. 	![[이벤트 처리 및 화면 업데이트.png]]

해당 시간에 다른 이벤트가 들어오면 병렬로 처리될 수 있습니다. 

![[이벤트 병렬 처리.png]]


## 3. 행의 원인

#### 3-1. Busy main thread hang

하나의 메인스레드가 긴 task를 수행하거나 (Too long), 적은 task지만 여러개를 수행한경우 (Too Often)
메인 스레드가 CPU를 많이 사용했기 때문에 Hang이 발생했습니다. (High CPU Usage)

![[Hang_Busy main thread.png]]

이 문제를 해결할 수 있는 방법은 두 가지.
- 지금 당장 앱에서 수행할 필요가 없는 작업을 식별하거나, (Identify work your app doesn’t need to do right now)
- 동일한 작업을 수행하되, 백그라운드 스레드로 활용하는 방법. 그런 다음 작업이 완료되면 메인 스레드에 알리는 방법. (Do the same work, but rely on using a background thread instead of the main thread. Then, when the work is complete, notify the main thread.)

#### 3-2. Blocked main thread hang

다른 스레드나 시스템자원에 의해 막힌 경우
해당 경우는 위의 경우와 다르게 메인 스레드가 CPU를 아주 적게 사용합니다. (Low CPU Usage)

![[Hang_Blocked main thread.png]]


#### 3-3. 행을 바라보는 다른 관점 : Asynchronous/Delayed hang

> 행이 일어나는 원인 (Synchronous/Direct) vs 행이 발생하는 시점 (Asynchronous/Delayed)
![[Hang_Async hang.png]]

- `Synchronous/Direct hang` 
  : 만약 이벤트가 들어왔을 때, 그 이벤트 (원인)를 처리하는데 오랜 시간이 걸리는 것
  
- `Asynchronous/Delayed hang` 
  : 이벤트 (원인)를 빨리 처리하더라도, 메인 스레드에서 일부 작업을 연기했거나 
  다른 메인 스레드를 작업하기로 한 예약된 일이 겹치면, 이전 작업이 끝날 때까지 기다렸다가 이벤트를 처리하게 됨.
  -> **이벤트 직후의 RunLoop 단계** (레이아웃/그리기/애니메이션 커밋, 메인 큐에 쌓인 작업 등)에서 길어져 **나중에** 터짐 (시점만 지연되는 Hang).

Delayed hang이라고 부르는 이유는 이 Hang이 생기는 작업이 비동기 작업과 연관되어있기 때문입니다.
-> 즉, 메인 스레드에 작업을 발생시키는 모든게 원인일 수 있다는 뜻입니다!

a. 메인 큐의 `dispatch_async` 작업 때문. ("dispatch_async"ing work on the main queue)
b. Swift Concurrency task가 비동기로 메인 actor에서 실행되었기 때문. (Swift Concurrency task that runs asynchronously on the main actor)



## 4. 개발 단계별 행을 분석하는 방법

각 개발 단계별로 행을 분석하는 적절한 도구들이 있습니다. 
![[개발단계별 행분석기.png]]

**Development 단계에선 스레드 성능 검사를 통해 앱을 디버깅하는 동안 능동적으로 추적하지 않고도 행을 일으키는 스레드 문제를 알려줍니다.**

또한, **Instruments에서는 행 탐지기를 통해 행을 감지하고 표시**해줍니다.

**Beta때는 Xcode를 사용하지 않거나 추적하지 않을때도 기기에서의 행 탐지기를 통해 실시간 행 알림과 진단**을 해줄 수 있습니다.

**Public release 단계에선 Xcode Reports Organizer를 통해 실제 사용자로부터 집계된 행 정보들의 리포트를 제공**해줍니다.

#### 4-1. Development tooling

Xcode 14이상에서는 스레드 성능검사를 통해 행이 걸리만한 메인 스레드의 Non-UI 작업이나 우선순위가 역전되는 작업을 감지해서 알려줍니다. 
예시로 우선순위가 역적되는 작업을 감지하여 동기화 시도에서 행이 걸리수 있다고 알려 줍니다.

![[우선 순위 역전 작업.png]]

설정 방법은 Thread Performance Checker를 활성화 해주면 됩니다.	![[Thread Performance Checker.png]]

Instrumnet의 [[Time Profiler]]는 콜 스택을 제공하면서 앱의 각 스레드가 시간에 따라 뭘하고 있는지 알 수 있습니다.

#### 4-2. Beta tooling

개발이 완료되고 앱을 TestFlight를 통해 개인들의 기기에 다운하여 테스트를 할때 iOS 16 이상부터는 개발자 설정에서 행 감지 기능을 사용할 수 있습니다.  만약 행이 걸리는 현상이 발생하면 앱 상단에 행이 걸리는 시간이 표시 됩니다.
	
 ![[행 알림 표시.png]]

설정 ➡️ 개발자  ➡️ 행 감지 기능 활성화
Hang Theadhold란 행을 감지하는 기준값을 말합니다. 250ms로 설정하면 250ms보다 낮게 걸리는 행을 무시하고 그 이상만 감지합니다. 

Monitered Apps은 행을 감지하는 앱들입니다. 

Available Hang Logs는 지금까지 감지한 비정상적인 행 항목을 시간 순으로 기록한 목록입니다. 
	
![[행 감지 설정.png]]


#### 4-3. Public release tooling

Xcode 14부터는 Xcode Organizer에서 Hang reports를 사용해 사용자들의 기기로부터 비정상 행진단을 집계해 전달 받을수 있습니다.
	 ![[organizer hang.png]]

물론 앱 분석 공유에동의한 사용자만 행 현상에 대한 메인 스레드 스택의 추적 정보가 제공됩니다. 

다른 방법으로는 [App Store Connect의 API](https://developer.apple.com/videos/play/wwdc2020/10057/)를 이용해 행 리포트 데이터를 검색할 수 있습니다. 
	


## References

- https://green1229.tistory.com/509?category=1022373
- https://green1229.tistory.com/508

- [WWDC23 - Analyze hangs with Instruments](https://developer.apple.com/videos/play/wwdc2023/10248/)
- [WWDC21 - Understand and eliminate hangs from your app](https://developer.apple.com/kr/videos/play/wwdc2021/10258/)
- [WWDC19 - Getting Started with Instruments](https://developer.apple.com/videos/play/wwdc2019/411)
- [WWDC22 - Track down hangs with Xcode and on-device detection](https://developer.apple.com/videos/play/wwdc2022/10082)