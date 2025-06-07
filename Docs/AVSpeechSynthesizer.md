>[!question]
>GQ1. AVSpeechSynthesizer는 어떤 역할을 하며, 어떻게 사용해서 텍스트를 음성으로 읽을 수 있을까?

## Description
---
AVSpeechSynthesizer는 iOS/macOS의 AVFoundation 프레임워크에 포함된 **TTS(Text-to-Speech) 엔진**이다.
텍스트를 음성으로 읽어주는 기능을 담당하며, AVSpeechUtterance와 함께 사용하여 말할 문장을 지정하고, AVSpeechSynthesisVoice로 어떤 목소리로 읽을지를 선택할 수 있다.

## 주요 기능
---
- `speak(_:)` 메서드로 텍스트를 음성으로 읽기 시작
- `.pauseSpeaking(at:)`, `.continueSpeaking()`, `.stopSpeaking(at:)`으로 말하기 제어
- `isSpeaking`, `isPaused` 속성으로 현재 TTS 상태 확인
- `AVSpeechSynthesizerDelegate`를 활용해 상태 및 현재 읽는 단어 범위 감지
- 여러 문장을 큐로 처리 가능 → 여러 utterance를 순차적으로 읽을 수 있음

## 예시 코드
---
- 기본 사용
```Swift
import SwiftUI
import AVFoundation

struct ContentView: View {
	let synthesizer = AVSpeechSynthesizer()
	var body: some View {
		VStack {
			Button("말하셈") {
				let utterance = AVSpeechUtterance(string: "안녕하세요. 제이입니다.")
				utterance.voice = AVSpeechSynthesisVoice(language: "ko-KR")
				utterance.rate = 0.5
				synthesizer.speak(utterance)
			}
		}
	}
}
```


## 읽고 있는 문장 하이라이팅을 하려면?
---
이를 구현하려면 AVSpeechSynthesizerDelegate의 메서드 중 하나인
`speechSynthesizer(_:willSpeakRangeOfSpeechString:utterance:)`를 활용하는 것이 핵심이다. 이 메서드는 현재 읽으려는 텍스트의 **범위(NSRange)** 를 전달해주며, 일반적으로는 **단어 단위**로 호출된다. 

하이라이팅 적용은 `NSMutableAttributedString` 을 사용해 해당 문장 범위에 색상을 입히는 방식으로 구현한다. 문제는 SwiftUI의 기본 Text 뷰가 이러한 실시간 하이라이팅 처리에 적합하지 않다는 점이다. 특히 `NSAttributedString` 을 기반으로 한 동적 스타일링은 제한적이기 때문에, UIKit의 UILabel을 활용하고 이를 `UIViewRepresentable`로 SwiftUI에 통합하는 방식이 가장 일반적이고 안정적인 대안이다.

```Swift
import SwiftUI
import AVFoundation

// MARK: - TTS Manager (Delegate)
class TTSHighlighter: NSObject, ObservableObject, AVSpeechSynthesizerDelegate {
    @Published var attributedText: NSAttributedString = NSAttributedString(string: "") // 하이라이팅된 텍스트를 View로 보내기 위한 변수
    
    private let synthesizer = AVSpeechSynthesizer()
    private var fullText: String = "" // 전체 문장 (하이라이팅 범위 계산용)

    override init() {
        super.init()
        synthesizer.delegate = self
    }

    func speak(text: String) { 
        self.fullText = text
        let utterance = AVSpeechUtterance(string: text) // 속도와 음성 언어 설정
        utterance.voice = AVSpeechSynthesisVoice(language: "ko-KR")
        utterance.rate = 0.5
        
        let attrString = NSAttributedString(string: text) // 초기에는 일반 색상으로 표시
        self.attributedText = attrString
        synthesizer.speak(utterance)
    }
    
    // 현재 단어 감지해서 하이라이팅
    func speechSynthesizer(
	    _ synthesizer: AVSpeechSynthesizer, 
	    willSpeakRangeOfSpeechString characterRange: NSRange, 
	    utterance: AVSpeechUtterance
	) {
		let mutable = NSMutableAttributedString(string: fullText)
		mutable.addAttribute(
			.foregroundColor, 
			value: UIColor.label, 
			range: NSRange(location: 0, length: fullText.count) 
		) // 전체 문장 : 기본 색상으로 초기화
		mutable.addAttribute(
			.foregroundColor, 
			value: UIColor.systemOrange, 
			range: characterRange
		) // 현재 읽는 단어 범위 : 강조 색상 적용
		DispatchQueue.main.async { // UI 업데이트는 무조건 메인 쓰레드에서만!
			self.attributedText = mutable
		}
    }
}
```

```Swift
import SwiftUI
import AVFoundation

struct ContentView: View {
    @StateObject private var tts = TTSHighlighter()
    let sampleText = "안녕하세요. 이 문장은 단어별로 하이라이팅 됩니다."
    
    var body: some View {
        VStack(spacing: 32) {
            AttributedTextView(attributedString: tts.attributedText)
                .padding()
            Button("읽기 시작") {
                tts.speak(text: sampleText)
            }
            .padding()
            .background(Color.orange)
            .foregroundColor(.white)
        }
        .padding()
    }
}

struct AttributedTextView: UIViewRepresentable {
    var attributedString: NSAttributedString
    func makeUIView(context: Context) -> UILabel {
        let label = UILabel() // UIKit 뷰 생성
        label.numberOfLines = 0
        label.textAlignment = .center
        return label
    }
    
    func updateUIView(_ uiView: UILabel, context: Context) { 
        uiView.attributedText = attributedString // 업데이트
    }
}
```


>[!info] UIViewRepresentable
>SwiftUI에서 UIKit 뷰(UILabel, UITextView 등)을 쓰고 싶을 때 사용하는 중간 래퍼(Bridge) 역할을 한다.

>[!info] NSMutableAttributedString
>텍스트의 **일부 범위만 선택해서 스타일**을 적용할 수 있는 클래스이자, 그 속성을 나중에 **자유롭게 수정 가능하게 만든 수정 가능**한 문자열 

>[!info] NSRange
>텍스트 내 특정 범위(시작 위치 + 길이)를 표현하는 구조체


## Keywords
---
* AVSpeechSynthesizer
* AVSpeechUtterance
* AVSpeechSynthesisVoice

## References
---
- [AVSpeechSynthesizer – Apple Developer Docs](https://developer.apple.com/documentation/avfoundation/avspeechsynthesizer)
- [AVSpeechSynthesizerDelegate – Apple Developer Docs](https://developer.apple.com/documentation/avfoundation/avspeechsynthesizerdelegate)
- [NSMutableAttributedString – Apple Developer Docs](https://developer.apple.com/documentation/foundation/nsmutableattributedstring)
- [NSRange – Apple Developer Docs](https://developer.apple.com/documentation/foundation/nsrange)
- [UIViewRepresentable – Apple Developer Docs](https://developer.apple.com/documentation/swiftui/uiviewrepresentable)
