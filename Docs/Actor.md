
비동기 작업을 안전하게 사용하기 위해 사용하는 방식이다.

기본적으로 백그라운드 스레드에서 실행되며, @MainActor와 같은 속성을 부여해 코드가 어떤 스레드에서 실행될지를 명시할 수 있다.

클래스와 마찬가지로 참조 타입이면서, 한번에 단 하나의 작업만 접근해 내부 상태를 변경시킬 수 있다는 점에서 다르다.

> 👀 한번에 단 하나의 작업만 접근 가능하다고?

Actor는 백그라운드 스레드에서 멀티스레딩 코드를 안전하게 실행하도록 하는 중요한 역할을 한다. 멀티스레딩 코드에서 가장 큰 문제점은 공유 상태의 문제로 속성이 하나 이상의 스레드에서 변경될 수 있다면, 코드는 일관성 없는 상태에 빠질 위험이 있다는 것이다. Actor는 이런 가능성을 방지하기 위해 속성을 격리 한다.

→ 아래에서 언급될 actor isolation를 말한다.

### 사용 방법

```swift
actor TemperatureLogger {
    let label: String
    var measurements: [Int]
    private(set) var max: Int

    init(label: String, measurement: Int) {
        self.label = label
        self.measurements = [measurement]
        self.max = measurement
    }
}

let logger = TemperatureLogger(label: "Outdoors", measurement: 25)
print(await logger.max) // Actor 외부에서 가변 상태 접근 시 await 필요
```

> 클래스처럼 actor 키워드를 사용해서 Actor를 정의할 수 있다.

이때, let인 label은 immutable한 변수이기 때문에 제한 없이 접근할 수 있다.

하지만 max는 mutable한 변수이기 때문에 await을 사용해 접근해야 한다.

즉, max에 접근하는 지점은 중단 지점이 되고 해당 작업은 Actor가 내부 작업을 모두 완료한 후에 실행된다.

```swift
extension TemperatureLogger {
    // 액터 내부이므로 await 필요 없음
    func update(with measurement: Int) {
        measurements.append(measurement)
        if measurement > max {
            max = measurement
        }
    }
}
```

update() 내부 함수는 액터 컨텍스트 내(동일한 스레드 환경 == 동일 액터의 직렬화된 큐 내)에서 실행되기 때문에 max에 접근할 때 await 표시를 하지 않아도 된다.

update 호출은 반드시 동기적으로 실행되기 때문에 update가 실행되는 동안 다른 접근들은 await 되어 동시성 문제가 발생하지 않는다.

#### ❓ 만약 액터의 외부 컨텍스트에서 await을 붙이지 않고, mutable 변수에 접근하게 된다면?

컴파일 에러를 발생시킨다.

액터 내부 코드만 액터 로컬 상태에 접근할 수 있도록 보장되어 있기 때문이다.

이런 보장 상태를 액터 분리 (actor isolation)이라고 한다.

---

## Actor Isolation

Actor의 mutable한 상태를, Actor를 고립시킴으로써 보호한다. 기본적으로 self를 통해서만 접근이 가능하다.

- Actor 인스턴스가 가변 속성(var)을 가지고 있다면, 해당 인스턴스만이 그 속성을 변경할 수 있다.
- Actor 외부의 코드는 Actor의 가변 속성을 가져올 수 있지만, 이 접근은 비동기적이다.
    - 즉, 코드는 반드시 await 사용해야 한다.
- Actor 외부의 코드는 Actor의 메소드들을 호출할 수 있지만, 이 접근도 비동기적이다.
    - 즉, 코드는 반드시 await을 사용해야 한다.
    - 이는 Actor가 메소드를 사용하며 자신의 가변 속성을 변경할 수 있기 때문이다.

