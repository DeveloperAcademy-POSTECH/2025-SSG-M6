030>[!question]
>GQ1. 접근 제어가 왜 필요할까?

## 접근 제어

> 접근 제어는 클래스, 구조체, 열거형, 변수, 함수 등에 대해 **외부에서 접근 가능한 범위를 제한하는 문법**
> 이 기능은 코드의 구현 세부를 숨기고 해당 코드에 접근하고 사용될 수 있는 기본 인터페이스를 지정할 수 있습니다. 


### 접근 제어는 왜 필요할까?

접근 제어는 코드의 안전성과 유지보수성, 의도된 사용을 보장하기 위해 반드시 필요한 기능입니다.

- **캡슐화**
	- 내부 구현을 감추고, 외부에 필요한 인터페이스만 공개합니다.
- **잘못된 접근 방지**
	- 외부에서 실수로 중요한 변수나 메서드에 접근해 망가뜨리는 일을 막습니다.
- **모듈 간 경계 설정**
	- 라이브러리나 프레임워크의 내부 구현과 외부 API를 구분할 수 있습니다.



Swift에서는 5가지의 다른 접근 레벨을 제공합니다.
**open**(가장 개방적) -> **public** -> **internal**(Default) -> **fileprivate** -> **private**(가장 제한적)

![[Access Control.png]]
### open

가장 개방적인 접근 수준
이 키워드가 붙은 클래스나 메서드는 다른 모듈에서도 접근할 수 있으며, 서브클래싱과 메서드 오버라이딩까지 허용됩니다.
~ 주로 프레임워크 제작자가 사용자에게 확장 가능한 API를 제공하고 싶을 때 사용합니다 ~

라이브러리 제작자가 
```swift
open class ButtonStyle {
    public init() {}

    open func backgroundColor() -> String {
        return "Blue"
    }

    open func cornerRadius() -> Double {
        return 8.0
    }
}
```
이렇게 open 키워드를 사용하여 제공하면

해당 라이브러리를 사용하는 사람은
```swift
import MyLibrary

class CustomButtonStyle: ButtonStyle {
    override func backgroundColor() -> String {
        return "Red"
    }

    override func cornerRadius() -> Double {
        return 20.0
    }
}

let style = CustomButtonStyle()
print("색상: \(style.backgroundColor())") // Red
print("모서리 반경: \(style.cornerRadius())") // 20.0
```
이렇게 상속하여 커스터마이징하여 사용할 수 있습니다

### public

public은 open처럼 외부 모듈에서도 접근 가능한 수준이지만, 상속하거나 오버라이딩하는 것은 허용하지 않습니다.
~ 외부에서는 사용할 수 있지만 수정하거나 확장할 수 없도록 보호합니다 ~

라이브러리 제작자가
```swift
public class Calculator {
    public init() {}

    public func add(_ a: Int, _ b: Int) -> Int {
        return a + b
    }

    public func multiply(_ a: Int, _ b: Int) -> Int {
        return a * b
    }
}
```
퍼블릭으로 계산기를 만들어 주면

해당 라이브러리를 사용하는 사람은
```swift
import MyLibrary

let calc = Calculator()
print(calc.add(3, 5)) 
print(calc.multiply(4, 7))

class MyCalculator: Calculator {} // 상속이나
override func add(...) { }  // 오버라이딩이 불가능
```
쓸 수는 있지만 바꾸지는 못 해요.

### internal 

디폴트로 적용되는 접근 수준
같은 모듈(= 하나의 빌드 단위) 내에서는 자유롭게 접근할 수 있지만, 외부 모듈에서는 접근할 수 없습니다.
~ 명시하지 않으면 자동으로 internal이 적용됩니다 ~

```swift
class AuthManager {
    func login(username: String, password: String) -> Bool {
        return username == "admin" && password == "1234"
    }
}
```

```swift
// 이건 같은 모듈 다른 파일 -> 접근 가능 !
let auth = AuthManager()
if auth.login(username: "admin", password: "1234") {
    print("로그인 성공")
}
```
이렇게 같은 모듈에서는 접근이 가능하지만 ~
다른 모듈에서 같은 코드를 사용하면 컴파일 에러가 발생합니다.

### fileprivate

같은 소스 파일 안에서만 접근이 가능한 접근 수준
같은 타입이 아니더라도 파일 안에 있기만 하면 접근할 수 있습니다.
~ 주로 타입 정의와 확장 같은 파일에 정의하면서, 다른 파일에서는 숨기고 싶을 때 사용합니다 ~
```swift
class Animal {
    fileprivate var species: String

    init(species: String) {
        self.species = species
    }

    func describe() {
        print("\(species)")
    }
}

// 같은 파일의 다른 타입에서 접근하고 싶을 때
struct AnimalPrinter {
    func printSpecies(of animal: Animal) {
        print("🔍\(animal.species)")
    }
}

// 같은 파일 안의 extension에서도 접근 가능
extension Animal {
    func shout() {
        print("\(species.uppercased())")
    }
}
```
### private 
가장 제한적인 접근 수준
같은 타입 내부에서만 접근할 수 있고, 같은 파일 안에 있더라고 다른 타입이나 확장에서는 접근할 수 없습니다.
~ 중요한 속성이나 내부 구현을 철저하게 감추고 싶을 때 사용합니다 ~

```swift
class SafeBox {
    private var secretCode: String = "1234"

    func unlock(with input: String) -> Bool {
        if input == secretCode {
            print("열림")
            return true
        } else {
            print("틀림")
            return false
        }
    }
}
```

```swift
let box = SafeBox()
print(box.secretCode) // 이건 안됨 

box.unlock(with: "0000") // 틀림
box.unlock(with: "1234") // 열림
```

##### 그럼 private(set)은 뭔데요?

읽기는 외부에서 허용하되, 쓰기는 내부에서만 가능하게 하고 싶을 때 사용하는 키워드 !
```swift
struct Counter {
    private(set) var count = 0

    mutating func increment() {
        count += 1
    }
}

var counter = Counter()
print(counter.count) // 읽기는 가능
counter.count = 10 // 수정은 불가능
counter.increment() // 내부 함수로만 수정
```

---
## References
[AccessControl](https://bbiguduk.gitbook.io/swift/language-guide-1/access-control#access-control-syntax)