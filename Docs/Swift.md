>[!question]
>GQ1. Swift는 무엇일까?
>GQ2. Swift는 언제 만들어졌을까?
>GQ3. 



## Description

**Swift**는 Apple이 2014년에 처음 발표한 프로그래밍 언어로, iOS, macOS, watchOS, tvOS 등 Apple 플랫폼에서 주로 사용된다. Swift는 빠르고 안전하며, 모던한 문법을 통해 사용자가 쉽게 이해하고 사용할 수 있도록 설계되었음. 기존의 Objective-C 언어를 대체할 목적으로 개발되었으며, 다음과 같은 주요 특징을 가지고 있다.

- **안전성 (Safety)**: 강력한 타입 검사와 [[옵셔널(Optionals)]]을 통해 널 포인터 에러를 방지할 수 있음
    
- **성능 (Performance)**: [[LLVM 컴파일러]]를 통해 빠른 코드 실행을 지원
    
- **현대적 문법 (Modern Syntax)**: [[클로저 (Closure)]], [[프로토콜 지향 프로그래밍 (POP)]], [[제네릭 (Generic)]], [[열거형 (Enumeration)]] 등의 최신 프로그래밍 개념을 지원
    
- **오픈 소스 (Open Source)**: Swift는 오픈 소스로 제공되며, 활발한 커뮤니티에서 발전하고 있음

## 주요 기능

1. [[Swift Standard Library]]
2. [[병렬 처리(Swift-Concurrency)]]
3. [[정규식 DSL]]
4. [[데이터 모델링 (Data Modeling)]]



## 코드 예시
```swift
// 공식 문서의 Swift 소개 코드
var interestingNumbers = [
    "primes": [2, 3, 5, 7, 11, 13, 17],
    "triangular": [1, 3, 6, 10, 15, 21, 28],
    "hexagonal": [1, 6, 15, 28, 45, 66, 91]
]

for key in interestingNumbers.keys {
    interestingNumbers[key]?.sort(by: >)
}

print(interestingNumbers["primes"]!)
// Prints "[17, 13, 11, 7, 5, 3, 2]"

```

## Keywords
+ [[옵셔널(Optionals)]]
+ [[LLVM 컴파일러]]
+ [[클로저 (Closure)]]
+ [[프로토콜 지향 프로그래밍 (POP)]]
+ [[제네릭 (Generic)]]
+ [[열거형 (Enumeration)]]
+ [[Swift Standard Library]]
+ [[병렬 처리(Swift-Concurrency)]]
+ [[정규식 DSL]]
+ [[데이터 모델링 (Data Modeling)]]

## References
- [Apple 공식 Swift 문서](https://developer.apple.com/documentation/swift)
- [Swift 오픈 소스 GitHub 저장소](https://github.com/apple/swift)
- [Swift.org - Open Source Project](https://swift.org/)