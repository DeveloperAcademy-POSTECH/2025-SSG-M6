멀티스레딩(Multithreading) 또는 [[동시성 (Concurrency)]] 환경에서 발생하는 문제 중 하나이다. 두 개 이상의 쓰레드가 동시에 공유된 가변 데이터(Shared Mutable State)에 접근하면서, 적어도 하나의 쓰레드가 데이터를 변경할 때 발생하는 오류 상황을 말한다.

> ❓공유된 가변 데이터 (Shared Mutable State)
> 
> Shared State : 여러 스레드나 작업이 동일한 메모리 위치(변수, 객체의 프로퍼티 등)에 접근
> Mutable State : 접근하는 데이터가 변경 가능한 상태

---
### 발생 조건

아래 세 가지를 모두 만족할 때 발생한다.

1. 두 개 이상의 스레드가 동시 실행됨
2. 동일한 메모리 위치에 접근함
3. 최소 하나 이상의 스레드가 해당 메모리에 쓰기 작업 수행

```swift
class Counter {
    var value = 0
}

let counter = Counter()

DispatchQueue.global().async {
    counter.value += 1
}

DispatchQueue.global().async {
    counter.value += 1
}
```

위 코드에서는 여러 쓰레드가 동시에 value에 접근해 값을 증가시키므로, 결과를 보장하지 못한다. 결과가 2가 될 것이라고 기대하지만, 각 블록이 언제 어느 스레드에서 실행될지 확신할 수 없기 때문에, 결과는 1이 될 수도, 2가 될 수도 있다.

### 위험한 이유?

- 비결정적 결과 : 실행 시점이나 스레드 스케줄링 순서에 따라 결과가 달라질 수 있다
- 디버깅의 어려움 : 간헐적으로만 문제가 발생하거나 특정 조건에서만 발생하기에 디버깅이 어렵다
- 앱 동작 문제 : 앱이 충돌하거나, 잘못된 데이터를 저장/표시할 수 있음

---

### 방지 방법

#### - Serial Queue 사용

하나의 큐에서 순차적으로 작업을 수행하게 하는 방식이다. 모든 작업을 한 번에 하나씩만 처리하기 때문에, 공유 자원에 대한 경쟁을 방지해 안전하게 사용할 수 있다.

```swift
let queue = DispatchQueue(label: "com.example.serial")

queue.async {
    counter.value += 1
}
```

#### - DispatchQueue.sync 사용

읽기와 쓰기 모두 큐에 sync를 이용해 직렬화하는 방식으로, 큐 내에서 작업이 순차적으로 처리되기 때문에 데이터 경합을 방지한다.

- 장점
    - 사용방법이 간단하다
    - [[GCD (Grand Central Dispatch)]]는 애플에서 제공하는 저수준 동시성 API로, 성능이 우수하다
    - sync, async를 모두 지원해 유연한 사용이 가능하다
- 단점
    - 큐 접근이 많아질수록 성능과 가독성 저하의 우려가 있다
    - 동기 사용 시 데드락을 주의해야 한다

```swift
class SafeCounter {
    private let queue = DispatchQueue(label: "com.example.safe")
    private var _value = 0

    var value: Int {
        queue.sync { _value }
    }

    func increment() {
        queue.sync {
            _value += 1
        }
    }
}
```

#### - NSLock 사용

명시적으로 lock(), unlock()을 호출해 사용하며 임계 구역을 직접 보호하는 방식이다. 임계 구역은 공유 자원에 접근하는 코드 블록을 말한다.

- 장점
    - 가장 기본적이고 강력한 동기화 방식이다
    - 락을 특정 코드 블록에만 범위별로 지정해 유연한 사용이 가능하다
- 단점
    - 락 누락 시 데드락의 위험이 있다. unlock() 호출을 잊거나, 예외 발생 시 unlock()이 실행되지 않으면 앱 크래시 위험이 있다. → 이를 방지가하기 위해 defer 사용
    - 세밀한 락 관리를 위해 코드 로직이 복잡해질 수 있고, 그로인해 버그 가능성이 증가한다

```swift
class LockCounter {
    private var _value = 0
    private let lock = NSLock()

    func increment() {
        lock.lock() // 락 걸기 -> 다른 스레드 대기
        _value += 1
        lock.unlock()
    }

    var value: Int {
        lock.lock()
        defer { lock.unlock() } // 함수 종료 시 락이 반드시 해제되도록 defer 사용
        return _value
    }
}
```

#### - Actor 사용

Swift Concurrency의 [[Actor]]는 기본적으로 데이터 경쟁 방지를 보장한다.
actor 내부 상태는 직렬적으로 접근되기 때문에, 외부에서 동시에 접근하더라도 Data Race가 발생하지 않는다.

- 장점
    - 언어 레벨에서 데이터 결쟁을 방지해 실수로 발생하는 데이터 경합을 컴파일 시점에 잡아준다
    - 개발자가 직접 락이나 큐를 관리할 필요가 없으며 actor, await 키워드만으로 구조적으로 스레드 안전하게 사용 가능하다
- 단점
    - 오버헤드 발생 가능성이 있다

```swift
actor ActorCounter {
    var value = 0

    func increment() {
        value += 1
    }
}

let counter = ActorCounter()

Task {
    await counter.increment()  // 안전한 접근
}
```