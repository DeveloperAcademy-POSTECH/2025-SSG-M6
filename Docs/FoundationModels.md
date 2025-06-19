>[!question]
> GQ1. FoundationModels는 또 뭐야?
>GQ2. 어떻게 쓰는거야?

## Description
- 이번 WWDC25에서 Foundation Models라는 게 발표됐습니다.
- [[대규모 언어 모델(LLM)]]인데요.
- On-Device(서버 필요없음)이고, 앱 개발에 사용할 수 있는 Framework입니다.

## 주요 기능
+ 프롬프트를 읽고 알맞은 답변을 내놓습니다. (prompt, response)
+ 이전 프롬프트/응답을 기억합니다. (session, transcript)
+ 데이터 구조에 맞춘 응답 생성. (@Generable, @Guide)

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
        to: "서울로 떠나는 3일짜리 여행계획 짜줘.",
        generating: Itinerary.self
    )
    
    for try await partial in stream {
        print(partial)
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

## Keywords
+ [[Tool]]

## References
- https://developer.apple.com/documentation/foundationmodels
- https://machinelearning.apple.com/research/apple-foundation-models-2025-updates
- https://arxiv.org/abs/2407.21075
- https://developer.apple.com/videos/play/wwdc2025/286/
- https://developer.apple.com/videos/play/wwdc2025/301/
- https://developer.apple.com/kr/videos/play/wwdc2025/259/