>[!question]
>GQ1. Protocol Extension이 뭘까?
>GQ2. 사용하면 뭐가 좋을까?

## Description
#### 개념
- 프로토콜에 **기본 구현**을 제공하는 기능
#### 장점
- 공통 코드를 한 번에 정의하여 코드 중복을 방지하고 일관성을 유지하게 한다.
### 예시
#### 1. 공통 기능을 묶은 `Ssg` 프로토콜
```swift
protocol Ssg {
    func study()
}
```

#### 2. `Ssg` 프로토콜을 확장해서 공통되는 동작은 기본 구현으로 제공
```swift
extension Ssg {
    func prepareForStudy() {
        print("주제를 정하고 이슈를 판다🔥")
    }
}
```

#### 전체 코드 (실행 가능 🙆‍♀️)
```swift
import Foundation

// MARK: - Protocol 정의
protocol Ssg {
    func study()
}

// MARK: - Protocol Extension: 공통 기능 기본 구현 제공
extension Ssg {
    func prepareForStudy() {
        print("주제를 정하고 이슈를 판다🔥")
    }
}

// MARK: - 각 사람별 Struct 정의 및 채택
struct Jam: Ssg {
    func study() {
        print("브랜치 파자마자 성실하게 공부 시작🧐")
    }
}

struct Gus: Ssg {
    func study() {
        print("팀장에게 점수를 따기 위해 노력함🕺")
    }
}

// MARK: - 테스트
let jam = Jam()
jam.prepareForStudy() // "주제를 정하고 이슈를 판다🔥"
jam.study()           // "브랜치 파자마자 성실하게 공부 시작🧐"

let gus = Gus()
gus.prepareForStudy() // "주제를 정하고 이슈를 판다🔥"
gus.study()           // "팀장에게 점수를 따기 위해 노력함🕺"

```


#### + Protocol Extension을 사용하지 않았더라면?
- 아래와 같이 공통되는 코드를 반복해서 구현해야 했겠죠!
```swift
struct Jam: Ssg {
	func prepareForStudy() {
	        print("주제를 정하고 이슈를 판다🔥")
	    }
    func study() {
        print("브랜치 파자마자 성실하게 공부 시작🧐")
    }
}

struct Gus: Ssg {
	func prepareForStudy() {
	        print("주제를 정하고 이슈를 판다🔥")
	    }
    func study() {
        print("팀장에게 점수를 따기 위해 노력함🕺")
    }
}

```


### 조건부 확장
#### 현실 세계 예시
#### 1. 팀장만 채택할 프로토콜 생성
```swift
protocol TeamLeader {
    func giveScore(to member: Ssg)
}
```

#### 2. `TeamLeader`만 `giveScore()`를 쓸 수 있도록 조건부 제한걸기
```swift
extension Ssg where Self: TeamLeader {
    func giveScore(to member: Ssg) {
        print("\(self.name) 팀장이 \(member.name)에게 점수를 준다")
    }
}
```

#### 전체 코드 (실행 가능 🙆‍♀️)
```swift
import Foundation

// MARK: - Protocol 정의
protocol Ssg {
    func study()
}

// MARK: - 팀장 전용 프로토콜 정의
protocol TeamLeader {
    func giveScore(to member: Ssg)
}

// MARK: - TeamLeader만 사용할 수 있는 기능 제공 (프로토콜 확장 + 제약조건)
extension Ssg where Self: TeamLeader {
    func giveScore() {
        print("팀원들에게 점수를 줌🎉")
    }
}

// MARK: - 각 사람별 Struct 정의 및 채택
struct Jam: Ssg {
    func study() {
        print("브랜치 파자마자 성실하게 공부 시작🧐")
    }
}

struct Gus: Ssg {
    func study() {
        print("팀장에게 점수를 따기 위해 노력함🕺")
    }
}

struct Jay: Ssg, TeamLeader {
	func study() {
        print("솔선수범하여 미리미리 공부함📑")
    }
}

// MARK: - 테스트
let jam = Jam()
jam.study()           // "브랜치 파자마자 성실하게 공부 시작🧐"

let gus = Gus()
gus.study()           // "팀장에게 점수를 따기 위해 노력함🕺"

let jay = Jay()
jay.study()           // "솔선수범하여 미리미리 공부함📑"
jay.giveScore() // "팀원들에게 점수를 줌🎉"

```

#### 실제 예시
- `Collection` 안의 요소가 `Equatable`일 때만 비교(`==`)가 가능
- 따라서 `Element: Equatable`일 때만 `allEqual()` 메서드를 사용할 수 있게함
```swift
extension Collection where Element: Equatable {
    func allEqual() -> Bool {
        guard let first = self.first else { return true }
        return !self.contains { $0 != first }
    }
}
```

>[!Equatable을 따르는 대표적인 타입]
>|타입|설명|
|---|---|
|`Int`, `Double`, `Float`|숫자 비교 가능|
|`String`, `Character`|문자열 비교 가능|
|`Bool`|true/false 비교 가능|
## Keywords
- [[Protocol-Oriented Programming (POP)]]
+ [[익스텐션 (Extension)]]

## References
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/#Protocol-Extensions