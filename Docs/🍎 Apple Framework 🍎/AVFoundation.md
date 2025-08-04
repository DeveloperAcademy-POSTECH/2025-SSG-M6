
>[!question]
>GQ1. iOS의 AVFoundation 프레임워크는 어떤 역할을 하고, 왜 TTS를 구현할 때 사용하는 걸까?

## Description
---
AVFoundation은 Apple이 제공하는 멀티미디어 프레임워크로,
iOS, macOS 등에서 **오디오/비디오 재생, 녹음, 편집, 캡처, TTS(Text-to-Speech)** 같은 기능을 처리할 수 있게 해준다.
텍스트를 음성으로 읽어주는 TTS 기능 또한 이 프레임워크에 포함되어 있으며, AVSpeechSynthesizer 등 관련 클래스들은 모두 AVFoundation에 속한다.

## 주요 기능
---
- **오디오 재생** : 음악 효과음, 녹음 파일 등 다양한 오디오 재생을 지원 (AVAudioPlayer)
- **오디오 녹음** : 마이크 입력을 받아 파일로 저장 (AVAudioRecorder)
- **TTS(Text-to-Speech)** : - 문자열을 음성으로 읽어줌 (AVSpeechSynthesizer, AVSpeechUtterance)
- **비디오 재생** :  로컬 및 스트리밍 비디오 콘텐츠 재생 가능 (AVPlayer)
- **미디어 캡처** : 카메라/마이크 입력을 실시간으로 수집하고 처리 (AVCaptureSession, AVCaptureDevice)
- **미디어 편집**: 오디오/비디오 자르기, 조합, 필터 등 후처리 가능 (AVAsset, AVAssetExportSession)
- **오디오 이펙트/믹싱**: 실시간 음성 처리 및 필터링 지원 (AVAudioEngine)

## Keywords
---
* AVSpeechSynthesizer
* AVSpeechUtterance
* AVSpeechSynthesisVoice

## References
---
* [Apple Developer Docs: AVFoundation](https://developer.apple.com/documentation/avfoundation)
