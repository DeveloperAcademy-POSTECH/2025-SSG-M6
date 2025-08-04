>[!question]
>GQ1. 애플은 왜 Observation이라는 개념을 Swift 5.9에서 도입한 것일까?
>GQ2. Observation의 기능을 최대한 잘 활용할 수 있는 SwiftUI에 적합한 아키텍처는 무엇일까?

## Description

- Observation은 SwiftUI에서 사용하는 데이터 모델 (State Object)을 추적/관찰하는 것을 의미합니다.
- Observation을 통해 SwiftUI는 데이터 모델이 변하면 -> UI를 다시 렌더링할 수 있습니다.
- SwiftUI의 Observation 개념은 `@Observable`과 `@Bindable` 키워드를 사용해 구현합니다.
- Observation은 iOS 17+, Swift 5.9 이후부터 사용가능합니다.


## 1. How? Observation?

+ ObservableObject를 프로토콜로 채택하는 방식이 아니라, 객체 앞에 `@Observable` 매크로를 붙이는 방식으로 구현합니다.
+ `@Observable` 매크로를 붙이면, `Observation` 프로토콜을 SwiftUI 컴파일러가 내부적으로 채택합니다.
+ Combine 프레임워크에 의존하고 있는 `@Published` 키워드를 사용하지 않아도 됩니다.

```Swift
// 🤨 BEFORE 
import Combine
import SwiftUI

class Library: ObservableObject { 
	@Published var books: [Book] = [Book(), Book(), Book()] 
} 

// 😀 AFTER
import SwiftUI

@Observable class Library { 
	var books: [Book] = [Book(), Book(), Book()] 
}
```

+ `@StateObject`는 `@State` 프로퍼티 래퍼를 사용해서 Observation을 사용하는 방식으로 대체 가능합니다.
+ 또한 데이터 모델을 class로 제약을 주지 않고, struct를 사용하는 방식으로 대체해서 만들 수도 있습니다.

```Swift
// 🤨 BEFORE : class 고정
class CounterViewModel: ObservableObject {
    @Published var count = 0
}

struct CounterView: View {
    @StateObject private var viewModel = CounterViewModel()

    var body: some View {
        VStack {
            Text("Count: \(viewModel.count)")
            Button("Increment") {
                viewModel.count += 1
            }
        }
    }
}

// 😀 AFTER : struct도 사용할 수 있습니다.
@Observable
struct CounterModel {
    var count = 0
}

struct CounterView: View {
    @State private var model = CounterModel()

    var body: some View {
        VStack {
            Text("Count: \(model.count)")
            Button("Increment") {
                model.count += 1
            }
        }
    }
}
```

- `@EnvironmentObject`를 사용하는 방식은 `@Environment`로 대체해서 사용할 수 있습니다.

```Swift
// 🤨 BEFORE
class UserSettings: ObservableObject {
    @Published var username: String = "mini"
} 

struct ProfileView: View {
    @EnvironmentObject var settings: UserSettings

    var body: some View {
        Text("Hello \(settings.username)")
    }
}

struct AppView: View {
    @StateObject private var settings = UserSettings()
    var body: some View {
        ProfileView()
            .environmentObject(settings)
    }
}

// 😀 AFTER
@Observable
class UserSettings {
    var username: String = "mini"
}

struct ProfileView: View {
    @Environment(UserSettings.self) private var settings

    var body: some View {
        Text("Hello \(settings.username)")
    }
}
  
struct AppView: View {
    @State private var settings = UserSettings()

    var body: some View {
        ProfileView()
            .environment(settings)
    }
}
```


## 2. Why? Observation?

1. Combine 프레임워크에 의존하지 않고, SwiftUI 수준에서 완전한 Observable 구조를 구현할 수 있게 하고 싶었습니다.
   : SwiftUI 수준에서 Swift 5.9에 새롭게 업데이트된 [[매크로 (Macro)]]의 기능을 최대한 활용해 구현한 것이죠!
   
2. SwiftUI의 View가 body 안에서 읽은 프로퍼티만 추적하여 의존성을 맺고자 하기 위함입니다.
   : 아래 코드 참고!
   BookView는 `book.title`에만 의존하는 구조로 한정 ! `book.author`나 `book.isAvailable`이 바뀌는 경우에는 아무런 업데이트를 수행하지 않도록 제한합니다.
   -> 불필요한 업데이트를 최대한 줄이고자 합니다!
   
```Swift
@Observable final class Book: Identifiable { 
	var title = "Sample Book Title" 
	var author = Author() 
	var isAvailable = true 
}

struct BookView: View { 
	var book: Book 
	var body: some View { 
		Text(book.title) 
	} 
}
```

3. 원래 `ObservableObject`는 class에서만 채택할 수 있었습니다만. 새로운 `@Observable`을 사용함으로써 struct에서도 데이터 모델을 지원할 수 있도록 합니다.
   : class보다 struct를 선호하는 것은 최근 Swift의 업데이트 흐름과 맞닿아있습니다 !
   struct의 성능이 기본적으로 더 낫다는 것 + [[Data Race]]로부터 안전한 것이 값 타입 (Value type)인 struct라는 점.

```Swift
// ❌ 에러발생
struct CounterViewModel: ObservableObject {
    @Published var count = 0
}

@Observable
struct CounterModel {
    var count = 0
}
```


## 3. Utilize! Observation!

+ 커밍쑨


## References

- [Apple Developer Documentation - Observation](https://developer.apple.com/documentation/observation)
- [WWDC23 - Discover Observation in SwiftUI](https://developer.apple.com/kr/videos/play/wwdc2023/10149/)
- [WWDC23 - Demystify SwiftUI performance](https://developer.apple.com/kr/videos/play/wwdc2023/10160/)
- [코딩하는 체대생 - iOS 17+부터 새로워진 SwiftUI의 Observation 상태 관리 알아보기 : @Observable @Bindable](https://mini-min-dev.tistory.com/341)