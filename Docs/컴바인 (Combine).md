
>[!question]
>GQ1. - **Combine 프레임워크**는 어떤 기능을 제공할까? 언제 필요할까?
>GQ2. - RxSwift가 있는데도 왜 애플에서 Combine을 만든 이유는 뭘까? + 차이점
## **Combine 프레임워크**는 어떤 기능을 제공할까? 언제 필요할까?
### Combine이란?

Apple이 만든 [[반응형 프로그래밍 (Reactive Programming)]] 프레임워크입니다.
예전 프로그래밍에서는 값이 고정되어 있고 그 값에 직접 접근하거나 가져오는 식으로 처리했다면, Combine은 시간에 따라 계속 변화하는 것들, 즉 **이벤트 스트림을 자동으로 전달 받으며 처리**합니다.

- iOS 13+, macOS 10.15+부터 사용 가능
- 콜백이나 델리게이트처럼 명령형 방식이 아닌, **데이터 흐름을 조립하는 방식**으로 이벤트 처리

##### 전통적인 방식 (UIKit 방식 -> Delegate 사용)

```swift
textField.delegate = self

func textFieldDidChange(_ textField: UITextField) {
    // 글자 바뀔 때마다 처리
}

/*
- textPublisher는 UITextField.textDidChangeNotification을 Combine Publisher로 래핑한 것입니다.
- 사용자가 텍스트 필드에 입력할 때마다 알림(Notification)이 발생합니다.
- debounce는 입력이 멈춘 후 300ms 동안 변화가 없을 때만 이벤트를 전달합니다.
- removeDuplicates는 같은 문자열이 연속해서 들어오면 무시합니다.
- 최종적으로 sink에서 최신 텍스트 값을 받아 출력합니다.
*/

```
- delegate를 설정하고 + 메서드 구현도 해야하고 + 직접 로직을 작성해야 함 👎🏻 
반응형 처리 방식 - **이벤트 발생 -> 메서드 호출**

##### Combine 사용 시 !

```swift
textPublisher
    .debounce(for: .milliseconds(300), scheduler: RunLoop.main)
    .removeDuplicates()
    .sink { print($0) }
/*
- NotificationCenter를 통해 텍스트 필드에 입력이 변경될 때마다 나오는 알림을 구독합니다.
- 알림에서 UITextField를 꺼내서 .text 값을 가져옵니다.
- 그 값을 Combine 스트림에서 받아서 처리합니다. (sink)
*/
```
- publisher 체인을 선언적으로 구성 + 흐름이 한 눈에 보임 👍🏻
반응형 처리 방식 - **데이터가 흘러가는 방식으로 처리**

---
### Combine의 구성 요소

#### - Publisher
데이터를 발행하는 존재 : 서버 응답, 버튼 클릭, 텍스트 입력 등 모든 이벤트가 Publisher가 될 수 있습니다.
#### - Subscriber
Publisher가 내보내는 데이터를 받아 처리하는 주체, 실제 데이터에 반응하는 곳입니다.
#### - Operator
Publisher와 Subscriber 사이에서 값을 가공하거나 흐름을 제어하는 곳입니다.

위 3개가 체인처럼 연결되어 데이터 흐름을 만듭니다.

###### (그 외 알아두면 좋은 요소)
###### - Subscription
Publisher ↔ Subscriber 연결과 수명 관리
###### - Cancellable
나중에 구독을 취소하기 위한 객체
###### - Schedulers
코드가 실행되는 스레드를 제어하는 데 사용됩니다.

---
### Code
자 그럼 이제 실제 (간단한) 데이터의 흐름을 보고 학습해보아요 ✨

```swift
import Combine // 프레임워크 import

var cancellables = Set<AnyCancellable>() // sink로 구독한 결과는 AnyCancellable 타입으로 반환

// 1. Publisher: Just는 한 번에 하나의 값을 발행
let publisher = Just("hello world")

// 2️. Operator: 문자열을 대문자로 변환
publisher
.map { value in
	    return value.uppercased()   // "HELLO WORLD"
	}
// 3️. Subscriber: 값을 받아서 출력
.sink { value in
        print("Subscriber이 받은 값: \(value)")  // Subscriber이 받은 값: HELLO WORLD
    }
// 4️. 메모리 관리를 위해 cancellables에 저장
.store(in: &cancellables)
```

##### ✨ 흐름 한 눈에 보기
Just("hello world")  →  .map { uppercased }  →  .sink { print }
(Publisher)                     (Operator)                        (Subscriber)

---
### 그래서 언제 써요?

Combine은 다양한 [[비동기 (Asynchronous)]]/이벤트 작업을 하나의 선언적 흐름으로 통합할 수 있는 프레임워크입니다.  
복잡한 콜백/델리게이트 구조를 명확하게 바꾸고 싶을 때 유용하며,  
예를 들어 검색창 입력 제어, API 응답 처리, UI 상태-데이터 바인딩처럼 **흐름이 이어지는 작업을 간결하게 구성**할 수 있습니다.

> Combine은 **비동기 흐름을 조립 가능한 데이터 스트림으로 바꾸어**, **복잡한 UI 이벤트, 네트워크 처리, 상태 바인딩을 단순하고 선언적으로 만들고자 할 때 가장 효과적**입니다.

