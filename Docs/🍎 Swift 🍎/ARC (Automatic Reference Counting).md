>[!question]
>GQ1. ARC (Automatic Reference Counting)란 무엇이고 왜 필요한 것이지?
>GQ2. ARC (Automatic Reference Counting)가 없었다면? = ARC가 생기기 전에는 어떻게 메모리 관리를 했을까?
  
## Description

- Swift의 ARC (Automatic Reference Counting)는 앱의 메모리 관리를 자동화하는 기술이다.
- Value Type인 구조체 (Struct), [[열거형 (enum)]]에는 해당되지 않으며, Reference Type인 클래스 (Class)에만 해당된다. = 즉. 참조 타입 (Referece Type)을 사용하면, ARC가 메모리 관리를 자동으로 한다.
- 즉, 우리 개발자는 메모리 관리에 대한 큰 부담 하나를 코딩을 할 때 덜고 가는 셈! 

##### 어떻게 이게 가능하냐고..?

- 컴파일 타임 (Compile Time)에 객체에 대한 retain/release 호출을 자동으로 삽입하고, 런타임 (Runtime)에는 이 자동으로 삽입된 호출을 통해, 참조 카운트를 조정하며 메모리를 관리하는 것.
- 즉, 메모리 관리를 자동화한다. 
- 하지만, 우리는 이 자동화된 메모리 관리에서 [[순환 참조 (Retain Cycle)]] 문제는 자동으로 해결되지 않으니 주의해야 한다.

##### 그렇다면, ARC 그 이전에는 어떻게 메모리 관리를 했었는데?

- Apple은 Reference Counting을 동일하게 메모리 관리 방식으로 채택했었는데요. Objective-C에서는 MRC (Manual Reference Counting)이라는 수동 관리를 사용했습니다.
- 코드를 알 거는 없고... 그냥 직접 `retain`, `release`, `autorelease` 키워드를 삽입해서 썼습니다..!


  

## 주요 기능

+ **Object’s lifetime begins at init() and ends at last use** 
  : 객체의 생명주기는 init()으로 클래스의 인스턴스가 생성되는 시점에 시작되어 ~ 마지막으로 **해당 객체를 참조하는 코드가 해제되는 시점**까지 이어진다.
  
+ **ARC deallocates an object after its lifetime ends**
  : ARC는 객체의 생명주기가 끝난 이후 (reference count가 0이 되는 지점)에 자동으로 메모리를 해제한다.
  이 과정에서 객체의 메모리를 `deinit`으로 명시적으로 해제하지 않아도, ARC가 알아서 처리한다.
  
+ **ARC tracks object’s lifetime with reference count**
  : ARC는 참조 카운트 (reference count)를 내부적으로 관리하여, 객체의 생명주기를 추적한다.
  얼마나 많은 변수, 상수, 클로저가 이 객체를 참조하고 있는지를 나태는 것이며 -> 참조 카운트가 0이 된다는 것은 이 객체를 사용하는 코드가 더 이상 없다는 의미와 동일하다!
  
+ **Swift compiler inserts retain/release operations**
  : Swift 컴파일러 ([[LLVM 컴파일러]])는 코드를 분석하여 적절한 위치에 `retain`, `release` 호출을 자동으로 삽입한다.
  여기서의 retain은 참조 카운트 1 증가, release는 참조 카운트 1 감소를 의미!
  이때 클래스 인스턴스를 참조하면서 참조 카운트가 증가되는 상황을 [[강한 참조 (Strong Reference)]]라고 부릅니다.
  
+ **Swift runtime deallocates object with 0 reference count**
  : 컴파일러가 release 호출을 통해 참조 수를 줄이면, **Swift 런타임은** 이를 추적하다가 **참조 수가 0이 되는 시점에 deinit을 호출**하고 메모리에서 해제한다.
  : 해당 해제는 **정확히 참조 수가 0이 된 시점에 결정적(deterministic)** 으로 수행되며, 가비지 컬렉션(GC)처럼 나중에 모아서 처리하지 않는다. ->  성능에 큰 오버헤드를 끼치지 못함!

  

