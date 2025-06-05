
## Description
---
#### Swift Concurrency

Swift 5.5부터 도입된 새로운 동시성 모델이다.
기존의 [[GCD (Grand Central Dispatch)]] 위에 구축된 고수준의 추상화이며, async/await과 Task를 기반으로 비동기 및 병렬 코드를 더 안전하고 간결하게 작성할 수 있게 해준다. 개발자가 저수준의 스레드 관리에 직접 관여하지 않고 동시성 문제를 해결할 수 있도록 해준다. 복잡한 코드 흐름을 더 읽기 쉽게 만들고, 유지보수하기 좋다.


> ❓ 비동기 / 병렬 / 동시성은 뭐가 다를까?

- [[비동기 (Asynchronous)]]
	- 현재 진행 중이던 작어버이 완료될 때까지 기다리지 않고, 다음 작업을 시작한다.
- 병렬(Parallel)
	- 여러 작업이 실제로 동시에 실행된다.
- 동시성 (Concurrency)
	- 위의 비동기와 병렬을 아우르는 상위 개념으로, 한 프로그램이 여러 작업을 동시에 / 유연하게 수행하는 방식 전체를 의미한다. 작업이 동시에 진행되는 것 처럼 보이게 하거나, 실제로 동시에 진행되게 할 수 있다. 이런 방식들을 통해 작업을 효율적으로 관리해 앱의 응답성과 성능을 향상시킨다.


> 💡 동시성은 왜 중요할까?
> 
> 사용자 경험을 향상시키기 위해서는 UI는 끊기지 않고 동작해야 한다. 하지만 이미지 다운로드, 파일 분석 등 시간이 오래 걸리는 작업도 수행할때는 어쩔 수 없이 앱이 멈추거나 딜레이가 생기는 것처럼 보이게 된다. 동시성은 이 문제를 해결해 UI는 계속 자연스럽게 흘러가도록 유지하면서 무거운 작업도 함께 처리할 수 있게 해준다.



## 구현
---

동시성은 잘못 구현하면 코드가 매우 복잡해지고, 디버깅 하기도 어려워진다.
예를 들어, 아래 코드처럼 클로저를 이용해 동시성 처리를 하게되면, 중첩도 때문에 점점 복잡해진다. 
이런 상황이 계속 이어지면 이로인해 콜백지옥도 발생하게 된다.

```swift
listPhotos(inGallery: "gallery") { photoNames in 
	let sorted = photoNames.sorted() 
	let name = sorted[0] 
	downloadPhoto(named: name) { photo in 
		show(photo) 
	}
}
```
> 사진을 리스트화 하고 해당 리스트의 첫번째 사진을 다운로드해 사용자에게 보여주는 코드

하지만 Swift는 컴파일 타임 검사와 async/await 키워드를 통해 안전하고 쉽게 동시성 코드를 작성하도록 돕느다. 아래는 동일한 동작을 Swift Concurrency를 이용해 작성한 코드이다. 콜백 중첩 없이 동기 코드처럼 선형적인 흐름으로 작성되어 가독성과 유지보수성을 높혔다.

```swift
let photoNames = await listPhotos(inGallery: "gallery") 
let sorted = photoNames.sorted() 
let name = sorted[0] 
let photo = await downloadPhoto(named: name) 
show(photo)
```


#### async

실행 도중 **일시적으로 중단(suspend)될 수 있는** 함수를 의미한다. throws가 오류를 발생시키는 함수임을 나타내듯이, async는 비동기 작업을 수행해 중단될 수 있는 함수임을 나타낸다.

```swift
func listPhotos(inGallery name: String) async -> [String] {
    // 네트워크 요청 등 비동기 작업 수행
}
```

이때, async와 throw를 함께 사용해 오류 처리까지도 가능하다.

```swift
func listPhotos(inGallery name: String) async throws -> [String]

// 호출
let photos = try await listPhotos(inGallery: "Gallery")
```


#### await

