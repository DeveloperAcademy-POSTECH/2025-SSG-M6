>[!question]
>GQ1. 제네릭은 재사용을 넘어서 어떤 장점이 있을까?
>GQ2. associatedtype과 제네릭 타입 파라미터의 차이는 뭘까?
>GQ3. Variadic Generics는 기존 제네릭과 뭐가 다를까?

## 제네릭이란?

제네릭(Generic)은 함수, 구조체, 클래스, 열거형 등이 다양한 타입에서 **재사용**될 수 있도록 해주는 기능입니다.
➡️ 여러 타입을 하나의 코드 구조로 일반화 하기 위해 사용

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}
```

T는 타입 매개변수로, 어떤 타입이든 들어올 수 있다는 뜻이에요. 
어떤 이름이든 가능하다만 보통 관습적으로는 한 글자 또는 Upper Camel Case를 사용한다고 합니다!



#### 제네릭 동작 방식

C++과 마찬가지로 **컴파일 타임**에 타입이 확정이 되어요. (런타임에 타입을 유추하지 않음 -> 정적 타입 바인딩)
그치만 제네릭이 [[프로토콜 (Protocol)]]이 연관될 때는 런타임의 제약도 감안하고 있어요.

1. <font color="DodgerBlue">컴파일 타임</font>에 결정되는 경우

```swift
func example<T>(_ input: T) -> T {
    return input
}

example(5)      // 컴파일러가 T = Int로 확정 → Int 버전 생성
example("Hi")   // T = String → String 버전 생성
```


2. 제네릭이 <font color="palevioletred">런타임</font>에 타입을 전달해야 하는 경우

```swift
protocol P {
    func act()
}

func run<T: P>(_ item: T) { // 프로토콜 제약이 붙어버림
    item.act()
}
```

위 코드에서 T는 제네릭 타입이지만, P라는 프로토콜을 따른다는 제약이 붙었어요.
즉 T가 어떤 구체적인 타입인지 몰라도, P프로토콜을 따른다는 것만 보장되면 돼 ! 라는 구조가 됨

근데 여기서 발생하는 문제 하나는 item.act()를 호출해야 하는데,
Swift는 지금 이게 어떤 타입의 act()인지 컴파일 타임에는 알 수 없어요. 
왜냐하면 T가 어떤 타입인지 컴파일 타임에는 정해지지 않아서 !

##### 그럼 Swift는 어떻게 처리하나요?

- Swift는 컴파일 타임에 이 타입이 어떤 메서드를 구현했는지 기록한 테이블을 만들어 둡니다.
	- 이 테이블을 **Witness Table**이라고 해요.
- 그리고 런타임에 실제 타입과 함께 이 테이블을 함수에 넘겨줍니다.
item.act()는 결국 이 테이블을 통해 찾아서 호출되기 때문에, 런타임에 호출이 결정되는 구조입니다 !

결론 ! ☠️

그냥 제네릭은 컴파일 타임에 바로 박아서 실행
프로토콜 제약이 붙은 제네릭은 런타임에 테이블을 보고 어떤 메서드를 실행할지 찾아서 실행

##### 1번과 2번의 속도
는 당연히 1번이 더 빨라요

1번 컴파일 타임에 결정되는 경우는 타입별로 특화된 함수 버전을 만들어 놓고 호출하기 때문에 제네릭이 아니라 일반 함수처럼 빠르게 실행되어요.

반대로 2번은 T의 정확한 타입을 알 수 없기 때문에 런타임에 witness table을 참조해서 적절한 구현을 찾아 호출하기 때문에 약간 느리다네용.




#### 타입 제약

제네릭이 어떤 타입이든간에 열려 있는 건 사실이다만, 너무너무 열려있어요. 그래서 제약을 걸어봅시다.

- **특정 프로토콜을 준수하는 타입만 허용하기**
```swift
func printDescription<T: CustomStringConvertible>(_ value: T) {
    print(value.description)
}

또는 여러 제약 한 번에 걸기 !
func doSomething<T>(_ value: T) where T: Equatable, T: Hashable {
    print(value.hashValue)
}
```
	T : CustomStringConvertible 이 부분이 타입 제약
	 이 함수는 description을 가진 타입만 받을 수 있어요.

- **클래스 상속 제약**
```swift
class Animal {
    func speak() {
        print("Animal sound")
    }
}