```swift
actor TemperatureLogger {
    let label: String // 상수 프로퍼티
    var measurements: [Int] // 가변 프로퍼티
    private(set) var max: Int // 가변 프로퍼티

    init(label: String, measurement: Int) {
        self.label = label
        self.measurements = [measurement]
        self.max = measurement
    }

    func update(with measurement: Int) {
        measurements.append(measurement)
        if measurement > max {
            max = measurement
        }
        print("Updated \\(label): current max is \\(max)")
    }
}

// 액터 인스턴스 생성
let logger = TemperatureLogger(label: "Outdoors", measurement: 25)

// 상수 (let) 프로퍼티 접근 -> await 필요 없음
// 상수는 변경되지 않으므로, 액터 격리 대상이 아님
print(logger.label)

// 가변 (var) 프로퍼티 읽기 -> await 필요
// 액터 외부에서 가변 상태를 읽을 때는 await을 사용해야 한다
Task {
    let currentMax = await logger.max
    print("현재 최고 온도: \\(currentMax)")
}

// 액터 메서드 호출 -> await 필요
// 메서드는 액터의 가변 상태를 변경할 수 있기에 await을 사용해야 한다.
Task {
    print("--- 온도 업데이트 시작 ---")
    await logger.update(with: 28)
    await logger.update(with: 32)
    let finalMax = await logger.max
    print("최종 최고 온도: \\(finalMax)")
    print("--- 온도 업데이트 종료 ---")
}

// 가변 (var) 프로퍼티 직접 변경 시도 -> 컴파일 에러 발생
// 액터 외부에서 액터의 가변 상태를 직접 변경하는 것은 불가능
// logger.max = 30 // 에러 발생!!
```

> Actor Isolation 예시 코드

- 상수
    - Actor의 상수(let) 속성은 Actor 외부에서 직접 접근 가능하다.
    - 상수 값 유형의 속성은 변하지 않기 때문에, 어떤 코드나 스레드에서든 안전하게 접근 가능하다.
- 가변
    - Actor의 가변 속성은 Actor 외부에서 await을 사용해 가져올 수 있다.
    - 실제 접근이 Actor 자신의 스레드에서 비동기적으로 이뤄지도록 동작한다.
    - Actor 외부에서는 await을 사용하더라도 변경 불가능하다. (읽기만 가능!)
- 외부
    - Actor의 메소드는 Actor 외부에서 호출할 수 있지만, await을 사용해야 한다.
    - 호출이 Actor 자신 스레드에서 비동기적으로 이뤄지도록 보장하기 위해서이다.

---

## Task

Swift Concurrency에서 Task는 비동기 작업을 정의하고 실행하는 기본 단위이다.

앱 내에서 동시에 실행될 수 있는 독립적인 코드 블록이라 볼 수 있으며, Task로 새로운 비동기 실행 흐름을 만들게 된다.

- 비동기 코드 실행
    - Task 내에 작성된 코드는 즉시 실행되지 않고, 나중에 시스템에서 적절한 스레드에 할당되어 비동기적으로 실행된다.
- 구조적 동시성
    - Task들은 부모-자식 관계를 형성해 계층적으로 관리된다.
    - 부모 Task가 취소되면 자식 Task 들도 자동으로 취소될 수 있다.
- 반환 값
    - Task는 실행이 완료된 후 값을 반환한다.
    - await 키워드를 사용해 Task의 결과를 기다리고 받을 수 있다.
- 취소 지원
    - Task는 취소를 지원한다.
    - 실행중인 Task에 외부에서 취소 요청을 할 수 있으며, 내부에서 이를 감지해 종료한다.

### 사용 방식

1. 현재 컨텍스트에서 비동기 작업 시작
    
    이미 비동기 컨텍스트(async 함수나 actor 내부, 혹은 다른 Task 내부)에 있거나,
    
    앱 시작 시점에 비동기 작업을 시작하고 싶을 때 사용하는 일반적인 방식이다.
    
    이 방식의 경우, Task는 현재 실행중인 코드와 동일한 우선순위와 액터 컨텍스트를 상속받는다.
    
    즉, 만약 현재 실행중인 코드 (fetchDataAndProcess)가 @MainActor에서 호출되었다면, Task도 기본적으로 메인 액터 컨텍스트에서 시작된다.
    
    ```swift
    // 일반적인 동기 함수 내에서 비동기 작업을 시작하고 싶을 때
    func fetchDataAndProcess() {
        Task { // 새로운 비동기 Task 시작
            print("데이터 다운로드 시작...")
            let data = await downloadData() // 비동기 함수 호출
            print("데이터 다운로드 완료!")
    
            print("데이터 처리 시작...")
            await processData(data) // 또 다른 비동기 함수 호출
            print("데이터 처리 완료!")
        }
    }
    
    // 비동기 함수 예시
    func downloadData() async -> String {
        try? await Task.sleep(nanoseconds: 2 * 1_000_000_000)
        return "다운로드된 데이터"
    }
    
    func processData(_ data: String) async {
        try? await Task.sleep(nanoseconds: 1 * 1_000_000_000)
        print("처리 중: \\(data)")
    }
    ```
    
