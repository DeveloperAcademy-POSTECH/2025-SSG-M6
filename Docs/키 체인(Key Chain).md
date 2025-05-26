### 인증 토큰 이란 : 

일종 로그인 세션 역할을 하는 값으로, 해당 토큰을 소유한 사람이라면 별도의 자격 증명을 제시하지 않고도 관련 서비스를 사용할 수 있습니다. OAuth 기반 서버에서 인증이 필요한 API를 호출할 때에는 반드시 이 토큰을 함께 전송해야 합니다. 

### 왜 키 체인으로 인증토큰을 만들어야 하나? : 

인증 토큰을 사용하기 위해서는 어딘가에 저장해 두어야 합니다. 매번 로그인하면서 이 값을 받아올 수 없을 테니까 인증 토큰은 어디다 저장하는 것이 좋을까요?

UserDefault는 저장이 간편하고 사용하기 좋으나 이 저장소는 데이터를 암호화하지 않을 뿐만 아니라 너무 쉽게 접근이 가능하기 때문에 인증 토큰을 저장하기에 적당하지 않습니다. 인증 토큰을 평문 그대로 저장하면 사용자의 기기가 도난 당했을 경우 인증 토큰도 같이 도난 당하기 때문인데,  그렇다면 "암호화 하면 되지 않나?" 라고 생각 할수 있지만 이를 위해 부가적인 작업이 필요하고 암호화키의 관리까지 고민해야 합니다. 

### 키 체인이란? :


![[Keychain Services API.png]]


애플 계열의 운영체제에서 동작하는 다양한 응용 프로그램에서 비밀번호나 계정 등 보안이 필요한 요소를 저장하는 데 사용되는 암호화된 저장소 입니다. iOS뿐만 아니라 macOS, tvOS, watchOS 등에서도 모두 제공되고 있습니다. 

아이클라우드의 로그인 정보가 키 체인을 통해 관리되고 있고 와이파이(WIFI)의 패스워드도 키 체인에 저장되며, 사파리에서 웹사이트별 패스워드를 관리할 때에도 키 체인이 사용됩니다. 

 #### 키체인의 특징
1. 기본적으로 애플리케이션은 자기 자신의 키 체인에만 접근할 수 있습니다. 
2. iOS에서 키 체인의 위치는 샌드박스 외부이므로, 앱을 삭제해도 키 체인에 저장된 정보는 삭제되지 않습니다. 
3. 앱의  프로비저닝 파일을 이용해서 앱 간의 사용 경로를 구분하기 때문에, 동일한 앱이라도 프로비저닝 파일을 변경해서 빌드하면 기존 정보를 더 이상 조회할 수 없습니다.
4. 키 체인 그룹을 사용하여 서로 다른 앱에서도 저장된 데이터를 공유할 수 있습니다. 
5. 비밀번호 또는 개인 키[^1]처럼 보호가 필요한 항목은 암호화되어 키 체인으로 보호되며, 인증서처럼 보호가 필요하지 않은 항목은 암호화되지 않은 채로 저장됩니다.
6. 키 체인은 잠글 수 있어서, 일단 잠기면 해제하기 전까지 저장된 데이터에 접근할 수 없지만 iOS에서는 기기의 잠금이 해제되는 순간 키 체인의 잠금도 함께 해제됩니다.

#### 키체인의 구조

![[Keychain Attributes.png]]

실질적으로  키 체인은 단순히 파일 시스템에 저장된 데이터베이스입니다. macOS에서 사용자나 애플리케이션은 원하는 만큼 키 체인을 만들 수 있지만. iOS에서는 모든 앱에서 사용할 수 있는 하나의 키 체인만 제공됩니다. 

아이클라우드 경우 사용자가 디바이스에서 아이클라우드 계정에 로그인하면 시스템은 논리적으로 구분되는 아이클라우드용 키 체인을 제공합니다. 

1.  키 체인 아이템 (Keychain Item) 키 체인에 저장되는 데이터로, 키 체인은 여러 개의 키 체인 아이템을 가질 수 있습니다. 

2. 아이템 클래스(Item Class) 저장할 데이터의 종류로 ID/PW, 인증서, 인터넷 비밀번호 및 일반 비밀번호 등을 선택할 수 있으며 임의로 아이템 클래스를 추가할 수 없습니다. 


