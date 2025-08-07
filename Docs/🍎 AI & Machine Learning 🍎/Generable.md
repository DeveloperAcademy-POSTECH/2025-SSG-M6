>[!question]
>GQ1. Guided Generation을 잘 활용해서 Foundation Models를 잘 쓰기 위해서는 @Genrable 매크로를 어떻게 활용해야할까?
>GQ2. Generable은 어떤 기능이 있지?

## Description

- [[Foundation Models]]의 응답값을 받을 때 원하는 구조에 맞춰 답을 받을 수 있는데요. 그때 그 구조를 설정할 때 붙이는 매크로가 `@Genreable`입니다.
  -> Generable is an easy way to let the model generate structured data, using Swift types.


## 주요 기능

+ `@Generable` 매크로를 붙여 LLM이 생성하는 결과를 구조화되도록 만들 수 있습니다. (자유 형식으로 응답 x)
+ `@Generable`은 구조체 (struct)와 열거형 (enum)에 사용할 수 있습니다.
+ `@Generable`로 선언된 타입 내부 프로퍼티를 `@Guide` 매크로를 활용해, 원하는 특성이나 값을 구체화할 수 있습니다.


## Code

+ @Generable로 선언한 구조는, 요청시 `respond(generating:)` 파라미터로 요청시 보낼 수 있습니다.
+ `response.content`는 `@Generable`로 선언한 NPC 타입으로 들어오기에, 별도의 파싱이 필요없습니다.

```Swift
@Generable 
struct NPC {
	let name: String
	let coffeeOrder: String
}

func makeNPC() async throws -> NPC {
	let session = LanuageModelSession(instructions: ...)
	let response = try await session.respond(generating: NPC.self) {
		"Generate a character that orders a coffee."
	}
	return response.content
}
```

- `@Generable`의 속성은 `@Generable` 타입으로 받아오는 것 또한 가능합니다.
- `@Guide`를 통해서 속성의 설명이나 제약을 주는 것이 가능합니다.

```Swift
@Generable 
struct NPC {
	@Guide(description: "A full name")  // 프롬프트에 속성을 설명하는 방법
	let name: String
	
	@Guide(.range(1...10))  // 프롬프트에 값의 범위 지정
	let level: Int
	
	@Guide(.count(3))  // 프롬프트에 개수 지정
	let attributes: [Attribute]
	
	let encounter: Encounter

	@Generable
	enum Attribute {
		case sassy
		case tired
		case hungry
		...
	}
	
	@Generable
	enum Encounter {
		case orderCoffee(String)
		case wantToTalkToManager(complaint: String)
	}
}
```



## @Guide

+ **숫자 타입 (Int/Float/Double/Decimal)** 
  : 최소값 (`.minimum(1)`), 최댓값 (`.maximum(10)`), 범위 (`.range(1...10)`)를 지정할 수 있습니다.
  
+ **배열 타입 (Array)**
  : 개수 제어 (`.minimumCount(1)` `.maximumCount(10)` `.count(3)`), element 가이드 (`.element(.minimum(1))`)를 지정할 수 있습니다.

- **문자열 타입 (String)**
  : 고정값 (`.constant("NPC")`), 배열 중 값 선택 (`.anyOf(["barista", "guest"])`), 정규 표현식을 사용한 패턴 (`.pattern(/(Mr|Mrs)\. \w+/)`) 등을 지정할 수 있습니다.
  -> 정규 표현식에 대한 설명은 [WWDC22 - Meet Swift Regex](https://developer.apple.com/videos/play/wwdc2022/110357/) 영상을 참고하실 수 있습니다!



## References

- [Apple Developer Documentation - Generable](https://developer.apple.com/documentation/foundationmodels/generable)