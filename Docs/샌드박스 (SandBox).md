>[!question] 
>GQ1. 샌드박스란 무엇이며, 왜 존재할까?

## Description
---

샌드박스(SandBox)는 애플리케이션이 **시스템의 다른 자원에 무분별하게 접근하거나 영향을 미치는 것을 막기 위해 격리된 환경에서 실행**하도록 하는 보안 기술이다. iOS에서 모든 앱이 기본적으로 샌드박스 환경에서 동작하며, 이로 인해 앱은 자신에게 허용된 디렉터리 외에는 파일을 생성하거나 접근할 수 없다.

샌드박싱은 앱이 시스템을 망가뜨리거나, 다른 앱의 데이터를 침범하거나, 민감한 사용자 정보를 무단으로 읽는 것을 방지하는 데 목적이 있다. 이를 통해 앱 간 간섭 없이 안정성과 보안을 높이고, 사용자의 프라이버시도 강화할 수 있다.

## 주요 기능
---
* iOS의 각 앱은 설치될 때 고유한 **컨테이너 디렉터리**가 생성된다.
* 해당 앱은 **오직 자신의 컨테이너 내에서만 읽기/쓰기 가능**하다.
* 기본 디렉터리 구조
	* `Documents/` : 사용자 생성 데이터 저장
	* `Library/` : 설정, 캐시 등
	* `tmp/` : 임시 파일 (재시작 시 삭제됨)
	* `Library/Preferences/` : UserDefaults 데이터 저장 (설정값)
* 앱이 다른 앱의 컨테이너나 시스셈 파일 접근을 시도하면 거부된다.

## 코드 예시
---
* iOS에서 외부 시스템 경로에 접근하려고 하면 항상 false가 반환되거나 에러가 발생한다. → 샌드박스에 의해 차단
```Swift

let unsafePath = "/private/var/root/secret.txt"

let fileExists = FileManager.default.fileExists(atPath: unsafePath)

  

print("파일 접근 가능? \(fileExists)")

```

## Keywords
---
* 보안
* 앱 간 데이터 접근 제한
* 파일 시스템
## References
---
* Apple Developer Documentation - [SandBox](https://developer.apple.com/documentation/security/app-sandbox)
- Blog - [iOS 샌드박스 구조](https://odinios.tistory.com/8)
- Blog - [iOS 앱 샌드박스란](https://lxxyeon.tistory.com/222)