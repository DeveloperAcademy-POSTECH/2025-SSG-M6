>[!question]
>GQ1. - AppStorage가 뭘까?
>GQ2. - UserDefaults는 뭘까?
>GQ3 - 어떻게 사용할까?


## [[UserDefaults]]
- An interface to the user’s defaults database, where you store key-value pairs persistently across launches of your app.
- 앱을 여러 번 껐다 켜도 유지되는, 사용자 설정(기본값)을 저장하는 데이터베이스에 접근하는 인터페이스.
  키-값 으로 데이터를 저장할 수 있다.
- **저장할 수 있는 타입**
	-  String, Int, Double, Float, Bool
	* Array, Dictionary (단, 요소는 plist 호환 타입이어야 함)
	* Data, Date, URL 등

## @AppStorage
- A property wrapper type that reflects a value from UserDefaults and invalidates a view on a change in value in that user default.
- UserDefaults에서 값을 가져와 연결하고, 그 값이 바뀌면 뷰를 자동으로 다시 그리게 만드는 프로퍼티 래퍼 타입이다.
- **저장할 수 있는 타입 (SwiftUI에서 안전하게 바인딩할 수 있는 타입만 지원)**
	- String, Int, Double, Bool
	- Data, URL
```swift
@AppStorage("isDarkMode") var isDarkMode: Bool = false

Toggle("Dark Mode", isOn: $isDarkMode) // 자동으로 UI 갱신
```

| 항목      | `UserDefaults`           | `@AppStorage`                    |
| ------- | ------------------------ | -------------------------------- |
| 역할      | 저장만 함 (UIKit/SwiftUI 공용) | 저장 + 상태 변화 감지 (SwiftUI 전용)       |
| 뷰 자동 갱신 | 불가능 (직접 감지해야 함)          | 가능 (값 변경 시 뷰 자동 갱신)              |
| 사용 대상   | 로직, 뷰 외부 데이터 저장          | SwiftUI 뷰와 바인딩하고 싶을 때            |
| 내부 구현   | 자체 저장소                   | 내부적으로 `UserDefaults.standard` 사용 |

>[!AppStorage가 왜 생겼을까?]
>SwiftUI는 **상태 기반 UI 프레임워크**로 값이 바뀌면 UI가 다시 그려져야 한다.
>하지만 `UserDefaults`는 값이 바뀌어도 UI에 아무런 영향을 주지 않는다.
>그래서 등장한 `@AppStorage`는 내부적으로 `UserDefaults`를 사용하지만 
>값이 바뀌면 **SwiftUI View를 자동으로 다시 렌더링**해준다.

### SwiftUI에서 `UserDefaults`를 사용해야할 때는?
####  1️⃣ 배열, 딕셔너리 저장시
> ex) **최근 검색어 저장**

❌ `@AppStorage`는 배열/딕셔너리 저장을 지원하지 않음
```swift
	@AppStorage("recentSearches") var recentSearches: [String] = []   //에러
```

✅ `UserDefaults`는 배열/딕셔너리 저장 지원
```swift
let searches = ["apple", "swift", "xcode"]
UserDefaults.standard.set(searches, forKey: "recentSearches")
```

####  2️⃣ 구조체 저장시 (Codable)
> ex) **유저 프로필 저장**

- **Codable**
	- 구조체나 클래스 등을 JSON이나 Data로 변환하고, 다시 복원할 수 있게 해주는 프로토콜
- **Encode** 
	- 데이터를 기기가 알아볼 수 있게 JSON이나 Data로 변환
- **Decode**
	- 데이터를 사람이 알아볼 수 있게 구조체로 복원

✅ `UserDefaults`로 저장
```swift
struct Profile: Codable {
    var name: String
    var age: Int
}

let profile = Profile(name: "Jamin", age: 25)

// 저장(Encode)
let data = try? JSONEncoder().encode(profile)
UserDefaults.standard.set(data, forKey: "userProfile")

// 불러오기(Decode)
if let savedData = UserDefaults.standard.data(forKey: "userProfile") {
    if let decodedProfile = try? JSONDecoder().decode(Profile.self, from: savedData) {
        print(decodedProfile.name)
        print(decodedProfile.age)
    }
}
```

## References
- https://developer.apple.com/documentation/swiftui/appstorage
- https://developer.apple.com/documentation/foundation/userdefaults