| kSecClassInternetPassword | 인터넷 아이디/패스워드 저장 |
| ------------------------- | --------------- |
| kSecClassCertificate      | 인증서 저장          |
| kSecClassGenericPassword  | 일반 비밀번호 저장      |


3. 어트리뷰트(Attributes) 아이템 클래스에 대한 속성입니다. 아이템 클래스에 따라 설정할 수 있는 어트리뷰트의 종류가 달라집니다.  


| kSecAttrAccessControl      | 접속 가능한 그룹 설정          |
| -------------------------- | --------------------- |
| kSecAttrCreationDate       | 생성일                   |
| kSecAttrModificationDate   | 수정일                   |
| kSecAttrDescription        | 저장값 대한 설명             |
| kSecAttrCreator            | 생성한 유저                |
| kSecAttrAccount            | 사용자 계정                |
| kSecAttrSecurityDomain     | 보안 도메인 정보             |
| kSecAttrServer             | 서버 정보를 정하는 데 사용되는 속성  |
| kSecAttrProtocol           | 프로토콜을 저장하는 데 사용되는 속성  |
| kSecAttrAuthenticationType | 인증 유형을 저장하는 데 사용되는 속성 |
| kSecAttrPort               | 서버 포트를 저장하는 데 사용되는 속성 |
| kSecAttrPath               | 해당 URL 경로             |

예) 

![[네이버 키체인 사용예시.png| 400x400]]


네이버 앱에서 로그인하면 네이버 연동된 다른 앱들이 로그인 되어 있음 kSecAttrAccessControl 값을 공유 할수 있는 그룹 설정하면 어플리케이션 간에 키 체인 데이터를 공유 할 수 있음

한사이트 내에서도 아이디/패스워드를 사용하는 곳이 여러 군데일 경우 각 경로별로 따로 값을 저장하고 불러올 수 있으며,  kSecAttrServer, kSecAttrProtocol, kSecAttrPort, kSecAttrPath, kSecAttrAccount를 이용하여 서버 도메인, HTTP/HTTPS 프로토콜, 접속포트, 경로, 사용자 계정등을 저장한 다음, 이를 현재 웹사이트 정보와 비교하기가 가능하기 때문임


### 인증 토큰에 사용되는 아이템 클래스 

kSecClassGenericPassword 이 타입에서 핵심 어트리뷰트는 
kSecAttrAccount와 kSecAttrService로 인증 토큰을 저장하고 식별하는 메인 키(Primary Key)역할을 합니다. 

kSecAttrAccount : 저장할 비밀번호에 대한 사용자 계정
kSecAttrService : 앱을 식별할 수 있는 서비스 아이디

kSecAttrAccount는 저장할 값에 대한 사용자 계정입니다. 하나의 앱이 여러 개의 계정을 가지는 상황을 고려하면, 비밀번호 저장 시 짝을 이루는 계정을 함께 저장해서 구분할 수 있어야 하는데 이때 구분하기 위한 값이라고 보면됩니다. 

kSecAttrService는 서비스를 구분하기 위한 값으로 모든 앱이 공통으로 사용하는 키 체인에서 해당 앱을 식별하기 위한 목적으로 사용됩니다. 보통은 앱의 번들 아이디를 넣는 경우가 있지만 강제사항은 아니고, 하나의 앱내에서도 저장 용도에 따라 kSecAttrService 값을 다양하게 사용해도 됩니다. 

| 잘못된 방법                                                                              | 올바른 방법                                                                                                                                                                                                |
| ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| - 액세스 토큰 <br>kSecAttrAccount = Gus@naver.com<br>kSecAttrService = Gus.awesome.right | - 액세스 토큰 <br>kSecAttrAccount = accessToken<br>kSecAttrService = Gus.awesome.right<br><br>또는<br><br>- 액세스 토큰 <br>kSecAttrAccount =  refreshToken<br>kSecAttrService = Gus.awesome.                     |
| - 리프레시 토큰<br>kSecAttrAccount = Gus@naver.com<br>kSecAttrService = Gus.awesome.right | - 리프레시 토큰<br>kSecAttrAccount = refreshToken<br>kSecAttrService = Gus.awesome.accessToken<br><br>또는<br><br>- 리프레시 토큰<br>kSecAttrAccount =  Gus@naver.com<br>kSecAttrService = Gus.awesome.refreshToken |
| 두 토큰은 모두 동일하므로, 키 체인에는 두 <br>값 중에서 하나만 저장됨                                          | 서비스값을 조금더 상세하게 하거나, 계정 대신 다른<br>값을 넣어주어야 함                                                                                                                                                            |

