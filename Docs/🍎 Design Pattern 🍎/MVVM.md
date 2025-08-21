>[!question]
>GQ1. ViewModel의 역할 범위는 어디까지인가?
>GQ2. VIewModel은 얼마나 비즈니스 로직을 가져도 되는가?


## MVVM 이란?
---
MVVM은 **UI와 비즈니스 로직을 분리**하여 코드의 재사용성과 유지보수성을 높이는 아키텍처 패턴이다. View는 사용자 인터페이스, ViewModel은 상태와 로직 관리, Model은 데이터와 도메인 규칙을 담당한다.

Swift에서는 **SwiftUI, Combine** 같은 프레임워크가 MVVM과 잘 맞으며, 데이터 바인딩을 자연스럽게 지원한다.


## MVVM 구성요소
---

MVVM은 이름 그래도 Model, View, ViewModel 3가지 구성 요소로 이루어진다.

1. 모델 (Model)
	* 데이터를 표현하는 순수 객체 
	* 네트워크에서 받은 JSON, 데이터베이스 엔티티, 비즈니스 도메인 로직을 담당
	* UI를 전혀 모르고, 그저 데이터와 규칙만 알고 있음
	
2. 뷰 (View)
	* 사용자에게 보여지는 화면
	* UI를 표현하는 데만 집중하고, 로직을 갖지 않음
	* ViewModel에서 제공하는 상태를 그대로 그려주기만 함
	
3. 뷰모델 (ViewModel)
	* View와 Model 사이의 **중재자(Mediator)** 역할
	* Model을 사용해 데이터를 가공하고, 상태를 @Published 같은 방식으로 노출
	* View는 이 상태를 구독하다가 값이 변하면 자동으로 UI를 갱신


![[MVVM Pattern.png]]


## 데이터 바인딩
---
MVVM에서 가장 중요한 개념 중 하나는 데이터 바인딩(Data Binding)이다.

데이터 바인딩은 View와 ViewModel을 자동으로 연결해주는 다리 역할을 한다. ViewModel에서 값이 바뀌면 View는 자동으로 화면을 업데이트하고, 사용자가 View에서 입력한 값은 자동으로 ViewModel로 전달된다.

SwiftUI에서는 이 개념을 더욱 단순하고 자연스럽게 표현된다. 예를 들어, TextField와 ViewModel의 username을 연결하면, 사용자가 텍스트를 입력할 때마다 ViewModel의 값이 업데이트되고, ViewModel 값이 바뀌면 TextField나 관련 UI도 자동으로 갱신된다.

```Swift
final class ViewModel: ObservableObject {
	@Published var username: String = ""
}

struct SomeView: View {
	@StateObject private var viewModel = ViewModel()
	
	var body: some View {
		VStack {
			TextField("이름 입력", text: $viewModel.username)
			Text("안녕하세요. \(username)")
		}
	}
}
```

이 예시처럼 SwiftUI와 Swift Concurrency (또는 Combine)은 MVVM의 데이터 바인딩을 언어 차원에서 지원해주기 때문에, MVVM의 모양은 쉽게 만들 수 있다.


## MVVM의 장단점
---
##### 장점

- **관심사 분리**: View는 화면 표현만, ViewModel은 상태와 로직만 → 코드가 깔끔해짐
- **테스트 용이성**: ViewModel은 UI와 독립적이므로 단위 테스트 작성이 쉬움
- **코드 재사용성**: 하나의 ViewModel을 여러 View에서 재활용 가능
- **가독성**: 데이터 바인딩으로 UI 업데이트 로직이 단순해져 코드량이 줄고 읽기 쉬워짐

##### 단점

- **소규모 앱엔 과함** → 단순한 앱은 MVC가 더 효율적일 수 있음
- **ViewModel 비대화 위험** → 로직이 몰리면 관리가 어려워짐
- **학습 난이도** → 데이터 바인딩, 비동기 처리 개념을 익혀야 함
