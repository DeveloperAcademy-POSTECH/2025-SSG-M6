>[!question]
>GQ1. VoiceOver에 대해 알아보자.


## Description
#### **VoiceOver란?**

**VoiceOver**는 Apple의 스크린 리더(Screen Reader)로, 시각적 정보 대신 UI 요소와 그 동작을 음성으로 안내해주는 iOS, iPadOS, macOS의 내장 보조 기술입니다. VoiceOver는 텍스트, 버튼, 이미지 등 인터페이스 요소를 탐색하고, 해당 요소의 이름, 역할, 상태, 힌트 등을 사용자에게 읽어줍니다.

개발자는 접근성 속성을 적절히 지정하여 VoiceOver가 올바른 정보를 전달하도록 해야 합니다.

---

#### **VoiceOver 활성화 및 사용법**

1. **설정 > 손쉬운 사용 > VoiceOver**에서 VoiceOver를 활성화할 수 있습니다.
    
2. 또는 **Siri**에게 “VoiceOver 켜줘”라고 말하면 빠르게 활성화할 수 있습니다.
    
3. **VoiceOver 동작 예시**
    
    - 한 번 터치: 요소 선택 및 정보 읽기
        
    - 두 번 탭: 선택한 요소 활성화(예: 버튼 클릭)
        
    - 세 손가락 스와이프: 화면 이동
        
    

VoiceOver가 켜져 있으면, 사용자는 화면의 각 UI 요소를 손가락으로 탐색하고, VoiceOver가 해당 요소의 **라벨, 역할, 상태, 힌트**를 음성으로 안내합니다.

## 주요 기능
#### **SwiftUI에서 접근성(Accessibility) 지원 방법**

SwiftUI는 접근성 기능을 지원하기 위한 다양한 Modifier를 제공합니다. VoiceOver 등 보조 기술을 통해 UI 요소의 의미와 동작을 명확하게 전달할 수 있도록 접근성 속성을 추가할 수 있습니다.

## 코드 예시
---

### **1. 접근성 라벨(accessibilityLabel)**

- UI 요소의 의미를 명확하게 전달하기 위한 텍스트를 제공합니다.
    

```
Text("삭제")
    .accessibilityLabel("삭제 버튼")
```

---

### **2. 접근성 힌트(accessibilityHint)**

- 요소의 동작에 대한 추가 설명을 제공합니다.
    

```
Button(action: {
    // 동작 구현
}) {
    Image(systemName: "trash")
}
.accessibilityLabel("항목 삭제")
.accessibilityHint("이 버튼을 누르면 선택한 항목이 삭제됩니다.")
```

---

### **3. 접근성 값(accessibilityValue)**

- 토글 등 상태 기반 UI 요소에 상태 값을 제공합니다.
    

```
Toggle(isOn: $isOn) {
    Text("알림")
}
.accessibilityValue(isOn ? "켜짐" : "꺼짐")
```

---

### **4. 이미지 접근성**

- 이미지 등 시각적 요소에 대체 텍스트를 제공합니다.
    

```
Image(systemName: "trash")
    .accessibilityLabel("휴지통 아이콘")
```

---

### **5. 커스텀 뷰 및 그룹**

- 여러 자식 뷰를 하나의 접근성 요소로 결합할 수 있습니다.
    

```
VStack {
    Text("총합")
    Text("\(total)원")
}
.accessibilityElement(children: .combine)
.accessibilityLabel("총합 \(total)원")
```

## Keywords
+ [Accessibility]
+ [SwiftUI]

## References
- https://developer.apple.com/documentation/uikit/supporting-voiceover-in-your-app
- https://developer.apple.com/documentation/swiftui/modifiedcontent
- https://developer.apple.com/documentation/accessibility/voiceover