### 키 체인 작업에 사용되는 코드


SecItemAdd : 저장
SecItemCopyMatching : 읽기
SecItemUpdate : 수정 
SecItemDelete : 삭제

이들은 모두 C 스타일로 코딩되어 있으며, 소위 '키 체인 쿼리(Key Chain Query)'라고 불리는 CFDictionary 타입의 데이터 집합을 인자값으로 받아 사용합니다.

```swift
// 키체인에 비밀번호를 저장하는 코드 예시

// 1. 프레임워크 반입
import Security

// 2. 키 체인 쿼리 정의
let keyChainQuery: NSDictionary = [
	kSecClass       : <아이템 클래스>,
	kSecAttrService : <서비스 아이디>,
	kSecAttrAccount : <사용자 계정>,
	kSecValueData   : <저장할 값>
]

// 3. 기존에 저장된 값 삭제
SecItemDelete(keyChainQuery)

// 4. 새로운 값 추가
SectItemAdd(keyChainQuery, nil)
```

1. 키 체인을 사용하려면 소스 코드 맨 위에 Security 프레임워크을 불러오는 구문을 작성해야 합니다. 

2. 키 체인 관련 작업에서 가장 먼저 정의해야 하는 것은 키 체인 쿼리입니다. 타입은 CFDictionary 이지만 사용 편의상 이를 상속받은 NSDictionary 또는 NSMutableDictionary 타입을 사용하는 경우가 많습니다. 어트리뷰트 키는 모두 CFString 타입으로 정의되어 있기 때문에, NSDictionary객체에 저장하기 위해서는 NSString 또는 String 타입으로 캐스팅 해야 합니다.  

   kSecClasss는 어떤 타입의 데이터를 저장할지 선택하는 아이템 클래스를 지정하는 항목입니다. 

   - kSecClassGenericPassword
   - kSecClassInternetPassword
   - kSecClassCertificate
   - kSecClassKey
   - kSecClassIdentity

   kSecValueData는 실제로 저장하는 값입니다.  Data 타입의 값을 입력 받기 때문에, 실제로 값을 저장할 때에는 Data 타입으로 인코딩하는 과정이 추가됩니다. 

```swift
let pw = "저장할 비밀번호"
let value = pw.data(using: .utf8, allowLossyConversion: false)
```

3. 키 체인은 새 값을 작성하면 자동으로 기존의 값을 덮어쓰는 구조가 아니므로 키가 중복되면 새로운 값을 저장할수 없기 때문에 항상 기존 값을 삭제해 주어야 합니다. 

4. SecItemAdd 함수는 처리 결과를 OSStatus 타입으로 반환하기 때문에 필요에 따라서는 저장한 결과를 확인할 수 있는 구문으로 바꾸어 쓸 수 있습니다. 

```swift
let status: OSStatus = SecItemAdd(keyChainQuery, nil)
assert(status == noErr, "토큰 값 저장에 실패했습니다.")
NSLog("status=\(status)")
```

```swift
// 키체인에 저장된 값을 읽어오는 코드

// 1. 키 체인 쿼리 정의 
let keyChainQuery: NSDictionary = [
	kSecClass       : <아이템 클래스>,
	kSecAttrService : <서비스 아이디>,
	kSecAttrAccount : <사용자 계정>,
	kSecReturnData  : kCFBooleanTrue,
	kSecMatchLimit  : kSecMatchLimitOne
]

// 2. 저장된 값 읽기
var dataTypeRef: AnyObject?
SecItemCopyMatching(keyChainQuery, &dataTypeRef)

// 3. 읽어온 값 변환
let retrivedData = dataTypeRef as! Data
let value = String(data: retrievedData, encoding: String.Encoding.utf8)

```
1. 저장된 값을 읽어올 때에도 가장 먼저  키 체인 쿼리를 정의하는 것입니다.  이때 아이템 클래스와 서비스 아이디, 사용자 계정은 저장 시 설정한 값을 그대로 사용해야 합니다. 

   kSecReturnData는 읽어올 데이터 타입을 결정하는 부분으로, 값으로 설정된 kCFBooleanTrue는 저장된 값을 CFData 형식으로 읽어오도록 지시하는 부분입니다. 나중에 이 타입 대신 호환 가능한 AnyObject 타입을 사용하고, 바로 Data타입으로 반환합니다. 

