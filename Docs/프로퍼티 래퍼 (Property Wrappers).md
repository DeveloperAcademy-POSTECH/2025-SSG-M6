
### Property Wrappers 란?

swift 5.1에 새로 추가된 기능으로 프로퍼티(property)에 
반복적으로 적용되는 로직을 재사용 가능하게 만드는 기능입니다. 

보일러 플레이트와 중복된 코드를 줄이고, 속성에 부가 기능(예: 값 검증, 디폴트 값, 변환등)을
쉽게 할 수 있습니다. 

swiftUI 에서 
@Published
@Binding
@ObservedObject
@State
이런놈들이 property wrapper들입니다.


### 왜 생겨 났나?

기존에는 속성에 부가 기능을 추가 할 때 마다 getter/setter 혹은 
별도 메서드를 정의 해야 했습니다.

```swift
var volume: Int {
    didSet {
        if volume > 100 { volume = 100 }
        if volume < 0 { volume = 0 }
    }
}
```
이런 로직이 여러 속성에 반복되면 **유지 보수와 재사용성이 떨어집니다.** 
Property Wrapper를 사용하면 이러한 부가 기능을 하나의 타입으로 
캡슐화할 수 있어 코드의 재사용성과 가독성이 크게 향상됩니다.

### Property Wrappers  사용방법
예시로 아래 School과 Learner구조체에 name이라는 이름을 대문자로 바꿔
주는 프로퍼티로 중복된 코드가 있습니다. 이 중복된 코드를 propertyWrapper
바꿔보겠습니다. 
```swift
struct School {

    private var _name: String = ""

    var name: String {
        get {
            self._name.uppercased()
        }
        set {
            self._name = newValue
        }
    }

    init(name: String){
        self.name = name
    }
}

struct Learner {

    private var _name: String = ""

    var name: String {
        get {
            self._name.uppercased()
        }
        set {
            self._name = newValue
        }
    }

    init(name: String){
        self.name = name
    }
}

let school = School(name: "postech")

let learner = Learner(name: "gus")

print(school.name, learner.name) // POSTECH, GUS
```

타입 앞에 붙힐 수 있음
이렇게 이타입은 특별하다고 알려줌
```swift
@propertyWrapper
struct Uppercase {}
```

타입 앞에 붙이는 것이 아닌
내가 어떤 프로퍼티에 하고싶은 행동에 @propertyWrapper를
붙입니다.  그리고 School과 Leaner 구조체에서 반복되는 구간을
wrappedValue라는 프로퍼트로 바꿔줍니다.

```swift
@propertyWrapper
struct Uppercase {

    private var value: String = ""

    var wrappedValue: String {
        get {
            self.value
        }
        set {
            self.value = newValue.uppercased()
        }
    }

    init(wrappedValue initialValue: String) {
        self.wrappedValue = initialValue
    }
}
```

이제 아래 코드처럼 사용하면 됩니다. 

```swift
struct School {
    @Uppercase var name: String
}

struct Learner {
    @Uppercase var name: String
}

let school = School(name: "postech")
let learner = Learner(name: "gus")

print(school.name, learner.name) // POSTECH GUS
```

@Uppercase도 하나의 struct로 만들어져 있기 때문에 
아래 코드 처럼 사용 가능합니다. 

```swift
struct School {
	@Uppercase(wrappedValue: "postech")
	var name: String
}

let school = School()
print(school.name) //POSTECH
```

아시다시피 연산 프로퍼티는 자기자신의 값을 바꿀 수 없기 때문에 
Private저장 프로퍼티를 하나 두고 그 프로퍼티를 가지고 연산을 했지만
그게 싫다면 didSet을 이용하는 방법도 있습니다. 

didSet은 init시에는 호출되지 않으므로 init에서도 uppercased를 호출
해줘야 합니다 

```swift
@propertyWrapper
struct Uppercase {

    var wrappedValue: String {
       didSet {
	       self.wrappedValue = wrappedValue.uppercased()
       }
    }

    init(wrappedValue initialValue: String) {
        self.wrappedValue = initialValue.uppercased()
    }
}

struct School {
    @Uppercase var name: String
}

let school = School(name: "apple academy")
print(school.name) // APPLE ACADEMY

```

### 🫡 보너스 실무 예시 코드

1. UserDefaults 연동
		앱 설정 저장시 사용
```swift
@propertyWrapper
struct UserDefault<T> {
    let key: String
    let defaultValue: T

    var wrappedValue: T {
        get { UserDefaults.standard.object(forKey: key) as? T ?? defaultValue }
        set { UserDefaults.standard.set(newValue, forKey: key) }
    }
}

struct Settings {
    @UserDefault(key: "hasSeenOnboarding", defaultValue: false)
    static var hasSeenOnboarding: Bool
}

```


2. 값 감시 및 로깅
```swift
@propertyWrapper
struct Logged<Value> {
    private var value: Value

    var wrappedValue: Value {
        get { value }
        set {
            print("Setting value to \(newValue)")
            value = newValue
        }
    }

    init(wrappedValue initialValue: Value) {
        self.value = initialValue
    }
}

struct DebugExample {
    @Logged var status: String = "Idle"
}

```