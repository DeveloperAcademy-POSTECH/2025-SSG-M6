>[!question]
> GQ1. Foundation Models란 또 뭐야. Apple은 왜 이 Model을 만든거지?
>GQ2. Foundation Models 어떻게 쓰는거야?
>GQ3. Foundation Models 어디에 쓰는거야?

![[Foundation Models.png]]

## Description

- 이번 WWDC25에서 Foundation Models라는 게 발표됐습니다.
- [[대규모 언어 모델(LLM)]]인데요. On-Device (별도의 서버 필요없음, 보안 안전, 앱 용량을 차지하지 않음)이고, 앱 개발 (macOS, iOS, iPadOS, visionOS)에  사용할 수 있는 Framework입니다.


## 주요 기능

+ 프롬프트를 읽고 알맞은 답변을 내놓습니다. (prompt, response)
+ **Stateful sessions** : 이전 프롬프트/응답을 기억합니다. (session, transcript)
+ **Guided Generation** : 데이터 구조에 맞춘 응답 생성. ([[@Generable]] - 모델이 생성할 타입, `@Guide` - 타입 속성의 가이드)

>[!question] 왜 Guided generation이 필요한거야?
> 언어 모델은 기본적으로 "비구조화" 되어있는 자연어를 답변으로 출력합니다.
> 즉, 사람이 봤을 때는 그냥 글이라 읽기는 쉽지만, 이 응답값을 뷰에 바로 적용하기에는 어려운 점이 있죠.
>
>이 문제를 해결하기 위해서는 프롬프트에 "어떠어떤 형태 (json과 같은..)로 답변을 만들어줘"와 같이 요청을 보낼 수 있지만,
>프롬프트 자체가 길어지고 / 모델의 생성값이 확률에 의존하고 있다는 문제가 있죠. (이 형태가 아닐 수도..!)
>
>그래서 애초부터 모델의 응답을 뷰에 적용하기 쉽고, 프롬프트의 길이를 줄이고, 응답 구조의 동일 안정성을 보장하기 위해 `@Generable`과 `@Guide` 매크로를 지원하게 된 것입니다 !

- 스트리밍 출력시 토큰이 델타 방식 (작은 문자 단위가 순차적으로 방출) 아니라, Snapshot (부분적으로 생성된 응답을 방출) 방식을 채택합니다.
  -> SwiftUI 선언형 프레임워크 방식에서 UI의 업데이트를 애니메이션과 함께 자연스럽게 보여주기 적합한 방식 !
  -> Swift 구조체 (struct)에 선언된 순서대로 값이 생성됩니다. (한번에 빡! 이 아니라 순서대로 쫘라락~)

	![[Foundation Models_Snapshot Streaming.png]]

- Tool calling : 모델에 [[Tool]]을 활용해서 추가적인 기능을 붙여 활용할 수 있습니다.
  (World knowledge, Recent Events, Personal Data와 같은 온디바이스 모델의 약점을 보완하는 기능)
  -> 예를 들어, 여행 앱에 붙이는 Foundation Models에는 MapKit을 통해 현재 위치를 받아올 수 있는 것!



## 코드 예시

```swift
import Playgrounds
import FoundationModels

#Playground {
    // 간단한 프롬프트 및 응답
    let session = LanguageModelSession()
    let response = try await session.respond(to: "프랭크는 미니의 룸메다.")

    // 맥락을 기억하는 지 테스트 해봄
    let response2 = try await session.respond(to: "프랭크의 룸메는 누구일까?")

    // 데이터 구조에 맞는 응답 생성하기
    let stream = session.streamResponse(
        to: "서울로 떠나는 3일짜리 여행계획 짜줘.",  // 프롬프트
        generating: Itinerary.self          // 희망 응답타입
    )
    
   // Snapshot streaming 방식으로 값 방출하기
    for try await partial in stream {
        print(partial)
        // Itineray.PartiallyGenerated(name: nil, days: nil)
        // Itineray.PartiallyGenerated(name: "Mt.", days: nil)
        // Itineray.PartiallyGenerated(name: "Mt. Fuji", days: nil)
        // Itineray.PartiallyGenerated(name: "Mt. Fuji", days: [])
        // Itineray.PartiallyGenerated(name: "Mt. Fuji", days: [Day.PartiallyGenerated(...)])
    }
}

@Generable
struct Person {
    var name: String
    var phone: String
}

@Generable
struct Itinerary {
    var destination: String

    @Guide(description: "이 여행 계획에 대한 한줄요약") // 이 값에 대한 프롬프트랄까
    var description: String

    @Guide(.maximum(12)) // 최대 12일
    var days: Int

    @Guide(.range(200000...500000)) // 20 ~ 50만원
    var budget: Int

    @Guide(.range(3...5)) // 3점이상 5점이하
    var rating: Double
    
    @Guide(.count(3)) // 액티비티는 3개만
    var activities: [String]
    
    var emergencyContact: Person
} 
```
+ 추신. MacOS 26 beta 대상으로 지원되는 코드입니다.



