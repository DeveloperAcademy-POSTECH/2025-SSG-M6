>[!question] Question
>GQ1. iOS에서 위치 정보는 어떤 조건에서 갱신될까?
>GQ2. 배터리 효율을 유지하면서 필요한 시점에만 위치를 업데이트하려면 어떤 전략을 써야 할까?

## Description
---
CoreLocation은 iOS에서 기기의 현재 위치를 가져오거나 지속적으로 추적할 수 있는 프레임워크다. 위치 갱신 방법은 **포그라운드·백그라운드·이벤트 기반** 등 상황에 따라 다르게 선택할 수 있으며, 배터리 효율과 정확도를 모두 고려하여 적절한 방식을 선택하는 것이 중요하다.


## 위치 갱신 방식 비교
---

1. 포그라운드 지속 갱신 - `startUpdatingLocation()`
	- 앱 실행 중 **계속 위치를 받는 방식**
	- GPS가 **계속 활성화**되므로 실시간 경로 추적·내비게이션에 적합
	- 배터리 소모 **매우 높음**
	* 배터리 절약 팁 :
		* desiredAccuracy를 kCLLocationAccuracyHundredMeters 등 낮은 값으로 설정 → GPS 사용 빈도↓
		* distanceFilter로 최소 이동 거리 지정 → 불필요한 이벤트 보고↓ (GPS는 계속 켜짐)
	
2. 포그라운드 단발 갱신 - `requestLocation()`
	- 호출 시 **한 번만** 현재 위치 반환
	- 호출 후 GPS 자동 종료 → **배터리 절약 효과 큼**
	- 지도 초기 위치 표시, 날씨 API 요청 등 짧은 순간 측정에 적합
	
3. 백그라운드 지속 갱신 - `startUpdatingLocation()`
	- 앱이 백그라운드에서도 위치 계속 추적
	- 러닝·워킹 앱, 장거리 이동 기록에 사용
	* 조건 :
		* `requestAlwaysAuthorization()` 로 Always 권한 받기
		* Xcode → Capabilities → Background Modes → **Location updates** 활성화
		* `allowsBackgroundLocationUpdates = true` 설정
	* 배터리 소모 **매우 높음**
	* 절약 방법 :
		* 정확도 낮추기 → GPS 사용 최소화
		* `distanceFilter` 로 보고 빈도 줄이기
		![[CoreLocation 백그라운드 모드 설정 1.png]]

4. 중요 위치 변화 감지 - `startMonitoringSignificantLocationChanges()`
	- **큰 이동**(셀룰러 기지국 전환 등) 시에만 보고
	- GPS 대부분 꺼져 있음 → 배터리 소모 **매우 적음**
	- 하루 종일 켜놔도 배터리 부담 거의 없음
	- 실시간성은 떨어짐 (수백 m 이상 이동해야 감지 가능)
	
5. 지오펜싱 – `startMonitoring(for:)`
	- 지정 반경 진입/이탈 시에만 이벤트 발생
	- Wi‑Fi·기지국 기반 측정 위주, 필요 시에만 GPS 사용
	- 중간 수준의 배터리 소모


