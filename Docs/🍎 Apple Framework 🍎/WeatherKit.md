
![[WeatherKit.jpg]]

##  Description
---
WeatherKit은 Apple에서 제공하는 **날씨 프레임워크**입니다.  
iPhone의 기본 날씨 앱이 사용하는 것과 동일한 고품질 날씨 데이터를, 우리가 만든 앱에서도 손쉽게 사용할 수 있게 해줍니다. 예를 들어, 오늘의 기온이나 자외선 지수, 일주일 간의 날씨 예보 같은 정보를 코들 몇 줄로 불러올 수 있습니다.

Swift에서 간단한 코드로 날씨를 받아올 수 있고, REST API를 지원해서 **애플 기기가 아닌 다른 디바이스**에서도 사용할 수 있습니다.

하지만, 실제 데이터를 가져오려면 애플 개발자로 등록되어있어야하며, [Apple Developer 사이트에서 App ID에 WeatherKit를 활성화](https://developer.apple.com/kr/help/account/services/weatherkit/)하고 [Xcode에서도 해당 권한(Capability)](https://developer.apple.com/documentation/xcode/adding-capabilities-to-your-app)를 추가해야만 정상 작동합니다. 

또한 위치 기반 날씨 데이터를 받아오려면, [Info.plist에 특정 키를 추가](https://developer.apple.com/documentation/bundleresources/information-property-list/nslocationwheninuseusagedescription?utm_source=chatgpt.com)해줘야합니다.

## 주요 기능
---
#### 어떤 데이터를 넘겨주는가?

* 실시간 날씨 정보
	* 기온, 체감온도, 날씨 상태, 풍속, 습도, 자외선 지수 등
* 시간별 / 일별 예보
	* 최대 10일치 예보까지 가져올 수 있음
* 자외선
	* 현재 자외선 지수
	* 오늘 최대 자외선 지수 예측
* 대기질 및 기상 경보
	* 미세먼지, 오존, 이산화탄소 등 대기질 재표
	* 지역별 특보나 주의보 정보 제공


많이 사용할 것들 몇 개만 써놨습니다. *~~웬만한 건 다 있습니다~~* 
자세한 내용이 궁금하다면 [여기](https://developer.apple.com/documentation/weatherkitrestapi/dataset)를 확인하세요 :)

## 코드 예시
---

WeatherKit은 전 세계 어느 위치든 **위도와 경도 좌표**만 있으면 날씨 정보를 가져올 수 있습니다.
예를 들어 서울, 뉴욕, 도쿄, 제주도 같은 지역의 위도·경도를 활용하면, 특정 지역 날씨를 앱에 미리 표시할 수 있습니다.

```Swift
let newYork = CLLocation(latitude: 40.7128, longitude: -74.0060)
let weather = try await weatherService.weather(for: newYork)
```


여기서부터는 일반적으로 사용할, **사용자의 현재 위치 기반 날씨 정보를 불러오는 코드**를 살펴보겠습니다.

1. 현재 위치 정보 가져오기 (여기서 다룰 내용 아니지만 돌려보실까봐...)

```Swift
import Foundation
import CoreLocation

class LocationManager: NSObject, ObservableObject, CLLocationManagerDelegate {
    var locationManager = CLLocationManager()
    @Published var authorisationStatus: CLAuthorizationStatus?
    var latitude: Double {
        locationManager.location?.coordinate.latitude ?? 36.00568611
    }
    
    var longitude: Double {
        locationManager.location?.coordinate.longitude ?? 129.3616667
    }

    override init() {
        super.init()
        locationManager.delegate = self
    }

    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        let status = manager.authorizationStatus
        authorisationStatus = status
        if status == .authorizedWhenInUse || status == .authorizedAlways {
            manager.requestLocation()
        }
    }

    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        print("위치 업데이트 됨: \(locations.first?.coordinate.latitude ?? 0), \(locations.first?.coordinate.longitude ?? 0)")
    }

    func locationManager(_ manager: CLLocationManager, didFailWithError error: any Error) {
        print("위치 오류: \(error.localizedDescription)")
    }
}
```


2. WeatherKit으로 날씨 정보 호출

```Swift
import Foundation
import WeatherKit

@MainActor
class WeatherKitManager: ObservableObject {
    @Published var weather: Weather?
    func getWeather(latitude: Double, longitude: Double) async {
        do { // 백그라운드에서 비동기적으로 작업 실행
            weather = try await Task.detached(priority: .userInitiated) {
	            // 위도, 경도 기반 현재 날씨 데이터 가져오는 API 호출
                return try await WeatherService.shared.weather( 
	                for: .init(
		                latitude: latitude, 
		                longitude: longitude
					)
				)
            }.value
        } catch {
            fatalError("\(error)")
        }
    }
    
    var symbol: String { // 날씨 상태에 따라 적절한 SF Symbol 이름 반환 (ex. sun, cloud)
        weather?.currentWeather.symbolName ?? "xmark"
    }

    var temp: String { // 섭씨 단위로 변환해서 문자열로 반환
        let temp = weather?.currentWeather.temperature
        let convertTemp = temp?.converted(to: .celsius).description
        return convertTemp ?? "Connecting to Apple Weather Servers"
    }

    var uvIndex: String { // 소수점 첫째 자리까지 문자열로 반환
        let uvIndex = weather?.currentWeather.uvIndex.value
        return String(format: "%.1f", uvIndex ?? 0.0)
    }
}
```

>[!info] Task.detached(priority: .userInitiated)
>`.userInitiated`는 사용자가 **명시적으로 요청한 작업**으로, **즉각적인 응답이 필요한 작업**에 사용됩니다.
>
>예시 코드의 경우, 날씨 데이터가 앱을 켜자마자 즉각적으로 필요한 작업이므로, iOS가 이 작업에 더 많은 리소스를 할당해 빨리 끝내주도록 유도하고 있습니다.

>[!info] WeatherService.shared라고 사용하는 걸 보니, 싱글톤인가?
>**WeatherService**는  [[싱글톤 패턴 (Singleton Pattern)]]을 따르는 대표적인 Apple 프레임워크 객체 중 하나입니다. 앱 전역에서 **공통으로 날씨 데이터를 요청하기 위한 객체**이기 때문에, **한 번만 만들고 재사용**할 수 있도록 싱글톤 패턴을 사용합니다.


3. View에서 표시하기

```Swift
import SwiftUI

struct ContentView: View {
    @StateObject var weatherKitManager = WeatherKitManager()
    @StateObject var locationManager = LocationManager()
  
    var body: some View {
        VStack(spacing: 16) {
            Label(
	            "기온: \(weatherKitManager.temp)", 
	            systemImage: weatherKitManager.symbol
			)
            Label(
	            "자외선 지수: \(weatherKitManager.uvIndex)", 
	            systemImage: "sun.max"
			)
            Text(
	            "위치: \(locationManager.latitude), \(locationManager.longitude)"
			)
        }
        .padding()
        .task {
            await weatherKitManager.getWeather(
                latitude: locationManager.latitude,
                longitude: locationManager.longitude
            )
        }
    }
}
```


## References
---
* [Apple Developer - WeatherKit](https://developer.apple.com/documentation/WeatherKit)
* [WWDC - WeatherKit 소개](https://developer.apple.com/kr/videos/play/wwdc2022/10003/)
* [WeatherKit SwiftUI Tutorial - How To Build A Weather App With SwiftUI & WeatherKit](https://www.youtube.com/watch?v=Awis63z1el0)
