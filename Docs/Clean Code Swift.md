>[!question]
>GQ1. Clean Code란 무엇이지?
>GQ2. 구체적인 Swift의 Clean Code 원칙은 뭐고, 이 Clean Code 원칙을 내 코드에서 어느정도 수준까지 지키면서 코딩할 수 있을까?

## Clean Code란 무엇인가?

- 클린 코드 (Clean Code)란 “가독성 (읽기 쉬움), 유지보수성 (기존 코드를 고쳐도 영향이 크지 않음), 확장성 (새로운 코드를 추가허기가 쉬움)”을 높이는 코드를 작성하는 것을 의미합니다.
- 클린 코드는 **읽는 사람 중심의 코드**를 의미합니다. 
  -> 나중에 본인 스스로, 혹은 동료 개발자, 혹은 이후에 투입될 유지보수 담당자가 주요 독자일 것을 가정.)
- “코드는 작성하기 위해서가 아니라 읽히기 위해 작성한다.”는 철학을 따릅니다.

-> 중요한 것은 제시하는 모든 원칙을 따를 필요는 없다는 점입니다.
보편적으로 이렇게 써야만한다는 합의된 원칙도 아니며, 단순히 지침이자 권장 사항으로 이해하면 됩니다.



>[!question] 내가 판단하는 Clean Code 평가지표
>판단 기준에 따라 다를 수 있지만, 제 기준에서 **중요도**에 기반한 우선순위에 따라 별점을 나누어 평가해보겠습니다.
>- ⭐️ (필요) : 해당 사항을 준수하기 위해 노력한다면, 클린 코드를 작성하는데 분명 도움이 되는 원칙
>- ⭐️ ⭐️ (중요) : "반드시"의 수준은 아니지만, 그럼에도 불구하고 중요한 원칙
>- ⭐️ ⭐️ ⭐️ (매우 중요) : 팀에서 최소한의 컨벤션을 설정하더라도 해당 내용만큼은 반드시 지켜야 할 원칙


## Clean Code (1) : Variables
---
##### 1️⃣ Use meaningful and pronounceable variable names ⭐️⭐️⭐️

+ 의미있고 발음하기 쉬운 변수 이름을 지으라는 뜻입니다.
+ Swift는 타입 안전(type safety)과 타입 추론(type inference)이 강력하기 때문에, 타입을 이름에 명시하기보다는 ***변수 이름만 보고 용도와 타입을 짐작할 수 있는 것*** 이 유지보수에 더 도움됩니다.

```Swift
// ❌ BAD 
// Str과 같은 축약어는 발음하기 어렵다 + 이름에서 문자열임을 암시하고 있지만, 실제로는 로컬라이즈된 날짜를 표현하는 문자열로 역할이 조금 다르다.
let currentDateStr = DateFormatter.localizedString(from: Date(), dateStyle: .short, timeStyle: .none)

// ✅ GOOD 
// 날짜를 나타내는 표현을 충분히 전달함 + 타입을 변수 이름에 굳이 쓸 필요없다.
let currentDate = DateFormatter.localizedString(from: Date(), dateStyle: .short, timeStyle: .none)
```

##### 2️⃣ Use the same vocabulary for the same type of variable ⭐️⭐️

+ 같은 타입 (의미)를 갖는다면, 같은 단어를 사용하라는 말입니다.
+ 팀 컨벤션에서 "우리 프로젝트에서 사용자는 뭐라고 칭할지. 각 화면 이름은 뭐라고 칭할지" 정하고 들어가는 것이 중요할 것입니다.

```Swift
// ❌ BAD 
// 어디는 User, 어디는 Client, 어디는 Customer로 쓰지 마!
func getUserInfo()
func getClientData()
func getCustomerRecord()

// ✅ GOOD 
// 그냥 팀끼리 앱에서의 사용자는 User로 하자.라고 정해서 사용
func getUser()
```

##### 3️⃣ Use searchable names ⭐️

+ 검색할 수 있는 이름을 사용해서 상수로 분리해 사용하라는 뜻입니다.

```Swift
// ❌ BAD 
// 86400000만 봤을 때 이게 무슨 의미인지. 어떻게 찾을지 엄두조차 나지 않음
setTimeout(blastOff, 86400000)

// ✅ GOOD 
// 86400000이 하루의 초단위라는 것을 명시적으로 네이밍해 상수로 분리함. 나중에 검색하기 용이.
let millisecondsPerDay = 86400000
setTimeout(blastOff, millisecondsPerDay)
```

