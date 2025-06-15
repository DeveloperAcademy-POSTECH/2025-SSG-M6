>[!question]
>GQ1. Swift 6.2의 새로운 기능은 무엇이 있을까?
>GQ2. (모든 내용 공통) Apple은 왜 이런 흐름으로 Swift 언어를 업데이트한거지?

## Contents

1. Development workflow : 코드 작성부터, 빌드, 디버깅까지 Swift를 사용하는 workflow 전반에 걸친 개선사항
2. New Library API : Swift와 연관된 Library 등에서의 개선사항
3. Swift throughout the stack : 외부와 연관되는 Swift의 확장성 흐름과 보안 이슈 해결 등
4. Language evolution : 성능적으로 혹은 동시성 파트에서 생기는 언어 개선사항 소개

---

## 1.Development workflow

### ✍🏻 Writing Code

+ VS Code의 Swift Extension 속 기능이 업데이트되었습니다. -> Background indexing, Contex-sensitive code completion, Built-in debugging 등이 있다고 함니다..! (VS Code로 Swift를 다룰 일은 많지 않으니 넘어가죠!)
  
+ VS Code Swift Extension은 오픈 소스입니다. 
  [swiftlang / vscode-swift](https://github.com/swiftlang/vscode-swift)에서 확인할 수 있다고 하는군요.

### 🏢 Building

+ [[매크로 (Macro)]]를 사용하는 프로젝트의 Build Performance를 개선했습니다.
	+ 기존 방식 : Swift PM이 Macro를 지원하는 swift-syntax 코드를 가져오고 (Fetch) -> Swift Syntax Build -> Macro Plugin Build -> Build Project 순
	+ 개선 방식 : Swift Syntax가 Pre-build된 Swift-syntax 코드를 가져오고 (Fetch) -> Macro Plugin Build -> Build Project 순으로 진행
	  
- 컴파일러 경고와 오류에 대한 설명, 해결법을 제공하는 [Diagnostics Category](https://docs.swift.org/compiler/documentation/diagnostics/diagnostic-descriptions) 문서를 확장했습니다.

- 모든 Warning들을 Error로 default 처리할 수 있도록 지정할 수 있게 되었습니다.
```Swift
import PackageDescription

let package = Package(
	name: "PackageExample",
	// ... 
	targets: [
		.target(
			name: "PackageExample",
			swiftSettings: [
				.treatWarnning("StrictMemorySafety", as: .error),
			]
		)
	]
)
```

### 🪲 Debugging

+ [[비동기 (Asynchronous)]] 코드에 대한 디버깅 experience가 크게 개선되었습니다.
  
+ 디버깅 세션에 명시적으로 빌드된 모듈 (모듈 의존성을 빌드할 때, 병렬 처리나 재사용을 가능하게 하는 기능이래요~) 덕분에 responsive성이 개선되었습니다. 
  ㄴ 이게 무슨 말이죠...? -> 더 자세한 내용을 이해하고 싶다면 [WWDC24 - Demystify explicitly built modules](https://developer.apple.com/videos/play/wwdc2024/10171/) 참고할 것.


---
 
## 2.New Library API

### 1️⃣ Subprocess

+ Swift를 사용해서 scripting task를 작성하는 경험을 개선하기 위해 하위 프로세스 (Subprocess)를 도입했습니다. -> 이것도 잘 이해가 안됩니다.. [swiftlang / swift-subprocess](https://github.com/swiftlang/swift-subprocess)를 통해 확인할 수 있습니다.

### 2️⃣ Foundation

+ [[UIKit]]의 [[NotificationCenter]]를 응답 (respond)하는 것은 iOS에서 일반적인 동작이지만, 이 알림을 Observe하는 코드에서 오류가 빈번하게 발생할 수 있었다고 합니다.
	+ 객체가 posting을 지원하는 알림 이름을 등록할 때 조심해야 합니다.
	  `NotificationCenter.addObserver(forName: 어쩌구 저쩌구)`에 해당하는 부분
	+ 알림이 메인 스레드에 posting되는 것이 보장되더라도, MainActor에 접근할 때 [[Data Race]] 에러가 발생할 수 있기 때문입니다.
	
![[Swift 6.2_NotficationCenter Before.png]]

- 하지만 이제 NotificationName과 payload는 concrete type를 지원하며, 이 문제를 해결할 수 있게 되었습니다!
	- `NotificationCenter.addObserver(of:, for:)` : 컴파일러를 통해 Notification을 지원하는 객체, 구체적인 타입을 확인하도록 합니다.
	- NotificationCenter payload (노티피케이션에 함께 전달되는 추가 데이터(정보), 이 코드에서는 startFrame과 endFrame이 해당되죠!)를 다룰 때, 
	  보일러플레이트 (boilerplate, **반복적으로 작성해야 하지만 실질적인 로직이나 의미는 거의 없는 코드**)를 제거합니다.
	  -> `notification.userInfo[UIResponder.keyboard어쩌구저쩌구` 코드를 단순하게 클로저로 넘어온 `{ keyboardState in keyboardState.startFrame }`으로 접근할 수 있다는 거쥬.

![[Swift 6.2_NotficationCenter After.png]]

### 3️⃣ Observation

+ Broadcasting System Notification은 등록된 Observer에 대해 보다 일반적인 (more general) [[관찰자 패턴 (Observer Pattern)]]을 기반으로 합니다. 
  -> 기존 NotificationCenter와 Combine 기반이 아니더라도, Observation 라이브러리와 `@Observable` 키워드를 사용한 구조로 객체의 상태 변화를 자동으로 추적하는 기능을 구현할 수 있었죠.
```Swift
import Observation

@Observable
class Player {
	let name: String
	var score: Int = 0
	var item: Item = .none 
}

let player = Player(name: "Holly")
let values = Observations {
	let score = "\(player.score) points"
	let item = 
		switch player.item {
		case .none: "no item"
		case .banana: "a banana"
		case .star: "a star"
		} 
	// Observe하고 싶은 최종 목적!
	return "\(score) and \(item)"
}
```

- Swift 6.2에서는 Observable 타입의 AsyncSequence로 상태 변경을 스트리밍하는 (stream state change) 방법을 도입합니다. 
- 이를 통해 얻을 수 있는 것은 **"Observation updates are transactional"** 입니다. 
  -> **"Transactional observation preserves consistency"** 가 보장할 수 있다고 합니다. 아래와 같은 원리루요.
```Swift
player.score += 2  // [Update begins] player.score.willSet 
// Other property changes
await Task.yield()  // [Update ends]
}
```


  - 즉, 여러 속성에 대한 동기 업데이트가 발생하더라도 / 변화가 진행중인 상태는 외부로 관찰되지 않고, 모든 속성에 대한 업데이트가 이루어져야 비로소 Observer에게 업데이트 내용이 전달되는 것이죠!
```Swift
// score와 item의 속성 변화가 있을 때 한번에 변화가 처리된 값만 Observe 된다는 의미!
return "\(score) and \(item)"  

// 0 points and no item

// 변화
player.score += 2
player.item = .banana

// 2 points and a banana
```


### 4️⃣ Testing
 
+ [[Swift Testing]]에서 테스트를 실팼는지 선별하기 어려운 경우가 있었다고 합니다. (예를 들어, CI 환경에서만 실패하는 경우) 
  -> 테스트 실패를 이해하려면 테스트에 사용된 데이터에 대한 context가 필요한데요.
  
+ Swift 6.2에서는 custmom Attachment (`Attachment.record`)를 사용해 테스트 실패에 대한 진단을 도울 수 있습니다.
  
+ 또한 [[Swift Testing]]에서 새로운 테스트 실행 프로세스를 사용할 수 있습니다.
  특정 종료 코드나 시그널과 함께 프로세스가 성공적으로 종료됐는지 / 실패 상태로 종료되었는지 검증할 수 있습니다. (`await #expect(processExitsWith: .failure`)

---

## 3.Swift throughout the stack

### 📦 Embeded Swift 

> Embeded Swift는 아주 제약적인 환경에서도 Swift를 (Kernel-level code) 돌릴 수 있도록 도와주는 환경
> Value/Reference Type, Closure, Error Handling, Generic 등 핵심 Swift 기능을 지원하는 컴파일 모드
> 
> [WWDC24 - Go small with Embeded Swift](https://developer.apple.com/videos/play/wwdc2024/10197/) 

+ Swift 6.2부터는 [[문자열 보간 (String interpolation)]]을 포함한 String 전체 기능, any 타입, 그리고 InlineArray, Span까지 지원하게 되었습니다.

### 🔒 Security

+ Swift 6.2에서는 엄격한 메모리 안전 (strict-memory-safety)을 위한 **"Opt-in" 기능**을 추가했습니다.
  -> 보안을 위해 필요한 부분이 어떤 것인지를 식별해주고, 안전하지 않은 API의 사용을 (`unsafe` 키워드를 붙여서) 명시적으로 인정하도록 하는 것이죠.

![[Swift 6.2_Opt-in.png]]

### 💾 Server

+ Java 서버 코드를 Swift로 바꾸고 어쩌구 저쩌구... [swift-java](http://github.com/swiftlang/swift-java)라는 프로젝트가 있고 어쩌구 저쩌구...
  사실 잘 와닿지 않아서 그냥 패쓰. 
  난 Swift로 만든 Server를 아직 우리나라에서 보지 못했거덩요. 아시는 분 있으면 소개좀.

![[Swift 6.2_Swift Java.png]]

- [Containerization](https://github.com/apple/containerization)이라는 라이브러리도 새롭게 릴리즈했다고 합니다.
  -> Mac에서 실행하는 Linux 컨테이너 기반 도구를 구축 시에 사용할 수 있는 컨테이너화 라이브러리입니다.

### 🍎 Platforms

+ FreeBSD라는 운영체제에 대한 공식 지원이 시작되었습니다.
+ 가상 머신 플랫폼인 WebAssembly, Wasm에 대한 Swift 지원을 확대하고 있습니다. (상황도 직접 보여주네요!)

---

## 4.Language evolution
### ↗️ Performance Update
##### InlineArray

+ `Array<Int>`는 동적 크기 조정 (Dynamic Resizing)이 가능하다는 유연성을 장점으로 갖는 대표적인 데이터 타입입니다.
  -> 하지만 이 유연성을 위한 Cost가 동반되는데 (Heap Memory Buffer에 참조를 저장. 고정된 배열 메모리를 사용하는 경우에 이런 방식이 메모리 낭비 우려)
  -> 이를 위해 새로운 **`InlineArray<3, Int>` 타입**이 추가되었습니다.

- `InlineArray<크기, 데이터타입>`는 새로운 "고정 크기"를 지정하는 Array 타입입니다. (제네릭인 Array Literal에서 입력된 값을 바탕으로 컴파일러는 크기와 타입을 추론합니다!)
  -> InlineArray는 추가적인 Heap 할당없이 Stack에 직접적으로 바로 저장됩니다.
  -> InlineArray는 Copyable과 Noncopyable 타입의 값을 모두 저장할 수 있습니다.
```Swift
public struct InlineArray<let count: Int, Element: ~Copyable>: ~Copyable
```

##### Span

- 메모리 안전을 저해하지 않으면서도, 연속되는 메모리에 대해 빠르고 직접적인 접근을 제공하는 기능입니다.
- Swift 6.2 이후부터는 모든 연속적인 저장공간에 대해 지원하게 됩니다!
- C++의 span처럼 Swift에서도 해당 메모리 위치를 슬라이스처럼 참조할 수 있는 읽기 전용 추상화 문법이죠!
- [[COW (Copy-On-Write)]] 기반입니다.
```Swift
extension Array {
	var span: Span<Element> { get }
}

extension ArraySlice {
	var span: Span<Element> { get }
}

extension ContiguousArray {
	var span: Span<Element> { get }
}

extension InlineArray {
	var span: Span<Element> { get }
}

extension String.UTF8View {
	var span: Span<Unicode.UTF8.CodeUnit> { get }
}
```

### 👥 Concurrency

+ 역시 [[Data Race]] 문제입니다.
  아래 상황은 `@MainActor`로 지정되어 있는 UI 호출 블럭에서  [[비동기 (Asynchronous)]] 코드를 호출하려고 하면, 데이터 레이스 문제가 발생할 수 있는 상황인데요.

```Swift
class PhotoProcessor {
    func extractSticker(data: Data, with id: String?) async -> Sticker? {
        ...
    }
}

@MainActor
final class StickerModel {
    let photoProcessor = PhotoProcessor()

    func extractSticker(_ item: PhotosPickerItem) async throws -> Sticker? {
        guard let data = try await item.loadTransferable(type: Data.self) else {
            return nil
        }

		// Error : Sending `self.PhotoProcessor` risks causing data races
        return await processor.extractSticker(data: data, with: item.itemIdentifier)
    }
}
```

- **Approchable concurrency**가 Swift 6.2부터 도입됩니다.
	- mutable state 타입에 비동기 함수 호출이 쉬워졌습니다. -> 해당 함수가 호출을 받은 Actor에서 자동으로 실행되는 것이죠! 우리가 일일이 신경을 써주지 않아도 됩니다.
	  (Run async functions on the caller's actor)
	  
	- 메인 액터의 적합성 구현이 더욱 쉬워졌습니다.(Made easier to implement conformances on main actor types)
	  
	- 전역 상태를 보호하기 위해 사용하는 `@MainActor`코드를 이제는 자동으로 추론합니다.
		![[Swift 6.2_MainActor.png]]
---

## References

- [WWDC25 - What's new in Swift](https://developer.apple.com/kr/videos/play/wwdc2025/245/)
- [[Swiftlang]](https://github.com/swiftlang/swift)