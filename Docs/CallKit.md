과 더불어 **Live Translation**(전화 중 실시간 번역)까지 ..

>[!question]
>GQ1. iOS 26에서 CallKit에 어떤 새로운 기능이 추가되었을까?
>GQ2. 통화 중 실시간 번역은 어떻게 작동할까?
>GQ3. Apple Intelligence와 어떤 관련이 있을까?

## CallKit이 뭔데 !

**CallKit**은 Apple이 제공하는 프레임워크로, VoIP (Voice over IP) 앱이 시스템 수준의 전화처럼 수신/발신 통화 UI를 제공하고, 다른 앱 및 시스템 동작(방해 금지 모드 등)과 자연스럽게 통합되도록 해줍니다.

쉽게 말해 나의 앱이 WhatsApp, FaceTime, 전화 앱처럼 보이고 동작하게 만들어줌

근데 여기서 VoIP 앱은 뭔가요?
- 인터넷을 통해 음성 통화를 가능하게 해주는 기술 = **전화번호망 (3G/LTE) 대신 Wi-Fi나 데이터 통신을 통해 전화 통화를 제공하는 앱**

그럼 CallKit이랑 VoIP 앱은 어떻게 연결되는데요?

🙋🏻‍♀️ (친구) -- 통화 요청 -> 서버 -> VoIP Push -> 내 앱 -> CallKit UI 호출

🙆🏻‍♀️ (나) -- 전화 받기 누름 -> VoIP 앱이 실제 음성 연결을 시작함

**결론**

- VoIP 앱 : **실제 음성 연결**을 처리함
- CallKit : **통화 UI**, 시스템 통화 이벤트 처리 담당 -> 그냥 거의 UI 담당
- PushKit : 수신 통화 알림을 **백그라운드에서** 받는 데 사용됨

중에서 CallKit의 기능

- UI 제공
- 시스템 통합
- 사용자 이벤트 감지
- 방해 금지 모드 대응
+
아래에서 더 다룰 것임 .


궁금해서 찾아봤는데 CallKit 기반 앱이 생각보다 많더라구요?
Zoom, Telegram, Skype, KakaoTalk, Discord, Line 등등 . . 메신저 내부에서 통화 기능이 가능한 앱들이 거의 자체 VoIP 엔진을 사용해 통화 기능을 직접 구현하고, 통화 UI를 CallKit 기반으로 하고 있었습니다.

## iOS 26에서 CallKit에 어떤 새로운 기능이 추가되었을까?

#### Call translation API

통화 중 번역을 자동으로 켜고 끌 수 있습니다.

핵심 클래스는 **CXSetTranslatingCallAction**.
통화 중 실시간 번역 기능을 앱에서 **직접 켜고 끌 수 있도록 하는** 새로운 액션 클래스

CallKit 기반 통화 중 사용자가 "번역 켜기"를 누르면, 이 액션을 통해 시스템에 전달되어요.
시스템은 이후 통화 오디오 스트림을 자동 인식하여 **자막 or 음성 번역을 표시**

-  init(call: isTranslating: localLocale: remoteLocale: )
- 통화 UUID, 현지 언어, 상대방 언어, 그리고 번역 활성 여부를 인자로 받을 수 있습니다.

```swift
let action = CXSetTranslatingCallAction(
    call: callUUID,
    isTranslating: true,
    localLocale: Locale(identifier: "ko-KR"),
    remoteLocale: Locale(identifier: "en-US")
)

transaction = CXTransaction(action: action)
callController.request(transaction)
```

#### 기본 통화 앱 지정 기능 지원

iOS 26부터 사용자가 iPhone 앱이나 FaceTime 외의 CallKit 기반 앱을 "기본 통화 앱"으로 설정 가능
앱에서 특정 설정을 하면, 시스템이 사용자의 전화 걸기 요청을 해당 앱에 넘김

머 예를 들면 연락처에서 전화 걸기 -> 사용자가 선택한 CallKit 기반 앱으로 연결해줌 

#### 프라이버시 데이터 공유에 대한 새로운 설명 키 추가

iOS 26부터는 사용자가 CallKit 기반 앱으로 누구와 통화했는지, 얼마나 자주 대화했는지 같은 정보가 Journal 앱의 회고 제안에 활용될 수 있습니다.

