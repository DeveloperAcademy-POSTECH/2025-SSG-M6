>[!question]
>GQ1. 트랜스포머 모델이란?

## Description
- 기존의 순차적으로 요소들을 학습하는 AI 학습 모델과 달리, 입력 데이터를 병렬로 한번에 학습하는 AI 학습 모델
- 병렬로 학습하기에 학습속도가 빠르다.
- 자연어같이 문맥에 따라 의미가 크게 바뀔 수 있는 분야에서 문맥에 따른 학습에 용이하다.

## 주요 기능
+ 학습 과정에서 문장 내 단어들이 '서로가 서로에게 얼마나 중요한 지'에 집중합니다. 이 아이디어를 'Self-Attention'이라고 합니다.
+ 이는 단순히 자연어뿐만이 아닌 사진 상황 분석 등의 분야에서도 쓰일 수 있습니다.

## 코드 예시
transformer의 self-attention에 대한 설명을 위한 가벼운 예시입니다.
#### 실행 코드
```
import Foundation

// 임의의 단어 리스트
let sentence = ["나는", "밥을", "먹었다"]

// 임의의 '유사도' 행렬 (실제로는 학습됨)
let attentionScores: [[Double]] = [
    [1.0, 0.2, 0.1],   // "나는"이 각 단어에 집중하는 정도
    [0.1, 1.0, 0.8],   // "밥을"이 각 단어에 집중하는 정도
    [0.0, 0.4, 1.0]    // "먹었다"가 각 단어에 집중하는 정도
]

// Attention score를 softmax로 변환 (합이 1이 되게)
func softmax(_ scores: [Double]) -> [Double] {
    let expScores = scores.map { exp($0) }
    let sum = expScores.reduce(0, +)
    return expScores.map { $0 / sum }
}

// 단어별 attention 분포 출력
for (i, word) in sentence.enumerated() {
    let attn = softmax(attentionScores[i])
    print("\(word) attends to:", terminator: " ")
    for (j, score) in attn.enumerated() {
        print("\(sentence[j]): \(String(format: "%.2f", score))", terminator: "  ")
    }
    print("")
}
```
#### 출력 예시
```
나는 attends to: 나는: 0.71  밥을: 0.16  먹었다: 0.13  
밥을 attends to: 나는: 0.12  밥을: 0.44  먹었다: 0.44  
먹었다 attends to: 나는: 0.10  밥을: 0.27  먹었다: 0.63  
```

해석 : 
+ '밥을'은 '나는'보다 '먹었다'를 0.32정도 더 중요하게 생각한다.
+ '먹었다'는 '나는'보다 '밥을'을 0.17정도 더 중요하게 생각한다.
## Keywords
- self-attention

## References
- [블로그, 초등학생도 이해하는 자연어처리](https://codingopera.tistory.com/43)
- (CloudFlare)[https://www.cloudflare.com/learning/ai/what-is-large-language-model/]