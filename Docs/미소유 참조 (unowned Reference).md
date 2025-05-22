>[!question]
>GQ1. 미소유 참조를 사용할 때 주의해야할 점은 무엇이 있을까?

## Description

- 미소유 참조 (unowned Reference)는 ***참조 대상이 항상 살아있다고 가정하는 방법***이다.
- 약한 참조 (weak)보다 더 직접적이고 위험하지만, 효율적인 참조 방식이다.


## 주요 기능

+ 인스턴스의 소유권을 가지지 않는다.
+ 참조 카운트 (Reference Count)를 증가시키지 않는다.
+ 참조하는 대상이 항상 살아있을 것이라 가정하기 때문에 nil이 될 수 없으며, 옵셔널 (Optional)로 선언되지도 않는다.

##### 언제 쓰나? (두가지 조건이 모두 충족되어야만 함!)

- 순환 참조는 예방하고 싶은데, **weak를 사용할 수 없는 경우** (= Optional이 아닌 걸로 선언하고 싶은 경우!)
- 상위 객체가 하위 객체보다 수명이 길거나 동일함을 보장할 수 있는 경우


## 코드 예시

+ `unowned` 키워드로 선언하면, 해당 프로퍼티는 클래스 인스턴스를 미소유로 참조하는 것!

```swift
class Traveler {
    var name: String
    var account: Account?
    init(name: String) { self.name = name }
}

class Account {
    unowned var traveler: Traveler
    var points: Int
    init(traveler: Traveler, points: Int) {
        self.traveler = traveler
        self.points = points
    }
}
```


## ⚠️ 주의해야할 점!

- 참조 대상이 먼저 해제되었는데, 미소유 참조로 접근하게 되면 크래시가 발생하게 된다!

```swift
var traveler: Traveler? = Traveler(name: "Lily")
let account = Account(traveler: traveler!, points: 1000)

traveler = nil // Traveler 해제됨
account.traveler.name // ❌ 접근 시 크래시
```


## References

- [WWDC21 - ARC in Swift: Basics and beyond](https://developer.apple.com/kr/videos/play/wwdc2021/10216/)
- [The Swift Programming Language - Automatic Reference Counting](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)