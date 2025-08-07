> 다양한 방식으로 음성 오디오 콘텐츠를 분석하고 분석 세션을 관리하는 **actor 클래스**

```swift
final actor SpeechAnalyzer
```

### Description
- 음성 분석 모듈(SpeechTranscriber, SpeechDetector 등)을 넣어주는 컨테이너이자 실행 관리자
- 대부분의 경우에는 **SpeechTranscriber** 하나만으로도 충분
### 역할
- 어떤 종류의 분석을 할지 정하는 **모듈(SpeechTranscriber 등) 보관소**
- 음성 데이터를 받아서 각 모듈에 전달 (`AsyncStream<AnalyzerInput>`)
- 언제 분석 시작할지, 언제 끝낼지, 중간에 멈출지 등을 **제어**

### 비동기 분석
- Analysis is **asynchronous**
- 오디오 입력 → 분석 → 결과 출력 → 세션 종료 **각각 독립된 흐름(Task)** 으로 진행됨
- 입력은 **AsyncStream**으로 실시간 공급
	- 직접 만든 `AsyncStream<AnalyzerInput>`을 통해 공급해야 함
- 결과는 **AsyncSequence**로 실시간 수신
- 한 번에 하나의 입력 시퀀스만 가능
	- 동시에 두 개의 음성 스트림을 분석하는 건 불가능
