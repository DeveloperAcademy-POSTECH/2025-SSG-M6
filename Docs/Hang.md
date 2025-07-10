1. 행이란 무엇인가? 

	행이라는 것은 응답이 없는 순간을 의미합니다.  사진이나 데이터를 화면에 표시하기 위해 서버로부터 불러와 나타날 때 **뷰에서 메인 스레드가 차단되어 사용자의 인터랙션을 받을수 없고, 화면이 멈추는 것처럼 어떤한 이벤트도 실행 할수 없을 때** 우리는 행에 걸렸다고 합니다.  

	애플에서는 일정 주기동안 응답이 없는 것을 행이라고 부른다. 

	사용자 입장에서는 행이 오래 걸리면 앱이 멈추거나 정상적으로 동작하지 않는다고 느껴지기 때문에 이를 개선하는 것이 중요합니다. 해결 순서는 찾고(Find) -> 분석(Analyze) -> 해결(Fix) 방식입니다.


2.  행의 임계값

	100ms 미만의 딜레이가 인발적으로 즉각적으로 반응한다고 느껴집니다. **250ms까지도 상황에 다르지만, 문제가 없다고** 느낄수 있습니다. 이보다 길면 무의식적으로 끊긴다고 느낌이 들지만 대부분 툴에선 기본적으로 **250ms부터 비정상적인 행으로 보아 행을 기록하지만 사실 무시하기 쉽기 때문에 마이크로 행**이라고 부릅니다. **500ms 부턴 확실히 즉각적으로 반응한다고 느껴지지 않기에 행**으로 부릅니다. 

	만약 즉각적인 느낌을 원하면 1**00ms 잡는것이 가장 이상적**이라고 합니다. 다만 서버의 통신 같은 요청과 응답 같은경우 500ms도 괜찮게 보기도 합니다. 
	
	![[행 임계.png]]

4. 이벤트 핸들링과 렌더링 루프

	행의 발생의 방식을 이해하기 위해 이벤트 처리 및 렌더링 루프에 대해서 알아봐야 합니다. UI 엘리먼트가 즉각적으로 반응할수 있도록 하려면 메인 스레드를 UI 이외의 작업으로부터 자유롭게 해주는 것이 중요합니다. 

	사용자의 인터랙션에 의해 하드웨어가 운영체제에 전달되고, 운영체제는 프로세스에서 이벤트를 처리하도록 전달합니다. 여기서 이벤트를 처리하는 것은 앱의 메인 스레드의 역할 즉 UI 업데이트를 결정합니다. 

	이 UI 업데이트는 개별 UI 레이어를 합성해 다음 프레임을 렌더링하는 별도의 프로세스인 렌더 서버로 전송됩니다. 마지막으로 디스플레이 드라이버는 렌더 서버에서 준비한 비트맵을 선택하고 이에 따라서 화면의 픽셀을 업데이트합니다. 
	![[이벤트 처리 및 화면 업데이트.png]]

	해당 시간에 다른 이벤트가 들어오면 병렬로 처리될 수 있습니다. 

	![[이벤트 병렬 처리.png]]


5. 행의 원인

- Busy main thread hang
	하나의 메인스레드가 긴 task를 수행하거나, 적은 task지만 여러개를 수행한경우`

- Blocked main thread hang
	다른 스레드나 시스템자원에 의해 막힌경우


3. 개발 단계별 행을 분석하는 방법

	각 개발 단계별로 행을 분석하는 적절한 도구들이 있습니다. 

	![[개발단계별 행분석기.png]]

	**Development 단계에선 스레드 성능 검사를 통해 앱을 디버깅하는 동안 능동적으로 추적하지 않고도 행을 일으키는 스레드 문제를 알려줍니다.**

	또한, **Instruments에서는 행 탐지기를 통해 행을 감지하고 표시**해줍니다.

	**Beta때는 Xcode를 사용하지 않거나 추적하지 않을때도 기기에서의 행 탐지기를 통해 실시간 행 알림과 진단**을 해줄 수 있습니다.

	**Public release 단계에선 Xcode Reports Organizer를 통해 실제 사용자로부터 집계된 행 정보들의 리포트를 제공**해줍니다.

	3-1.  Development tooling
	Xcode 14이상에서는 스레드 성능검사를 통해 행이 걸리만한 메인 스레드의 Non-UI 작업이나 우선순위가 역전되는 작업을 감지해서 알려줍니다. 

	예시로 우선순위가 역적되는 작업을 감지하여 동기화 시도에서 행이 걸리수 있다고 알려 줍니다.

	![[우선 순위 역전 작업.png]]

	설정 방법은 Thread Performance Checker를 활성화 해주면 됩니다.
	![[Thread Performance Checker.png]]

	Instrumnet의 [[Time Profiler]]는 콜 스택을 제공하면서 앱의 각 스레드가 시간에 따라 뭘하고 있는지 알 수 있습니다.

	3-2. Beta tooling

	개발이 완료되고 앱을 TestFlight를 통해 개인들의 기기에 다운하여 테스트를 할때 iOS 16 이상부터는 개발자 설정에서 행 감지 기능을 사용할 수 있습니다.  만약 행이 걸리는 현상이 발생하면 앱 상단에 행이 걸리는 시간이 표시 됩니다.
	
	 ![[행 알림 표시.png]]

	설정 ➡️ 개발자  ➡️ 행 감지 기능 활성화
	Hang Theadhold란 행을 감지하는 기준값을 말합니다. 250ms로 설정하면 250ms보다 낮게 걸리는 행을 무시하고 그 이상만 감지합니다. 

	Monitered Apps은 행을 감지하는 앱들입니다. 

	Available Hang Logs는 지금까지 감지한 비정상적인 행 항목을 시간 순으로 기록한 목록입니다. 
	
	![[행 감지 설정.png]]


	3-3. Public release tooling

	Xcode 14부터는 Xcode Organizer에서 Hang reports를 사용해 사용자들의 기기로부터 비정상 행진단을 집계해 전달 받을수 있습니다.
	 ![[organizer hang.png]]

	물론 앱 분석 공유에동의한 사용자만 행 현상에 대한 메인 스레드 스택의 추적 정보가 제공됩니다. 
	다른 방법으로는 App Store Connect의 API(https://developer.apple.com/videos/play/wwdc2020/10057/)를 이용해 행 리포트 데이터를 검색할 수 있습니다. 
	

https://green1229.tistory.com/509?category=1022373
https://green1229.tistory.com/508

- [Analyze hangs with Instruments](https://developer.apple.com/videos/play/wwdc2023/10248/)
- [Understand and eliminate hangs from your app](https://developer.apple.com/kr/videos/play/wwdc2021/10258/)
- [Getting Started with Instruments](https://developer.apple.com/videos/play/wwdc2019/411)
- [Track down hangs with Xcode and on-device detection](https://developer.apple.com/videos/play/wwdc2022/10082)