>[!question]
>GQ1. Mediator Pattern이란 뭐고? 어떤 상황에서 사용할 수 있는 디자인 패턴이지?
>GQ2. 실제 코드에서는 어떻게 활용할 수 있는 디자인 패턴일까?

## Description

- 중재자 패턴 (Mediator Pattern)이란 **"중재자 (Mediator)"라는 객체를 통해서만 객체 간 상호작용을 가능하게 함으로써** / 객체 간 직접적인 연관성을 낮추기 위해 활용할 수 있는 디자인 패턴입니다.
- 여러 객체 간 상호작용은 Mediator를 통해서만 이루어지록 구성되므로,
  객체 간 **상호 의존성은 줄어들고** / 상호작용은 **중앙에 집중**되고 / 통신 과정은 **독립적으로 동작**할 수 있게 이루어집니다.


## 주요 파일 구조

+ `Mediator`
  : `Colleague` 객체간 통신 규칙 (규약, 방법)을 정의하는 곳이며, 프로토콜 (인터페이스)로 선언합니다.
  
+ `Concrete Mediator`
  : `Mediator`프로토콜에서 정의했던 객체간 통신 규칙의 구현부를 정의하는 곳입니다.
  구체적으로는 `Colleague` 객체간 인터랙션을 조율해주는 역할이죠.
  
+ `Colleague`
  : `Mediator`에 의존하고 있으며, 상호간은 서로 알지 못하는 상태입니다.
  `Colleague`간 상호작용을 수행하고자 할 때는 반드시 `Mediator`를 통해서만 이루어져야 합니다.

![[Mediator Pattern.png]]


## 1. 예시 Code

- 여러 대의 비행기(객체)가 공항에 착륙하거나 이륙하려고 함
- 비행기들이 **서로 직접 통신하면서** 이착륙을 조율한다면? -> “나 먼저 갈게!”, “안 돼, 나 연료 없어!” → 혼란 발생
- 그래서 **공항에는 관제탑**이 중간에 있어 모든 비행기를 **조율**할 수 있는 역할을 수행하는 것.

> 직접 서로 통신하지 않고, 관제탑을 통해서만 의사소통을 주고받는 **"비행기"** -> **"Colleague"** 가 해당
> 비행기들의 요청을 받아 이착률을 조율하는 **"공항 관제탑"** -> **"Mediator"** 가 해당


```Swift
// 공항 관제탑
protocol AirTrafficControl {
    func requestTakeOff(from airplane: Airplane) async
}
```

```Swift
// 공항 관제탑 구현부
final class ControlTower: AirTrafficControl {
    private var queue = [Airplane]()
    private var isRunwayAvailable = true

    func requestTakeOff(from airplane: Airplane) async {
        if isRunwayAvailable {
	        // 착륙 Action
            isRunwayAvailable = false
            print("🛫 \(airplane.name) is taking off!")
			try? await Task.sleep(for: for: .seconds(duration))
			
			self.isRunwayAvailable = true
            print("✅ Runway is now available.")
			await processNextInQueue()
        } else {
            print("🕓 Runway busy. \(airplane.name) added to queue.")
            queue.append(airplane)
        }
    }   

    private func processNextInQueue() {
        guard !queue.isEmpty else { return }
        let next = queue.removeFirst()
        await requestTakeOff(from: next)
    }
}
```

```Swift
final class Airplane {
    let name: String
    let controlTower: AirTrafficControl

    init(
	    name: String, 
	    controlTower: AirTrafficControl
	) {
        self.name = name
        self.controlTower = controlTower
    }

    func requestTakeOff() {
        print("📡 \(name) requesting takeoff.")
        Task {
            await controlTower.requestTakeOff(from: self)
        }
    }
}
```


## 2. 활용 Code

+ Mediator에 Colleague 간에 상호작용을 담당할 메서드 정의

```Swift
protocol CameraMediator {
    func notify(sender: CameraModelColleague, event: CameraEvent)
}

protocol CameraModelColleague {
    var mediator: CameraMediator? { get set }
}

enum CameraEvent {
    case didCaptureImage    // 이미지 캡처 이벤트
    case didAnalyzeText     // 텍스트 분석 수행 이벤트
}
```

- Concrete Mediator에서는 Mediator에서 정의했던 메서드의 구현부를 정의
- 하위 Colleague들을 의존하고 있으며, 상호작용은 CameraEvent라는 열거형으로 정의해 메서드 내 분기처리해서 구분하고 있음.

```Swift
final class SemanticCameraMediator: CameraMediator {
    var cameraModel: CameraModel
    var visionModel: VisionModel

    init(
	    cameraModel: CameraModel, 
	    visionModel: VisionModel
	) {
        self.cameraModel = cameraModel
        self.visionModel = visionModel
        self.cameraModel.mediator = self
        self.visionModel.mediator = self
    }

    func notify(sender: CameraModelColleague, event: CameraEvent) {
        switch event {
        case .didCaptureImage:
            visionModel.analyzeImage(cameraModel.capturedImage)
        case .didAnalyzeText:
			print("recognized Text: \(visionModel.recognizedTextObservations.map { $0.transcript })")
        }
    }
}
```

- 하위 Colleague에서는 mediator에 약한 참조로 의존하도록해 순환 참조를 방지합니다.
- 비즈니스 로직 처리에는 mediator에 notify를 호출해 넘겨주도록 합니다.

```Swift
final class CameraModel: CameraModelColleague {
    weak var mediator: CameraMediator?

    var capturedImage: UIImage?

    func capture() {
        // 이미지 캡처 로직 어쩌구 저쩌구
        capturedImage = UIImage(named: "sample")
        mediator?.notify(sender: self, event: .didCaptureImage)
    }
}

final class VisionModel: CameraModelColleague {
    weak var mediator: CameraMediator?
    
    var detectedText: String = ""

    func analyzeImage(_ image: UIImage?) {
        // 분석 로직 어쩌구 저쩌구
        detectedText = "Hello"
        mediator?.notify(sender: self, event: .didAnalyzeText)
    }
}
```


## References

- [Mediator in Swift / Design Patterns](https://refactoring.guru/design-patterns/mediator/swift/example)
- [The Mediator Pattern in Swift: A Comprehensive Guide](https://swiftlynomad.medium.com/the-mediator-pattern-in-swift-a-comprehensive-guide-5c88eb105d97)