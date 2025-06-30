
### 기본 문법
#### async
에러가 발생할 수 있는 함수를 나타내기 위해 `throws`를 붙여주듯이, 비동기 함수를 나타내기 위해서는 `async`를 붙여준다. 에러가 발생할 수 있는 비동기 함수라면 `throws` 전에 `async`를 붙여준다.

```swift
func listPhotos(inGallery name: String) async throws -> [String] {}
```

#### await
에러를 실제로 던지는 즉, 에러가 발생할 수 있는 함수를 호출할 때 `try`를 붙여주듯이, 비동기 함수를 호출할 때에는 즉, 중단될 가능성이 있을 때는 `await`을 붙여준다. 에러가 발생할 수 있는 비동기 함수라면 `await` 전에 `try`를 붙여준다.

```swift
let photoNames = try? await listPhotos(inGallery: "Summer Vacation")
```

---

### 작업 생성
#### Task
**비동기 작업의 단위(A unit of asynchronous work)** 이다.
비동기 함수를 동기 함수 내부에서 사용하려면 `Task {}`를 사용할 수 있다.

> 버튼 클릭시 예시
```swift
Button("Show First Photo") {
    Task {
        let names = await listPhotos(inGallery: "Gallery")
        let name = names.first!
        let photo = await downloadPhoto(named: name)
        show(photo)
    }
}
```

---

### 데이터 보호
#### actor
swift에서 동시성을 안전하게 다루기위해 도입된 타입이다.
여러 비동기 작업이 공유 가능한 데이터에 동시에 접근하면 생기는 문제(Data Race)를 막아준다.

> Data Race 발생할 수 있는 예시
- 여러 Task가 동시에 `increment()`를 호출하므로 `value += 1` 중에 충돌이 발생하여 잘못된 값이 저장될 수 있다.
- 즉, value의 결과가 2,000이 아닐수도 있다.
```swift
class UnsafeCounter {
    var value = 0

    func increment() {
        value += 1
    }
}

let counter = UnsafeCounter()

Task {
    for _ in 0..<1_000 {
        counter.increment()
    }
}

Task {
    for _ in 0..<1_000 {
        counter.increment()
    }
}
```

> actor 사용하여 안전하게 처리한 예시
```swift
actor SafeCounter {
    private var value = 0

    func increment() {
        value += 1
    }

    func getValue() -> Int {
        return value
    }
}

let counter = SafeCounter()

Task {
    for _ in 0..<1_000 {
        await counter.increment()
    }
}

Task {
    for _ in 0..<1_000 {
        await counter.increment()
    }
}

```

> [!question] increment()에 async 없는데 어떻게 await을?
> actor는 내부 상태 보호를 위해 동시에 여러 스레드가 접근하지 못하도록 자동 직렬화(serialized) 해준다.
> 따라서 actor 안의 함수는 자동으로 비동기로 작동하므로 await을 붙여줘야한다.


#### Main Actor
변경 가능한(mutable) 데이터에 대한 접근을 보호하기 위해, 코드가 차례로 접근하게 강제하는 객체인 `actor` 중 **앱에서 가장 중요한 액터**이다.

앱에서는 **UI를 구성하는 모든 데이터**가 메인 액터에 의해 보호된다.
메인 액터는 **UI를 렌더링하고, UI 이벤트를 처리하며, UI를 업데이트하는 코드**를 차례대로 실행한다.

함수가 항상 메인 액터에서 실행되도록 하려면, `@MainActor` 속성을 사용한다.

```swift
@MainActor func show(_: Data) { // ... UI code to display the photo ... }
```

Main Actor 함수를 외부에서 사용하려면 `await`이 필요하다.
```swift
func downloadAndShowPhoto(named name: String) async { 
	let photo = await downloadPhoto(named: name) 
	await show(photo) //await 붙여줌
}
```


#### [[Sendable]]
Swift [[Concurrency]]에서 사용하는 [[프로토콜 (Protocol)]]로 **값이 여러 스레드 간 안전하게 이동될 수 있게 보장**하는 약속
>[[Sendable]]을 채택한 타입은 다른 스레드/Task로 복사되거나 전달될 때 데이터 충돌 없이 안전하게 사용 가능함을 보장