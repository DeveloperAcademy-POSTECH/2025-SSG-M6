>[!question]
>GQ1. MLX가 대체 뭘까용?
>GQ2. MLX.. python도 있고 swift도 있고.. 어디서 불러오는 방식도 요상하고..

## Description
MLX가 대체 뭔지 너무 궁금했습니다.  벌써 [[CoreML]] 다음의 머신러닝 프레임워크인가?

MLX는 애플에서 만들고 애플 깃허브에 올린 공식 프로젝트긴 합니다. 다만 [[CoreML]], SwiftUI 등등 처럼 Apple 플랫폼 개발을 위한 공식 SDK 프레임워크는 아닙니다.

이 점은 MLX라는 이름에서도 알 수 있는데요. Machine Learning eXperimentation의 약자로 애초에 실험용이라는 얘기가 있습니다.

#### MLX는?
첫 출발은 python의 numpy같은 다차원 배열(Tensor) 연산 프레임워크였습니다. 언어도 같은 python이었구요. 다른 점은 Apple Silicon활용에 최적화되어 있다는 점입니다.
이후 업데이트를 지속하면서 Pytorch, TensorFlow가 가지고 있는  머신러닝 모델 학습을 위한 기능들이 추가되었습니다.
그리고 마침내 Swift도 지원하게 업데이트되면서 Swift언어만을 활용해 머신러닝 연구를 진행할 수 있게 되었습니다.

## 주요 기능
+ Tensor 연산 (NDArray)
+ 자동 미분
+ Optimizer (SGD, Adam 등)
+ Layer 정의 (Linear 등)
+ GPU 가속 (M 시리즈)
+ Lazy evaluation
+ 모델 학습 / 추론
+ LLM / Diffusion 예제

이런 게 있다고 하는데요. 너무 어렵습니다.
그냥 주요 기능은 기존 Python 생태계에서의 머신러닝 프레임워크(Pytorch, TensorFlow)와 거의 유사하다고만 이해했습니다.

#### 주목할만한 기능
Apple ML LAB에서는 머신러닝 모델 플랫폼 Hugging Face에 여러 종류의 모델들을 꾸준히 업로드하고 있는데요.
MLX에는 이 모델을 허깅페이스 URL쿼리만을 통해 프로젝트에 바로 불러오는 기능이 있습니다.
이 기능은 MLX 오픈소스 예제에도 탑재되어있고, 최근 세간에 큰 주목을 받은 FastVLM 프로젝트에도 탑재되어 있습니다.

#### 있을 법한데 없는 기능
MLX로 학습한 모델을 CoreML모델로 변환하는 기능이 없습니다.
Apple 생태계 디바이스에서 효율적인 모델 활용을 위해선, ML모델을 CoreML모델로 변환하며 해주는 최적화가 반드시 필요한데요. 이 과정이 공식적으로 지원되지 않아 쉽지 않다고 합니다. 

*MLX는 실험용이라고 선긋느라 일부러 지원하지 않는 다는 소리가 있긴 하던데, 솔직히 나중가면 다 지원해줄 것 같긴합니다.
배포용 앱을 위한 온디바이스 모델은 기존 Python 생태계에서 모델을 생성 후 변환하고 있다고 합니다.

## 코드 예시
#### 1. 배열, 수치 연산
```
import MLX               // 배열
import MLXRandom         // 난수

let a = MLXRandom.randn([3, 3])
let b: MLX = 2
let z = (a + b).relu()           // 연산은 파이프라인처럼 체인
print(z.mean().item(Float.self)) // 필요할 때만 평가 → lazy-JIT
```
#### 2. 온디바이스 LLM - LLMEval 중 일부
```
import MLXLLM          // 예제 repo 안의 서브패키지
import MLXRandom

// ① 모델 + 토크나이저 자동 다운로드 (Hugging Face)
//    원하는 모델 ID만 바꿔 끼우면 끝.
let (model, tokenizer) = try LLM.from(
    pretrained: "mlx-community/phi2-mlx", // 위에서 언급한 허깅페이스 쿼리 ㅎㄷㄷ
    dtype: .float16   // M-시리즈 GPU에 맞춰 FP16
)

// ② 프롬프트 토큰화
let prompt = "Instruct: Explain Swift Concurrency in 3 lines."
var tokens = MLXArray(tokenizer.encode(text: prompt))

// ③ 토큰 스트림 생성
for token in TokenIterator(
        prompt: tokens, model: model, temperature: 0.8, maxTokens: 64) {

    let nextId = token.item(Int.self)
    if nextId == tokenizer.eosTokenId { break }
    print(tokenizer.decode(tokenIds: [nextId]), terminator: "")
}
print()  // generated text 완료
```
## Keywords
+ [[CoreML]]

## References
- https://devocean.sk.com/blog/techBoardDetail.do?ID=165572
- https://www.swift.org/blog/mlx-swift/?utm_source=chatgpt.com
- https://opensource.apple.com/projects/mlx/
- https://github.com/apple/ml-fastvlm
- https://github.com/ml-explore/mlx?tab=readme-ov-file
- https://e-string.com/articles/who-is-apples-mlx-for?utm_source=chatgpt.com
- https://huggingface.co/apple