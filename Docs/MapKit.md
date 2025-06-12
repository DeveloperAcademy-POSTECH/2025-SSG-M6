>[!question]
>GQ1. **MapKit**은 사용자의 위치를 어떻게 가져올까?
>GQ2. 지도 중심 좌표를 이동시키는 방식은 frame 변경 vs region 설정 중 무엇이 더 안정적일까?
>GQ3. 사용자의 제스처 ( 줌 / 스크롤 / 탭 )는 어떻게 앱 내부 로직과 연결될까?

![[MapKit.png]]

## MapKit이 뭔데 !

Apple에서 제공하는 지도 프레임워크로, iOS, iPadOS, macOS, watchOS에서 지도를 표시하고 다양한 지도 관련 기능을 앱에 통합할 수 있도록 도와주는 도구입니다. 
이 프레임워크를 통해 지도를 표시하고, 위치 기반 정보를 시각화하거나, 사용자 상호작용을 처리할 수 있습니다.

#### MapKit의 기능

- 지도에 사용자 위치 표시
- 특정 위치에 핀(Marker, Annotation) 꽂기
- 경로 그리기 (길 찾기)
- 위성 지도, 3D 지도 등 다양한 스타일의 지도 사용
- Custom Overlay(원, 다각형, 경로)로 특정 지역 강조
- 탐색 기능

#### MapKit HIG 🔆

**MapKit** 전용 HIG 문서는 아니지만, Maps 섹션 + 위치 데이터 처리 섹션 + 컴포넌트 UI 가이드라인을 참고한 HIG입니다 !

**사용자 중심 탐색**
- 지도를 사용할 땐 사용자가 명확하게 위치를 인식하고, 주변 맥락(랜드마크, 거리, 교통 등)을 파악할 수 있도록 도와야 합니다.
- Look Around나 위치 기반 경로 안내 같은 탐색 도구는 가급적 상황에 맞게 제공되어야 합니다.
- 지도가 앱에 처음 나타날 때는 사용자의 **현재 위치나 앱의 주요 목적지**가 보이도록 설정합니다.

**마커 및 주석**
- 지도에 너무 많은 마커를 동시에 표시하면 시각적 과부하를 일으킬 수 있으므로, 중요도에 따라 필터링하거나 클러스터링이 권장됩니다.
- Callout(말풍선 스타일의 정보)는 직관적으로 누가, 무엇을 클릭했는지 즉시 보여주는 정보여야 하며, 지나치게 많은 내용을 담지 말아야 합니다.
- 마커나 오버레이는 명확하게 터치 대상임을 인식할 수 있도록 충분한 크기와 피드백을 제공해야 합니다.

**위치 권한 요청**
- 앱이 위치 정보를 사용하는 이유를 명확히 설명하는 권한 요청 메세지를 제공해야 합니다.
- 위치를 사용하는 기능은 사용자가 언제든지 끌 수 있어야 하며, 앱이 위치를 어떻게 활용하는지 명확히 보여줘야 합니다.


## MapKit은 사용자의 위치를 어떻게 가져올까?

MapKit은 사용자의 위치를 직접 가져오지 않고, 위치 정보를 얻기 위해 Core Location 프레임워크와 함께 사용됩니다. 
##### <font color="#228B22">사용자의 위치 = Core Location, 지도에 표시 = MapKit</font>

```swift
/// CLLocationManager를 사용해 위치 권한 요청 및 위치 수신 처리

import Foundation
import MapKit

class LocationManager: NSObject, ObservableObject, CLLocationManagerDelegate {
    private let manager = CLLocationManager()
    
    @Published var mapCameraPosition = MapCameraPosition.region(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 37.5665, longitude: 126.9780), // 서울 시청 -> 초기값
            span: MKCoordinateSpan(latitudeDelta: 0.01, longitudeDelta: 0.01)
        )
    )

    override init() {
        super.init()
        manager.delegate = self
        manager.desiredAccuracy = kCLLocationAccuracyBest
    }

    func requestPermission() {
        manager.requestWhenInUseAuthorization()
        manager.startUpdatingLocation()
    }
    
	/// 사용자 위치 받아오기
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        guard let location = locations.last else { return }
        DispatchQueue.main.async {
            self.mapCameraPosition = .region(
                MKCoordinateRegion(
                    center: location.coordinate,
                    span: MKCoordinateSpan(latitudeDelta: 0.01, longitudeDelta: 0.01)
                )
            )
        }
    }
}
```

서울 시청은 한국 앱 개발 시 기본 지도 중심지로 자주 사용합니다 !
미국 앱이라면 샌프란시스코 애플 본사 좌표를 쓰기도 합니다 ㅎ ㅎ

```swift
/// Map 뷰에서 .showsUserLocation(true)처럼 설정하여 사용자 위치를 지도에 표시

import SwiftUI
import MapKit
import CoreLocation

struct ContentView: View {
    @StateObject private var locationManager = LocationManager()
    
    var body: some View {
        Map(
            position: $locationManager.mapCameraPosition,
            showsUserLocation: true
        )
        .onAppear {
            locationManager.requestPermission()
        }
    }
}
```



## 지도 중심 좌표를 움직이는 방법은 뭘까?
#### frame 방식 

지도의 중심을 바꾸는 것이 아니라, 지도를 감싼 View의 레이아웃 위치를 바꾸는 방식입니다. 
Map 자체는 그대로고, SwiftUI 레이아웃에서 뷰의 위치나 크기를 조정합니다. 
-> 지도는 그대로 있지만 뷰의 위치만 바뀌는 시각 효과 느낌 .. 

```swift
Map(position: $position)
    .frame(width: 300, height: 300) // 지도 크기만 바뀜
    .offset(x: 50, y: 100) // 지도 시각적 위치만 이동
```