>[!info] 배터리 절약을 위한 서비스 최적화
>- **desiredAccuracy** (정확도): 낮출수록 GPS 사용 빈도가 줄어 배터리를 아낄 수 있음
>	- 높은 값 → GPS 위주 (배터리↑, 정확도↑)
>	- 낮은 값 → Wi‑Fi/기지국 위주 (배터리↓, 정확도↓
>- **distanceFilter**: 위치 측정은 계속하지만, **보고 횟수만 제한** → 배터리 절감 효과는 적음, 앱 부하 감소
>- 배터리를 절약하려면 **갱신 주기·방식·정확도**를 함께 조정하는 것이 중요

>[!info] 주요 desiredAccuracy 상수 
>- **kCLLocationAccuracyBestForNavigation** : 내비게이션 전용, 5~10m
>- **kCLLocationAccuracyBest** : 최고 정확도, 5~10m
>- **kCLLocationAccuracyNearestTenMeters** : 10~20m
>- **kCLLocationAccuracyHundredMeters** : 50~300m
>- **kCLLocationAccuracyKilometer** : 500m~1km
>- **kCLLocationAccuracyThreeKilometers** : 3~5km

## 코드 예시
---
```Swift
import SwiftUI
import CoreLocation

// MARK: - Location Manager
final class LocationManager: NSObject, ObservableObject, CLLocationManagerDelegate {

    @Published var currentLocation: CLLocation?
    @Published var authorizationStatus: CLAuthorizationStatus?

    private let locationManager = CLLocationManager()

    override init() {
        super.init()
        setupLocationManager()
    }

    // 기본 설정
    private func setupLocationManager() {
        locationManager.delegate = self
        // 기본값: 정확도 최고, 모든 이동 보고 → 배터리 많이 씀
        // 필요에 따라 조정 권장
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.distanceFilter = kCLDistanceFilterNone
    }

    // MARK: - 권한 요청
    func requestPermission(always: Bool = false) {
        if always {
            locationManager.requestAlwaysAuthorization() // 포그라운드+백그라운드 가능
        } else {
            locationManager.requestWhenInUseAuthorization() // 포그라운드만 가능
        }
    }

    // MARK: - 갱신 방식
    /// 1. 포그라운드 지속 갱신
    /// - 앱 실행 중 계속 위치 업데이트
    func startForegroundContinuousUpdates() {
        locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters 
        locationManager.distanceFilter = 20 // 최소 20m 이동 시 보고
        locationManager.startUpdatingLocation()
    }

    /// 2. 포그라운드 단발 갱신
    /// - 호출 시 1회만 위치 반환 후 종료
    func requestSingleLocation() {
        locationManager.requestLocation()
    }

    /// 3. 백그라운드 지속 갱신
    /// - Always 권한 필요
    /// - Capabilities → Background Modes → Location updates 켜야 함
    func startBackgroundContinuousUpdates() {
        locationManager.requestAlwaysAuthorization()
        locationManager.allowsBackgroundLocationUpdates = true
        locationManager.pausesLocationUpdatesAutomatically = false
        locationManager.desiredAccuracy = kCLLocationAccuracyHundredMeters
        locationManager.distanceFilter = 50 // 50m 이상 이동 시 보고
        locationManager.startUpdatingLocation()
    }

    /// 4. 중요 위치 변화 감지
    /// - 큰 이동(셀룰러 기지국 변경 등) 있을 때만 보고
    func startSignificantChangeMonitoring() {
        locationManager.startMonitoringSignificantLocationChanges()
    }

    /// 5. 지오펜싱
    /// - 지정 반경 진입/이탈 감지
    func startGeofencing() {
        let center = CLLocationCoordinate2D(latitude: 37.3318, longitude: -122.0312)
        let region = CLCircularRegion(center: center, radius: 100, identifier: "ApplePark")

        region.notifyOnEntry = true
        region.notifyOnExit = true
        locationManager.startMonitoring(for: region)
    }

    // MARK: - CLLocationManagerDelegate
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {

        currentLocation = locations.last
        if let loc = currentLocation {
            print("위치 갱신: \(loc.coordinate.latitude), \(loc.coordinate.longitude)")
        }
    }

    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("위치 갱신 실패:", error.localizedDescription)
    }

    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        authorizationStatus = manager.authorizationStatus
    }

    func locationManager(_ manager: CLLocationManager, didEnterRegion region: CLRegion) {
        print("진입: \(region.identifier)")
    }

    func locationManager(_ manager: CLLocationManager, didExitRegion region: CLRegion) {
        print("이탈: \(region.identifier)")
    }
}
```


## References
---
* [Core Location – Apple Developer](https://developer.apple.com/documentation/corelocation)