2. 새로운 독립적인 비동기 작업 시작
    
    현재 실행 중인 컨텍스트와 독립적으로, 완전히 새로운 비동기 작업을 시작하고 싶을 때 사용한다.
    
    예를 들어, 부모 Task의 취소에 영향을 받지 않아야하는 작업이나, 완전히 다른 스레드에서 실행시키고 싶을 때 사용한다.
    
    Task.detached로 부모-자식 간의 관계를 끊어내는 형태로 동작한다.
    
    ```swift
    func performBackgroundWork() {
        Task.detached { // 현재 컨텍스트와 분리된 새로운 Task 시작
            print("백그라운드 작업 시작...")
            let result = await performHeavyCalculation() // 무거운 비동기 계산
            print("백그라운드 작업 완료! 결과: \\(result)")
        }
    }
    
    func performHeavyCalculation() async -> Int {
        print("복잡한 계산 중...")
        try? await Task.sleep(nanoseconds: 3 * 1_000_000_000)
        return 12345
    }
    ```
    
    Task 관리의 복잡성을 증가시키기 때문에, 필요한 경우가 아니라면 기본 Task { } 사용이 권장된다.
    

> 💡 Task와 Actor

- Task : 무엇을 비동기적으로 실행할지에 대한 단위로, 코드 블록 자체로 표시
- Actor : 어디서 안전하게 실행할지에 대한 단위로, 공유 가능한 상태를 보호하기 위해 사용

Task는 Actor의 메서드를 호출하거나 Actor의 속성에 접근할 때 await을 사용한다. 이는 Task가 Actor에게 작업을 요청하고, Actor가 해당 작업을 안전하게 처리할 때까지 Task가 멈춰 기다림을 말한다.

Task와 Actor는 Swift Concurrency 구현에 필수 요소들이다. 함께 사용될 때 복잡한 멀티스레딩 환경에서 데이터 안전성을 보장하고, 코드 가독성과 관리를 쉽게 만들어준다.

---

## @MainActor

Swift의 동시성 모델에서는 개발자들이 Actor를 생성하면, 일반적으로 해당 코드가 백그라운드 스레드에서 실행된다고 앞서 언급했다.

그 뒤에는 메인 스레드에서 코드를 실행하는 MainActor라는 보이지 않는 전역 액터가 존재한다.

코드를 메인 스레드에서 실행하려면 메인 액터에 그 코드를 격리해야 한다.

이때, 격리를 위해 사용하는 것이 @MainActor 어트리뷰트이며, 사용 가능 지점은 다음과 같다.

- 속성
    - 속성이 메인 스레드에서만 액세스되어야 함을 명시한다.
    - 인스턴스 속성이나 정적, 클래스 속성일 수 있다.
- 메서드
    - 메서드가 메인 스레드에서만 호출되어야 함을 명시한다.
    - 인스턴스 메서드나 정적, 클래스 메서드일 수 있다.
    - 클래스에서 오버라이드하는 경우, 오버라이드된 메서드가 @MainActor로 지정되어 있으면 이 메서드도 그 지정을 상속한다.
- 전체
    - 클래스, 구조체, 열거형 전체의 모든 속성과 메서드가 메인 스레드에서만 액세스되어야 함을 명시한다.
    - 서브클래스가 명시적으로 @MainActor로 표시되면, 그렇게 표시된 슈퍼클래스에서 파생되어야 하거나, 그 선언된 슈퍼클래스가 NSObject여야 한다.
    - Actor의 경우에는 이미 자체적인 직렬화 큐를 가지고 있기 때문에 붙일 필요가 없으며, 사용이 불가능 하다.
- 전역 변수나 함수
    - 전역 변수나 함수에도 적용해 해당 요소들이 메인 스레드에서 실행되도록 강제할 수 있다.

메인 스레드에서 실행된다는 것은 코드 블록이 직렬로 실행되기에 멀티스레딩 환경에서 data race로부터 안전하다는 의미이다.

따라서 await을 사용하지 않고 직접 접근이 가능하다.

예를 들어, ViewController의 viewDidLoad 메서드는 메인 스레드에서 실행되는데, 그 이유는 ViewController가 @MainActor 클래스인 UIViewController의 서브 클래스이기 때문이다.

또한, 인터페이스와 관련있는 코코아 클래스는 코드가 메인 스레드에서 실행되어야 하기 때문에 @MainActor가 이미 붙어져 있음을 확인할 수 있다.

### 그렇다면, MainActor.run은 무슨 역할일까?