## 그래서 이 Foundation Models 어디에 활용할 수 있는데?

WWDC25에서 Apple은 해당 Foundation Models를 활용할 수 있는 사례들에 대해 소개하고 있습니다.

> It's not designed for world knowledge (세계 지식, 상식) or advanced reasoning (복잡한 문제 추론),
> which are tasks you might typically use server-scale LLMs for

눈 여겨 볼만한 점은 Foundation Models를 활용해서 새로운 앱을 개발하는 것보다,기존 앱의 기능을 강화하기 위한 목적으로 모델을 소개하고 있다는 점이네요. 
-> 앱 용량 차지 X 별도 서버 X 나는 온-디바이스 모델이다 ! 라는 점을 강조하기 위함인 것 같습니다.

총 14개 사례인데, 더 확장해볼 수도 있을 거구요. 
처음 볼 때는 와 진짜 대박이다라고 느겨졌는데, 보면 볼수록 ~~성능의 부족함을 온디바이스로 매꾸는 느낌이랄까..~~

- **콘텐츠 생성 (Content generation)** : 자연어를 활용해 앱에서 보여주는 용도의 새로운 콘텐츠를 생성하는 기능
- **생성형 대화 (Generative dialog)** : 사람과 자연스럽게 대화를 주고받을 수 있는 챗봇 (단순 질의응답을 넘어서 맥락을 이해하는 대화를 이어나갈 수 있음. ex 개인 비서) 
- **앱 내 사용자 가이드 (In-app user guides)** : 사용자가 앱을 어떻게 사용할지 자연어로 설명하는 가이드 기능
- **생성형 퀴즈 (Generative quizzes)** : 앱 내 텍스트나 문서를 분석해 자동으로 문제를 만들어주는 기능
- **개인화 (Personalization**) : 사용자 선호, 히스토리, 위치, 취향 등을 학습해 맞춤형 컨텐츠를 제공하는 기능
- **분류 (Classification)** : 입력된 데이터를 카테고리로 자동 분류
- **요약 (Summarization)** : 긴 텍스트를 짧게 요약해주는 기능
- **의미 기반 검색 (Semantic search)** : 키워드로 단순 동일함을 매칭해주는 것이 아니라, 문장의 의미와 맥락을 이해해서 검색해주는 기능
- **개선 (Refinement)** : 이미 존재하는 앱 내의 텍스트를 더 자연스럽게 혹은 더 정중하게 등의 요구대로 다시 작성해서 보여줄 수 있음
- **질문 답변 (Question answering)** : 사용자가 물어보는 질문에, 텍스트를 읽고 정확하게 답변을 해줌.
  (ex. 계약서 텍스트를 넣고, "독소 조항이 될 수 있는 내용은 뭐야?"라고 물으면 해당 부분 답변해줌.)
- **맞춤 학습 (Customized learning)** : 학습자의 실력, 스타일, 목표에 맞춘 교육 컨텐츠를 자동 생성해줌
- **태그 생성 (Tag generation)** :텍스트로부터 키워드/태그를 자동 추출해주는 기능
  (ex. 블로그 글에서 내용을 읽고 iOS, Swift, WWDC25 등의 키워드를 추출해주는 상황)
- **엔티티 추출 (Entity extraction)** : 텍스트로부터 구체적인 정보를 엔티티화해서 뽑아내기
  (ex. 2025년 3월 10일 포항에서 애플 디벨로퍼 아카데미가 시작되었다. -> 날짜: 2025년 3월 10일, 위치: 포항, 이벤트: 애플 디벨로퍼 아카데미 시작)
- **주제 감지 (Topic detection)** : 현재 문서, 자연어 등이 무엇에 관한 내용인지 주제를 자동 판별해주는 기능


## Availability

- Foundation Models는 Apple Intelligence를 지원하는 기기와 지역에서만 활용할 수 있는 기능입니다.
그래서 기능을 제공하기 전 사용 가능 여부를 반드시 확인해야 하죠.