2. SecItemCopyMatching 함수는 저장된 값을 읽어 올때 사용합니다. 이 함수는 OSStatus 타입의 처리 결과를 반환하기 때문에, 키체인에 저장된 값은 매개변수를 이용하여 반환해야 합니다. 이를 위해 두번째 매개변수를 inout 타입[^2]으로 정의합니다. 

3. AnyObject 타입으로 읽어온 값을 Data 타입으로 변환하고, 이를 다시 String 타입으로 변환하면 됩니다. 


2~3 방법은 빈 데이터를 불러올 가능성이 있으므로 체크하여 정상적으로 값을 읽어왔을 때만 String 타입으로 변환하는 것이 좋습니다. 

```swift
let status = SecItemCopyMatching(keyChainQuery, &dataTypeRef)

if (status == errSecSuccess) {
	let retrievedData = dataTypeRef as! Data
	let value = String(data: retrievedData, encoding: String.Encoding.utf8)
} else {
 print("키 체인에서 불러올 데이터가 없습니다. Status code \(status)")
}
```


## 추가팁 !
### 키 체인 래핑하기 


키 체인을 다루기 위해 필요한 코드가 좀 많아서 실무에서 편하게 사용하기 위해 래핑된 클래스를 사용합니다. 

```swift

class keyChaingTokenUtils {

    // 키 체인에 값을 저장하는 메소드
	func save(_ service: String, account: String, value: String) {
		let keyChainQuery : NSDictionary = [
			kSecClass : kSecClassGenericPassword,
			kSecAttrService : service,
			kSecAttrAccount : account,
			kSecValueData : value.data(using: .utf8, allowLossyConversion: false)!
		]
		
		// 현재 저장되어 있는 값 삭제
		SecItemDelete(keyChainQuery)

		// 새로운 키 체인 아이템 등록
		let status: OSStatus = SecItemAdd(keyChainQuery, nil)
		assert(status == noErr, "토큰 값 저장에 실패했습니다.")
		NSLog("status=\(status)")
	}

	// 키 체인에 저장된 값을 읽어오는 메소드
	func load(_ service: String, account String) -> String? {

		// 1. 키 체인 쿼리 정의
		let keyChainQuery: NSDictionary = [
			kSecClass : kSecClassGenericPassword,
			kSecAttrService : service,
			kSecAttrAccount : account, 
			kSecReturnData : kCFBooleanTrue!, // CFDataRef
			kSecMatchLimit : kSecMatchLimiOne
		]

		// 2. 키 체인에 저장된 값을 읽어온다. 
		var dataTypeRef: AnyObject?
		let status = SecItemCopyMatcing(keyChainQuery, &dataTypeRef)

		// 3. 처리 결과가 성공이라면 
		if (status == errSecSuccess) {
			let retrievedData = dataTypeRef as! Data
			let value = String(data: retrievedData, encoding: String.Encoding.utf8)
			return value
		} else { // 4. 처리 결과가 실패라면 nil을 반환한다. 
			print("키 체인에서 불러올 데이터가 없습니다. Status code \(status)")
		}
	}

	// 키 체인에 저장된 값을 삭제하는 메소드
	func delete(_ service: String, account: String) {
		let keyChainQuery: NSDictionary = [
			kSecClass : kSecClassGenericPassword,
			kSecAttrService : service,
			kSecAttrAccount : account, 
		]

		// 현재 저장되어 있는 값 삭제
		let status = SecItemDelete(keyChainQuery)
		assert(status == noErr, "토큰 값 삭제에 실패")
		NSLog("status=\(status)")
	}

	// 키 체인에 저장된 액세스 토큰을 이용하여 헤더를 만들어주는 메소드
	func getAuthorizationHeader() -> HTTPHeaders? {
		let serviceID = "Gus.Awesome.right"
		
		if let accessToken = self.load(serviceID, account: "accessToken") {
			return ["Authorization" : "Bearer \(accessToken)"] as HTTPHeaders
		} else {
			return nil
		}
	}
}

```
[^1]: 비대칭 암호화에서 사용되는 짧은 바이트 문자열
[^2]: 함수 내부에서 수정된 인자값을 함수 외부에서도 참조 할수 있는 것을 말함
https://developer.apple.com/documentation/security/keychain-services
https://developer.apple.com/documentation/security/keychain-items