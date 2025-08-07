>[!question]
>GQ1. 순환 참조 (Retain Cycle)는 왜 문제인 것일까? 순환 참조가 생기면, 어떤 상황이 벌어지는 것이지?
>GQ2. 순환 참조의 해결 방법, [[약한 참조 (weak Reference)]]와 [[미소유 참조 (unowned Reference)]] 등은 어떤 원리로 이 문제를 해결하는 것일까?
>GQ3. Xcode에서 순환 참조가 발생했다는 것을 어떻게 알 수 있을까?


## Description

- 순환 참조 (Retain Cycle)란 두개 이상의 참조 타입 인스턴스 간 서로 [[강한 참조 (Strong Reference)]] 관계가 형성되며, 참조 카운트 (Reference Count)가 0으로 떨어지지 않아 ***메모리에서 영구적으로 해제되지 않는 상황***을 의미한다.
- 메모리에서 영구적으로 해제되지 않는 상황을 우리는 ***"메모리 누수 (Memory Leak)"*** 라고 부른다.

##### 이 메모리 누수 (Memory Leak) 상황을 예방하려면?

- 클래스의 인스턴스간 연결관계 혹은 클로저와의 관계를 [[약한 참조 (weak Reference)]]와 [[미소유 참조 (unowned Reference)]] 관계로 설정한다.


## 코드 예시 (1) : 클로저 내부에서 self를 캡처하는 상황

+ ViewController 클래스로부터 생성된 인스턴스 vc는 onTap이라는 클로저를 내부적으로 갖고 있다.
+ onTap 클로저는 self (인스턴스 vc)를 캡처하여 내부에서 사용하고 있다.
+ 즉, vc -> onTap -> self (vc)라는 관계의 순환 참조 (Retain Cycle)가 발생하는 것!

```swift
class ViewController {
    var onTap: (() -> Void)?

    init() {
        onTap = {  // [weak self] in을 붙여서 순환 참조를 해결할 수 있다!
            print("Button tapped in \(self)")  // self를 캡처 (강한 참조)
        }
    }

    deinit {
        print("ViewController deinitialized")
    }
} 

func createViewController() {
    let vc = ViewController()
    vc.onTap?()  // vc.onTap이 self를 캡처하고 있고, vc는 onTap 클로저를 속성으로 갖고 있으므로 서로 강한 참조 → 순환 참조 발생
}
```


## 코드 예시 (2) : 클래스 인스턴스의 상호 참조 상황

+ Boy 클래스는 내부에서 Optional 타입의 Girl을, Girl 클래스는 내부에서 Optional 타입의 Boy를 참조한다.
+ 각각 클래스에서 생성된 boy, girl 인스턴스가 존재
+ boy는 girl을, girl은 boy를 상호적으로 참조하여 (Reference Count가 각 1씩 증가함)
+ 인스턴스가 nil이 되더라도, 각 인스턴스는 상호 참조되어 있기에 메모리에서 해제되지 않음

```swift
class Boy { 
    let name: String 
    var girlFriend: Girl? 
    init(name: String) { self.name = name } 
    deinit { print("\(name) is being deinitialized") } 
} 
class Girl { 
	let name: String 
	var boyFriend: Boy? 
	init(name: String) { self.name = name } 
	deinit { print("\(name) is being deinitialized") } 
} 

var boy: Boy? = Boy(name: "미니") 
var girl: Girl? = Girl(name: "민니")

// 순환 참조 발생 (boy는 girl을 참조하고, girl은 boy를 참조한다)
boy?.girlFriend = girl 
girl?.boyFriend = boy

// 둘다 메모리에서 남아있는 상황. 메모리 누수 발생
boy = nil 
girl = nil
```


## Keywords

+ [[강한 참조 (Strong Reference)]]
+ [[약한 참조 (weak Reference)]]
+ [[미소유 참조 (unowned Reference)]]

  

## References

- [WWDC21 - ARC in Swift: Basics and beyond](https://developer.apple.com/kr/videos/play/wwdc2021/10216/)
- [The Swift Programming Language - Automatic Reference Counting](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)