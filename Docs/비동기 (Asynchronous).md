>[!question] 
>GQ1. 동시성 프로그래밍에서 동기(Synchronous)와 비동기(Asynchronous)의 차이점은 무엇인가?
>GQ2. iOS에서 비동기 작업을 처리하는 방법은 무엇인가?


## Description
---
#### 비동기

어떤 작업을 실행할 때, 그 작업의 완료 여부와 관계없이 다음 코드를 실행하는 방식이다.
즉, 결과를 기다리지 않고 다음 코드 작업을 실행한다.

#### 동기

어떤 작업을 실행할 때, 해당 작업이 완료될 때까지 기다린 후 다음 작업을 실행하는 방식이다.
즉, 작업이 순차적으로 처리되어 순서를 보장하고, 한 작업이 완료 되어야지만 다음 작업이 실행된다.

#### ❓왜 비동기 처리가 필요할까?

- 사용자 경험 개선
	- 비동기 처리를 하지 않으면, 사용자는 자신이 요청한 작업에 대한 처리가 완료될 때 까지 무한정 기다려야 한다. 이런 동작은 사용자 경험을 저해시키는 큰 문제이기에 비동기 처리를 통해서 사용자의 요청에 빠르게 반응하는 것처럼 보이도록 만들어야 한다.
- 네트워크 통신
	- 서버와의 데이터 통신 과정에서, 비동기 처리를 하지 않으면, 서버의 응답을 기다리는 동안 다른 작업을 전혀 수행할 수가 없다. 이런 동작은 앱 동작의 반응성을 해친다. 따라서 비동기 적으로 서버로부터 데이터를 받아와 앱 반응성을 향상시켜야 한다.
- 병렬 처리
	- 비동기 처리를 통해 여러 작업을 동시에 처리할 수 있다. 이를 통해서 시간이 오래 걸리는 작업을 병렬적으로 처리할 수 있으며, 결과적으로 작업 시간을 줄일 수 있다.
- 에러 처리
	- 비동기적으로 처리할 때 에러가 발생하면, 해당 에러를 쉽게 처리할 수 있다. 에러 발생 시, 에러를 처리할 수 있는 콜백 함수를 실행하거나, 에러를 캐치해 처리할 수 있어 프로그램의 안정성을 높일 수 있다.



## 코드 예시
---
### 비동기 처리 방법

Swift에서 비동기 처리 방법은 다양하며, 각각 점점 더 개선된 방법을 제공한다.
일반적으로 프로그램은 작업을 순차적으로 실행하는 동기적인 방식으로 동작한다.
즉, 앞의 일이 끝나야 다음 일을 시작한다.
하지만, 비동기 처리에서는 작업을 병렬적으로 실행해, 작업의 완료를 기다리지 않고 다음 작업을 실행한다.


#### 1. Completion Handler

Completion Handler는 비동기 작업이 완료된 후 완료된 작업을 처리해주는 방식이다.
즉, 비동기 작업이 완료된 후 호출되는 콜백 함수를 의미하며, [[클로저 (Closure)]]를 이용해 구현된다.
작업이 완료되면 Completion Handler가 호출되고, 결과나 에러를 전달한다.

```swift
func downloadImage(from url: URL, completion: @escaping @Sendable (Result<UIImage, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, response, error in
        if let error = error {
            DispatchQueue.main.async {
                completion(.failure(error))
            }
            return
        }

        guard let data = data, let image = UIImage(data: data) else {
            DispatchQueue.main.async {
                completion(.failure(NSError()))
            }
            return
        }

        DispatchQueue.main.async {
            completion(.success(image))
        }
    }.resume()
}

downloadImage(from: url) { result in
    switch result {
    case .success(let image):
        print("이미지 다운로드 성공: \(image)")
    case .failure(let error):
        print("다운로드 실패: \(error.localizedDescription)")
    }
}
```
> 네트워크 요청이 끝난 후 이미지 결과 전달하는 경우 - Completion Handler

@escaping 클로저를 사용하며, 메서드 내부 호출시에 Reference Counting을 신경써야한다.
그렇지 않으면 [[순환 참조 (Retain Cycle)]]가 발생되기 때문이다. 따라서 weak self 캡처리스트를 사용한다.
또한, Swift의 동시성 환경에서는 클로저가 여러 스레드에서 동시에 실행될 수 있기 때문에, @Sendable 키워드를 사용해 클로저가 Thread-safe하도록 선언할 수 있다. @Sendable은 클로저 내에서 캡처된 값들이 스레드에 안전하게 작동할 수 있도록 제한을 둔다.

