## @MainActor

Swift의 동시성 모델에서는 개발자들이 Actor를 생성하면, 일반적으로 해당 코드가 백그라운드 스레드에서 실행된다고 앞서 언급했다. 그 뒤에는 메인 스레드에서 코드를 실행하는 MainActor라는 보이지 않는 전역 액터가 존재한다.
```swift
@globalActor
final public actor MainActor: GlobalActor {
    public static let shared: MainActor
}
```

코드를 메인 스레드에서 실행하려면 메인 액터에 그 코드를 격리해야 한다.
이때, 격리를 위해 사용하는 것이 `@MainActor`이며, 사용 가능 지점은 다음과 같다.

- 속성
    - 속성이 메인 스레드에서만 액세스되어야 함을 명시한다.
    - 인스턴스 속성이나 정적, 클래스 속성일 수 있다.
```swift
class MyViewModel {
    @MainActor var text: String = "text"
}
```


- 메서드
    - 메서드가 메인 스레드에서만 호출되어야 함을 명시한다.
    - 인스턴스 메서드나 정적, 클래스 메서드일 수 있다.
    - 클래스에서 오버라이드하는 경우, 오버라이드된 메서드가 @MainActor로 지정되어 있으면 이 메서드도 그 지정을 상속한다.
```swift
class Logger {
    @MainActor
    func updateUI(_ message: String) {
        print("로그:\(message)")
    }
}
```

- 전체
    - 클래스, 구조체, 열거형 전체의 모든 속성과 메서드가 메인 스레드에서만 액세스되어야 함을 명시한다.
    - 서브클래스가 명시적으로 @MainActor로 표시되면, 그렇게 표시된 슈퍼클래스에서 파생되어야 하거나, 그 선언된 슈퍼클래스가 NSObject여야 한다.
    - Actor의 경우에는 이미 자체적인 직렬화 큐를 가지고 있기 때문에 붙일 필요가 없으며, 사용이 불가능 하다.
```swift
@MainActor
class AppCoordinator {
    var path = NavigationPath()

    func navigate() {
        path.append("다음 화면")
    }
}
```

- 전역 변수나 함수
    - 전역 변수나 함수에도 적용해 해당 요소들이 메인 스레드에서 실행되도록 강제할 수 있다.
```swift
@MainActor
var sharedState = 0

@MainActor
func showAlert() {
    print("Main thread alert")
}
```

메인 스레드에서 실행된다는 것은 코드 블록이 직렬로 실행되기에 멀티스레딩 환경에서 data race로부터 안전하다는 의미이다.

따라서 await을 사용하지 않고 직접 접근이 가능하다.

예를 들어, ViewController의 viewDidLoad 메서드는 메인 스레드에서 실행되는데, 그 이유는 ViewController가 @MainActor 클래스인 UIViewController의 서브 클래스이기 때문이다.

### `nonisolated`는 무슨 역할일까?
@MainActor 내부에 있지만 메인 스레드에서 실행시키고싶지 않다면 `nonisolated`를 사용하면 된다.
```swift
@MainActor
class AppCoordinator {
    var path = NavigationPath()

    func navigate() {
        path.append("다음 화면")
    }

	//nonisolated로 격리 해제!
	nonisolated func version() -> String {
        "AppCoordinator v1.0"
    }
}
```

### 그렇다면, `MainActor.run`은 무슨 역할일까?

특정 코드 블록을 메인 스레드에서 실행되도록 강제하는 함수이다.
```swift
extension MainActor {
    public static func run<T>(
        resultType: T.Type = T.self,
        body: @MainActor @Sendable () throws -> T
    ) async rethrows -> T where T: Sendable
}
```

주로 백그라운드 스레드에서 실행중인 코드에서 UI 업데이트와 같이 메인 스레드에서만 실행되어야 하는 작업을 수행할 때 사용한다. 이를 이해하기 위해서 Task의 컨텍스트 스위칭 방법에 대해 정리하자면, 2가지 방법이 있다.

1. Task의 암시적인 컨텍스트 스위칭
   
	Swift 코드는 개발자가 그 사실을 인식하지 못하는 상태에서도 메인 스레드와 백그라운드 사이를 전환할 수 있다. 어떤 스레드에서 실행되는지에 대한 명시가 없을 수 있으며, 그 인식 자체가 반드시 필요하지 않기도 하다. 하지만, 특정 메소드가 백그라운드 스레드에서 실행되기를 원한다고 가정한다면 이때 actor를 사용할 수 있다.

2. Task의 명시적인 컨텍스트 스위칭
   
	Swift에서는 코드가 실행되는 스레드의 종류를 명시적으로 지정해야 할 때가 있다.
	- 백그라운드 스레드에서 작업하기
	    - 기존에 메인 스레드에서 실행되고 있지만, 이를 백그라운드에서 실행하고 싶다면 Task.detached를 사용할 수 있다.
	    - Task.detached가 Task 객체와 주변 컨텍스트 간의 관계를 끊어 백그라운드 스레드에서 실행이 가능하게 한다.
	- 메인 스레드에서 작업하기
	    - 코드가 백그라운드 스레드에서 실행되고 있고, 특정 코드 블록이 메인스레드에서 실행되도록 하고 싶다면 MainActor.run을 사용할 수 있다.

```swift
// 백그라운드에서 실행되는 코드
Task.detached {
    let heavyCalculationResult = performHeavyCalculation()

    // UI 업데이트는 메인 스레드에서 이루어져야 함
    await MainActor.run {
        self.updateUI(with: heavyCalculationResult)
    }
}
```

