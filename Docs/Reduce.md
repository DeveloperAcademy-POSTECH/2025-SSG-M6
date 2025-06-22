>[!question]
>GQ1. 왜 reduce(into: )가 따로 존재할까?
>GQ2. 같은 동작을 하는 코드를 여러 가지로 써서 비교하면 어떤 차이를 보일까?

## Description
---
`reduce` 연산자는 컬렉션의 요소들을 하나의 결과 값으로 **축약(누적)** 하는 고차함수이다. 초기값과 함께 클로저를 전달하여, 누적 계산을 순차적으로 수행한다.


## 함수 정의
---
![[reduce 함수 정의.png]]

*  `initialResult` : 누적 계산의 시작값
* `nextPartialResult` : 누적 클로저 (이전 값, 현재값 → 다음값)


## 예시 코드
---
```Swift
/// 합계
let sum = [1, 2, 3, 4].reduce(0, +) // 10

/// 문자열 연결
let words = ["Swift", "UIKit", "SwiftUI"]
let sentence = words.reduce("") { $0 + " " + $1 } // Swift UIKit SwiftUI
```


## 성능 주의점
---
```Swift
let arr = [1, 2, 3, 4, 5]

let doubled = arr.reduce([]) { $0 + [$1 * 2] } 
// O(n^2)
```

* Swift는 **값 타입 기반 언어**이므로, + 연산이 배열 복사를 유발한다.
* 반복될수록 전체 복사량이 누적되어 **심각한 성능 저하**가 발생한다.


## 해결책 : `reduce(into:\_:)`
---
![[reduce into 함수 정의.png]]

```Swift
let doubled = arr.reduce(into: []) { $0.append($1 * 2)}
// O(n)
```

* `reduce(into: )` 는 내부에서 `inout` 을 사용한다.
* 배열/딕셔너리를 직접 수정하므로 복사가 발생하지 않는다.


## ➕ reduce(into: ) 안 쓰고도 해결할 수 있는 방법은?
---
1. **`map` 을 활용한 대체**
	* 목적이 요소 변환이라면, `map`이 가독성과 성능 모두 뛰어나다.
	* 내부적으로도 inout을 활용한 최적화 구조로 설계되어 있다.
2. **`for` 루프를 이용한 명시적 누적**
	* 직접 누적 배열을 선언하고 수정하면 복사 없이 빠르게 처리할 수 있다.
	* Swift는 내부적으로 append 시 [[COW (Copy-On-Write)]]를 이용한다.
	* 수정이 실제 발생하기 전까지는 복사되지 않아 호율적이다.
	* 조건 분기나, 중단이 필요한 경우에는 `for` 문을 사용하자.
3. **`forEach`로 외부 값 누적**
	* `forEach`는 리턴값이 없지만, 외부 변수에 누적하는 용도로 자주 사용된다.
	* 단순 누적일 때는 `forEach`로도 충분하다.

```Swift
let doubled = arr.map { $0 * 2 }
```

```Swift
var doubled: [Int] = []

for num in arr {
	doubled.append(num * 2)
}
```

```Swift
var doubled: [Int] = []

arr.forEach { doubled.append($0) }
```

#### ✅ 벤치마킹 돌려본 결과 

* `reduce`는 O(n^2) → 매번 새 배열 생성 때문에 느림
* `reduce(into:)`는 O(n) → 기존 배열을 수정, 빠름
* `map`, `for`, `forEach` 모두 O(n)
	* 대신 `for` 를 사용하면 일정 개수가 넘으면 새 메모리 블록을 복사해야해서 시간 소모가 발생

```Swift
import Foundation

let arr = Array(1...100)

func benchmark(name: String, block: () -> Void) {
    let start = CFAbsoluteTimeGetCurrent()
    block()
    let end = CFAbsoluteTimeGetCurrent()
    let time = (end - start) * 1000
    print("\(name): \(String(format: "%.2f", time)) ms")
}

benchmark(name: "reduce") {
    _ = arr.reduce([]) { $0 + [$1 * 2] }
}

benchmark(name: "reduce(into:)") {
    _ = arr.reduce(into: []) { $0.append($1 * 2) }
}

benchmark(name: "map") {
    _ = arr.map { $0 * 2 }
}

benchmark(name: "for") {
    var result: [Int] = []
    for i in 0..<arr.count {
        result.append(arr[i] * 2)
    }
}

benchmark(name: "forEach") {
    var result: [Int] = []
    arr.forEach { result.append($0 * 2) }
}

/*
	reduce: 30.20 ms
	reduce(into:): 0.38 ms
	map: 0.29 ms
	for: 9.51 ms
	forEach: 0.58 ms
*/

```

## References
---
* [Apple Developer Documentation – reduce(_:_:)](https://developer.apple.com/documentation/swift/sequence/reduce\(_:_:\))
* [Apple Developer Documentation – reduce(into:_:)](https://developer.apple.com/documentation/swift/sequence/reduce\(into:_:\))
