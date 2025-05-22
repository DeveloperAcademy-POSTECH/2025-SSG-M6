>[!question]
>GQ1. 클로저의 캡처(Capture) 기능은 무엇인가요?
>GQ2. [@escaping](https://github.com/escaping) 클로저와 non-escaping 클로저의 차이점은 무엇인가요?
>GQ3. 트레일링 클로저(Trailing Closure) 문법은 어떤 경우에 유용한가요?


## Description

### 클로저

변수나 상수에 저장할 수 있는 일급 시민 함수로, 이름이 없는 **익명 함수** 혹은 **코드 블록**을 의미한다.
함수처럼 인자로 넘기거나 반환값으로 사용할 수 있다. 두 가지 형태의 클로저가 존재하며, 종류는 아래와 같다.

- Named Closure → `func hello() {…}`
    
    위와 같은 형태로 작성되는 이름이 붙은 클로저로, 일반적인 함수를 의미한다.
    
- **Unnamed Closure** → `let hello = { print(”hello”) }`
    
    위와 같은 형태로 작성되는 이름 없는 클로저로, 익명 함수(즉, 보통 클로저라고 불리는 경우)를 의미한다.
    

#### 구조

```swift
{ (parameters) -> returnType in
    // 실행 코드
}
```

#### 사용

```swift
let closure = { () -> () in
    print("HEllO")
}

// 위 형태처럼 파라미터와 반환값이 없는 경우는 () 생략 가능하다.
let closure = { print("HELLO") }

// 사용
closure()
```


#### 주의!
클로저는 Argument Label을 사용하지 않기 때문에, 파라미터가 있는 경우 사용에 주의해야 한다.

```swift
let closure = { (name: String) -> String in
    return "HELLO, \\(name)"
}

closure("Berry")
closure(name: "Berry") // error!!!
```

#### 일급 시민 함수

앞서, 클로저는 일급 시민 함수라고 언급했었다. 일급 시민 함수란 뭐고 각각 클로저에서 어떻게 확인될까?

→ First-Class Citizen. 일급 객체를 의미하며 아래와 같은 특징을 갖는다.

- 변수에 저장 가능
- 함수의 인자 혹은 반환값으로 사용 가능
- 실행 중에 생성되거나 전달 가능

각각을 클로저에서는 아래처럼 확인할 수 있다.

- 변수/상수에 저장 가능
    
    ```swift
    let closure = { print("HELLO") }
    let helloClosure = closure
    
    helloClosure() // HELLO 출력됨
    ```
    
- 파라미터로 전달 가능
    
    ```swift
    func closureEx(actions: () -> Void) {
        action()
    }
    
    closureEx(action: {
        print("Play Action")
    })
    ```
    
- 반환값으로 클로저 사용
    
    ```swift
    func closureEx() -> () -> Void {
        return {
            print("Return")
        }
    }
    
    let closure = closureEx()
    closure()
    ```
    
- 실행 중 생성된다. 
	-> 리터럴 처럼 보이지만, 함수가 실행되는 런타임 시점에 생성된다.
    
```swift
func closureEx(for word: String) -> () -> Void {
    return {
        print("\(word)")
    }
}

// 이 시점에 생성된다.
let closer = closureEx(for: "hello")
closer()
```


### 특징

#### 경량 문법

타입 추론, 후행 클로저, 단축 인자 이름 등을 지원해 다양한 형태의 축약이 가능하다.

- 기본 클로저 → Inline 클로저 방식
    
    ```swift
    func doSomething(closure: (Int, Int) -> Int) {
        closure(1, 2)
    }
    
    // 함수 호출
    doSomething(closure: { (a: Int, b: Int) -> Int in
        return a + b
    })
    ```
    
    > Inline Closure?
    
    클로저가 파라미터의 값 형식으로 함수 호출 구문 () 안에 작성되어 있는 경우
    
- Parameters, returnType 생략
    
    ```swift
    doSomething(closure: { (a, b) in
        return a + b
    })
    ```
    
- Shortand Argument Names 사용
    
    단축 인자 이름 사용해 파라미터명 대체하고 in 키워드 제거.
     -> 파라미터가 존재하는 경우에는 in 키워드 제거 불가능하며, 단축인자를 사용해 파라미터가 없는 경우에만 제거가 가능하다.
    
    ```swift
    doSomething(closure: {
        return $0 + $1
    })
    ```
    
- return 생략
    
    클로저 내부 코드가 return 부분만 남은 단일 리턴문의 경우에는 생략 가능하다.
    
    ```swift
    doSomething(closure: {
        $0 + $1
    })
    
    // 단일 리턴문이 아닌 경우는 생략 불가능
    doSomething(closure: {
        print("--")
        $0 + $1
    } // 오류 발생
    ```
    
- Trailing Closure 사용
    
    함수의 **마지막 파라미터가 클로저**일때, 이를 파라미터 값 형식이 아닌 함수 뒤에 붙여 작성 가능하다.
    이때, **Argument Label은 생략**된다.
    
    ```swift
    doSomething() {
        $0 + $1
    }
    ```
    
- () 생략
    
    파라미터가 하나인 경우 () 생략이 가능하다.
    
    ```swift
    doSomething {
        $0 + $1
    }
    ```
    

#### 클로저 캡처

클로저는 외부 변수를 캡처해서 사용 가능하며, 변수의 복사본이 아닌 참조를 캡처한다.
즉, 아래 코드에서 count는 함수 내부에 있지만, 클로저가 참조하기 때문에 함수가 끝나도 메모리에 남아 있다.
이런 동작으로 인해 메모리 누수(순환 참조)가 발생하게 될 수도 있다.

```swift
func makeCounter() -> () -> Int {
    var count = 0
    return {
        count += 1
        return count
    }
}

let counter1 = makeCounter()
print(counter1()) // 1
print(counter1()) // 2

let counter2 = makeCounter()
print(counter2()) // 1
```

#### 순환 참조

클로저가 self를 강하게 참조하면 메모리 누수 발생한다.
클로저는 self를 캡처하기 때문에, weak self로 참조 순환을 방지해야 한다.
특히 UIViewController에서 클로저 사용 시점에 주의해야 한다.

```swift
class Downloader {
    var onDownload: (() -> Void)?
    
    func start() {
    onDownload = { [weak self] in
        self.doSomething()
    }
    
    func doSomething() { print("Downloaded") }
}
```

- [weak self]
    
    약한 참조로 클로저가 self를 소유하지 않도록 한다. 순환 참조 방지를 위해 사용.
    
- [unowned self]
    
    self가 반드시 존재하는 경우에 사용 가능하다. self가 먼저 해제되면 크래시가 발생할 수 있다.
    

#### 탈출 클로저

일반적으로 클로저는 non-escaping Closure이다. 즉, 함수가 종료되고 나서 클로저가 실행될 수 없다.

함수 실행이 끝난 후에도 클로저가 나중에 실행될 수 있을 때 사용되며, 중첩함수에서 실행 후 중첩 함수를 리턴하는 경우, 변수 상수에 대입하고 싶은 경우, 비동기 처리의 경우에 많이 사용된다.

```swift
func downloadImage(completion: @escaping () -> Void) {
    DispatchQueue.global().async {
        completion()
    }
}
```

> non-secaping Closure ?

- 함수 내부에서 직접 실행하기 위해서 사용되기에 파라미터로 받은 클로저를 변수나 상수에 대입할 수 없다.
- 중첩 함수에서 클로저를 사용할 경우, 중첨 함수를 리턴할 수 없다.
- 함수의 실행 흐름을 탈출하지 않아, 함수가 종료되기 전에 무조건 실행 되어야 한다.
    - 함수가 종료되고 나서는 클로저가 실행될 수 없다.

#### 주의!!

@escaping 키워드 없으면 클로저가 함수 종료 전에 실행된다고 가정하기 때문에 에러 발생한다.

→ 메모리 관리와 관련되어 있다.

#### Autoclosure

파라미터로 전달된 표현식(일반 구문과 함수)을 자동으로 클로저로 래핑할 때 사용한다.
클로저 파라미터 앞에 @autoclosure 작성해 사용 가능하며, 반드시 파라미터가 없어야 사용 가능하다.
주로 지연 평가에 유용하며, 일반 구문은 작성되자마자 실행되어야 하지만 autoclosure 형태로 작성하면 함수가 실행될 시점에 구문을 클로저로 만들어 주기 때문에 함수 내에서 클로저를 실행할 때 까지 구문이 실행되지 않는다.

즉, 바로 실행되어야 할 구문이 지연되어 실해되게 된다.

아래와 같은 코드에서 predicate는 실제 클로저를 전달받지 않지만, 클로저를 래핑 했기 때문에 클로저처럼 사용 가능하다.

```swift
func logIfTrue(_ predicate: @autoclosure () -> Bool) {
    if predicate() {
        print("True")
    }
}

logIfTrue(3 > 1)
```

#### 클로저의 실제 사용 예


- 애니메이션
    
```swift
UIView.animate(withDuration: 0.3) {
    view.alpha = 0
}
```

- 비동기 콜백
	  
  ```swift
fetchData { data in
    self.updateUI(with: data)
}
```

- 고차 함수
	
	클로저는 고차 함수의 핵심 동료이다. 클로저를 이해해야 Swift 컬렉션 처리에 능숙해질 수 있다.

```swift
let doubled = [1, 2, 3].map { $0 * 2 }
let evens = [1, 2, 3, 4].filter { $0 % 2 == 0 }
```



### 클로저 사용 주의점

- 클로저는 값 캡처를 통해 강한 참조 순환을 유발할 수 있다
- 지나친 중첩 또는 클로저 트리거 방식은 디버깅과 추적을 어렵게 한다
- 클로저가 self를 캡처하는 경우는 항상 의도적으로 접근해야 한다

