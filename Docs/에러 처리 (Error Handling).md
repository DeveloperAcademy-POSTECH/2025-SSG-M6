>[!question]
>GQ1. Error가 뭘까?
>GQ2. Error를 어떻게 활용할까?

## 개념
-  에러 처리 (Error Handling)는 에러 조건에서 응답하고 복구하는 프로세스이다.
- Swift는 런타임에 복구 가능한 에러를 던지고(throwing), 포착하고(catching), 전파하고(propagating), 조작하기(manipulating) 위한 최고 수준의 지원을 제공한다.
- 즉 에러가 발생하였을 때 크래시가 발생하지 않도록 돕는다.

## 에러 표현과 던지기 (Representing and Throwing Errors)
Swift에서 에러는 **Error** 프로토콜을 따르는 타입의 값으로 표현된다.
[[열거형 (Enumerations)]]이 관련 에러 조건의 그룹을 모델링 하기에 특히 적합하다.

>[!왜 열거형이 적합할까?]
> - 에러는 보통 정해진 케이스 중 1개로 명확하게 종류를 나눌 수 있다
> - case로 분기 처리를 할 수 있어서 catch문 에서 바로 쓰기 편하다

- **Error 프로토콜**
```swift
protocol Error : Sendable
```

>[!Sendable]
>Swift [[Concurrency]]에서 사용하는 [[프로토콜 (Protocol)]]로 **값이 여러 스레드 간 안전하게 이동될 수 있게 보장**하는 약속
>[[Sendable]]을 채택한 타입은 다른 스레드/Task로 복사되거나 전달될 때 데이터 충돌 없이 안전하게 사용 가능함을 보장
>**➡ 따라서 Error는 Sendable을 채택함으로서 스레드/Task 간에 이동해도 안전하다는 것이 보장된다!**

- **자동 판매기(VendingMachine)** 에러
```swift
enum VendingMachineError: Error {
    case invalidSelection   // 유효하지 않은 항목 선택시
    case insufficientFunds(coinsNeeded: Int)   //투입한 금액이 부족할 때
    case outOfStock   //선택한 상품의 재고가 없을 때
}
```

- **네트워크** 에러
```swift
enum NetworkError: Error {
    case timeout
    case badURL
    case noConnection
}
```

## 에러 처리 (Handling Errors)
-  **`throws`** 이용한 에러 전파`
- **`do`-`catch`** 구문 이용하여 에러 처리
- **`try?`** 키워드 이용하여 에러 옵셔널 처리
- **`try!`** 키워드 이용하여 에러 전파 비활성화 처리

### `throws`이용한 에러 전파
>  함수, 메서드, 초기화 구문 선언시 에러가 발생할 수 있음을 나타내기 위해 파라미터 뒤에 `throws` 키워드를 작성한다
> 본문에서 `throw` 키워드는 내부에서 발생한 에러를 실제로 던져 호출 범위로 전파시킨다

- `throws`: 에러가 발생할 수 있다고 선언시 알려줌
- `throw`: 실제로 에러를 던짐

- 함수/메서드 선언
```swift
func canThrowErrors() throws -> String {
    return "OK"
}
```
- 초기화 구문 선언
```swift
init(parameters) throws {
}
```

> `throws`, `throw` 사용 예시 

- **init(name:)**
```swift
enum UserInitError: Error {
    case emptyName
}

struct User {
    let name: String

    init(name: String) throws {
        if name.isEmpty {
	        // 실제로 에러를 던짐
            throw UserInitError.emptyName
        }
        self.name = name
    }
}
```

- **checkAge(age:)**
```swift
enum AgeError: Error {
    case invalid
    case tooYoung(min: Int)
    case tooOld(max: Int)
}

func checkAge(age: Int) throws {
    if age < 0 {
        throw AgeError.invalid
    } else if age < 18 {
        throw AgeError.tooYoung(min: 18)
    } else if age > 100 {
        throw AgeError.tooOld(max: 100)
    }
}
```

checkAge(age:)는 발생하는 에러를 전파하기 때문에 이 함수를 호출하는 코드에서는 에러를 처리(`do`-`catch` / `try?` / `try!`) 해주거나, 계속 전파해야한다.

- **`validateUserAge(age:)`** 는 던지기 함수로 **`checkAge(age:)`** 에서 발생한 에러는 **`validateUserAge(age:)`** 가 호출된 지점까지 전파될 것이다
```swift
func validateUserAge(age: Int) throws {
    // checkAge가 던지기 함수이므로 try로 호출 + 전파
    try checkAge(age: age)
}
```

### **`do`-`catch`** 구문 이용하여 에러 처리
> **`do`** 절 에서 에러가 발생되면 **`catch`** 절 에서 비교하여 에러를 처리한다
>  에러를 종류에 따라 다르게 처리할 때 사용한다


```swift
func processUserAge(_ age: Int) {
    do {
        try checkAge(age: age)
        print("✅ 나이 \(age)는 유효합니다!")
    } catch AgeError.invalid {
        print("나이가 음수입니다.")
    } catch AgeError.tooYoung(let min) {
        print("나이가 너무 어립니다. 최소 \(min)세 이상이어야 합니다.")
    } catch AgeError.tooOld(let max) {
        print("나이가 너무 많습니다. 최대 \(max)세 이하이어야 합니다.")
    } catch {
        print("알 수 없는 에러: \(error)")
    }
}
```

> catch 절은 do 절에서 발생할 수 있는 모든 에러를 처리할 필요는 없다
```swift
func processUserAge(_ age: Int) throws {
    do {
        try checkAge(age: age)
        print("✅ 나이 \(age)는 유효합니다!")
    } catch is AgeError {
        print("❌ 나이 관련 에러 발생 (AgeError 처리)")
    }
    // AgeError 아닌 건 throws로 전파
}

do {
    try processUserAge(30)
} catch {
    print("❌ 최종 do-catch에서 AgeError 아닌 에러 처리: \(error)")
}
```


### **`try?`** 표현식 이용하여 에러 옵셔널 처리
> 에러를 옵셔널로 변환하여 처리하기 위해 **`try?`** 표현식을 사용한다. 에러가 발생하면 표현식의 값은 **`nil`** 이다.
>  간결하지만 어떤 에러인지는 알 수 없으므로 **에러 발생 여부만 알면 될 때** 사용한다

```swift
func fetchData() -> Data? {
    if let data = try? fetchDataFromDisk() { return data }
    if let data = try? fetchDataFromServer() { return data }
    return nil
}
```


### **`try!`** 표현식 이용하여 에러 전파 비활성화 처리
> 던지는 함수 / 메서드 / 초기화 구문이 실제로 런타임 에러를 발생하지 않는다면 에러 전파를 비활성화 처리하기 위해 **`try!`** 를 작성할 수 있다.
>  **절대 에러가 나지 않을거라 확신**할 때만 써야한다. 에러 발생시 바로 앱 크래시💀

```swift
let photo = try! loadImage(atPath: "./Resources/John Appleseed.jpg")
```


>[!try]
> 에러가 발생할 수 있는 코드를 시도(try) 해볼 때 꼭 붙여야하는 키워드이다.
> 즉, throws를 통해 에러가 발생할 수 있는 메서드를 작성했다면 무조건 try를 붙여서 사용해야한다.

## References
- https://bbiguduk.gitbook.io/swift/language-guide-1/error-handling
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/errorhandling/
- https://developer.apple.com/documentation/swift/error