특정 코드 블록을 메인 스레드에서 실행되도록 강제하는 함수이다.

주로 백그라운드 스레드에서 실행중인 코드에서 UI 업데이트와 같이 메인 스레드에서만 실행되어야 하는 작업을 수행할 때 사용한다.

이를 이해하기 위해서 Task의 컨텍스트 스위칭 방법에 대해 정리하자면, 2가지 방법이 있다.

1. Task의 암시적인 컨텍스트 스위칭

Swift 코드는 개발자가 그 사실을 인식하지 못하는 상태에서도 메인 스레드와 백그라운드 사이를 전환할 수 있다.

어떤 스레드에서 실행되는지에 대한 명시가 없을 수 있으며, 그 인식 자체가 반드시 필요하지 않기도 하다.

하지만, 특정 메소드가 백그라운드 스레드에서 실행되기를 원한다고 가정한다면 이때 actor를 사용할 수 있다.

1. Task의 명시적인 컨텍스트 스위칭

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

---

## Sendable

한 동시성 도메인에서 다른 동시성 도메인으로 안전하게 전달될 수 있는 타입을 전송 가능 타입이라고 한다.

> ❓동시성 도메인(concurrency domain) 이란?

Data Race로부터 안전하게 보호되는 코드의 영역을 말한다. Task나 Actor의 내부 변수, 프로퍼티를 변경하는 가능성을 포함한 코드 부분을 동시성 도메인(concurrency domain)이라고 한다.

sendable 타입은 Sendable 프로토콜을 채택해 구현되며 Sendable 프로토콜을 준수하는 객체는 다음과 같은 특징을 만족한다.

- 값 타입이고 변경 가능한 상태는 다른 전송 가능한 데이터로 구성된다.
    - 예를 들어, 전송 가능한 저장된 프로퍼티가 있는 구조체 또는 전송 가능한 연관된 값이 있는 열거형이 있다.
    - 구조체, 열거형 내부 프로퍼티 또한 sendable 프로토콜을 준수하는 타입이어야 한다.
- 변경 가능한 상태가 없으며 변경 불가능한 상태는 다른 전송 가능한 데이터로 구성된다.
    - 예를 들어, 읽기 전용 프로퍼티만 있는 구조체 또는 클래스가 있다. (let 프로퍼티만 있는 클래스 등)
- 변경 가능한 상태의 안정성을 보장하는 코드를 가지고 있다.
    - 예를 들어, 자체적으로 직렬화 큐를 통해 상태를 보호하는 Actor, 메인스레드에서만 접근을 보장하는 @MainActor가 있다.

### 왜 Sendable을 사용할까?

Data Race가 발생하는 원인은 Data가 공유 가변 상태이기 때문이다.

하지만, 값 타입일 경우 값의 복사가 일어나고 참조가 발생하지 않기 때문에 Data Race가 발생하지 않는다.

즉, 주어진 타입의 값이 안전하게 동시성 코드에서 사용해야하고 안전한지 아닌지를 컴파일러가 알게하기 위해서 Sendable을 사용한다.

Sendable type이 아닌 타입의 경우에는 데이터 레이스와 같은 예상치 못한 결과를 발생시킬 수 있어 주의가 필요하다.

### @unchecked Sendable 이건 뭘까?

일반적으로, 클래스가 Sendable 프로토콜을 채택하면, 해당 클래스는 Sendable 프로토콜의 규칙을 충족해야한다.

그렇지 않으면 다중 스레드 환경에서 데이터 일관성과 안전성을 보장하기 위해 컴파일 에러가 발생한다.

특정 경우에 이런 체크를 우회해야 할 필요가 있는데, 이때 @unchecked 를 사용할 수 있다.

이 어노테이션을 채택하면 컴파일러는 Sendable 프로토콜의 확인을 건너뛰게 된다.

여기서 중요한 점은, @unchecked Sendable을 사용한다면 직접 스레드 안전성을 보장해야 한다는 것이다.

만약 따로 스레드 안전성 보장을 위한 처리 (커스텀 큐 만드는 등)를 하지 않고 해당 어노테이션을 채택한다면, 데이터 불일치나 동시성 문제가 발생하게 될 수도 있다.



### 참고

[동시성 (Concurrency) | Swift](https://bbiguduk.gitbook.io/swift/language-guide-1/concurrency)

[[Swift] Actor, @MainActor, Task, Sendable의 모든것](https://jimmy-ios.tistory.com/46)