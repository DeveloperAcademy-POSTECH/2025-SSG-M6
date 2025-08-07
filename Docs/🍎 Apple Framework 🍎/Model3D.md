
|항목|`Model3D`|`RealityView`|
|---|---|---|
|기반 엔진|✅ RealityKit|✅ RealityKit|
|사용 목적|간단한 3D 모델 로딩 및 표시|복잡한 3D 씬 구성, 상호작용, 물리 엔진 활용 가능|
|API 복잡도|매우 단순 (SwiftUI 스타일)|복잡 (Entity, Anchor, Component 구조 활용 필요)|
|상호작용 처리|없음 또는 매우 제한적|충돌, 제스처, Physics 등 풍부한 상호작용 지원|
|커스터마이징|제한적 (프레임, 회전 등 기본 UI 수준)|자유도 높음 (애니메이션, 움직임, 시뮬레이션 등)|
|성능|상대적으로 가벼움|무거울 수 있으나 최적화 가능|

### Model3D 란?

3D 모델을 View에 표시하기 위해 USD파일이나 Reality 파일을  비동기적으로 로드 하여 SwiftUI에서 사용하는 구조체 입니다. 


ProgressView를 먼저 보여주고 로드가 완료되면 3D Model를 표시함 
```swift
Model3D(named: "Gus") { model in 
	model
		.resizable() // 크기 조정
		.aspectRatio(contentMode: .fit) // 콘텐츠 크기와 맞게
} placeholder: { 
	ProgressView() 
}
```
Model3DPhase를 사용하면 로드 상태에 따라 알맞은 View를 표시 할 수 있습니다. 

```swift
Model3D(named: "Gus") { phase in 
	if let model = phase.model { 
		model // 모델 로드 
	} else if phase.error != nil { 
		Color.red // 에러 표시 
	} else { 
		Color.blue // placeholder 
	} 
}
```


### RealityView란?

RealityKit 엔진을 활용하여 3D 씬을 렌더링하고, 객체 간 상호작용 및 제스처 처리 등을 가능하게 하는 구조체 입니다.  


RealityView는 Model3D처럼 placeholder 코드를 작성하지 않아도 자동으로 placeholder 뷰를 표시함 만약 커스텀하려면 placeholder 파라미터를 사용하면 됩니다. 
```swift
struct ModelExample: View { 
	var body: some View { 
		RealityView { content in 
			if let robot = try? await ModelEntity(named: "robot") { 
				content.add(robot) 
			} 
			Task { 
			// 비동기 적으로 수행할 추가적인 작업 
			// 렌더링 후 콘텐츠 
			} 
		}
	} 
}
```
update 클로저를 사용하면 뷰의 상태가 변경되면 RealityKit 콘텐츠를 수정 할수 있습니다. 
subscribe를 사용하여 매 프레임마다 SceneEvents.Update로 이벤트를 받아 코드를 동작시킬 수 있습니다. 

```swift
RealityView { content in 
	let entity = ModelEntity(mesh: .generateSphere(radius: 0.1)) 
	content.add(entity) 
	_ = content.subscribe(to: SceneEvents.Update.self) { event in
		entity.position.y -= Float(event.deltaTime)
	} 
}
```
RealityView의 3D location은 meter가 아닌 SwiftUI의 local 좌표계 point로 나타나기 때문에  
새로운 라벨의 위치를 계산하기 위해서는 RealityView의 scene으로 값을 변환해줘야합니다.
targetedToEntity modifier로  SwiftUI local space를 scene의 coordinate space로 손 쉽게 만들 수 있습니다.

```swift
var body: some View {
	RealityView { content, attachments in 
	...
	} update { content, attachments in 
	...
	for place in GusPlaces {
		if let placeEntity = attachments.entity(for: place.id) {
			content.add(placeEntity)
			placeEntity.look(at: .zero, from: place.location, relativeTo: placeEntity.parent)
		}
	}
	} attachments: {
		ForEach(GusPlaces) { place in
			Text(place.name)
				.padding()
				.glassBackgroundEffect()
				.tag(place.id)
		}
	}.gesture(SpatialTapGesture()
		.targetedToEntity(earthEntity)
		.onEnded( value in
			let location = value.location3D
			let convertedLocation = value.convert(location, from: .local, to: .scene)
		)
	)
}
```

|축(Axis)|방향(Direction)|의미|
|---|---|---|
|+X|오른쪽(Right)|사용자를 기준으로 오른쪽 방향|
|+Y|위쪽(Up)|중력의 반대 방향|
|+Z|**앞쪽(Forward)**|사용자가 바라보는 방향|
|-Z|뒤쪽(Backward)|사용자의 뒤|

| 시스템                       | +Z 방향  | 설명            |
| ------------------------- | ------ | ------------- |
| **RealityKit (visionOS)** | **앞쪽** | 사용자가 보는 방향    |
| **ARKit (iOS)**           | **뒤쪽** | 카메라에서 멀어지는 방향 |

