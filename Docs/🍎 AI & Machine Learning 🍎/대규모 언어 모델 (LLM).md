>[!question]
>GQ1. LLM이란 뭘까?

## Description
- [[대규모 언어 모델 (LLM)]]은 텍스트를 인식하고 생성하는 등의 작업을 수행할 수 있는 일종의 [[인공 지능 (AI)]]프로그램입니다. LLM은 방대한 데이터 세트를 학습하므로 "대규모"라는 이름이 붙었습니다. 
- LLM은 [[머신 러닝 (Machine Learning)]], 특히 [[트랜스포머 모델 (Transformer Model)]]이라고 하는 일종의 신경망을 기반으로 합니다.
- LLM은 인간의 언어나 기타 복잡한 데이터를 인식하고 해석할 수 있을 만큼 충분한 예제를 제공받은 컴퓨터 프로그램입니다. 많은 LLM은 인터넷에서 수집된 수천 또는 수백만 기가바이트에 달하는 텍스트로 학습됩니다. 
- 그런 다음 튜닝을 통해 추가 학습이 이루어집니다. 질문을 해석하고 답변을 생성하거나 한 언어에서 다른 언어로 텍스트를 번역하는 등 프로그래머가 원하는 특정 작업에 맞게 미세 조정되거나 프롬프트 조정됩니다.

## 주요 기능
+ 감정 분석
- DNA 연구
- 고객 서비스
- 챗봇
- 온라인 검색

## 코드 예시
```
import OpenAIKit            // https://github.com/MacPaw/OpenAI
import Foundation

@main
struct DemoLLM {
    static func main() async throws {
        let client = OpenAI(apiToken: ProcessInfo.processInfo.environment["OPENAI_KEY"]!)

        let chat = Chat(messages: [
            .system("요점만 간단히 대답하기"),
            .user("LLM이 뭐야?")
        ])

        // 스트리밍으로 출력 받기—콘솔에 바로 print
        for try await chunk in client.chats.stream(chat) {
            if let delta = chunk.choices.first?.delta.content {
                print(delta, terminator: "")
            }
        }
    }
}
```

## Keywords
+ 인공 지능(AI)
+ 머신 러닝(Machine Learning)
+ 트랜스포머 모델(Transformer Model)

## References
- [Github MacPow/OpenAI](https://github.com/MacPaw/OpenAI)
- [CloudFlare](https://www.cloudflare.com/learning/ai/what-is-large-language-model/)