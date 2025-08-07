>[!question]
>GQ1. 테스트 함수의 종류는 뭐가 있을까?
>GQ2. `import XCTest`와 `import Testing`의 차이점
>GQ3. `@testable import`를 사용할 때 주의할 점은?
>


## XCTest

> Apple의 공식 테스트 프레임워크로, iOS/macOS 앱에서 유닛 테스트, 통합 테스트, UI 테스트 등을 작성하고 실행할 수 있게 해주는 도구입니다.

단축키 : **Command + U**

### 테스트 코드 작성 5대 원칙 (FIRST)

**Fast** : 빠르게
**Independent** : 테스트끼리 서로 영향을 주지 않고
**Repeatable** : 환경에 관계 없이 언제든 같은 결과를 내야 하고
**Self - Validating** : 테스트 결과가 명확해야 하고
**Timely** : 적절한 시기에 작성해야 함

## 테스트 함수의 종류는 뭐가 있을까?
#### 검증 함수

테스트의 결과가 기대한 값과 일치하는지를 확인

`XCTAssertTrue` : 값이 true인지 확인
`XCTAssertFalse` : 값이 false인지 확인
`XCTAssertEqual` : 두 값이 같은지 확인
`XCTAssertNotEqual` : 두 값이 다른지 확인
`XCTAssertNil` : 두 값이 nil인지 확인
`XCTAssertNotNil` : 두 값이 nil이 아닌지 확인
`XCTAssertFail` : 무조건 실패 처리
`XCTAssertThrowsError` : 에러를 던지는지 확인
`XCTAssertNoThorw` : 에러 없이 실행되는지 확인

#### 비동기 테스트 함수

`expectation(description:)` : 비동기 작업을 기다리기 위한 기대값 생성
`wait(for:timeout:)` : 특정 기대값이 충족될 때까지 대기
`fulfill()` : 기대값이 충족되었음을 표시

#### 성능 측정 함수

```swift
measure {
	// 여기에 성능 측정하고 싶은 코드
}
```

####  초기화 및 정리용 메서드

`override func setUp()` : 각 테스트 **시작 전**에 실행됨
`override func tearDown()` : 각 테스트 **끝난 후**에 실행됨


## import Testing vs import XCTest

![[XCTest_Testing.png]]
#### import XCTest

> 전통적인 테스트 방식

- 테스트를 class로 만들고, test로 시작하는 함수 사용
- XCTestCase를 상속해야 사용 가능

```swift
import XCTest

final class MathTests: XCTestCase {
    func testAdditionIsCorrect() {
        let result = 1 + 2
        XCTAssertEqual(result, 3)
    }
}
```

#### import Testing

> Swfit 5.9부터 등장한 새로운 문법의 테스트 방식

- 테스트를 구조체로 만들고, `#Test`라는 걸 붙여서 테스트를 정의
- expect() 구문으로 결과 검증
- Swift Package와 호환성이 좋음
- 
```swift
import Testing

#Test("테스트 설명란") {
    let result = 1 + 2
    expect(result) == 3
}
```

## `@testable import`를 사용할 때 주의할 점은?

```swift
@testable import Yut
```

테스트 코드에서 실제 앱 모듈에 있는 내부 코드까지 접근 가능하게 해주는 import
@testable을 붙이면 모듈의 internal 접근 수준까지 테스트에서 접근이 가능함 !

- 캡슐화 원칙을 해칠 수 있음
	- 외부에서는 보이지 않아야 할 내부 로직까지 테스트에서 접근할 수 있기 때문에, 잘못 사용하면 코드의 구조가 테스트에 의존적인 방향으로 설계될 위험이 있음

- 테스트 대상 모듈의 Build Setting에서 Enable Testability가 Yes여야 함
	- @testable은 Debug 환경에서만 유효하며, Release 빌드에서는 internal 접근이 막혀 있음.

- 모듈이 잘 분리되어 있어야 효과적임
	- 하나의 거대한 모듈에 모든 기능이 몰려 있다면, @testable 사용으로 인해 모듈 내부 구현에 테스트가 과도하게 결합될 수 있음
## References
- [XCTest](https://developer.apple.com/documentation/xctest)
- [Swift Testing](https://developer.apple.com/kr/xcode/swift-testing/)