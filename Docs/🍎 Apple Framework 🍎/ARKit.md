>[!question]
>GQ1. ARKit은 어떻게 실제 세계 공간을 인식할까?
>GQ2. ARKit은 인식한 실제 공간 위에 어떻게 가상 객체를 배치할까?
>GQ3. 두 기기간 AR 공간을 동기화하려면 어떤 기술이 필요할까?

해당 문서는 SceneKit 기반으로 작성되었습니다. (렌더링 부분 빼고 !)

![[ARKit.png]]
## ARKit이 뭔데 !

ARKit은 증강 현실(AR) Apple Framework로, 장치에 카메라, 자이로 센서를 활용해 실제 환경을 이해하고 가상 콘텐츠를 덧씌우는 AR 경험을 만들 수 있습니다.

내가 지금 어디 있는지, 무엇이 바닥이고 벽인지 등을 실시간으로 계산 !

- **월드 트래킹**
	- 기기의 위치와 방향을 추적해 고정된 AR 콘텐츠 표시
- **평면 감지**
	- 바닥, 책상 등 수평/수직 면을 인식하여 AR 오브젝트 배치 가능
- **LiDAR 스캐닝**
	- 정밀한 거리 측정 + 깊이 정보 인식 -> Pro 기기에서만 가능
- **모션 캡처**
	- 단일 카메라로 전신 추적, 귀 위치까지 인식
- **듀얼 카메라** 
	- 전 · 후면 카메라를 동시에 사용하여 양방향 트래킹 
- **Multiuser AR**
	- 여러 기기가 같은 AR 공간을 공유

위와 같은 기능을 통합하여 복잡한 AR 구현을 간단하게 만들어줍니다.



## 그럼 실제 세계 공간은 어떻게 인식하나요?

먼저 **VIO(Visual-Inertial Odometry)** 는 카메라와 IMU 센서를 동시에 활용하여 사용자의 움직임을 3D 공간상에서 정밀하게 추적합니다. 이를 통해 **위치와 방향이 실시간으로 파악**이 되어요.

![[Visual-Inertial Odometry.png]]

**카메라**는 외부 환경을 관찰하여 **특징점을 추적**하여 현실 세계의 지점들(모서리, 질감, 무늬 등)을 분석하고, 이 특징점들을 활용해 주변의 공간 구조를 구성합니다. **IMU 센서**는 가속도계 + 자이로스코프로 구성되어 있어 기기의 선형 가속도와 **회전 속도를 빠르게 측정**합니다.

ARKit 내부 알고리즘이 이 둘을 결합하여 현재 위치와 방향을 정밀하게 추정하여 Base 기준으로 표현됩니다.

추가적으로 깊이 인식과 공간 재구성 기술이 함께 작동되기도 하는데, **LiDAR 스캐너**가 탑재된 기기에서는 레이저가 표면에 반사되어 돌아오는 시간을 측정함으로써 실제 거리 정보를 계산하고, 이를 기반으로 현실 공간을 3D 메쉬 형태로 정밀하게 재구성합니다. -> 여기서 오클루전(가림 효과), 조명 추정 같은 고급 기능이 가능해짐 !

![[LiDAR Scanner.png]]

이러한 분석들을 기반으로 사용자가 실제 공간에서 의미 있는 위치를 인식했을 경우, 해당 지점을 **ARAnchor로 저장**할 수 있어요. 이렇게 고정된 앵커는 사용자가 움직이더라도 가상 객체가 마치 현실 세계에 머무는 것처럼 자연스럽게 보이도록 해줍니다.

#### 앵커의 종류

- **ARAnchor** : 가상 객체의 위치 기준점
- **ARPlaneAnchor** : 감지된 평면 위에 고정
- **ARImageAnchor** : 인식된 이미지를 기준으로 오브젝트 배치
- **ARFaceAnchor** : 얼굴의 위치와 표정을 추적
- **ARBodyAnchor** : 전신 추적용




## ARKit은 인식한 실제 공간 위에 어떻게 가상 객체를 배치하나요?

ARKit은 실제 세계의 공간을 인식한 뒤, 가상 객체가 현실과 일관되게 고정되어 보이도록 배치합니다.

#### ARSession : 각 기기에서 AR 세션을 실행하며 공간 추적

#### ARAnchor: 가상 객체의 위치 기준점

ARAnchor은 3D 공간 내 특정 위치와 방향을 가지는 고정점입니다. 
ARKit은 transform(4X4 행렬) 값을 통해 위치, 회전을 표현하며,  가상 객체는 이 앵커에 붙어 있게 됩니다.

```swift
let anchor = ARAnchor(transform: simdTransform)
sceneView.session.add(anchor: anchor)
```

#### Raycasting : 화면을 터치 -> 현실 좌표 변환