##### 4️⃣ Use explanatory variables ⭐️

+ 중간에 변수를 사용해 코드의 의도를 설명하는 부분을 추가하라는 의미입니다.

```Swift
// ❌ BAD 
// address.match(..)[1]은 어떤 것을 의미하는지 정규표현식으로는 알 수가 없죠.
let address = "One Infinite Loop, Cupertino 95014"
let cityZipCodeRegex = try! NSRegularExpression(pattern: "^[^,\\]+[,\\\\\\s]+(.+?)\\s*(\\d{5})?$")
saveCityZipCode(
	address.match(cityZipCodeRegex)![1], address.match(cityZipCodeRegex)![2]
)

// ✅ GOOD 
// 이런 경우 코드 줄이 늘어나더라도 각 부분을 중간 변수를 분리해 의미를 드러내는 것이 좋습니다.
let address = "One Infinite Loop, Cupertino 95014"
let cityZipCodeRegex = try! NSRegularExpression(pattern: "^[^,\\]+[,\\\\\\s]+(.+?)\\s*(\\d{5})?$")
if let match = cityZipCodeRegex.firstMatch(in: address) {
    let city = (address as NSString).substring(with: match.range(at: 1))
    let zipCode = (address as NSString).substring(with: match.range(at: 2))
    saveCityZipCode(city, zipCode)
}
```

##### 5️⃣ Avoid mental mapping ⭐️⭐️⭐️

+ 읽는 사람이 머릿속으로 자의적 해석하지 않도록 코드에 명확히 드러낼 것을 주문합니다.

```Swift
// ❌ BAD 
// l은 무엇을 의미하는지 직접 드러나지 않고 있음. l -> location 변환은 머릿속에서 자의적인 해석 
let locations = ["Austin", "New York", "San Francisco"]
locations.forEach { l in
    doStuff()
    dispatch(l)
}

// ✅ GOOD 
// location이라는 이름만으로 이 값이 도시 이름이라는 것을 바로 드러냄.
let locations = ["Austin", "New York", "San Francisco"]
locations.forEach { location in
    doStuff()
    dispatch(location)
}
```

##### 6️⃣ Avoid Unnecessary Context ⭐️

- 중복적인 클래스, 타입 이름을 변수에 또 반복해서 사용하지 말라고 설명합니다.
- 단순히 코드의 반복이 문제가 아니라 / 유지보수 시에 User -> Person으로 이름을 변경하는 경우, 타입에 종속적인 변수 이름은 문제를 의미적인 충돌을 일으킬 수 있기 때문입니다.

```Swift
// ❌ BAD 
// User 클래스의 username과 userAge 네이밍은 불필요한 반복
class User {
    var userName: String
    var userAge: Int
}

// ✅ GOOD 
// 클래스 이름이 이미 User이므로 name, age만으로도 충분히 의미가 전달됨.
class User {
    var name: String
    var age: Int
}
```



## Clean Code (2) : Functions
---
##### 1️⃣ Function Arguments (Max 2 params)

+ 함수 파라미터는 최대 2개까지만 설정하라는 의미입니다.

```Swift
// ❌ BAD 
// 인자가 많아 무엇이 무엇인지 헷갈리기 쉽다.
func createMenu(title: String, body: String, buttonText: String, cancellable: Bool) { ... }

// ✅ GOOD 
// MenuViewData 구조체로 관련 데이터를 의미 단위로 묶어 -> 유지보수가 용이해짐
struct MenuViewData {
    let title: String
    let body: String
    let buttonText: String
    let cancellable: Bool
}

func createMenu(viewData: MenuViewData) { ... }
```

##### 2️⃣ Use Default Arguments Instead of Short-Circuiting or Using Conditionals

+ ??나 조건문을 사용하기보다, 함수의 기본 파라미터를 사용하는 것을 권장합니다.
+ 코드의 "논리"가 추가됨으로써 코드의 흐름을 따라가기 어려워지는 부분이 존재하기 때문이죠.

```Swift
// ❌ BAD 
func createMicrobrewery(name: String?) {
    let breweryName = name ?? "Hipster Brew Co."
    // ...
}

// ✅ GOOD 
func createMicrobrewery(breweryName: String = "Hipster Brew Co.") {
    // ...
}
```

##### 3️⃣ Functions Should Do One Thing

- 함수는 한 가지 일만 하도록 선언합니다.
- 단일 책임 원칙 (SRP)에 부합하기 때문에 너무나도 알고 있는 내용이지만, 막상 구현하다보면 쉽게 적용하기가 어려운 내용인 것 같아요. 아무튼 클린 코드 원칙에서는 중요합니다!

