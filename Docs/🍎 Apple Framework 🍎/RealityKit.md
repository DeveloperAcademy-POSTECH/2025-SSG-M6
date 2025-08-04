>[!question]
>GQ1. SceneKit이 있는데 RealityKit이 왜 나왔을까?
>GQ2. RealityKit은 ECS 패턴을 왜 채택했을까?

## RealityKit이 뭔데 !

![[RealityKit.webp]]


> **RealityKit**은 사진처럼 생생한 렌더링과 카메라 효과, 애니메이션, 물리적인 요소 등을 갖추고 특별히 증강 현실을 위한 Apple Framework 입니다. [[ARKit]]과 서로 보완적인 역할을 하며 사용됩니다.

- **렌더링 엔진**
	- Metal 기반으로 동작하여 고성능 3D 그래픽 지원
	- 실시간 그림자, 반사, PBR(물리 기반 렌더링) 재질 지원
- **Physics Simulation**
	- 중력, 충돌 감지, 관성 등 현실 세계와 유사한 시뮬레이션 제공
- **애니메이션**
	- 엔티티 이동, 회전, 스케일 변화 애니메이션 지원
	- Keyframe 기반 애니메이션 지원 가능
- **ARKit 통합**
	- 평면 감지, 얼굴/손 추적, 이미지 객체 인식 등 ARKit의 센서 기반 기능 사용 가능
	- ARView는 RealityKit의 핵심 뷰이며, ARKit 세션과 자동으로 연동
- **공간 오디오**
	- 공간상의 위치에 따라 소리의 방향과 거리감 재현
	- 몰입감 있는 AR 경험 구현


Realitykit이랑 하는 거 비슷한 프레임워크가 하나 더 있어요 바로바로
#### [[SceneKit]] !

둘 다 3D 그래픽 렌더링 프레임워크인데요,
이미 있던 SceneKit 대신 RealityKit을 굳이 만든 이유가 궁금했어요.



## SceneKit이 있는데 RealityKit이 왜 나왔을까?

기존 SceneKit은 좋은 범용 3D 프레임워크지만, AR이라는 새로운 기술 흐름에 맞지 않는 한계들이 있었기 때문에 RealityKit을 만들었대요.

예를 들면,
ARKit과 분리된 구조, 무거운 객체 구조, 렌더링 품질 제한, 오래된 설계, 멀티 플랫폼 비효율 등 . .,
위와 같은 이유들로 AR과 Spatial Computing 환경에 최적화된 프레임워크가 필요했기 때문이에요.

SceneKit을 보완하여

- **ARKit 완전 통합**
	- ARView 하나면 카메라 제어 + 현실 공간 인식 + 가상 객체 렌더링 모두 해결 ^~^
- **Swift 친화적 설계**
	- 스위프트 구조체, 컴포넌트 기반 패턴을 반영
	- 선언적 코드 스타일로 스유랑 잘 맞음
- **PBR + 고품질 렌더링**
	- 광원, 그림자, 반사, 금속 재질 등 현실감 높은 표현 제공 (Metal 기반 최적화)
- **AR/VR/Spatial에 특화**
	- RealityComposer Pro와의 연동으로 no-code/low-code 환경 제공
- **컴포넌트 기반 구조**
	- Entity + Component 시스템 -> 뒤에 ECS 구조와 연관되므로 생략


## SceneKit vs RealityKit 비교

| 항목             | SceneKit                  | RealityKit                    |
| -------------- | ------------------------- | ----------------------------- |
| **구조**         | Scene Graph (노드 트리 구조)    | Entity-Component System (ECS) |
| **프로그래밍 스타일**  | OOP 기반, 명령형 스타일           | 선언형, Swift 친화적                |
| **SwiftUI 연동** | 제한적                       | 원활 (`ARView` 사용)              |
| **렌더링 엔진**     | OpenGL → Metal 전환 중       | Metal 기반 최신 엔진                |
| **물리 엔진**      | 있음 (충돌, 중력 등 수동 설정)       | 컴포넌트 기반 물리 시스템 내장             |
| **ARKit 통합**   | `ARSCNView` 사용 (직접 구성 필요) | `ARView` 하나로 완전 자동 통합         |
| **앵커 처리**      | ARAnchor 수동 생성 및 관리       | `AnchorEntity`로 자동 배치 및 추적    |
그렇다고 모두 대체하느냐 ? 그건 아니라서 선택적으로 사용해야 함 !



## Realitykit의 구조

### ECS 기반 구조

객체지향 방식과는 반대되는 상속보다 구성 원칙을 따르는 **데이터 지향적 아키텍처**
ECS에서 System은 **System 프로토콜을 채택한 타입**이고, 이 구조 덕분에 개발자가 직접 시스템을 만들고, 여러 엔티티의 상태를 제어하는 로직을 분리하여 구현할 수 있게 되었습니다.

그래서 앱을 효율적이고 유연하게 구성할 수 있는 최적의 아키텍처라서 RealityKit은 **ECS 구조**를 채택하고 있답니다.

![[RealityKit_ECS.png]]

E (Entity) - AR Scene의 모든 객체

C (Component) - 속성 부여 (모델, 충돌, 물리 등) 

S (System) - 동작 로직을 처리하는 시스템 (RealityKit 내부에 자동 내장됨)

SwiftUI랑 잘 맞는다고 해놓고 UIKit 예제 코드를 들고 왔네요 , , ㅋ . . 쨋든 ..

```swift
// System 프로토콜의 구조
@available(iOS 15.0, *)
public protocol System {
    init(scene: Scene)
    func update(context: SceneUpdateContext) // 매 프레임마다 RealityKit이 자동 호출하여 시스템 로직을 수행함
}
```


```swift
// Entity + Component
let modelEntity = ModelEntity(mesh: .generateSphere(radius: 0.25))

modelEntity.components[PhysicsBodyComponent.self] = .init()
modelEntity.components[PhysicsMotionComponent.self] = .init()

print(modelEntity.components)
```


```swift
// System
@available(iOS 15.0, *)
class ChangeMeshAndColorSystem: System {
    static let query = EntityQuery(where:
        .has(ModelComponent.self) && .has(PhysicsBodyComponent.self)
    ) // 저 둘을 만족하는 것이 대상

    required init(scene: Scene) {
        print("System 등록됨")
    }

    func update(context: SceneUpdateContext) {
        context.scene.performQuery(Self.query).forEach { entity in
            guard let entity = entity as? ModelEntity else { return }

            entity.model?.mesh = .generateSphere(radius: 0.1) // 해당하는 엔티티를 구로 바꾸기

            var material = PhysicallyBasedMaterial()
            material.baseColor.tint = .systemOrange // 해당하는 엔티티를 주황색으로 바꾸기
            entity.model?.materials = [material]
        }
    }
}
```


## References
[ECS 구조](https://medium.com/macoclock/realitykit-911-entity-component-system-ecs-bfe0520e0e8e)
[RealityKit](https://developer.apple.com/documentation/realitykit)