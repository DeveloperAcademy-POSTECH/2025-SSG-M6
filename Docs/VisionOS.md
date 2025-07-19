### VisionOS 란?

핵심 프레임워크(UIKit, SwiftUI, ARKit 및 RealityKit 포함)와 포비티드 렌더링 및 실시간 상호 작용을 위한 MR 관련 프레임워크에서 파생된 혼합현실 운영체제 입니다. 

SwiftUi + RealityKit + ARkit



### 주요기능
- 모든 방향에서 평면을 감지합니다. 
- 주변 표면에 객체를 고정할 수 있습니다. 
- Room Anchor를 사용하면 사용자의 주변 환경을 실내 단위별로 고려할 수 있습니다. 
- Object Tracking API를 이용해 사용자 주변의 개별 객체에 콘텐츠를 첨부할 수 있습니다.
- 기업용 엔터프라이즈 API에서 공간 바코드 스캔, Apple Neural Engine,  물체 추적 매개변수를 제공합니다. 
- TabletopKit 사용하여 테이블 게임 같은 공동경험을 제작할 수 있습니다.

### Interaction

![[visionOS_Interaction.png]]

Hover Effect : 사용자가 element를 응시할 때 시각적인 피드백을 제공하여 상호작용을 개선하는 데 사용

Extension Gesture : 사용자는 머리를 움직여 기기 밝기나 볼륨을 조절하는 등 더욱 다양한 동작을 설정하여 사용할 수 있습니다. 

Custom Hand Gesture :  개발자는 사용자가 자신에게 최적화된 방식으로 기기를 제어할 수 있도록 커스텀 제스처를 설정할 수 있습니다.

```swift
struct GusView: View {

@State var dragTranslation: Vector3D = .zero

	var body: some View {
		RealityView {...}
		.gesture(
		 DragGesture()
		 .targetedToEntity(gus)
		 .onChanged { event in // 여러번 호출 이전 호출 차이만큼 이동
			 let transDiff = event.translation3D - dragTranslation
			 dragTranslation = event.translation3D
		 }
		 .onEnded {_ in // 끝나면 원래대로
			 dragTranslation = .zero
		 }
		)
	}
}


```

### Spatial Computing

![[Spatial Computing.png]]

디지털을 넘어서 오브젝트들이 마치 실제 공간에 있는 것처럼 화면이 크기에 제약 없이 배치 및 사용이 가능합니다. 

### Scene Types

- Windows 

![[VisionOS_Windows.png]]
각 앱에 적어도 하나는 열려 있어야 합니다. Resize 가능하며 2D 및 3D Content를 함께 표현이 가능합니다.

```swift

// App에 Window 추가 하기 
import SwiftUI

@main
struct GusApp: App {
	var body: some Scene {
		WindowGroup {
			HomeView()
		}
		.windowStyle(.plain)
		.defaultSize(width: 300, height: 200, depth: 1, in: .meters)
	}
}

```

- Volumes

![[visionOS Volumes.png]]

3D 컨텐츠를 정해진 바운드 내에서 보여줄 수 있습니다. 다양한 각도에서 컨텐츠를 보여줄 수 있습니다.

```swift

// App에 Volume 추가 하기 
import SwiftUI

@main
struct GusApp: App {
	var body: some Scene {
		WindowGroup {
			HomeView()
		}
		.windowStyle(.volumetric)
		.defaultSize(width: 300, height: 200, depth: 1, in: .meters)
	}
}

```

```swift

// 두 번째 Scene 띄우기 
import SwiftUI

@main
struct GusApp: App {
	var body: some Scene {
		WindowGroup {
			HomeView()
		}
		.windowStyle(.plain)
		
		WindowGroup {
			SecondView()
		}
		.windowStyle(.volumetric)
	}
}

```

- Spaces

![[visionOS_Spaces.png]]

앱에 몰입감을 더하는 새로운 방법으로 Shared Space는 여러앱이 한 공간에 배치가 가능하면, Full Space를 열면 다른 앱들이 숨겨지고 자신의 앱만 보입니다. 전체 공간을 활용하여 창, 볼륨, 컨테츠를 원하는 위치에 배치 가능 하며 ARKit을 이용해 더 많은 데이터와 기능을 활용 할 수 있습니다. 

![[visionOS_Immersion.png]]