```Swift
struct AvailabilityExampleView: View {
	private let model = SystemLanguageModel.default  // 해당 속성 접근해서 확인

	var body: some View {
		switch model.availability {
		case .available:
			Text("Model is available").foregroundStyle(.green)
		// reason에는 .appleIntelligenceNotEnabled .modelNotReady .deviceNotEligible 등으로 분기처리를 할 수 있습니다.
		case .unavailable(let reason):
			Text("Model is available").foregroundStyle(.red)
			Text("Reason: \(reason)")
		}
	}
}
```

- 또한 모델을 호출할 때 발생할 수 있는 에러 (`GenerationError`)도 존재합니다. 분기 처리를 역시 해줘야합니다.
	- **Guardrail violation** : `guardrailViolation`
	  : 부적절한 컨텐츠나 보안상 민감 정보 등 Foundation Models가 생성해서는 안되는 자연어의 경우 반환되는 에러
	  * 생성형 모델의 안정성에 대한 내용은 [Apple Developer Documentation - Improving safety from generative model output](https://developer.apple.com/documentation/foundationmodels/improving-safety-from-generative-model-output?changes=_10_5) 아티클 내용 참조 !
	  
	- **Unsupported language** : `unsupportedLanguageOrLocale`
	  : Foundation Models가 지원하지 않는 언어 (현재는 한국어, 영어, 일본어, 중국어-간체 등 사용할만한 언어를 모두 지원해서 크게 신경쓸 필요는 없을 듯!)

```Swift
// 언어 지원 확인하는 예제 코드
var session = LanguageModelSession()

do {
    let response = try await session.respond(to: userInput)
    print(response.content)
} catch LanguageModelSession.GenerationError.unsupportedLanguageOrLocale {
	// 언어를 지원하지 않는 경우 발생시킬 에러 처리
}

let supportedLanguages = SystemLanguageModel.default.supportedLanguages
guard supportedLanguages.contains(Locale.current.language) else {
	// 메시지 보여주기 코드 추가
	return
}
```

	- **Context window exceeded** : `exceededContextWindowSize`
	  : 이전 맥락을 기억할 수 없는 상황 (LLM은 내부적으로 제한된 기억 버퍼를 가지기 때문. 과거 맥락을 짤라냄.)

>[!question] exceededContextWidowSize Error가 발생했을 때 그럼 어떻게 처리를 해야해?
> `GenerationError.exceededContextWindowSize` 에러에 대해 분기 처리를 해주고, 별도의 session을 다시 시작하는 방법이 있습니다.
> 하지만, 이런 경우 새로 시작된 session은 이전 대화 기록을 모두 잊게 된다는 문제가 발생하게 되죠.
> 
> 이 경우 두 가지 방법을 수행할 수 있습니다.
> 1. 이전 Session의 Entry 첫 번째 (instructions)와 마지막 값 (Response)을 새로운 Session의 transcript로 전달한다.
> 2. 이전 Session의 모든 transcript를 요약해서 (Summarize) 새로운 Session의 trnascript로 전달한다.

```Swift
// 1번 case 예시 코드
var session = LanguageModelSession(instructions: "블라블라")

do {
    let response = try await session.respond(to: prompt)
    print(response.content)
} catch LanguageModelSession.GenerationError.exceededContextWindowSize {
	session = LanguageModelSession(instructions: "블라블라")
}

private func newSession(previousSession: LanguageModelSession) -> LanguageModelSession {
	// 엔트리란 사용자가 모델이 입력을 주는 한번의 요청 (allEntries는 이전 세션의 모든 요청)
	let allEntries = previousSession.transcript.entries
	var condencedEntries = [Transcript.Entry]()

	// 첫번째 Entry인 instruction과 마지막 Entry인 응답값 (Response)을 가져옴
	if let firstEntry = allEntries.first {
		condencedEntries.append(firstEntry)
		if allEntries.count > 1, let lastEntry = allEntries.last {
			condencedEntries.append(lastEntry)
		}
	}

	// 이전 Entry에서 추출한 값들을 새로운 Session으로 정보를 전달함
	let condensedTranscript = Transcript(entries: condencedEntries)
	return LanguageModelSession(transcript: condensedTranscript)
}
```


## Keywords

![[Foundation Models_Core Features.png]]

+ [[Tool]]


## References

- https://developer.apple.com/documentation/foundationmodels
- https://machinelearning.apple.com/research/apple-foundation-models-2025-updates
- https://arxiv.org/abs/2407.21075
- https://developer.apple.com/videos/play/wwdc2025/286/
- https://developer.apple.com/videos/play/wwdc2025/301/
- https://developer.apple.com/kr/videos/play/wwdc2025/259/