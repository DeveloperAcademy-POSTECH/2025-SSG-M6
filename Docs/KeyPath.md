>[!question] 
>GQ1. KeyPath가 뭘까?
GQ2. KeyPath를 왜 쓸까?

## Description
---
**KeyPath**는 **객체의 특정 속성으로 접근하는 경로를 표현하는 기능**이다. 
`person.name` 처럼 직접 값을 꺼내는 것이 아니라, `\Person.name` 처럼 **속성으로 가는 길**을 변수로 저장하거나 함수로 전달할 수 있다.

KeyPath는 KeyPath<Root, Value> 형태로 정의된다.
- Root : 경로의 시작점이 되는 타입
- Value : 최종적으로 접근하는 속성의 타입

KeyPath는 데이터 흐름이 자동으로 연결도는 환경인 SwiftUI, [[컴바인 (Combine)]], RxSwift, [[Realm]] 등에서 많이 사용된다.

## 주요 기능
---
#### 1. KeyPath 정의 및 값 읽기
```Swift
struct Person {
	var name: String
	var age: Int
}

let person = Person(name: "제이", age: 32)
let path = \Person.name

print(person[keyPath: path]) // "제이"
```

#### 2. WritableKeyPath를 통한 값 수정
```Swift
struct Car {
    var model: String
}

var car = Car(model: "Tesla")
car[keyPath: \Car.model] = "BMW"
print(car.model)  // "BMW"
```

WritableKeyPath는 읽기와 쓰기가 모두 가능한 경로를 의미하며, 대상 속성이 var로 선언되어 있어야 한다. 이때 값 타입(struct)이라면 WritableKeyPath, 참조 타입(class)라면 ReferenceWritableKeyPath가 사용된다.

| **KeyPath 종류**           | **읽기** | **쓰기** | **대상 타입**    | **비고**          |
| ------------------------ | ------ | ------ | ------------ | --------------- |
| KeyPath                  | 가능     | 불가능    | 모든 타입        | 읽기 전용           |
| WritableKeyPath          | 가능     | 가능     | struct (var) | 변경 가능 속성        |
| ReferenceWritableKeyPath | 가능     | 가능     | class (var)  | 클래스 내부 속성 수정 가능 |

#### 3. 고차 함수와 함께 사용
```Swift
let people = [
    Person(name: "A", age: 20),
    Person(name: "B", age: 25)
]

let names = people.map(\.name)  // ["A", "B"]
```

#### 4. Optional과 함께 사용
```Swift
let optionalList: [Person?] = [Person(name: "C", age: 30), nil]
let names = optionalList.compactMap(\.?.name)  // ["C"]
```

## 📌  왜 SwiftUI의 ForEach에서 경로(KeyPath)를 주는걸까?
---
```Swift
ForEach(users, id: \.id) { user in
    Text(user.name)
}
```

그것은 Swift가 `.id` 와 `\.id` 를 전혀 다르게 해석하기 때문이다.
`.id` 는 그 순간 속성 값을 꺼내는 코드다.  `user.id` 같은 코드는 즉시 값을 읽어온다.
반면 `\.id` 는 그 속성에 도달하는 경로를 값으로 만든 표현이다. Swift에서 `KeyPath<User, Int>` 같은 타입으로 표현되며, 나중에 어떤 `User` 객체에 대해 `user[keyPath: \.id]` 처럼 동적으로 값을 꺼낼 수 있는 경로 정보가 된다.

SwiftUI는 뷰가 바뀔 때마다 ForEach를 순회하면서 각 요소의 id 값을 직접 꺼내 이전 값과 새로운 값을 비교하면서 "어떤 게 그대로고, 어떤 게 바뀌었는지"를 알아낸다. 만약 우리가 id 값을 미리 꺼내서 넘겨준다면, SwiftUI 입장에서는 "이 id가 어떤 user에서 나온거야?!" 라는 문제가 생긴다.

즉, 값을 넘기는 건 **이미 꺼낸 결과일 뿐**이고 SwiftUI는 각 항목에 접근하면서 그때그때 직접 꺼낼 수 있는 수단이 필요하다. 그래서 우리는 그걸 꺼낼 방법을 알려줘야하고 바로 그게 **KeyPath** 다!

## Keywords
---
- ForEach
- 고차 함수
- KVC

## References
---
-  [Apple 공식 문서: Swift KeyPath](https://developer.apple.com/documentation/swift/keypath)
- [Apple 공식 문서: SwiftUI ForEach](https://developer.apple.com/documentation/swiftui/foreach)
- [KeyPath 이해하기 – 김종권 블로그](https://ios-development.tistory.com/982)
- [SwiftUI ForEach의 id에 대해서 – programoom](https://littlemoom.tistory.com/5)
- [Swift KeyPath와 고차함수 – eunjin 블로그](https://eunjin3786.tistory.com/590)