- mixed : 기본값, passthrough 위에 컨텐츠를 보여줌 
- progressive :  mixed와 full의 혼합, 초기에 passthrough를 허용, 디지털 크라운으로 Level조정
- full : passthrough를 숨김, App의 Contents만 보여줌


```swift
// Space 추가하기
@main
struct GusApp: App {

	@State private var immersionStyle: ImmersionStyle = .full

	var body: some Scene {
		WindowGroup {
			HomeView()
		}
		.windowStyle(.plain)

		ImmersiveSpace(id: "gusSpace") {
			GusView()
		}
		.immersionStyle(selection: $immersionStyle, in: .full)
	}
}

// Space 열기
struct HomeView: View {

	@Environment(\.openWindow) private var openWindow

	var body: some View {
		VStack{

			Button {
				openWindow(id:"gusSpace") // id로 구분
			} label : {
				Text("Gus님 강림 하신다!")
			}
		}
	}
}
```

### RealityKit

![[ECS.png]]

iOS, iPadOS에서 사용하는 코드가 조금 다릅니다. 

```swift
// RealityView 기본 구조

struct GusView: View {
	var body: some View {
		RealityView { content, attachments in
		
		} update: { content, attachments in
		
		} attachments: { 
		
		}
	}
}
```

```swift
// 3D Model 로드 하기 

struct GusView: View {
	var body: some View {
		RealityView { content in

			let gus = try! await Entity(named: "gus_space", in: realityKitContentBundle)

			gus.scale = SIMD3(repeating: 0.1)
			content.add(gus)
		}
	}
}
```

```swift
// Animation 재생하기

struct GusView: View {
	var body: some View {
		RealityView { content in

			let gus = try! await Entity(named: "gus_space", in: realityKitContentBundle)

			gus.scale = SIMD3(repeating: 0.1)
			let animation = gus.availableAnimations[0]
			gus.playAnimation(animation)
			content.add(gus)
		}
	}
}
```

```swift
// Attachment - SwiftUI View를 Entity

struct GusView: View {
	var body: some View {
		RealityView { content, attachment in

			let gus = try! await Entity(named: "gus_space", in: realityKitContentBundle)

			gus.scale = SIMD3(repeating: 0.1)
			let animation = gus.availableAnimations[0]
			gus.playAnimation(animation)
			content.add(gus)
			
		if let gusGuide = attachment.entity(for: "gusID") { // id로 접근 가능
				gusGuide.position = [0, 0.1, 0]
				content.add(gusGuide)
			}
		} attachments: { // 사용하게 되면 파라미터를 추가해줘야 함
			Attachment(id: "gusID") {
				Text("눌러")
				.font(.system(size: 40))
				.blinking()
			}
		}
	}
}
```
```swift
// Swift -> RealityKit 값을 전달 -> angle 값이 변하면 update 클로저 호출 Entity를 회전
struct GusView: View {

let angle: Float
let gus: Entity

	var body: some View {
		RealityView { content, attachment in

			let gus = try! await Entity(named: "gus_space", in: realityKitContentBundle)

			gus.scale = SIMD3(repeating: 0.1)
			let animation = gus.availableAnimations[0]
			gus.playAnimation(animation)
			content.add(gus)
			
		if let gusGuide = attachment.entity(for: "gusID") { // id로 접근 가능
				gusGuide.position = [0, 0.1, 0]
				content.add(gusGuide)
			}
		} update: {	content, attachment in
			gus.orientation = simd_quatf(angle: angle, axis: SIMD3(x: 0, y: 1, z: 0))
		} attachments: { // 사용하게 되면 파라미터를 추가해줘야 함
			Attachment(id: "gusID") {
				Text("눌러")
				.font(.system(size: 40))
				.blinking()
			}
		}
	}
}

```

### 어떻게 SwiftUi 좌표를 RealityKit 좌표로 변환할까 ?

![[coordinate-conventions.png]]

**알려줄까 말까? 반응보고 결정~**

### 제한사항

개인정보보호로 카메라의 프레임 데이터에 접근하지 못하기 때문에 실시간 위치 추적에 불가능 합니다. 원래는 카메라로 촬영한 이미지를 기반으로 공간 맵 정보와 비교하여 현재 위치를 파악해야 하지만, 이러한 방식이 불가능합니다.