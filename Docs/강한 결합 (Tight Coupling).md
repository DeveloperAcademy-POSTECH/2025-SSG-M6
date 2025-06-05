>[!question]
>GQ1. 강한 결합 (Tight Coupling)은 무엇이고, 이것에 대비되는 느슨한 결합 (Loose Coupling)은 무엇인지?
>GQ2. Swift에서 강한 결합 (Tight Coupling) 문제를 해결하기 위해서는 어떠한 해결책을 시도해볼 수 있을까?

## Description

- ***"한 모듈을 변경하면 다른 모듈에 연쇄적인 변경을 강요하는 상황"***
-  한 모듈의 수정이 다른 여러 모듈에게도 끼치는 **파급 효과 (Ripple Effect)** 가 커져 유지보수를 어렵게 만들 때, 이 모듈들은 서로 강한 결합 (Tight Coupling) 관계에 있다고 말한다.

>[!question] 느슨한 결합 (Loose Coupling) 개념 정확하게 잡고 가기
>컴포넌트 (객체, 모듈을 포함) 간의 ***의존성이 최소화된 상태*** 라고 정의할 수 있다.
>- 한 컴포넌트에서 발생한 수정사항이 다른 컴포넌트에 연쇄적으로 수정사항이 미치지 않는 것 (사실 완전 미치지 않는 것이 베스트지만, 현실적으로 쉽지 않기에 "최소화" 정도로 문구를 정함)을 의미
>- 다르게 말하면, 하나의 컴포넌트가 다른 컴포넌트의 내부 구조 (구체적인 구현)를 모른채 협업할 수 있는 관계를 의미
>- 상호 독립성이 보장되어 있기 때문에 테스트 시에도 Mock을 주입하기가 용이한 관계



## [[의존성 주입 (DI)]]과 [[프로토콜 (Protocol)]] 기반 설계로 해결하기

#### 1. 문제 상황 분석

+ [[싱글톤 패턴 (Singleton Pattern)]]을 사용할 때 발생하는 두 객체 간 강한 결합 사례
+ Service 클래스는 `Singleton.shared`를 직접 사용하며, Singleton 클래스에 강하게 의존하고 있다.

```Swift
class Singleton { 
    static let shared = Singleton()
	private init() {}
	func doSomething() {
		print("싱글톤에서 어떤 일을 했군요...")
	}
}

class Service {
	func useSingleton() {
		Singleton.shared.doSomething()  
	}
}
```

#### 2. 문제 해결 방향 설정 (프로토콜 정의)

- Service 클래스는 `doSomething()`이라는 역할이 필요해 Singleton 객체를 의존하고 있음.
  즉, 객체가 직접적으로 필요해서 의존하는 것이 아니라, doSomething() 이라는 역할을 필요로 해서 의존하는 것.
  -> 타입 (Class)이 아닌 역할 (Role) 을 기반으로 프로토콜을 정의하는 방식으로 수정해볼 수 있다.

```Swift
protocol DoSomethingProtocol {
    func doSomething()
}
```

#### 3. 싱글톤 역할 재정의

- 싱글톤은 DoSomethingProtocol이라는 추상화된 역할을 수행하는 "하나의 구현체"로 다시 재정의

```Swift
class Singleton: DoSomethingProtocol { 
    static let shared = Singleton()
	private init() {}
	func doSomething() {
		print("싱글톤에서 어떤 일을 했군요...")
	}
}
```

#### 4. Service에서는 의존성 주입 (DI)을 사용해 느슨한 결합 관계로 연결

- Service 클래스 생성자에 어떤 것을 의존 (DoSomethingProtocol)하고 있는지 명시적으로 보여주기 때문에, 의존 관계를 명확하게 개발자가 확인할 수 있다.
- Service 객체에 `Singleton.shared`가 아닌 다른 Mock도 주입할 수 있기에 유연성 또한 커진다.

```Swift
class Service {
    private let worker: DoSomethingProtocol

    init(worker: DoSomethingProtocol) {
        self.worker = worker
    }

    func useWorker() {
        worker.doSomething()
    }
}

let service = Service(worker: Singleton.shared)
service.useWorker()
```


## References

- [위키페디아 - Coupling (computer programming)](https://en.wikipedia.org/wiki/Coupling_(computer_programming)#:~:text=1,dependent%20modules%20must%20be%20included)]