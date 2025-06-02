## 1. Network Condition
   
서버 요청/응답이 필요한 앱의 경우 여러 네트워크 상태에 대해서 테스트가 필요합니다. (네트워크가 좋지 않을 경우, 네트워크가 끊겼을 경우 앱의 동작 확인 등)

Xcode → Window → Devices and Simulator → DEVICE CONDITIONS 에서 네트워크 상태를 조절할 수 있습니다.

네트워크 100% 유실 뿐만 아니라 2G 환경, 3G 환경 등등 다양하게 선택할 수 있습니다.


![[디버깅 network.jpg]]


## 2. Network Instrument

 서버 요청/응답에 대한 정보 알고 싶을 때 유용합니다. iOS 15 이상 디바이스에서만 확인이 가능합니다. (시뮬레이터는 불가능합니다.) `Instruments` 앱을 실행하고 `Network`를 선택합니다.

![[Instruments선택.png]]
![[Instruments에서 network 선택.png]]

