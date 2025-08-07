>[!question]
>GQ1. 싱글톤 패턴 (Singleton Pattern)이 필요한 상황을 명확하게 정의할 수 있을까? 
>GQ2. 싱글톤 (Singleton) 객체를 남용해서 사용했을 때 발생할 수 있는 문제는 무엇이 있을까?
>GQ3. 싱글톤 (Singleton) 객체가 멀티스레드 환경에서 안정성 (Thread Safety)을 보장할 수 있도록 하는 방법은 무엇일까?

## Description

- 싱글톤 패턴 (Singleton Pattern)이란 앱 전체에서 전역적 (global)으로 접근 가능한 **"공유 클래스 인스턴스 (shared instance of a class)"** 를 만드는 디자인 패턴이다.


## 싱글톤 코드 예시

+ `static` 키워드를 붙인 "타입 프로퍼티 (Type Property)"를 사용해 싱글톤 객체를 선언한다.
+ 외부에서 Singleton 클래스 타입의 객체 생성을 불가하도록 만들기 위해 생성자 `init`의 접근 수준을 `private`로 지정해야한다.

```Swift
class Singleton { 
    static let shared = Singleton()
	private init() {}
	/*
	초기화 외에 추가적인 설정을 수행해야 하는 경우 (클로저 호출 결과를 전역 상수에 할당)
	static let shared: Singleton = { 
		let instance = Singleton() 
		// setup code 
		return instance 
	}()
	*/
}
```


- 사용할 때는 `shared`라는 위에서 선언한 타입 프로퍼티로 인스턴스에 접근하기만 하면 된다.

```Swift
Singleton.shared.어쩌구메서드()
```

>[!question] 싱글톤 객체의 이름은 항상 shared여야 하는가?
>아니다. 그냥 관습적으로 쓰는 이름이라, 다른 이름을 사용해도 상관은 없다고 한다.
>
>단 Apple이 공식 프레임워크에서 싱글톤을 사용하는 경우를 생각해보면,
>- UserDefaults.standard (대표 설정이나 규칙을 의미)
>- UIScreen.main (메인을 의미)
>- FileManager.default (기본값 설정)
>- NotificationCenter.default  (기본값 설정)
>- UIApplication.shared (가장 일반적. 전역적으로 사용하는 싱글톤)
>- URLSession.shared (가장 일반적. 전역적으로 사용하는 싱글톤) 등으로 매우 다양하다.
>  
>  즉, 그런 점에서 볼 때 꼭 `shared`키워드를 사용해 싱글톤 객체를 만들 필요는 없다고 정의할 수 있겠다.
>  각 키워드는 Apple 프레임워크에서 정의된 싱글톤 이름을 참고해 만들면 좋겠쥬?



## 싱글톤 사용시 장단점

### 👍🏻 장점

+ 하나의 인스턴스로만 사용하기에, 불필요한 리소스 (메모리 낭비)를 유발하지 않을 수 있다.
+ **여러 자원에서 같은 객체의 사용이 필요한 공유 자원 (앱 전역에서 접근가능해야 하는 상태, 앱 설정 등)의 경우**, 싱글톤 객체의 사용을 활용하면 용이하게 만들 수 있다. -> 상태의 일관성을 유지하기가 용이하다.
+ 인스턴스가 최초 접근 시 (지연 초기화되어) 한 번만 생성되기에 불필요한 초기화를 피할 수 있다.

>[!question] 싱글톤 객체의 지연 초기화에 대하여 깊게 파보기
>[[지연 초기화 (lazy initialization)]]란 "프로퍼티에 최초로 접근하는 비로소 값을 생성하는 방식"이다. 
>즉, 프로그램 시작 시점이 아니라 / 필요해지면, 비로소 그제서야 만드는 (메모리에 올리는) 개념!
>Swift에서는 [[lazy]]라는 키워드 혹은 [[static let]] 키워드를 붙인 싱글톤 객체가 "지연 초기화" 방식을 채택한다. 
>
>그렇다면 왜?
>
>- 메모리 절약 되니까! 아직 사용하기 전인데 메모리 공간을 차지할 필요는 없으니까!
>  
>  사실 이것보다 더 중요한 것은...
>- [[스레드 안전 (Thread Safety)]]하게 작동할 수 있도록 돕기 때문에!
>  
>	- `static let`이 붙어있으면 자동으로 LLVM이 자동으로 ***Thread-safe*** Lazy Initialization을 적용해주도록 구현되어 있기 때문이다!
> 	- 위와 같은 상황을 원자적 초기화 (Atomic Initialization)라고 부르는데, 내부적으로 `dispatch_once`와 유사한 방식을 사용한다!
> 	- 여기서 `dispatch_once`란 Swift 초기에서 "해당 코드를 앱 전체에서 한 번만 실행하도록 보장하도록, 스레드 안전의 목적"으로 수동으로 호출하던 메서드를 의미.


### 👎🏻 단점

+ 멀티스레드 환경에서 **동기화 이슈**가 발생할 수 있다. 
  -> 올바르게 구현하지 않으면 동시에 여러 스레드에서 인스턴스를 생성할 위험이 있으며, 스레드 안전을 위해 추가 구현이 필요하다.
  -> 이 추가 구현이 위에서 봤던 [[static let]] 키워드를 사용한 [[지연 초기화 (lazy initialization)]] 전략으로 만들어진 것!

+ 전역 상태 (Global State)라는 특성. 싱글톤을 사용하는 클래스 사이에  **암묵적 의존성 (의존성 은닉)** 이 생김. + **모듈 간 결합도**가 높아지는 [[강한 결합 (Tight Coupling)]] 문제를 야기할 수 있다. 
  -> 즉, 코드 가독성·유지보수가 어려워지며, 프로그램의 상태를 예측하기 어려워진다.

>[!question] 암묵적 의존성 (의존성 은닉) 이해하기
>싱글톤을 사용하면 클래스 내부에서 싱글톤 인스턴스를 **직접 참조**하기 때문에, 해당 클래스가 어떤 의존성을 갖는지 외부에서 명시적으로 드러나지 않게 된다.
>이 상황을 ***암묵적 의존성 (의존성 은닉)*** 이라고 부르는 것!
>
>어떤 클래스가 `MyService.shared`를 사용해도 이것이 생성자나 메서드 상에서 `MyService`에 의존하고 있다는 사실을 알 수 없다는 의미!
  
+ 싱글톤 인스턴스는 애플리케이션 종료 시까지 메모리에 상주하므로, 잘못 사용하면 **메모리 누수**로 이어질 수 있다.

- 전역 상태를 가진 코드는 단위 테스트가 어렵다. 
  테스트 간 전역 상태가 남아 있으면 (오염되면) 테스트 결과가 달라지거나 순서에 의존하게 되기 때문이다. 



## References

- [Apple Developer Documentation - Managing a Shared Resource Using a Singleton](https://developer.apple.com/documentation/swift/managing-a-shared-resource-using-a-singleton)
- [Refactoring.Guru - Singleton](https://refactoring.guru/design-patterns/singleton#:~:text=1,a%20database%20or%20a%20file)
- [Apple iOS — Singleton Design Pattern](https://imad-ali.medium.com/apple-ios-singleton-design-pattern-2dec5b33f7)
- [Hidden Dependency in Single Design Pattern](https://medium.com/@amitaswal87/singleton-class-hidden-dependency-7bfbd82c85b8)
- [Demystifying Singleton Design Pattern: Implement Singletons in iOS](https://medium.com/@afonso.script.sol/demystifying-singleton-design-pattern-implement-singletons-in-ios-e1c38929ed4c)