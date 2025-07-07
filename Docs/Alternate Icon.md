>[!question]
>GQ1. 유저에게 앱 아이콘 선택권을 주고싶어잉~

## Description
- 듀오링고앱 아이콘은 막 바뀌더라구요? 저도 바꿔보고 싶었습니다.
- 검색해보니까 alternate icon이라는 키워드를 발견했어요.
## 주요 기능
1. 이렇게 asset에 앱 아이콘 여러 개 올려놓으면 준비 끝!
![[AlternateIcons.png]]
2. 이제 앱아이콘 바꿔주면 됩니다.
## 코드 예시
+ 앱 아이콘 바꿔주는 코드~
```swift
UIApplication.shared.setAlternateIconName(iconName) { (error) **in**
	if let error = error {
		print("Failed request to update the app’s icon: \(error)")
	}
}
```
레퍼런스 링크타고 들어가서 샘플 프로젝트를 다운받아보실 수 있습니다.

## Keywords
+ alternate icon

## References
- https://developer.apple.com/documentation/xcode/configuring_your_app_to_use_alternate_app_icons#4047496