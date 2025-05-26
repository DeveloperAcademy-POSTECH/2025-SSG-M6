>[!question]
>GQ1. 소유권(Ownership)과 빌림(Borrowing)은 어떻게 다른 것일까?
>GQ2. 메모리 안전성을 보장하기 위한 Swift의 메커니즘(대여 검사, 소유권 검사 등)은 무엇일까?
>GQ3. 메모리 안전성 규칙을 위반하는 경우와 해결 방법을 예시와 함께 설명해주세요.

## Description

- 기본적으로 Swift 코드는 발생할 수 있는 안전하지 않은 동작을 방지한다.
- Swift는 **메모리의 위치를 수정하는 코드가 해당 메모리에 대한 독점 접근 권한을 갖도록 요구함**으로써 동일한 메모리 영역에 대해 다중 접근이 충돌나지 않도록 합니다.
- 하지만, 잠재적으로 코드에서 충돌이 발생할 수 있는 상황을 이해해야 컴파일이나 런타임에서 메모리에 접근 충돌이 일어나지 않도록 예방할 수 있다. -> [Thread Sanitizer](https://developer.apple.com/documentation/xcode/diagnosing-memory-thread-and-crash-issues-early) 사용해서 다중 스레드 충돌을 예방하는 방법!


## 주요 기능

#### 1) 소유권 (Ownership)

#### 2) 빌림 (Borrowing)


## 코드 예시

+ 실제 코드 예시를 작성


## Keywords

+ 파생된 키워드들을 작성


## References
- [Swift 공식 문서 - 메모리 안정성 (Memory Safety)](https://bbiguduk.gitbook.io/swift/language-guide-1/memory-safety)
- 