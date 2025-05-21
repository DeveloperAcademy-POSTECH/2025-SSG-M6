>[!question] 
>GQ1. UserDefaults는 어떤 목적과 구조로 설계된 저장 방식인가?
>GQ2. UserDefaults를 사용할 때 주의해야 할 점은 무엇인가?

## Description
---
UserDefaults는 iOS에서 간단한 사용자 설정 값이나 앱 상태 값을 저장하기 위해 제공되는 경량화된 로컬 저장 방식이다. 내부적으로는 키-값(Key-Value) 구조로 작동하며, 기본 데이터 타입을 간단하게 저장하고 불러오는 데 적합하다. 설정값을 저장할 수 있는 인터페이스를 제공하며, 앱이 삭제되지 않는 한 값은 유지된다. 일반적으로 앱의 환경설정, 플래그, 마지막 로그인 상태 등 간단한 정보를 저장하는 데 사용된다.

UserDefaults는 내부적으로 앱의 [[SandBox]] 내 `~/Library/Preferences/<bundleID>.plist` 파일에 저장하며, 이 파일은 암호화되지 않고 일반 텍스트(XML) 또는 바이너리 plist 형식으로 저장된다. 이로 인해 보안이 필요한 정보는 쉽게 추출될 수 있으며, 탈옥된 기기나 macOS 시뮬레이터에서는 사용자가 직접 접근하여 읽고 수정할 수 있다.

>[!샌드박스는 보안구조인데, 어떻게 정보가 쉽게 추출될 수 있는 말일까?]
>샌드박스는 외부로부터의 접근은 차단하지만, 내부 저장 데이터를 암호화하지는 않기 때문에, 앱이 보안이 필요한 정보를 UserDefaults 같은 단순 저장소에 저장하면, 탈옥된 기기나 시뮬레이터 환경에서는 그 값이 그대로 노출될 수 있다!

## 주요 기능
---
UserDefaults는 다음과 같은 데이터를 저장할 수 있다.

* String, Int, Double, Float, Bool
* Array, Dictionary (단, 요소는 plist 호환 타입이어야 함)
* Data, Date, URL 등

또한 `UserDefaults.standard` 인스턴스를 통해 전역적으로 접근 가능하며, 앱 전체에서 동일한 값에 접근할 수 있다. 활용 예시로는 다음과 같은 것들이 있다.

- 앱 최초 실행 여부 판단 (`hasLaunchedBefore`)    
- 사용자 로그인 상태 (`isLoggedIn`)
- 마지막으로 선택한 탭 또는 메뉴 저장
- 간단한 필터 조건 기억 등

>[!보안이 왜 안되는데?]
>UserDefaults는 암호화되지 않은 `.plist` 파일에 데이터를 저장한다. 예를 들어, 시뮬레이터에서는 다음 경로에서 해당 파일을 쉽게 확인할 수 있다. 
>`~/Library/Developer/CoreSimulator/Devices/{UDID}/data/Containers/Data/Application/{AppUUID}/Library/Preferences/{bundleID}.plist`
>이 파일은 macOS의 텍스트 에디터, Xcode의 plist 편집기 같은 도구로 열 수 있다.

>[!대용량 데이터에 왜 적당하지 못해?]
UserDefaults는 기본적으로 **전체 데이터를 메모리에 캐싱해두었다가 주기적으로 비동기 저장**한다. 이 구조는 큰 데이터(ex. 이미지, 영상, 대규모 배열 등)을 저장하면 앱 실행 시 메모리 로딩 지연, 비동기 저장 중 충돌 혹은 누락 가능성, 내부 plist 파일 전체가 덮어써져 성능 저하 등이 발생할 수 있다.

## 코드 예시 
---
```Swift
// 데이터 저장
UserDefaults.standard.set(true, forKey: "isLoggedIn")
UserDefaults.standard.set("ko-KR", forKey: "selectedLanguage")
UserDefaults.standard.set(["Swift", "iOS"], forKey: "preferredTopics")

// 데이터 읽기
let isLoggedIn = UserDefaults.standard.bool(forKey: "isLoggedIn")
let language = UserDefaults.standard.string(forKey: "selectedLanguage") ?? "en"

// 데이터 삭제
UserDefaults.standard.removeObject(forKey: "isLoggedIn")

// 커스텀 객체 저장 
struct User: Codable {
	let name: String
	let age: Int
}

let user = User(name: "J", age:26)
if let encoded = try? JSONEncoder().encode(user) {
	UserDefaults.standard.set(encoded, forKey: "currentUser")
}

if let data = UserDefaults.standard.data(forKey: "currentUser"), 
	let decoded = try? JSONDecoder().decode(User.self, fromL data) {
		print(decoded.name)
}

```

## Keywords
---
- 비동기
- 메모리 캐싱
- plist
- iOS [[SandBox]]
- Codable
* 인코딩, 디코딩

## References
---
- Apple Developer Documentation - [UserDefaults](https://developer.apple.com/documentation/foundation/userdefaults)
- 블로그 - [UserDefaults 알아보기](https://jeong9216.tistory.com/520)
- 블로그 - [UserDefault 사용해서 데이터 저장, 사용하기](https://lxxyeon.tistory.com/203)
