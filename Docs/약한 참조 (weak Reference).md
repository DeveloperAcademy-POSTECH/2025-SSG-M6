>[!question]
>GQ1. 약한 참조를 사용할 때도 주의해야할 점이 있을까?

## Description

- 약한 참조 (Weak Reference)는 ***참조 카운트를 증가시키지 않는 방법***이다.


## 주요 기능

+ 참조하고 있던 (= 참조 대상) 객체가 메모리에서 해제되면 자동으로 nil이 할당된다.
+ nil이 대입될 수 있기 때문에 weak로 선언되는 프로퍼티는 항상 [[옵셔널 (Optional)]] 타입이어야 한다.


## ⚠️ 약한 참조 사용 시 주의점

+ Account는 Traveler의 생명주기를 보장하지 않을 수 있다.
+ 즉, Traveler가 Account보다 먼저 사라질 수 있기 때문에 (Swift 컴파일러가 임의로 Travler가 함수 내에서 사용되지 않는다고 판단되면 -> 임의로 release를 삽입해 메모리에서 해제될 수 있다)  
	+ 컴파일러가 traveler의 **마지막 사용 시점 이후** 바로 release 최적화를 적용하면?
	+ Swift의 ARC 옵티마이저가 개선되면서 **traveler의 lifetime을 더 일찍 종료**하면?
	+ 클로저로 넘기거나 async 코드로 실행하면?
- ***traveler는 nil (= 이미 해제된 인스턴스 객체)를 참조할 수 있는 가능성이 남아있는 셈!*** -> 크래시가 발생할 수 있다!

```swift
class Traveler {
    var name: String
    var account: Account?
} 

class Account {
    weak var traveler: Traveler?    // 약한 참조 (retain count 증가 X)
    var points: Int
    func printSummary() {
        print("\(traveler!.name) has \(points) points")
    }
}

func test() {
    let traveler = Traveler(name: "Lily") 
    
     // account -> travel : weak reference
    let account = Account(traveler: traveler, points: 1000) 
    // traveler → account : strong reference
    traveler.account = account  

    account.printSummary()   // Apple은 이 부분은 우연히 해제되지 않아 정상 동작한다고 표현!
}
```

- 옵셔널 바인딩 (Optional Binding)을 사용해 해결할 수 있을 것 같지만,
  오히려 명시적인 크래시 자체가 발생하지 않으니 버그가 발생했을 때 그 위치를 정확하게 찾기가 더 어렵다!

```swift
func printSummary() {
	if let traveler {
	    print("\(traveler.name) has \(points) points")
	}
}
```

- 이 문제를 해결하기 위한 방법을 두 가지 제안한다.
	- [[withExtendedLifetime()]] : 객체의 lifetime을 안전하게 연장하는 방법
	- Redisign to access via strong reference / to avoid weak/unowned reference : 근본적으로 강한 참조 구조 내에서 순환 참조가 발생하지 않도록 ***"전체 구조의 재설계"*** 를 제안!


## References

- [WWDC21 - ARC in Swift: Basics and beyond](https://developer.apple.com/kr/videos/play/wwdc2021/10216/)
- [The Swift Programming Language - Automatic Reference Counting](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)