completion Handler를 사용하는 방식은 신경써야 하는 부분이 꽤나 많으며, 비동기 작업이 여러 단계로 중첩되는 경우에 콜백 함수들이 중첩되어서 코드의 가독성이 상당히 떨어지는 콜백 지옥이 발생할 수도 있다.

작업의 성공/실패 여부에 따라 각각 다른 콜백 함수로 처리하면서 에러 처리가 복잡해질 수도 있으며, 비동기 작업의 흐름이 콜백함수에 의해 이뤄지기 때문에 코드 자체가 실행 흐름을 파악하기에 비직관적이라는 단점이 있다.

콜백을 사용한 기본적인 방식이며, Swift 초기 버전부터 사용 가능하다는 장점이 있다.


#### 2. [[컴바인 (Combine)]]

iOS13 / macOS10.15 / watchOS6 이상 /  Swift 5부터 가능

비동기 작업의 시퀀스를 처리하기 위한 기능을 제공한다.
비동기 작업의 결과를 스트림으로 처리하고, 다양한 Operator를 사용해서 데이터 조작 및 변환이 용이하다.
Publisher & Subscriber를 기반으로 동작하기 때문에 데이터의 흐름을 '선언적인 방식으로 조작'할 수 있다.
Combine은 함수형 프로그래밍, 반응형 프로그래밍의 개념을 활용해 비동기 작업을 처리하는 방식이다.

```swift
import Combine

func downloadImagePublisher() -> AnyPublisher<String, Never> {
    return Future { promise in
        DispatchQueue.global().async {
            sleep(1)
            promise(.success("이미지 다운로드 완료"))
        }
    }
    .receive(on: DispatchQueue.main)
    .eraseToAnyPublisher()
}

var cancellable: AnyCancellable?

cancellable = downloadImagePublisher()
    .sink { value in
        print(value)
        print("이미지 표시")
    }

```
> 네트워크 요청이 끝난 후 이미지 결과 전달하는 경우 - Combine

굉장히 강력한 라이브러리이지만, 개념과 사용법이 복잡하다는 문제가 있다.
RxSwift와는 다르게 Apple 공식 프레임워크라는 점과, 선언적 방식으로 데이터 흐름을 관리할 수 있다는 장점을 가진다.


#### 3. Swift Concurrency (async/await)

Swift5.5 부터 가능

비동기 작업을 직관적이고 '동기적인 코드와 유사한 구조' 형태로 처리할 수 있다.
- async 키워드 : 비동기 함수를 선언
- await 키워드 : 비동기 작업의 완료 기다림

```swift
func downloadImage() async -> String {
    try? await Task.sleep(nanoseconds: 1_000_000_000)
    return "이미지 다운로드 완료"
}

Task {
    let result = await downloadImage()
    print(result)
    print("이미지 표시")
}

```
> 네트워크 요청이 끝난 후 이미지 결과 전달하는 경우 - Swift Concurrency

코드의 가독성이 뛰어나고, 콜백 지옥을 피할 수 있다.
추가적인 라이브러리나 프레임워크 사용이 필요 없다는 장점도 가진다.

간단하게 사용 가능한 좋은 방식이지만, Swift5.5 이후부터 사용하다는 점과 내부 동작 과정에서 추가적인 메모리 사용이 필요하다는 점, 여러 개의 비동기 작업을 동시에 실행하는 것이 까다로울 수 있다는 단점을 가진다.


>[!Result] 
위의 3가지는 사실 뭐가 더 확실히 좋으니깐 그것만 써요! 라고 말하기는 어렵다.
각 프로젝트에 맞게 선택해 사용하면 된다.




## Keywords
---
- 동기 / 비동기
- [[클로저 (Closure)]]
- [[순환 참조 (Retain Cycle)]]
- [[GCD (Grand Central Dispatch)]]
- [[컴바인 (Combine)]]
- Swift [[동시성 (Concurrency)]]


## References
---
- https://bbiguduk.gitbook.io/swift/language-guide-1/concurrency
- https://toughie-ios.tistory.com/290?category=1095613
- https://velog.io/@woojusm/Swift-Concurrency-Download-Images-w-Async-Await