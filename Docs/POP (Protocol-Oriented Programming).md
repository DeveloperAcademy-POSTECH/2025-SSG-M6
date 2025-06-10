>[!question]
>GQ1. POP가 뭘까?
>GQ2. POP가 왜 좋을까?

## Description
### 개념
- 프로토콜 지향 프로그래밍
- POP는 **프로토콜을 중심으로 설계하고 구현**하는 프로그래밍 스타일
- OOP(객체 지향 프로그래밍, Object Oriented Programming)가 가진 단점을 보완한 Swift 맞춤형 설계 방식
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