## 코드 예시

- ARC의 동작은 어떻게 이루어지는가?

```swift
class Person {
    let name: String
    init(name: String) {
	    self.name = name
	    print("\(name) is being initialized")
    }
    deinit {
        print("\(name) is being deinitialized")
    }
} 

// 이 상황에서는 Person 인스턴스를 참조하지 않음 (왜냐하면, 옵셔널 타입이기 때문에! 할당 전!)
var reference1: Person?
var reference2: Person?
var reference3: Person?

// Person Reference Count = 1
reference1 = Person(name: "John Appleseed"). // Prints "John Appleseed is being initialized"

// Person Reference Count = 3
reference2 = reference1
reference3 = reference1

// Person Reference Count = 1
reference1 = nil
reference2 = nil

// Person 인스턴스가 메모리에서 해제됨
reference3 = nil  // Prints "John Appleseed is being deinitialized"
```

+ Swift 컴파일러가 코드를 분석하여 적절한 위치에 retain, release 호출을 자동으로 삽입하는 것의 의미?

```swift
class Traveler {
    var name: String
    var destination: String?
} 

// 으로 표시된 부분은 Swift Compiler가 자동으로 삽입되는 부분. 
// 개발자는 이 부분을 직접 보면서 코딩하지는 않지만, 컴파일된 바이너리 안에는 이러한 동작이 들어가 있다.
func test() {
    let traveler1 = Traveler(name: "Lily")
    // %0 = alloc Traveler(name: "Lily")
    // retain(%0)  |  Reference Count = 1
    let traveler2 = traveler1
    // retain(%0)  |  Reference Count = 2
    traveler2.destination = "Big Sur"
    // release(%0)  |  Reference Count = 1
    print("Done traveling")
    // release(%0)  |  Reference Count = 0 (deinit 호출 -> 메모리 해제)
}
```
  

- 하지만, 관찰 가능한 객체의 수명 (Observable Object Lifetimes)에 의존하는 코드는 위험하다.
  : 개발자가 눈으로 보는 실행 결과 (= 어떤 시점에서 객체가 deinit되는지 관찰한 결과)를 기준으로 로직을 작성하면 문제가 생길 수 있다. -> "눈으로 보는 실행 결과가 즉, 관찰 가능한 객체의 수명을 의미"

```swift
class Thing {
    deinit {
        print("Thing deinitialized")
    }
} 

func trigger() {
    let thing = Thing()
    // thing은 해당 클로저 안에서 사용되니까 메모리에 살아있겠지라고 생각하지만, 
    // 만약 Swift 컴파일러가 thing을 캡처하지 않는다고 판단한다면,
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
        print(thing) // 여기 도달 전에 thing이 이미 해제될 수도 있음
    }
}
```

>[!tip]
> 관찰가능한 객체의 수명 (**Observed lifetime**)이 아니라 보장된 수명 (**Guaranteed lifetime**)을 기준으로 의존해야 한다.
> 즉, 객체가 마지막으로 명시적으로 사용된 지점까지 / 그 이후는 컴파일러 최적화에 따라 언제든지 해제될 수 있음을 전제로 코드를 작성해야만 합니다!
> -> [[클로저 (Closure)]] 안에서 객체가 필요하다면, 반드시 "***명시적***"으로 캡처해야 한다!!!


## Keywords

+ [[순환 참조 (Retain Cycle)]]
+ [[강한 참조 (Strong Reference)]]
- [[LLVM 컴파일러]]
  

## References

- [WWDC21 - ARC in Swift: Basics and beyond](https://developer.apple.com/kr/videos/play/wwdc2021/10216/)
- [The Swift Programming Language - Automatic Reference Counting](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)
- [(iOS) 메모리 관리 (3/3) - MRC/MRR(Manual Reference Counting](https://babbab2.tistory.com/28)
