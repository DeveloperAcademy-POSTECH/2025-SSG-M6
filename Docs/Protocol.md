>[!question]
>GQ1. 프로토콜이 무엇일까?
>GQ2. 프로토콜은 언제 필요할까?
>GQ3. 프로토콜은 어떻게 사용할까?

## Description
- 프로토콜은 타입이 반드시 구현해야 **요구사항**을 **추상화**한 것이다.  
	- 프로토콜에서 요구(정의)할 수 있는 항목은 다음과 같다.
		- **Property**
		- **Method**
		- **Initializer**
		- Subscript
		- associatedtype
- 프로토콜은 **클래스, 구조체, 열거형**에 의해 채택될 수 있다.

#### Protocol **Syntax**
> 프로토콜 **구문**

- 프로토콜 **정의**
```swift
protocol Learner {
	//protocol definition
}
```
 
 - **채택시(class, struct, enum)**
	- 여러 프로토콜들은 컴마로 구분된다.
	- class가 상위 class를 가진 경우에는 맨 앞에 상위 class를 작성한다.
```swift
class someClass: UIViewController, Learner, Service {
	// class definition
}
```

- 프로토콜 **네이밍 규칙**
	- 프로토콜은 Type이기 때문에 **UpperCamelCase**로 만들어준다.
	- 프로토콜은 개념/역할을 중심으로 네이밍
	- 구현체는 구체적인 기능/상황을 중심으로 네이밍

- swift 내의 **프로토콜 - 구현체** 예시
	- Collection - `Array`, `Dictionary`, `Set`
	- Comparable - `Int`, `Double`, `String`
	- Error - `enum NetworkError: Error { ... }`

ex) Collection
```swift
protocol Collection {}
struct Array<Element>: Collection {}
struct Dictionary<Key: Hashable, Value>: Collection {}
struct Set<Element: Hashable>: Collection {}
```

ex) UserService
```swift
protocol UserService {}
struct NetworkUserService: UserService {}
struct MockUserService: UserService {}
struct CachedUserService: UserService {}
```


#### **Property** Requirements
> **프로퍼티** 요구사항

- var로 정의되어야 한다
- gettable과 settable 프로퍼티는 타입 선언 뒤에 `{ get set }` 작성
- gettable 프로퍼티는 `{ get }` 으로 작성
```swift
protocol LearnerProtocol {
	static var generation: Int { get }
	var nickname: String { get set }
}

struct Learner: LearnerProtocol {
	static let generation = 4
	var nickname: String
}
var learner = Learner(nickname: "Jam")
```

#### **Method** Requirements
> **메서드** 요구사항

```swift
protocol Tappable {
    func didTap()
}

struct ButtonHandler: Tappable {
    func didTap() {
        print("Button tapped!")
    }
}
```

```swift
protocol RandomNumberGenerator {
    func random() -> Double
}

class LinearCongruentialGenerator: RandomNumberGenerator {
    var lastRandom = 42.0
    let m = 139968.0
    let a = 3877.0
    let c = 29573.0
    func random() -> Double {
        lastRandom = ((lastRandom * a + c)
            .truncatingRemainder(dividingBy:m))
        return lastRandom / m
    }
}

let generator = LinearCongruentialGenerator()
print("Here's a random number: \(generator.random())")
// Prints "Here's a random number: 0.3746499199817101"
print("And another one: \(generator.random())")
// Prints "And another one: 0.729023776863283"
```
#### **Mutating Method** Requirements
> **변경 메서드** 요구사항
- 변경 메서드 사용 조건
	- 값 타입(Value Type)이 프로토콜을 채택할 가능성이 있음
    - 해당 메서드가 프로퍼티 또는 self를 수정할 예정임

```swift
protocol Toggleable {
    mutating func toggle()
}

struct Switch: Toggleable {
    var isOn: Bool = false

    mutating func toggle() {
        isOn.toggle()
    }
}

var mySwitch = Switch()
print("처음 상태:", mySwitch.isOn) // false

mySwitch.toggle()
print("토글 후 상태:", mySwitch.isOn) // true
```

#### **Initializer** Requirements
> 초기화 구문 요구사항

```swift
protocol SomeProtocol {
    init()
}

class SomeSuperClass {
    init() {
        // initializer implementation
    }
}

class SomeSubClass: SomeSuperClass, SomeProtocol {
    required override init() {
        // initializer implementation
    }
}
```

## Protocol in Practice
[[Delegation]]
[[Protocol Extension]]
[[Protocol-Oriented Programming(POP)]]
## Keywords
+ Property, Method, Initializer
+ Class, Struct, Enum
+ Getter, Setter
+ Delegation
+ [[Method]]

## References
- [Swift.org](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/)
- https://bbiguduk.gitbook.io/swift/language-guide-1/protocols