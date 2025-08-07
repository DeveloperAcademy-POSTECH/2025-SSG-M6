>[!question]
>GQ1. 매크로 (Macro)란 무엇이고, 언제 사용하는 Swift 문법 개념일까?
>GQ2. Apple이 Swift 5.9에 매크로 (Macro)라는 문법 요소를 도입한 이유는 무엇일까?
>GQ3. 프로퍼티 래퍼 (Property Wrapper)와 매크로 (Macro)는 무엇이 다른 것일까?
>GQ4. 내 코드에서 매크로 (Macro)를 활용할 수 있는 방법은 무엇이 있을까?

## What is Macro?

- 매크로는 Swift 5.9 (WWDC 23)에 도입된 기능으로 컴파일 시간 (Compile Time)에 생성되는 코드 조각을 의미합니다.

- Swift는 이미 "Built-in code expansion", 즉 "컴파일 타임에 자동으로 코드가 확장되거나 생성되는 내장 기능"을 지원하고 있었습니다. ([[프로퍼티 래퍼 (Property Wrappers)]] @ViewBuilder와 같은 [[Result Builder]] 등이 해당되죠.)
- 하지만 Swift 컴파일러가 컴파일 시점에 기본으로 지원하지 않는 기능일 때, 
  직접 Swift Package로 배포하여 Built-in code expansion을 커스텀해 사용할 수 있도록 도입된 기능이 바로 매크로 (Macro)입니다.

![[매크로_기본 원리.png]]



## Macro Design philosophy (Goals)

Apple은 WWDC에서 매크로 (Macro)를 설계하면서 네 가지 목표에 집중했다고 합니다.
직접 Macro를 배포하는 경우, 아래 네 가지 원칙에 위배되지 않는지 고려하며 만들 필요가 있겠죠...?

#### 1. Distinctive use goals : 매크로를 사용할 때, 명확할 것

`#`나 `@` 기호를 보고 매크로 (Macro)가 사용되었구나!
더 자세하게는 `#` 키워드가 사용되었으니 Freestanding Macro이고, 여기는 `@` 키워드가 사용되었으니 Attached Macro구나! 라고 명확하게 사용시 인지할 수 있어야 한다는 의미입니다.

#### 2. Complete, type-checked, validated : 완전하고 검증된

매크로에 전달되거나, 매크로로부터 전달되는 코드가 완전해야 하고 오류가 없다는 것이 검증되어야한다는 의미입니다.
너무나도 당연한 내용이죠?

#### 3. Inserted in predictable ways : 예측 가능한

매크로는 프로그램의 가시적인 코드 (visible, 겉으로 명시적으로 드러나는 코드)에만 추가할 수 있습니다.
쉽게 말해, 구조체 (Struct)에서 자동으로 생성되는 initializer는 눈으로 보이지 않으니 매크로를 사용할 수 없다는 의미이죠!

이 원칙을 통해 매크로의 사용이 코드 자체로 예측 가능해야한다고 합니다.

아래 코드를 보면 `#someUnknownMacro()`는 무엇을 수행하는 녀석인지는 몰라도,
적어도 `startDoingThingy()`에 대한 동작이 끝난 이후, `finishDoingThingy()`에 대한 호출이 삭제되거나 중지되지 않는다는 것을 알 수 있죠?
이 정도의 예측 가능함 (predictable)을 Apple은 Macro 사용에 있어 소개합니다.

```Swift
func doThingy() {
	startDoingThingy()

	#someUnknownMacro()
	
	finishDoingThingy()
}
```

#### 4. Macros are not magic : 매크로는 마법이 아니다

매크로는 단지 프로그램에 코드를 추가하는 것을 의미합니다.
대단한 마법을 부릴 수 있는 것이 아니라, Xcode 내에서 확장을 통해 바로 추가된 코드를 확인할 수 있는 요소죠.



## 매크로 종류

#### Freestanding Macro (#)

```Swift
return #unwrap(icon, message: "should be in the app bundle")
```

- 코드 내의 표현식처럼 직접 호출해 사용합니다.
- 새로운 값을 표현 식으로 생성하거나, 독립적인 선언을 삽입하면 -> 그 결과가 코드로 확장되는 역할!
- Expression Macro와 Declaration Macro는 여기에 해당됩니다. (더 자세한 설명은 아래에!)

#### Attached Macro (@)

```Swift
@AddCompletionHandler func sendRequest() async throws -> Response
```

- 타입이나, 함수, 변수 등의 선언에 속성처럼 붙여서 사용합니다.
- 기존 타입에 선언되어있는 속성을 수정하거나, 새로운 기능을 추가할 수 있는 역할!



## 더 세부적인 매크로 종류 (Macro roles)

구현부 없이... 러프한 느낌만 봐주시죠
구현부는 [[SwiftSyntax]]에 대한 학습이 필요해서... 다음주에 설명한 다음에 이어서 설명하도록 할게요!

##### @freestanding(expression) 
: 표현식처럼 사용되어 값을 반환하는 매크로

```Swift
// Optional unwrapping할 값과 failure 처리를 위한
let image = #unwrap(downloadImage, message: "was already checked")

// Macro 선언
@freestanding(expression)
macro unwrap<Wrapped>(_ expr: Wrapped?, message: String) -> Wrapped
```

##### @freestanding(declaration) 
: 하나 이상의 선언 (함수, 변수, 타입 등)을 생성하는 매크로

```Swift
// N차원 배열의 struct를 자동으로 만들어주는
#makeArrayND(n: 2)
#makeArrayND(n: 3)
#makeArrayND(n: 4)
#makeArrayND(n: 5)

// Macro 선언
@freestanding(declaration, names: arbitrary)
macro makeArrayND(n: Int)
```