사용자 앞 1m에 배치하고 싶다면?
```swift
RealityView { content in
    // RealityKit content 추가
}
.attach(view: {
    Text("Hello VisionOS")
        .font(.title)
        .padding()
}, placement: .world(
    transform: Transform(
        scale: [0.001, 0.001, 0.001], // 1m 단위 → mm 크기로 축소
        rotation: simd_quatf(angle: 0, axis: [0, 1, 0]),
        translation: [0.0, 0.0, 1.0]  // 사용자 앞 1m
    )
))

```



아래는 y축 기준으로 좌우 30도 회전 가능한 동일한 기능을 model3D와 RealityView 코드 비교 예시 입니다.
```swift
// Model3D로 y축 기준으로 30도 회전
import SwiftUI
import RealityKit

struct GusModel3DView: View {
    var body: some View {
        Model3D(named: "Gus")
            .frame(width: 200, height: 200)
            .rotation3DEffect(.degrees(30), axis: .y)
            .scaleEffect(1.5)
    }
}
```

```swift
// RealityView로 y축 기준으로 30도 회전
import SwiftUI
import RealityKit
import RealityKitContent

struct GusModel3DView: View {
    var body: some View {
        RealityView { content in
            // 비동기적으로 모델 로드
            if let entity = try? await Entity(named: "Gus", in: realityKitContentBundle) {
                // 크기, 회전, 위치 조정
                entity.scale = [1.5, 1.5, 1.5]
                entity.transform.rotation = simd_quatf(angle: .pi / 6, axis: [0, 1, 0])
                
                // 앵커에 추가
                content.add(entity)
            }
        }
        .frame(width: 300, height: 300)
    }
}
```


model3D와 RealityView 를 컴포넌트 형태로 파일을 분리한 경우 loadedEntity가 업데이트가 되면 model3D는 자동으로 렌더링 되지만,  RealityView는 자동으로 렌더링 되지 않는 문제 발생했습니다.

```swift
// Model3D
import SwiftUI
import RealityKit
import RealityKitContent

private let targetSizeInMeters: Float = 0.26
private let modelScale: CGFloat = 0.7
private let modelOrientation: SIMD3<Double> = [0, 0, 0]

struct ToyPreview: View {
    
    var modelName: String = ""
    @State private var loadedEntity: Entity?
    
    var body: some View {
        Model3D(named: modelName) { model in
            model.resizable()
                .scaledToFit()
                .rotation3DEffect(
                    Rotation3D(
                        eulerAngles: .init(angles: modelOrientation, order: .xyz)
                    )
                )
                .scaleEffect(modelScale)
        } placeholder: {
            ProgressView()
        }
        .dragRotation(yawLimit: .degrees(360), pitchLimit: .degrees(360))
    }
}
```

RealityView는 update 클로저를 사용하여 기존 Entity를 삭제후 새로운 Entity로 갱신해줬습니다.
그리고  `@Binding`, `@ObservedObject`를 loadedEntity로 사용했을 때도 제대로 갱신이 되지 않거나 렌더링 되는 시점이 원하는 시점과 정확히 맞지 않았습니다. 

```swift
// RealityView
import SwiftUI
import RealityKit
import RealityKitContent

private let targetSizeInMeters: Float = 0.26

struct Toy: View {
 
    var modelName: String = ""
    @State private var loadedEntity: Entity?
    
    var body: some View {
    
        RealityView { content in
            if let entity = loadedEntity {
                content.add(entity)
            }
           
        } update: { content in
            content.entities.removeAll()
            if let entity = loadedEntity {
                           content.add(entity)
                       }
        
        } placeholder: {
            ProgressView()
        }
        .task(id: modelName) {
            await loadModel()
        }
        .gesture(
            RotateGesture3D()
                .targetedToAnyEntity()
                .useGestureComponent()
        )

    private func loadModel() async {
              if let entity = try? await Entity(named: modelName) {
                  
                  // 모델 원본 크기 측정
                  let originalBounds = entity.visualBounds(relativeTo: nil)
                  let originalSize = originalBounds.extents
                  
                  // 모델 크기 중 가장 큰 축을 기준으로 스케일 계산
                  let maxOriginalDimension = max(originalSize.x, originalSize.y, originalSize.z)
                  let scaleFactor = targetSizeInMeters / maxOriginalDimension
                  
                  entity.scale = [scaleFactor, scaleFactor, scaleFactor]
                  entity.position =  [0.21, -0.13, 0]
                  
                  enableGesturesRecursively(for: entity)
                  
                  await MainActor.run {
                      loadedEntity = entity
                  }
              }
      }
}
```

문제: 이번 C4에서 Model3D -> RealityView -> Model3D 다시 돌아온 이유는?


https://developer.apple.com/documentation/realitykit/model3d/
https://developer.apple.com/documentation/realitykit/realityview
https://velog.io/@0923alswl/Take-SwiftUI-to-the-next-dimension#:~:text=RealityView의%203D%20location은%20meter가%20아닌%20SwiftUI의%20local,위치를%20계산하기%20위해서는%20RealityView의%20scene으로%20값을%20변환해줘야한다.