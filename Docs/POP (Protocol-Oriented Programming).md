>[!question]
>GQ1. POP가 뭘까?
>GQ2. POP가 왜 좋을까?

## Description
### 개념
- 프로토콜 지향 프로그래밍
- POP는 **프로토콜을 중심으로 설계하고 구현**하는 프로그래밍 스타일
- OOP(객체 지향 프로그래밍, Object-Oriented Programming)가 가진 단점을 보완한 Swift 맞춤형 설계 방식
- Swift의 핵심인 [[구조체 (Struct)]], [[열거형 (enum)]], [[프로토콜 (Protocol)]], [[확장 (Extension)]] 이 POP의 4총사

> [!POP와 OOP의 차이점]
> |구분|OOP|POP|
|---|---|---|
|중심 구조|클래스 + 상속|프로토콜 + 조합|
|다형성|상속 기반|프로토콜 기반|
|코드 재사용|부모 클래스|프로토콜 익스텐션|
|타입 중심|참조 타입(class)|값 타입(struct)|
|예측 가능성|낮음 (공유)|높음 (복사)|
|Swift와의 궁합|중간|최상|

#### 코드로 비교하기
```swift
// OOP
class Animal {
    func speak() { print("...") }
}

class Dog: Animal {
    override func speak() { print("멍멍") }
}

// POP
protocol Animal {
    func speak()
}

struct Dog: Animal {
    func speak() { print("멍멍") }
}

```

## 장점

+ 상속없이 **공통 기능 재사용** 가능
```swift
	protocol Drivable {
    func drive()
}

extension Drivable {
    func drive() {
        print("운전 중")
    }
}

struct Car: Drivable {}
let myCar = Car()
myCar.drive() // 운전 중
```

+ 여러 기능 조합 가능 (**다중 프로토콜**)
	+ 상속은 1개만 가능
```swift
protocol Flyable { func fly() }
extension Flyable {
    func fly() { print("날아갑니다") }
}

struct Drone: Drivable, Flyable {}
let drone = Drone()
drone.drive()
drone.fly()
```

- 테스트와 유지보수에 강력함
	- 테스트용 `MockAPIClient`를 활용해 실제 네트워크 없이 테스트 가능
```swift
protocol APIClient {
    func fetchData() async throws -> [String]
}

struct RealAPIClient: APIClient {
    func fetchData() async throws -> [String] {
        // 네트워크 코드
        return ["Real 데이터"]
    }
}

struct MockAPIClient: APIClient {
    func fetchData() async throws -> [String] {
        return ["Mock 데이터"]
    }
}

```

- [[의존성 주입 (DI)]]에 최적
	- ViewModel은 APIClient가 어떤 구현인지 몰라도 됨
```swift
protocol APIClient {
    func request(endpoint: String) async throws -> String
}

class ViewModel {
    let api: APIClient // 프로토콜에만 의존

    init(api: APIClient) {
        self.api = api
    }

    func fetchData() async {
        let data = try? await api.request(endpoint: "/hello")
        print(data ?? "오류")
    }
}
```


### 상속과 프로토콜의 차이
##### ❓왜 하필 프로토콜을 써야해?
상속에서도 공통 기능을 재사용할 수 있는데 왜 하필 프로토콜을 써야할까?

1. 상속은 계층구조로 유연성이 낮다. 즉 부모의 작은 변경이 자식 모두에게 영향을 끼친다
- Animal 클래스
```swift
class Animal {
    func sound() { print("기본 소리") }
}

class Dog: Animal {}
class Cat: Animal {}
```

- Animal 클래스 변경
```swift
class Animal {
    func sound() { print("🐾 Animal이 새롭게 소리냄") }
}
```
➡️ Dog와 Cat이 바뀐 Animal 클래스의 영향을 받는다

##### ❓override로 해결하면 되는거 아니야?
- override한 Dog 클래스
```swift
class Dog: Animal {
    override func sound() { print("멍멍") }
}
```
➡️ 맞음. 하지만 override를 하지 않은 상태인 Cat, Hamster는..?(🐹: 당황찌)

##### ❓하위클래스 싹다 override하면 되는거 아냐?
➡️ 현실적으로 불가능하다. 수십 개의 하위 클래스를 매번 override하고 이 규칙을 직접 우리가 기억해야한다.
- 깊은 상속구조 예시
```swift
class Animal {
    func sound() { print("소리") }
}

class Mammal: Animal {}
class Canine: Mammal {}
class Dog: Canine {} // Depth: 3

let dog = Dog()
dog.sound()
```
➡️ Animal의 `sound()` 하나 바꾸면 Dog도 바뀜
➡️ Dog()는 Animal에 의존하고 있는지도 몰랐음 = 숨겨진 의존성
➡️ 높은 결합도 + 낮은 유연성

##### ❓그럼 프로토콜에선 어떻게 하는데?
- POP 스타일 예시
```swift
protocol Soundable {
    func sound()
}

extension Soundable {
    func sound() {
        print("소리")
    }
}

struct Dog: Soundable {}

let dog = Dog()
dog.sound() // "소리"
```
➡️ 상속 구조가 아닌 **역할** 중심으로 분리

##### ✨ 즉, POP는 상속보다 "기능 중심 재사용"
- 상속은 **"is-a"** 관계가 강함 → Dog **is-a** Animal
    
- 프로토콜은 **"can-do"** 관계 → Dog **can bark**, Bird **can fly**


##### ❓꼭 상속을 써야할 때는 없나?
**➡️ UIKit 사용시**에는 꼭 사용을 해야할 때가 있다
- UIKit 자체가 iOS 2시절 Objective-C 기반으로 만들어진 OOP 기반 프레임워크이기 때문

**대표 예시**
- `UIViewController` 커스터마이징
```swift
class HomeViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
    }
}
```

- 뷰 커스터마이징
```swift
class MyButton: UIButton {
    override func layoutSubviews() {
        super.layoutSubviews()
        // 버튼 디자인 변경
    }
}
```

- 셀 커스터마이징
```swift
class MyCell: UITableViewCell {
    override func prepareForReuse() { ... }
}
```

- Target-Action / Responder Chain 관련
	- 버튼 클릭, 텍스트 입력, 터치 이벤트 감지 등은  `UIResponder` 상속 구조에 연결돼 있음
	- 클래스 기반 아니면 아예 동작 안 함