시스템이 CallKit 통화의 수신자 정보를 Journal 앱 등과 공유할 수 있기 때문에, 앱에서 명시적으로 이를 방지하고 싶으면 **SRResearchDataGeneration = NO로 설정** 가능 (SRResearchDataGeneration은 통화 중 사용자의 음성 데이터를 연구용으로 수집할 수 있는지 여부를 제어하는 info.plist임)

등등 ! 
저는 여기서 실시간 번역이 어떻게 작동하는지에 대해 더 찾아보았습니다.

## 통화 중 실시간 번역은 어떻게 작동할까? + <font color = "slateBlue">Apple Intelligence</font>

- 통화 중 "번역 켜기" 버튼을 누름
- 앱이 `CXSetTranslatingCallAction`을 시스템에 전송
- iOS가 통화 오디오 스트림을 분석함
- 온디바이스 <font color = "slateBlue">Apple Intelligence</font>가
	- 음성 -> 텍스트 (STT)
	- 언어 감지
	- 실시간 번역
- 결과 표시
	- 자막 형태로 보여주거나, 번역된 음성을 재생하거나 (설정)

```swift
import CallKit

class TranslationManager {
    private let callController = CXCallController()

    /// 실시간 번역 상태 설정
    func setLiveTranslation(for callUUID: UUID, isTranslating: Bool) {
        let action = CXSetTranslatingCallAction(call: callUUID, isTranslating: isTranslating)
        let transaction = CXTransaction(action: action)

        callController.request(transaction) { error in
            if let error = error {
                print("\(error.localizedDescription)")
            } else {
                print("\(isTranslating))")
            }
        }
    }
}
```

```swift
let manager = TranslationManager()

manager.setLiveTranslation(for: currentCallUUID, isTranslating: true)

manager.setLiveTranslation(for: currentCallUUID, isTranslating: false)
```

**생각보다 CallKit이 통화 중 실시간 번역을 하기 위한 행위로는 그냥 번역 켰/껐 버튼 제어하기(트리거) 밖에 없었습니다 ..** 왜냐하면 번역 처리, 자막 렌더링, 음성 재생 모두 시스템에서 자동 처리되어요.



이렇게 끝나기에는 아쉬우니 **Apple Intelligence**가 번역을 어떻게 하는지 알아보아요.

Apple Intelligence는 온디바이스 AI 시스템이에요. 
사용자의 텍스트, 이미지, 음성을 개인 정보 보호를 유지한 채 이해하고 생성할 수 있도록 설계된 Apple의 퍼스널 AI ! 입니다. 대부분의 모델 추론은 기기 자체에서 처리되고, 복잡한 요청은 서버로 보낼 수 있지만, 그 서버는 Apple이 암호화된 상태에서만 처리해요.


그럼 음성은 어떻게 실시간으로 번역하고 처리하냐 !

Live Translation은 Apple Intelligence의 일부인 음성 언어 처리 스택 위에서 작동합니다.

1. **음성 입력 -> STT**
	- 사용자의 실시간 입력이 마이크로 들어오면, 온디바이스 STT 모델이 바로 텍스트로 변환
	- Apple은 온디바이스에서 작동하는 자체 음성 인식 모델을 사용하며, ~~구조적으로 Whisper와 유사한 Transformer 기반 STT 모델일 것으로 **추정**~~

2. **언어 감지 (번역 전 전처리 단계)**
	- 텍스트로 바뀐 음성은 현재 언어를 자동으로 감지함
	- 예를 들어 한국어로 말하면 "ko-KR"로 인식 !
	- 혼합 언어 사용 시 구간별로 언어가 다르게 감지될 수 있음

3. **실시간 번역**
	- 감지된 언어를 기반으로, Apple은 공개되지 않은 모델을 사용하여 실시간으로 타켓 언어로 번역된 텍스트를 생성합니다.

4. **자막 또는 음성 선택**
	- 번역된 텍스트는 자막처럼 화면에 표시되거나, 번역된 음성으로 들려줄 수 있습니다. 
	- TTS 음성은 Siri와 동일한 Apple Neural Voice 기반임미다.

## References
- [CXSetTranslatingCallAction](https://developer.apple.com/documentation/callkit/cxsettranslatingcallaction)
- [CallKit](https://developer.apple.com/documentation/callkit/)