사용자가 화면을 터치하거나 특정 지점을 바라볼 때, 해당 지점이 현실에서 어디인지를 계산해야 합니다.
- RaycastQuery 또는 hitTest(_ : types: )를 통해 2D 화면 좌표를 3D 공간 좌표로 변환합니다.

```swift
let results = sceneView.hitTest(location, types: [.existingPlaneUsingExtent])
if let result = results.first {
    let anchor = ARAnchor(transform: result.worldTransform)
    sceneView.session.add(anchor: anchor)
}
```

#### 좌표계 : 현실과 가상 간 정렬 유지

월드 좌표계를 기준으로 공간을 표현하는데, 모든 가상 객체는 이 좌표계 안에서 위치를 유지합니다.
이 좌표계는 ARSession이 처음 시작될 때 기준이 정해지며, 세션이 유지되는 동안 상대적인 위치 추적이 계속됩니다.

#### 렌더링 엔진으로 가시화

SceneKit 또는 RealityKit을 통해 가상 객체를 시각적으로 표현합니다.
SceneKit에서는 SCNNode, RealityKit에서는 Entity를 사용하며, 둘 다 앵커와 연결됩니다.

```swift
let anchorEntity = AnchorEntity(world: position)
let model = ModelEntity(mesh: .generateBox(size: 0.1))
anchorEntity.addChild(model)
arView.scene.anchors.append(anchorEntity)
```

#### + 오클루전(가림 효과)

LiDAR 기반 기기는 깊이 인식으로 실제 물체 뒤에 가상 객체가 가려지는 오클루전 효과를 낼 수 있습니다.
```swift
arView.environment.sceneUnderstanding.options.insert(.occlusion)
```




## 두 기기간 AR을 동기화하려면 어떤 기술이 필요할까?

ARWorldMap : 공간 구조를 저장한 맵으로, 공유 가능 (MultipeerCunnectivity통해 수신)
MultipeerConnecttivity : ARWorldMap 및 앵커 데이터를 네트워크를 통해 전송하는 프레임워크

**간단한 전체 흐름**
```text
A의 AR 정보를 B에게 공유하는 과정
┌──────────────┐       ┌──────────────┐
│ Device A     │       │ Device B     │
├──────────────┤       ├──────────────┤
│ ARSession    │       │ ARSession    │
│ WorldMap     │──────►│ Receive Map  │
│ Add Anchor   │──────►│ Add Anchor   │
│ Update Obj   │──────►│ Sync Obj     │
└──────────────┘       └──────────────┘
```

1. **ARWorldMap 저장 및 전송** (이미 각 세션은 초기화된 상태 -> 지금은 서로 다른 곳을 가리킴)
```swift
arSession.getCurrentWorldMap { worldMap, error in
    guard let map = worldMap else { return }
    if let data = try? NSKeyedArchiver.archivedData(withRootObject: map, requiringSecureCoding: true) {
        multipeerSession.sendToAllPeers(data)
    }
}
```

2. **수신한 WorldMap으로 세션 재시작**
```swift
func receivedWorldMap(_ data: Data) {
    guard let worldMap = try? NSKeyedUnarchiver.unarchivedObject(ofClass: ARWorldMap.self, from: data) else { return }
    let configuration = ARWorldTrackingConfiguration()
    configuration.initialWorldMap = worldMap
    arSession.run(configuration, options: [.resetTracking, .removeExistingAnchors])
}
```

3. **Anchor 추가 및 동기화**
```swift
let anchor = ARAnchor(name: "sharedCube", transform: transform)
arSession.add(anchor: anchor)

// 전송
if let anchorData = try? NSKeyedArchiver.archivedData(withRootObject: anchor, requiringSecureCoding: true) {
    multipeerSession.sendToAllPeers(anchorData)
}
```

4. **수신한 Anchor 추가**
```swift
func receivedAnchor(_ data: Data) {
    guard let anchor = try? NSKeyedUnarchiver.unarchivedObject(ofClass: ARAnchor.self, from: data) else { return }
    arSession.add(anchor: anchor)
}
```

### MultipeerConnectivity의 단점

- 로컬 네트워크 기반으로, 멀리 떨어진 기기 간 동기화는 불가함
- 5명 이상의 동시 연결은 불안정할 수 있음
- 중간에 블루투스 간섭, 와이파이 단절로 세션이 끊길 수 있음
- 영상 스트리밍과 같이 실시간 대용량 전송에는 부적합


## References
- [WWDC22_ARKit6](https://developer.apple.com/kr/videos/play/wwdc2022/10126/?time=1147)
- [Visual-Inertial Odometry](https://www.mathworks.com/help/nav/ug/monocular-visual-inertial-odometry-using-factor-graph.html)