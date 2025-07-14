
## APNs 란?
---
<span style="color:orange">Apple Push Notification service (APNs)</span> 는 Apple이 제공하는 알림 서비스 플랫폼으로, 서드 파티 서버 개발자가 사용자 디바이스로 푸시 알림을 보낼 수 있게 해주는 클라우드 서비스이다.

보통 Push Server가 **앱에 푸시 알람**을 보낸다고 생각하면 Server에서 App으로 바로 보내면 될 것 같지만, Apple은 Server가 앱에 직접 알람을 보내는 것을 용납하지 못하고 **APNs라는 것을 통해서만** 보낼 수 있게 해놓았다.


## 디바이스 토큰 기반 푸시 동작 방식
---
#### ✔️ Push를 받기 위한 동작

1. App이 APNs에 Device Token을 요청
2. APNs가 App에게 Device Token 전송
3. App이 Server에게 Device Token 전송

![[APNs Device Token 등록 흐름도.png]]
>[!info] Device Token
>Device Token은 Push를 제대로 App에 보내기 위한 고유 주소 역할(64자리 16진수 문자열)을 한다.
>기기와 앱 모두 고유한 주소를 가지며, 두 앱이 같은 기기에 있더라도 A앱의 토큰을 B앱에서 사용할 수 없다.
>앱이 시작되면 APNs에 요청해 Devcie Token을 얻을 수 있고 이걸 서버로 보내 저장하고 있도록 한다.
>Device Token은 앱이 백업될 때, 새로 발급된다.
>* 토큰 요청은 UIApplication의 [registerForRemoteNotifications()](https://developer.apple.com/documentation/uikit/uiapplication/1623078-registerforremotenotifications) 메서드 사용
>* 토큰 받기는 App Delegate의 [application(_:didRegisterForRemoteNotificationsWithDeviceToken:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622958-application) 메소드 사용
>
>를 하면 된다고 한다.


#### ✔️ Push 발생 동작

1. Server는 APNs에게 `Device Token + Data` 를 전송
2. APNs에서 Device Token으로 Data를 전송
3. App에서 Push 동작 
	- Background 상태면 OS에서 처리
	- Foreground 상태면 AppDelegate에서 정의한 대로 App 위에 띄워짐

![[APNs Device Token 푸시 전송.png]]


#### ✔️ Push 알림의 데이터 형식 (APNs 페이로드)

Device Token에 데이터를 전달할 때는 256byte를 초과하지 않는 JSON 데이터 형태여야 한다.

```JSON
{
  "aps": {
    "alert": "텍스트 알림",
    "badge": 1,
    "sound": "default",
    "category": "MESSAGE_CATEGORY",
    "thread-id": "thread-id-123"
  }
}
```

[Payload 키 값에 대한 추가 설명](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/PayloadKeyReference.html)


#### ✔️ 인증 방식 : 인증서 방식 vs 토큰 방식의 차이

APNs로 푸시를 보내기 위해서는 서버 인증 방식을 선택해야하는데, 크게 2가지가 있다

##### 1. 인증서 기반 `.cer` + `.p12` (SSL 인증서 기반)

* 사용 파일 : 인증서 `.cer`, 개인키 포함된 `.12`
* 인증 방식 : SSL 인증서 기반 APNs 서버와 TLS 소켓 연결
* 유효 기간 : 보통 1년, 갱신 필요
* 앱 연결 방식 : 앱 번들 ID마다 인증서 필요
* 사용처 : Firebase 등

##### 2. 토큰 기반 방식 `.p8` (JWT 기반)

* 사용 파일 : `.p8` (Apple Push Notification Auth Key)
* 인증 방식 : JWT (JSON Web Token) 기반 인증 (HTTP/2 요청)
* 유효 기간 : .p8 파일은 만료 없음, JWT는 만료가 있음
* 앱 연결 방식 : 하나의 키로 여러 앱 지원 가능 (Key ID, Team ID 필요)
* 사용처 : Node.js, Python 등 직접 서버를 구축해서 HTTP 요청을 날릴 때

>[!info] 서버에서 APNs로 요청을 보낼 때 꼭 필요한 헤더
>- `Authorization` : JWT 토큰
>- `apns-topic` : 앱 번들 ID
>- `apns-push-type` : alert, background, voip, liveactivity 등
>- `apns-priority` : 10 (즉시), 5(절전, 백그라운드에서 처리)
>- `apns-expiration` : 푸시 유효기간 (0은 즉시 만료됨)

```http
POST /3/device/<device_token> HTTP/2
Host: api.push.apple.com
Authorization: bearer <JWT>
apns-topic: com.example.MyApp
apns-push-type: alert
apns-priority: 10
apns-expiration: 0
```


## 정리
---
* APNs는 개발자가 디바이스에 직접 푸시를 보내는게 아니라 Apple이 대신 전달해주는 구조
* Device Token은 디바이스 + 앱 조합의 주소
* 서버에서 APNs에 데이터를 보낼 때 인증 방식은 .p12 방식과 .p8 방식 2가지가 존재
* Firebase는 .p12 방식만 지원, 직접 서버 호출은 .p8 방식을 권장