func letItSpeak<T: Animal>(_ creature: T) {
    creature.speak()
}
```
	T는 Animal 클래스를 상속받은 타입이어야 해요.
	 class와 protocol은 동일한 방식으로 제약을 걸 수 있어요.

- **연관 타입 (associatedType) 제약**
```swift
protocol Container {
    associatedtype Item: Hashable
    var items: [Item] { get }
}
```
	 제네릭과 유사하지만, 프로토콜 내부에서 타입을 추상화하고 싶을 때 사용하는 방식입니다.
	 프로토콜 내부에서 사용하는 타입에 이름만 정해놓고, 구현체가 실제 타입을 결정합니다.
	 Item이 반드시 Hashable을 따라야 한다는 의미입니다.
	 
	 아래에서 제네릭 타입 파라미터와의 차이점을 살펴보겠읍니다.




## 제네릭은 재사용을 넘어서 어떤 장점이 있을까?

1. **정적 타입 안전성 향상** -> 위에 설명한 장점임 !
	- 컴파일 타임에 타입을 확정하여 런타임 오류를 줄이고, 잘못된 타입 사용을 사전에 방지할 수 있습니다.
	  
2. **타입 추론을 통한 코드 간결화**
	- 제네릭은 호출 시 타입을 자동으로 추론하므로, 코드가 더 짧고 읽기 쉬워요.
	  
3. 프로토콜 제약 + where절 -> **타입 제어 가능**
	- 제네릭에 제약을 주면 의미 있는 타입 사용만 허용 가능해요!




## associatedtype과 제네릭 타입 파라미터의 차이는 뭘까?

| 구분           | 제네릭 파라미터 (`<T>`)               | `associatedtype`                            |
| ------------ | ------------------------------ | ------------------------------------------- |
| **위치**       | 함수/타입 정의부                      | 프로토콜 내부                                     |
| **타입 결정 시점** | 호출 시점 (함수 외부에서 명시)             | 프로토콜 채택 시점 (내부 구현에서 결정)                     |
| **사용 상황**    | 함수, 구조체, 클래스, 열거형 등 모든 타입에서    | 프로토콜 전용                                     |
| **문법**       | `func swap<T>(_ a: T, _ b: T)` | `protocol Stack { associatedtype Element }` |
| **다형성 활용**   | `T: Equatable` 등 타입 제약 명시      | `associatedtype Element: Equatable`처럼 제약 가능 |

위에서 제네릭은 많이 다루었으니 associatedtype에 대해서 더 알아보자면

```swift
protocol Stack {
    associatedtype Element
    
    mutating func push(_ item: Element)
    mutating func pop() -> Element?
}

struct IntStack: Stack {
    private var items: [Int] = []
    
    // 여기서 associatedtype Element == Int 로 암묵 결정됨
    // 여긴 암묵적으로 결정되는 건데 명시적으로 지정할 수도 있어요
    mutating func push(_ item: Int) {
        items.append(item)
    }

    mutating func pop() -> Int? {
        return items.popLast()
    }
}
```
 
 Stack이라는 프로토콜은 하나의 제네릭 개념을 담고 있어요, 근데 Swift의 프로토콜 자체는 제네릭 파라미터를 받을 수 없어요.
 
 그래서 associatedtype Element를 선언해 **이 프로토콜이 다루는 타입은 어떤 것이든 정의될 수 있다**고 선언합니당 -> 나는 어떤 타입을 다룰 건데, 그 타입은 이 프로토콜이 채택하는 사람이 나중에 정해줘 ! 입니다

프로토콜에서는 제네릭 파라미터 사용이 불가능하기 때문에 연관 타입이 프로토콜 제네릭의 유일한 방법이라 사용합니당


## Variadic Generics는 기존 제네릭과 뭐가 다를까?

#### Variadic Generics은 뭔데 !
제네릭 타입 매개변수를 여러 개 한 번에 받을 수 있게 해주는 기능이에요.
즉, 타입 매개변수의 개수 자체가 가변적이라는 뜻입니다.

기존에는 하나씩 또는 몇 개 고정된 타입 매개변수만 받을 수 있었지만, Variadic Generic을 사용하면 가변 길이 타입을 한 번에 다룰 수 있어요. + 매개변수가 안 들어오는 것까지 가능 !

```swift
func describeTwo<T, U>(_ a: T, _ b: U) {
    print("a는 \(type(of: a)), b는 \(type(of: b))")
}

describeTwo(1, "Hello")
// 출력: a는 Int, b는 String
```

기존 제네릭은 이렇게 고정된 개수의 매개변수만 받을 수 있었다면,

Variadic Generic 방식은
```swift
func describeAll(_ values: Any...) {
    for value in values {
        print("타입은 \(type(of: value)), 값은 \(value)")
    }
}

describeAll(1, "Hello", true, 3.14)
// 출력:
// 타입은 Int, 값은 1
// 타입은 String, 값은 Hello
// 타입은 Bool, 값은 true
// 타입은 Double, 값은 3.14
```

이렇게 때에 따라 자유로운 개수의 매개변수를 허용합니다 !