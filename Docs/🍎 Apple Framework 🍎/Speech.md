>[!question]
>GQ1. STT 어떻게 할까?
>GQ1. Speech 프레임워크에 뭐 있을까?

## Description
- Perform **speech recognition** on live or prerecorded audio, and receive **transcriptions**, alternative interpretations, and **confidence levels** of the results.
- 실시간 또는 녹음된 오디오에 대해 음성 인식을 수행하고, **텍스트 변환 결과**와 함께 대안 해석 및 **신뢰도**(confidence level)를 제공한다.
- Speech 프레임워크를 사용하여 **오디오를 텍스트로 변환할 수 있다. (speech-to-text)**

### 주요 기능
[[SpeechAnalyzer]] 
- 여러 방식으로 **음성 오디오 콘텐츠를 분석**하고 분석 세션을 관리
[[AssetInventory]] 
- 각 모듈이 작동하기 위해 필요한 언어 모델 등의 **리소스**를 설치,다운로드,해제하는 클래스
[[SpeechTranscriber]] 
- **자연스러운 대화체를 실시간으로 텍스트**로 바꾸는 데 특화된 모듈. 최신 디바이스와 가장 잘 호환됨
[[DictationTranscriber]]
- 시스템 키보드의 **받아쓰기**와 비슷한 방식으로 동작하는 모듈이며, 구형 디바이스에서도 사용 가능하도록 만들어짐
[[SpeechDetector]]
- 사용자가 **실제로 말을 하는지 아닌지**를 감지하는 데 사용됨. 음성이 들어오기 전까지 기다리거나 말이 끝났는지를 감지할 때 활용


### 사용 방법
1. 분석 모듈 생성 및 구성
	- **SpeechTranscriber**
2. 필요한 리소스 설치 확인
	- **AssetInventory**
3. 오디오 입력 스트림 생성
4. 분석기 생성 및 설정
	- **SpeechAnalyzer**
5. 오디오 공급
6. 분석 시작
7. 결과 처리
8. 분석 종료

- 공식문서 간단 예시
```swift
import Speech
import AVFoundation

// 1. 모듈 설정
guard let locale = SpeechTranscriber.supportedLocale(equivalentTo: Locale.current) else { /* 언어 미지원 처리 */ }
let transcriber = SpeechTranscriber(
    locale: Locale(identifier: "ko-KR"),
    transcriptionOptions: [],
    reportingOptions: [.volatileResults],
    attributeOptions: [.audioTimeRange]
)

// 2. 리소스 설치
if let request = try await AssetInventory.assetInstallationRequest(supporting: [transcriber]) {
    try await request.downloadAndInstall()
}

// 3. 오디오 입력 스트림 생성
let (inputSequence, inputBuilder) = AsyncStream.makeStream(of: AnalyzerInput.self)

// 4. 분석기 생성
let format = await SpeechAnalyzer.bestAvailableAudioFormat(compatibleWith: [transcriber])
let analyzer = SpeechAnalyzer(modules: [transcriber])

// 5. 오디오 공급
Task {
    while /* 오디오 남아 있음 */ {
        let pcmBuffer = /* 변환된 AVAudioPCMBuffer */
        inputBuilder.yield(AnalyzerInput(buffer: pcmBuffer))
    }
    inputBuilder.finish()
}

// 6. 분석 시작
let lastSampleTime = try await analyzer.analyzeSequence(inputSequence)

// 7. 결과 처리
Task {
    for try await result in transcriber.results {
        let best = result.text
        print(String(best.characters)) // 텍스트 출력
    }
}

// 8. 분석 종료
if let lastSampleTime {
    try await analyzer.finalizeAndFinish(through: lastSampleTime)
} else {
    try analyzer.cancelAndFinishNow()
}
```

- 실사용 간단 예시
```swift
import AVFoundation
import Speech
import SwiftUI

@Observable
final class SpokenWordTranscriber: Sendable {
    private var inputSequence: AsyncStream<AnalyzerInput>?
    private var inputBuilder: AsyncStream<AnalyzerInput>.Continuation?
    private var transcriber: SpeechTranscriber?
    private var analyzer: SpeechAnalyzer?
    private var recognizerTask: Task<(), Error>?

    var analyzerFormat: AVAudioFormat? // 오디오 포맷
    var converter = BufferConverter() // 포맷 변환기

    var finalizedTranscript: AttributedString = "" // 최종 텍스트

    // MARK: - 분석 초기화 및 시작
    func setUp() async throws {
        let localeKR = Locale(languageCode: "ko_KR")

        // 분석 모듈 생성 및 구성
        transcriber = SpeechTranscriber(
            locale: localeKR,
            transcriptionOptions: [],
            reportingOptions: [.volatileResults],
            attributeOptions: [.audioTimeRange]
        )
        guard let transcriber else { return }

        // 필요한 리소스 설치 확인
        if let downloader = try? await AssetInventory.assetInstallationRequest(supporting: [transcriber]) {
            try? await downloader.downloadAndInstall()
        }

        // 오디오 포맷 설정
        self.analyzerFormat = await SpeechAnalyzer.bestAvailableAudioFormat(compatibleWith: [transcriber])

        // 오디오 입력 스트림 생성
        (inputSequence, inputBuilder) = AsyncStream.makeStream()
        guard let inputSequence else { return }

        // 분석기 생성 및 설정
        analyzer = SpeechAnalyzer(modules: [transcriber])

        // 텍스트 처리
        recognizerTask = Task {
            for try await result in transcriber.results {
                if result.isFinal {
                    finalizedTranscript += result.text
                    print("텍스트: \(String(result.text.characters))")
                }
            }
        }

        // 분석 시작
        try await analyzer?.start(inputSequence: inputSequence)
    }

    func streamAudioToTranscriber(_ buffer: AVAudioPCMBuffer) async throws {
        guard let inputBuilder, let analyzerFormat else { return }
        let converted = try converter.convertBuffer(buffer, to: analyzerFormat)
        inputBuilder.yield(AnalyzerInput(buffer: converted))
    }

    func finishTranscribing() async throws {
        inputBuilder?.finish()
        try await analyzer?.finalizeAndFinishThroughEndOfInput()
        recognizerTask?.cancel()
        recognizerTask = nil
    }
}

// MARK: - 마이크 입력 처리 예시

final class LiveSTTController {
    let engine = AVAudioEngine()
    let transcriber = SpokenWordTranscriber()

    func start() {
        Task {
            try? await transcriber.setUp()
            startMicStream()
        }
    }

    func stop() {
        Task {
            try? await transcriber.finishTranscribing()
            stopMicStream()
        }
    }

    func startMicStream() {
        let inputNode = engine.inputNode
        let format = inputNode.outputFormat(forBus: 0)
        
        inputNode.installTap(onBus: 0, bufferSize: 1024, format: format) { buffer, _ in
            Task {
                try? await self.transcriber.streamAudioToTranscriber(buffer)
            }
        }
        try? engine.start()
    }

    func stopMicStream() {
        engine.inputNode.removeTap(onBus: 0)
        engine.stop()
    }
}

// MARK: - 사용 예시
// let controller = LiveSTTController()
// controller.start()
// ... 음성 인식 중 ...
// controller.stop()

```