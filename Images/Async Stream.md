>[!question]
>GQ1. 끝도 없이 들어오는 연속적인 데이터를 어떻게 다루면 좋을까?

## Description
- 모든 데이터를 처리하기엔 데이터 입력 속도가 처리보다 빨라요.
- 그럴땐 Async Stream
## 주요 기능
+ Async Stream을 만든다.
+ 거기에 연속적인 데이터를 집어넣는다.
+ Async Stream에 들어있는 데이터를 하나하나 느긋하게 꺼내서 처리한다.

## 코드 예시
```swift
// Async Stream을 만든다. continuation은 stream에 데이터 집어넣는 통로.
var exampleStream = AsyncStream<Int>?
var exampleContinuation = AsyncStream<Int>.Continuation?

exampleStream = AsyncStream<Int> { continuation in
	exampleContinuation = continuation
}

.
.
.

// 어딘가에서 데이터 집어넣는 예시
Task {
	for i in 1...5 {
	// yield << 집어넣는 함수
		exampleContinuation.yield(i)
		try? await Task.sleep(for: .seconds(1))
	}
	// 이제 통로 닫기
	continuation.finish()
}

.
.
.

// 또 다른 어딘가에서 집어넣어진 데이터를 꺼내서 처리하는 예시
Task {
	// exampleStream에서 하나씩 끄집어내는 코드
    for await value in exampleStream {
        print(value)
    }
    print("✅ 다 끝났음") // finish 실행없으면 이거 실행 안 되고 for await에서 대기해요.
}

```

## Keywords
+ [[Async Stream]]
+ [[Await]]

## References
- [카메라 프레임 처리에 기깔나게 AsyncStream을 사용한 레포를 발견했습니다.](https://github.com/DeveloperAcademy-POSTECH/2025-C4-M13-Visionable) 
  