---
##### @attached(peer) 
: 적용된 선언 (함수, 변수, 타입 뿐만 아니라 import나 연산자 선언 등도 포함)과 함께 새로운 선언을 추가하는 매크로

```Swift
// async 함수에도 completion Handler를 지원하는
@AddCompletionHandler(parameterName: "onCompletion")
func fetchAcatar(_ username: String) async -> Image? { ... }

// Macro 선언
@attached(peer, names: overload) 
macro AddCompletionHandler(parameterName: String = "completionHandler")
```

##### @attached(accessor) 
: 프로퍼티에 접근자 (get, set, didSet, willSet)를 추가하는 매크로

```Swift
// 각 Dictionary의 필드에 get/set 추가
struct Person: DictionaryRepresentable {
	init(dictionary: [String: Any]) { self.dictionary = dictionary }
	var dictionary: [String: Any]

	@DictionaryStorage var name: String
	@DictionaryStorage var height: Measurement<UnitLength>
	@DictionaryStorage(key: "birth_date") var birthDate: Date?
}

// Macro 선언
@attached(accessor) 
macro DictionaryStorage(key: String? = nil)
```

>[!question] 프로퍼티 래퍼 (Property Wrapper)를 사용하지 않고, 매크로 (Macro)를 사용한 이유는?
>일단 프로퍼티 래퍼는 런타임에 동작하고, 매크로는 컴파일 시점에 코드가 생성된다는 점에서 다릅니다.
>
>그것보다 더 명확한 차이점은
>프로퍼티 래퍼는 값을 감싸고 있는 구조 -> 그 자체가 독립적인 타입이고, 프로퍼티 래퍼로 선언되면, 프로퍼티 래퍼 타입 인스턴스 내부에 저장됩니다.
>하지만, 매크로는 코드가 생성되는 구조 -> self에도 접근할 수 있는 등 외부 참조가 가능하죠.
>
>위 경우에는 dictionary라는 (프로퍼티 래퍼 입장에서) 외부 데이터 저장소에 의해 name, height, birthDate를 접근하는 것이지만,
>그 자체가 독립적인 저장소인 프로퍼티 래퍼를 사용하는 경우 / 이 외부 데이터 저장소 dictionary에 접근할 수가 없는 것입니다!

##### @attached(memberAttribute) 
: 적용된 타입 또는 익스텐션의 선언에 다른 속성을 추가하는 매크로

```Swift
// 각 Dictionary의 필드에 get/set 추가 -> 타입 자체에 붙여서 사용하기
@DictionaryStorage
struct Person: DictionaryRepresentable {
	init(dictionary: [String: Any]) { self.dictionary = dictionary }
	var dictionary: [String: Any]

	var name: String
	var height: Measurement<UnitLength>
	@DictionaryStorage(key: "birth_date") var birthDate: Date?
}

// Macro 선언
@attached(memberAttribute)
@attached(accessor) 
macro DictionaryStorage(key: String? = nil)
```

>[!question] 하나의 매크로가 여러 Role을 갖는 것이 가능한 것인가?
>Attached Macro (@)는 여러 개를 조합할 수 있고, Freestanding Macro (#)는 조합할 수 없습니다.
>
>Macro에 적용된 Role은 모두 맥락에 맞춰 확장됩니다.
>여기서 말하는 맥락은 타입 자체에 적용된다던지 / 프로퍼티에 적용된다던지 등을 의미하는 것이죠!

##### @attached(member) 
: 적용된 타입 또는 익스텐션 내부에 새로운 선언 (메서드, 프로퍼티, initializor 등)을 추가하는 매크로

```Swift
// N차원 배열의 struct를 자동으로 만들어주는 + 그런데 init이랑 dictionary도 생략하는
@DictionaryStorage
struct Person: DictionaryRepresentable {
	var name: String
	var height: Measurement<UnitLength>
	@DictionaryStorage(key: "birth_date") var birthDate: Date?
}

// Macro 선언
@attached(member, names: named(dictionary), named(init(dictionary:)))
@attached(memberAttribute)
@attached(accessor) 
macro DictionaryStorage(key: String? = nil)
```

##### @attached(extension) 
: 타입 또는 익스텐션에 프로토콜 적합성을 추가하는 매크로

```Swift
// N차원 배열의 struct를 자동으로 만들어주는 + 그런데 init이랑 dictionary도 생략하는 + 그런데 DictionaryRepresentable Protocol도 자동으로 추가된
@DictionaryStorage
struct Person {
	var name: String
	var height: Measurement<UnitLength>
	@DictionaryStorage(key: "birth_date") var birthDate: Date?
}

// Macro 선언
@attached(extension)
@attached(member, names: named(dictionary), named(init(dictionary:)))
@attached(memberAttribute)
@attached(accessor) 
macro DictionaryStorage(key: String? = nil)
```


## References

- [Swift Documentation - Macro](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/macros/)
- [WWDC23 - Write Swift Macros](https://developer.apple.com/kr/videos/play/wwdc2023/10166/)
- [WWDC23 - Expand on Swift macros](https://developer.apple.com/videos/play/wwdc2023/10167/)
- [Medium - Najin, Swift 매크로](https://sujinnaljin.medium.com/swift-%EB%A7%A4%ED%81%AC%EB%A1%9C-5e232b78dc5b)
- [매크로 부수고 부서지기 - KWDC23](https://www.youtube.com/results?search_query=swift+%EB%A7%A4%ED%81%AC%EB%A1%9C+kwdc)