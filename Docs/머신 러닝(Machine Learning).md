>[!question]
>GQ1. 머신 러닝(Machine Learning)이란?

## Description
- 머신 러닝은 AI의 한 분야로, 구조화되거나 레이블이 지정된 데이터를 프로그램에 입력하여 사람의 개입 없이 해당 데이터를 식별하는 방법을 프로그램에 학습시키는 작업을 말합니다.
+ 머신 러닝은 알고리즘이라고 하는 사전 정의된 프로세스의 사용에 의존합니다. 머신 러닝 프로그램은 알고리즘 설정 방법에 따라 약간 다르게 "학습"합니다.

## 주요 기능
+ 오브젝트 감지
+ 다음 컨텐츠 추천
+ 음성 인식

## 코드 예시

### Core ML 오브젝트 디텍션 예시
```
import Vision
import CoreML
import UIKit

let model = try! VNCoreMLModel(for: YOLOv3().model)
let request = VNCoreMLRequest(model: model) { request, _ in
    guard let results = request.results as? [VNRecognizedObjectObservation] else { return }
    for object in results {
        print("감지된 오브젝트: \(object.labels.first?.identifier ?? "unknown")")
    }
}
let handler = VNImageRequestHandler(cgImage: image.cgImage!, options: [:])
try? handler.perform([request])
```

### MLX로 LLM 실행 (python)

```
import mlx.core as mx
from mlx_lm import load, generate

model, tokenizer = load("mlx-community/Meta-Llama-3-8B-Instruct-4bit")
prompt = "What is machine learning?"
response = generate(model, tokenizer, prompt)
print(response)
```

## Keywords
+ CoreML
+ MLX

## References
- [CloudFlare](https://www.cloudflare.com/learning/ai/what-is-artificial-intelligence/)