#### Region 방식

지도 중심 좌표와 확대 수준을 설정하는 대표적인 방식입니다.
MKCoordinateRegion 구조체를 사용해서 **지도 중심**과 **확대 범위**를 설정합니다.
-> frame과 반대로 어디를 중심으로, 얼마나 넓은 영역을 보여줄 것인가에 대한 수치적 정의임 !
##### MKCoordinateRegion 구조체

```swift
MKCoordinateRegion(
    center: CLLocationCoordinate2D, // 지구 상의 한 지점을 표현할 수 있음
    span: MKCoordinateSpan // 지도에서 표시할 지리적 범위를 지정하는 구조체
)
```

##### 그럼 둘 중에 뭐가 더 안정적일까 ? 

frame 방식은 단순히 SwiftUI에서 뷰의 외형을 조절하는 수단으로, 지도의 내부 상태에는 아무런 영향을 주지 않습니다. SwiftUI의 레이아웃 시스템과 독립적으로 동작하기 때문에, 지도 자체의 움직임과 분리된 시각적 효과만 유발하게 됩니다.

iOS 17부터 도입된 Map(position:) 방식에서는 **지도 상태(position)가 명확하게 앱의 상태와 연결되기 때문에, 외부에서 임의로 뷰를 움직이는 방식은 SwiftUI의 뷰 전환 애니메이션, GeometryReader 기반 레이아웃 연산 등과 충돌**을 일으킬 수 있습니다. 결과적으로 지도 뷰가 예기치 않게 겹치거나, 이동하는 듯 보이지만 실제 위치는 그대로인 등 UX가 깨지는 현상이 발생할 수 있ㅇㅡㅁ미다.

이와 달리 region 방식은 지도 중심(center)과 줌 수준(span)을 함께 정의하는 방식으로, **지도 내부의 논리 상태를 직접 조작하는 정식 인터페이스**입니다. 사용자가 지도 위에서 줌 또는 스크롤을 하더라도 region 값이 자동으로 반영되고, 앱에서 상태값을 갱신하면 지도도 그에 따라 움직이기 때문에 UI와 논리 상태 간의 일관성이 유지됩니다. @State 또는 @Binding을 통해 SwiftUI와도 자연스럽게 연결되며, 명확하고 안정적인 지리적 제어를 가능하게 합니다.

따라서 지도 중심 이동, 확대/축소, 위치 기반 피드백을 정확히 제공해야 하는 대부분의 앱에서는 frame이 아닌 region을 통해 지도 상태를 제어하는 것이 구조적으로 더 안전 ! 😷 해요.

<font color="#228B22">요약 Region 방식이 더 좋다. frame도 필요한 상황이 있답니다. 그게 언제냐먄 ..</font>


## 사용자의 제스처는 어떻게 앱 내부 로직과 연결될까? (SwiftUI)

지도 위에서 사용자가 수행하는 제스처 동작은 거의 3가지

- **줌** - 관심 영역을 더 자세히 보거나 넓게 보기
- **스크롤** - 다른 위치 탐색하기
- **탭** - 장소 선택, 상세 정보 보기

#### 줌 / 스크롤

```swift
struct ZoomScrollMapView: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 37.5665, longitude: 126.9780), // 서울
        span: MKCoordinateSpan(latitudeDelta: 0.01, longitudeDelta: 0.01)
    )

    var body: some View {
        Map(coordinateRegion: $region)
            .onChange(of: region) { newRegion in
                print("지도 이동됨  ! !  중심: \(newRegion.center)")
            }
    }
}
```

사용자가 지도를 움직이거나 확대 / 축소하면, 바인딩된 @State 값이 자동으로 갱신됩니다.
이 값을 바탕으로 앱은 지도 상태를 저장하거나, 외부 API 호출 등 로직을 트리거할 수 있습니다.
(요기 Region 방식이 반영된 것을 볼 수 있음)

제가 만들다가 발견했는데 .onChange는 **변화 감지가 가능한 Equatable을 채택한 타입**이어야만 작동한대요. 


##### Core Flow
> **사용자 제스처 발생** -> region 변경 -> .onChange 트리거 -> 앱 로직 실행

#### 마커 탭
```swift
struct Location: Identifiable, Hashable {
    let id = UUID()
    let name: String
    let coordinate: CLLocationCoordinate2D
}

struct MarkerSelectionMapView: View {
    @State private var selectedLocation: Location? = nil

    let locations = [
        Location(name: "카페", coordinate: .init(latitude: 37.5665, longitude: 126.9780)),
        Location(name: "서점", coordinate: .init(latitude: 37.5651, longitude: 126.9895))
    ]

    var body: some View {
        Map(selection: $selectedLocation) {
            ForEach(locations) { location in
                Marker(location.name, coordinate: location.coordinate)
                    .tag(location)
            }
        }
        .sheet(item: $selectedLocation) { location in
            VStack {
                Text("\(location.name) 정보 보기")
            }
        }
    }
}
```
 
 사용자가 마커를 탭하면 바인딩된 객체가 업데이트됩니다.
 이를 통해 선택된 장소 정보를 바탕으로 sheet를 열거나 상세 뷰를 띄우는 UI 연결이 가능해요.

##### Core Flow
> **사용자 마커 탭** -> selectedLocation 업데이트 -> sheet 또는 Navigation **트리거**

## References
- [Documentation](https://developer.apple.com/documentation/mapkit)
- [Maps HIG](https://developer.apple.com/design/human-interface-guidelines/maps)
- [Location Data HIG](https://developer.apple.com/design/human-interface-guidelines/privacy#Location-data)