```Swift
// ❌ BAD 
// client 조회 + 활성 여부 확인 + 이메일 발송 → 너무 많은 일을 하나의 함수가 함
func emailClients(clients: [Client]) {
    clients.forEach { client in
        let clientRecord = database.lookup(client)
        if clientRecord.isActive() {
            email(client)
        }
    }
}

// ✅ GOOD 
// emailActiveClients는 활성 클라이언트 이메일 전송에 집중. isActiveClient 는 활성 여부만 확인
func emailActiveClients(clients: [Client]) {
    clients
        .filter { isActiveClient(client: $0) }
        .forEach { email(client: $0) }
}

func isActiveClient(client: Client) -> Bool {
    let clientRecord = database.lookup(client)
    return clientRecord.isActive()
}
```

##### 4️⃣ Function Names Should Say What They Do ⭐️⭐️⭐️

+ 변수와 마찬가지로 함수 이름만 보고 이 함수가 무슨 일을 하는지 알 수 있어야 합니다.

```Swift
// ❌ BAD 
// addToDate 라는 이름은 무엇을 더하는지 알 수 없음
func addToDate(date: Date, month: Int) { ... }
addToDate(date: Date(), month: 1)

// ✅ GOOD 
// month 를 date 에 더한다는 의도가 이름에서 명확히 보임.
func addMonthToDate(month: Int, date: Date) { ... }
addMonthToDate(month: 1, date: Date())
```

##### 5️⃣ Functions Should Have One Level of Abstraction ⭐️

+ 함수는 한 단계의 추상화 수준만 가져야합니다.
+ 고수준 (비즈니스 로직, 예를들어 사용자를 결정하는 등)과 저수준 (파일 쓰기, 문자열 파싱 등) 작업을 하나의 함수에 담지 말 것을 설명합니다.

```Swift
// ❌ BAD 
// 파싱(로우레벨)과 저장(하이레벨)을 하나의 함수가 같이 수행. 추상화 계층이 섞임.
func parseInput(input: String) {
    // parsing
    let inputData = ...
    // saving
    saveData(inputData)
}

// ✅ GOOD 
// parseInput 은 파싱만, saveData 는 저장만 담당. 각 함수가 동일한 추상화 수준에서 작동.
func parseInput(input: String) -> Data {
    // parsing
    return ...
}

func saveData(data: Data) {
    // saving
}
```

##### 6️⃣ Remove Duplicated Code

+ 중복 코드를 제거하라는 원칙입니다. 너무도 당연!
+ 이 부분에서는 [[열거형 (enum)]]을 활용해서 역할별 로직을 분리하는 방식이 코드의 중복을 줄이는데 도움을 줄 수 있을 것 같네요 !

```Swift
// ❌ BAD 
// 비슷한 구조와 중복 코드가 반복되고 있는 느낌이네요!
func showDeveloper(name: String) {
    print("Developer: \(name)")
    print("Coding...")
}

func showManager(name: String) {
    print("Manager: \(name)")
    print("Meeting...")
}

// ✅ GOOD 
// enum 선언을 통해 역할별 분기 처리를 나눠주고, 하나의 메서드에서 공통된 동작을 처리합니다.
enum Role {
    case developer
    case manager

    var description: String {
        switch self {
        case .developer: return "Coding..."
        case .manager: return "Meeting..."
        }
    }
}

func showPerson(name: String, role: Role) {
    print("\(role): \(name)")
    print(role.description)
}
```

##### 7️⃣ Avoid Using Flags as Function Parameters

+ Bool 값 (true/false)을 통해 프로그램의 흐름 (로직)을 분기처리하는 Flag를 함수의 매개변수로 사용하지 말 것을 소개합니다.
+ Bool flag가 함수의 파라미터라는 것은 "이 함수가 두 가지 이상의 역할을 한다"는 신호이기 때문이죠.
+ Bool 값을 사용하는 것보다, 위에서 설명한 enum 기반 상태 분기처리를 관리하면 / 각각의 상태가 더욱 명확하게 드러나 관리가 편해질 것 같다는 개인적인 생각도 추가 !