비동기 함수가 언제 끝날지 모르니, 완료될 때까지 기다리자는 의미를 명시적으로 전달하는 역할이다. Swift에서는 await 키워드를 사용해 비동기 함수의 중단 가능한 지점을 명시적으로 표시한다. 이렇게 함으로써 중단 가능한 곳이 명확해져서 코드 가독성을 향상시키고 개발자가 어디서 작업이 멈출 수 있는지를 잘 알 수 있게 해준다. 또한, 컴파일러가 중단 지점을 파악해 동시성 문제를 미리 감지할 수 있게 되어 버그 발생의 가능성이 줄어든다.

```swift
let names = await listPhotos(inGallery: "Gallery") // 이 시점에 일시 중단이 가능하다
let name = names.first!
let photo = await downloadPhoto(named: name) // 이 시점에도 일시 중단이 가능하다
show(photo)
```


#### async await 코드의 실행

앞서서 예시로 들었던 코드를 토대로, async await을 사용한 동시성 코드의 실행 순서는 아래와 같다.
이 흐름대로 하나의 코드에서 중단이 발생해도, 전체 앱의 흐름은 멈추지 않고 실행되고 있음을 알 수 있다.

1. await listPhotos() 호출
	- 네트워크 요청을 보내고 일시 중단
2. 중단된 동안, 다른 동시 작업 실행
3. listPhotos 함수가 완료되면 코드 재개
4. await downloadPhoto() 호출
	- 다운로드 요청 보내고 일시 중단
5. 중단된 동안, 다른 동시 작업 있다면 실행
6. downloadPhoto 함수가 완료되면 코드재개 -> 이미지 표시


---

#### Task.yield()

비동기 함수에서 긴 반복 작업이나 연산이 많은 작업을 수행할 때, 중간중간 다른 작업에게 CPU 시간을 양보**하는 것이 좋다. 이는 특히 메인 스레드에서 UI 업데이트가 필요한 경우, 앱의 반응성을 유지하는 데 중요하다.

Task.yield()는 현재 실행 중인 작업을 잠시 멈추고, 다른 동시 작업이 실행될 수 있게 기회를 준다.


```swift
func generateSlideshow(forGallery gallery: String) async {
    let photos = await listPhotos(inGallery: gallery)
    for photo in photos {
        // 긴 반복 작업이나 연산이 많은 작업
        await Task.yield()
    }
}
```


#### Task.sleep()

네트워크 요청이나 다른 비동기 작업의 테스트를 위해서 의도적인 지연이 필요할 때 사용할 수 있다.
Task.sleep()은 오류를 던질 수 있는(cancellable) 함수이기 때문에, 호출 시 `try`를 함께 써야 한다.

```swift
try await Task.sleep(for: .seconds(1))
```


#### Result 매핑

에러를 do-catch 없이 처리하기 위해서 Result로 매핑하는 방식을 사용할 수 있다.

```swift
func availablePhotos() -> Result<[String], Error> { 
	return Result { 
		try listDownloadedPhotos(inGallery: "Weekend") 
	} 
}
```


#### Thread Safe

Swift는 `actor`, `Sendable`, `@MainActor` 같은 키워드를 제공하여 **메모리 안전한 방식**으로 동시성 코드를 작성할 수 있도록 설계되어 있다. 따라서 Swift Concurrency를 사용하면 복잡한 잠금 (lock)이나 메모리 접근 오류 없이도 동시성 코드를 비교적 쉽게 작성할 수 있다.

- Actor
	- 공유 가능한 mutable 상태에 대한 접근을 자동으로 직렬화해 경쟁조건 (Race Condition)을 방지한다.
	- Actor 내부의 데이터는 Actor의 메서드를 통해서만 접근할 수 있으며, 이 메서드 호출은 자동으로 직렬화되어 스레드 안전성을 보장한다.
- Sendable
	- 데이터 타입이 동시성 경계를 넘어 안전하게 전달될 수 있음을 나타내는 프로토콜이다.
	- 컴파일러가 Sendable 규칙을 강제해 데이터 레이스 (Data Race)를 방지하는 데 도움을 준다.
- @MainActor
	- 특정 클래스, 구조체, 함수 또는 클로저가 항상 메인 스레드에서 실행되도록 보장한다.
	- UI 업데이트 등 메인 스레드에서 안전하게 실행되어야 하는 경우 사용할 수 있다.


