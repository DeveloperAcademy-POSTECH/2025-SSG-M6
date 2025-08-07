>[!question]
>GQ1. 메모리 안정성 (Memory Safety)이란?
>GQ2. 소유권 (Ownership)은 뭐고, 빌림 (Borrowing)은 뭐지?
>GQ3. 메모리 안전성을 보장하기 위한 Swift의 메커니즘(대여 검사, 소유권 검사 등)은 무엇일까?
>GQ4. 메모리 안전성 규칙을 위반하는 경우와 해결 방법의 예시가 있을까?

## GQ1. 메모리 안정성 (Memory Safety)이란?

- 기본적으로 Swift 코드는 발생할 수 있는 안전하지 않은 동작을 방지한다.
- Swift는 **독점적 메모리 접근** 규칙을 통해 같은 메모리 위치에 대한 다중 쓰기/읽기 충돌을 방지한다. 즉, 변수나 프로퍼티를 수정하려면 해당 메모리 위치에 “단 하나의” 접근만 허용된다는 의미!
- 즉, **메모리 안정성 (Memory Safety**)이란 **"프로그램이 잘못된 메모리에 접근하지 않도록 보장하는 언어 설계 원칙"**을 의미한다.


## GQ2. 소유권 (Ownership)은 뭐고, 빌림 (Borrowing)은 뭐지?

##### 소유권 (Ownership)

> "어떤 값이 누구에게 속해 있으며 (할당), 누가 그것을 해제할 책임 (해제)이 있는가?"

- Swift의 참조 타입 (Reference Type)인 클래스 인스턴스는 [[ARC (Automatic Reference Counting)]]로 관리되었다.
- 반면, 값 타입 (Value Tyoe)은 복사할 때 새로운 인스턴스를 생성하여 소유권을 분리한다.

```swift
struct User {
    var name: String
} 

var user1 = User(name: "Mini")
var user2 = u1  // u2는 u1의 복사본 (user1과 user2는 다른 메모리 공간에 존재하므로, 소유권 분리!)

user2.name = "Jam"
print(user1.name) // Mini
```

##### 빌림 (Borrowing)

> 

- 전통적 Swift에서는 함수 호출 시 매개변수 값을 복사하거나 참조를 retain해 넘기지만, Swift 5.9부터 borrowing 매개변수를 통해 호출자가 원본 값을 유지한 채로 인자를 넘길 수 있다. 
- borrowing 매개변수로 넘기면 호출자(Caller)가 값을 소유(리테인하지 않음)하고, 피호출자(Callee)는 별도 해제 없이 접근만 한다. 
- 반대로 consuming 매개변수는 피호출자가 값을 소유하거나 retain해야 하며, 끝내 해제해야 한다 . 
- 이들은 기본적으로 호출 방식(ownership inference)에 해당하며, 복사 또는 retain 비용을 개발자가 제어할 수 있는 기능이다.



## References
- [Swift 공식 문서 - 메모리 안정성 (Memory Safety)](https://bbiguduk.gitbook.io/swift/language-guide-1/memory-safety)
- 