```Swift
// ❌ BAD 
// 해당 함수를 호출할 때, temporary에 따라 어떻게 동작이 나누어질지 상상하기가 어려움
func createFile(name: String, temporary: Bool) {
  if temporary {
    // 임시 파일 생성
  } else {
    // 영구 파일 생성
  }
}

// ✅ GOOD 
// 함수 이름이 명확하고, 두 함수는 각각 하나의 책임만을 가집니다.
func createTemporaryFile(name: String) {
  // 임시 파일 생성
}

func createPermanentFile(name: String) {
  // 영구 파일 생성
}
```

##### 8️⃣ Avoid Side Effects

+ Side Effect란 "함수가 입력값만 받아 리턴값만 반환하는 것이 아니라, 외부 상태 (ex 전역)를 변경하거나 의존하는 것"을 의미합니다.
+ 함수에서 발생하는 Side Effect는 발생하지 않도록 하라는 것이 요지입니다.
+ 해당 함수의 동작이 -> 다른 외부의 변화를 일으킨다면 코드를 예측하기 어렵고, 테스트 또한 어려워지겠죠.

+ 특히! Struct와 Class의 값타입/참조타입 복사 개념에 대해 주의할 필요가 있습니다.

```Swift
// ❌ BAD 
// 함수가 외부 전역 변수 name을 직접 수정하고 있음
var name = "Matheus Gois"
func splitIntoFirstAndLastName() {
  name = name.split(separator: " ").joined(separator: " ")
}

// ✅ GOOD 
// 해당 함수는 순수 함수 (Pure Function)로, 단순 입력 -> 출력만 수행하는 역할임
func splitIntoFirstAndLastName(name: String) -> (firstName: String, lastName: String) {
  let components = name.split(separator: " ").map { String($0) }
  guard components.count >= 2 else { return (firstName: name, lastName: "") }
  return (firstName: components[0], lastName: components[1...].joined(separator: " "))
}
```

##### 9️⃣ Do Not Write to Global Functions

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 🔟 Favor Functional Programming over Imperative Programming

+ 명령형 프로그래밍 (Imperative Programming)보다 함수형 프로그래밍 (Functional Programming) 방식을 사용하라고 설명합니다.

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 1️⃣1️⃣ Encapsulate Conditionals

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 1️⃣2️⃣ Avoid Negations in Conditionals

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 1️⃣3️⃣ Avoid Conditionals

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 1️⃣4️⃣ Avoid Type Checking

+ `is` `as?` 와 같은 수동 타입 체킹을 함수에서 사용하는 상황을 피하라는 뜻입니다.
+ 만약, 

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 1️⃣5️⃣ Remove Dead Code

+ 사용하지 않는 죽은 코드를 코드 베이스에 남겨두지 말라는 뜻입니다.
+ Git이 있으니 언제든지 히스토리 복원이 가능할뿐만 아니라, 괜히 코드에 혼란만 주기 때문이죠.

```Swift
func oldRequestModule(url: String) { /* ... */ }  // ❌ BAD - Dead Code!
func newRequestModule(url: String) { /* ... */ }
let req = newRequestModule
```



## Clean Code (3) : Objects and Data Structures
---
##### 1️⃣ Use Pure Objects

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 2️⃣ Make decisions based on an object

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 3️⃣ Use Getters and Setters

- 

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```



## Clean Code (4) : Classes
---
##### 1️⃣ Method Chaining

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 2️⃣ Prefer Composition Over Inheritance

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```



## Clean Code (5) : SOLID
---
##### 1️⃣ Single Responsibility Principle (SRP)

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 2️⃣ Open/Closed Principle (OCP)

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 3️⃣ Liskov Substitution Principle (LSP)

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 4️⃣ Interface Segregation Principle (ISP) with Protocols

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 5️⃣ Dependency Inversion Principle (DIP)

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```



## Clean Code (6) : Testing (생략)



## Clean Code (7) : Concurrency
---
##### Async/Await is even cleaner than Promises

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```



## Clean Code (8) : Error Handling
---
##### Don't ignore caught errors

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```



## Clean Code (9) : Formatting
---
##### 1️⃣ Use spaces instead of tabs

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 2️⃣ Use black lines effectively

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 3️⃣ Limit line length

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 4️⃣ Use consistent spacing

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 5️⃣ Avoid excessive whitespace

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 6️⃣ Follow the formatting recommended by SwiftLint

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```



## Clean Code (10) : Comments
---
##### 1️⃣ Only comment on things that have business logic complexity

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 2️⃣ Don't leave commented-out code in your codebase

+ 실제 활

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```

##### 3️⃣ Avoid change markers

+ 실제 활용을 작성

```Swift
// ❌ BAD 
// 

// ✅ GOOD 
// 

```



## References

- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)