### Keyword
[[비동기 (Asynchronous)]]
[[반응형 프로그래밍 (Reactive Programming)]]

## [[RxSwift]]가 이미 널리 쓰이고 있었는데, 왜 [[컴바인 (Combine)]]프레임워크를 새로 만들었을까?

취업 공고를 들어가면 보이는 .. "RxSwift 가능하신 분 🙋🏻‍♀️ Combine 사용해보신 분 🙋🏻‍♂️" 보고 의문을 가지게 되었어요.

우선 RxSwift와 Combine 모두 [[비동기 (Asynchronous)]] 프로그래밍과 [[반응형 프로그래밍 (Reactive Programming)]]을 지원하는 도구들입니다.

각자의 장단점을 알아보자면,
#### [[RxSwift]]
##### 장점
- 성숙도
	- RxSwift는 오랜 시간 동안 사용되어 안정성이 좋고 오랜 시간동안 쌓인 풍부한 문서와 커뮤니티 지원을 확보하고 있음
- 다양한 연산자
	- ReactiveX 기반으로 다양한 연산자를 제공하기 때문에 복잡한 비동기 작업을 간결하게 처리할 수 있음
- 멀티플랫폼 지원
	- RxSwift를 기반으로 한 코드는 다른 ReactiveX 라이브러리로 쉽게 확장 가능해서 멀티플랫폼 개발에서도 유리함
##### 단점
- 의존성 추가 필요
	- 외부 라이브러리(SPM, CocoaPods 등)를 설치해야 함
- 메모리 관리 어려움
	- bind 사용 시 [[순환 참조 (Retain Cycle)]] 발생 가능
- 추상화가 과도할 수 있음
	- 내부 흐름 디버깅이 어려운 경우가 있고, Observable 내부 상태 추적이 어렵기도 함
- 공식 지원이 아님
	- Apple 생태계와 완전히 통합된 것이 아니고 SwiftUI와 직접 연동이 불편함

#### Combine
##### 장점
- Apple 공식 프레임워크
	- Swift, Foundation, SwiftUI와 긴밀하게 통합되어 있음
- 타입 안전성
	- Publisher<Output, Failure> 타입 기반으로, Swift 언어 특성과 잘 어우러짐
- Cancellable로 메모리 해제 간편
	- .store(in:) 사용으로 메모리 누수 방지 쉬움
##### 단점
-  iOS 13 이상에서만 사용 가능
	- 구형 기기나 프로젝트에서는 사용 불가
- 연산자 종류 적음
	- RxSwift에 비해 operator이 부족함
- 테스트 지원 부족
	- 공식 테스트 툴 없음

### 차이점

| 항목                 | RxSwift                                       | Combine                                           |
|----------------------|-----------------------------------------------|---------------------------------------------------|
| **Framework Consumption** | **Third-party**                          | **First-party**                                   |
| **Maintained by**         | Open-Source / Community                   | Apple / Community (커뮤니티 거점 있음)             |

#### 핵심 구성 요소

| 항목               | RxSwift                     | Combine                                             |
| ---------------- | --------------------------- | --------------------------------------------------- |
| 던지는 애            | `Observable`                | `Publisher`                                         |
| 받는 애             | `Observer`                  | `Subscriber`                                        |
| Subject (초기값 없음) | `PublishSubject`            | `PassthroughSubject` (`value`, `error`, `finished`) |
| Subject (초기값 있음) | `BehaviorSubject`           | `CurrentValueSubject`                               |
| 구독               | `.subscribe`                | `.sink`, `.assign`                                  |
| 구독 해제            | `.disposed(by: disposeBag)` | `.store(in: &cancellables)`                         |

#### 스레드 전환

| 항목                 | RxSwift       | Combine                                            |
| ------------------ | ------------- | -------------------------------------------------- |
| 스레드 전환 (down only) | `observeOn`   | `receive(on:)`                                     |
| 스레드 전환 (up & down) | `subscribeOn` | `*subscribe(on:)` *(주의: Combine은 downstream only)* |

#### 대표 연산자

| 항목         | RxSwift                                      | Combine               |
| ---------- | -------------------------------------------- | --------------------- |
| 공통 연산자     | `just`, `combineLatest`, `zip`, `debounce` 등 | 대부분 동일하게 존재           |
| 병합 (merge) | `merge(A, B, C)`                             | `A.merge(with: B, C)` |
| 사이드 이펙트    | `do`                                         | `handleEvents`        |
| 건너뛰기       | `skip`                                       | `dropFirst`           |
| 제한         | `take`                                       | `prefix`              |
| 중복 제거      | `valueChanged` *(RxCocoa 전용)*                | `removeDuplicates()`  |

### 그래서 왜 만들었는데요?

> 🍎 **Combine은 Apple 생태계 전반에서 통일된 비동기 흐름을 구성할 수 있도록,  
> RxSwift의 기능을 Apple의 철학과 언어 체계에 맞게 재설계한 공식 반응형 프레임워크입니다.**
> Apple은 RxSwift가 존재함에도 **Combine을 직접 만든 이유는**,
> **외부 라이브러리에 의존하지 않고**, **Swift 언어, SwiftUI, Foundation, 시스템 API와 깊이 통합되는 반응형 프로그래밍 모델**이 필요했기 때문입니다.

