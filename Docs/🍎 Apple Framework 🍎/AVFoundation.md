
>[!question]
>GQ1. iOS의 AVFoundation 프레임워크는 어떤 역할을 하고, 왜 TTS를 구현할 때 사용하는 걸까?
>GQ2. AVFoundation에서 제공하는 기술들은 무엇이 있나?

## Description
---
<img src = "AVFoundation.png" width = "600" >

AVFoundation은 Apple이 제공하는 멀티미디어 프레임워크로, 시간 기반의 시청각 미디어를 생성하고 재생하는 등 미디어 작업을 위한 모든 기능을 제공한다.

iOS, macOS, iPadOS, tvOS, visionOS, watchOS에서 **오디오/비디오 재생, 녹음, 편집, 캡처, TTS(Text-to-Speech)** 같은 기능을 처리할 수 있게 해준다.
텍스트를 음성으로 읽어주는 TTS 기능 또한 이 프레임워크에 포함되어 있으며, AVSpeechSynthesizer 등 관련 클래스들은 모두 AVFoundation에 속한다.

AVFoundation에서는 CoreMedia 프레임워크의 CMTime 타입을 사용한다.
이는 부동소수점의 부정확성으로 인한 디테일한 시간 작업의 문제점을 막기 위함이다.


> 🔥 AVFoundation 관련 메서드들의 사용 특징
> 
> - Swift Concurrency 기반으로 API들이 변경된 부분들이 존재.
> - enum 형태를 이용한 타입 안전성 있는 API 구조
> - Error 처리 대신 try catch를 통한 에러 핸들링의 명확화
> - Objective-C 기반에서 Swift 중심 구조로 전환


## 주요 기능
---
- **오디오 재생** : 음악 효과음, 녹음 파일 등 다양한 오디오 재생을 지원 (AVAudioPlayer)
- **오디오 녹음** : 마이크 입력을 받아 파일로 저장 (AVAudioRecorder)
- **TTS(Text-to-Speech)** : - 문자열을 음성으로 읽어줌 (AVSpeechSynthesizer, AVSpeechUtterance)
- **비디오 재생** :  로컬 및 스트리밍 비디오 콘텐츠 재생 가능 (AVPlayer)
- **미디어 캡처** : 카메라/마이크 입력을 실시간으로 수집하고 처리 (AVCaptureSession, AVCaptureDevice)
- **미디어 편집**: 오디오/비디오 자르기, 조합, 필터 등 후처리 가능 (AVAsset, AVAssetExportSession)
- **오디오 이펙트/믹싱**: 실시간 음성 처리 및 필터링 지원 (AVAudioEngine)


## 구성
---
### 📍 AVAsset

시간이 지정된 시청각 미디어를 모델링하는 객체

```swift
class AVAsset: NSObject
```

Asset은 QuickTime 동영상이나 MP3 오디오 파일과 같은 파일 기반 미디어와 HLS(HTTP Live Streaming)을 사용해 스트리밍된 미디어를 모델링한다. 즉, Asset은 균일하게 입력된 미디어 트랙을 모델링하는 하나 이상의 AVAssetTrack 인스턴스에 대한 컨테이너 객체이다.

비디오, 오디오 같은 미디어라고 불리는 것들에 대한 데이터 부분들을 '트랙'이라고 한다.
일반적으로 사용되는 트랙은 비디오, 오디오 관련이지만 캡션이나 자막, 시간 등의 트랙도 포함할 수 있다.

사이즈가 큰 Asset의 트랙 속성을 비동기적으로 로드해 Asset의 트랙을 로드할 수 있다.


<img src = "AVFoundation_AVAsset.png" width = "600" >


##### AVAsset 주요 메서드

- URL 기반 AVAsset 생성
```swift
convenience init(url: URL) // Deprecated
AVAsset(url: URL)
```

- 에셋의 전체 재생 시간 및 타이밍
```swift
// Asset의 기간(전체 길이)을 나타내는 시간 값
let duration = try await asset.load(.duration)
// Asset이 정확한 기간과 타이밍을 제공하는지 여부를 나타내는 Bool 값
let providesPreciseTiming = try await asset.load(.providesPreciseDurationAndTiming)
// 최신 라이브 스트리밍 콘텐츠에 얼마나 가깝게 재생이 진행되는지(딜레이의 정도)를 나타내는 시간 값
let minimumTimeOffset = try await asset.load(.minimumTimeOffsetFromLive)
```

- 트랙 정보 로딩 및 접근
```swift
// 모든 트랙을 비동기로 로드
let tracks = try await asset.load(.tracks)

// 특정 ID의 트랙을 로드
asset.loadTrack(withTrackID: CMPersistentTrackID) { track, error in ... }
// 미디어 타입에 따라 특정 트랙을 로드
asset.loadTracks(withMediaType: AVMediaType) { tracks, error in ... }
// 미이더 특징에 따라 특정 트랙을 로드
asset.loadTracks(withMediaCharacteristic: AVMediaCharacteristic) { tracks, error in ... }
// 새로운 트랙 추가를 위한 ID 찾기
asset.findUnusedTrackID { id, error in ... }
```

- 메타데이터 접근
```swift
// 모든 메타데이터 식별자에 대한 asset에 포함된 메타데이터 항목
static var metadata: AVAsyncProperty<Root, [AVMetadataItem]>
// 일반적인 메타데이터 식별자에 대한 asset에 포함된 메타데이터 항목
static var commonMetadata: AVAsyncProperty<Root, [AVMetadataItem]>
// asset의 생성 날짜를 나타내는 메타데이터 항목
static var creationDate: AVAsyncProperty<Root, AVMetadataItem?>
// 지정된 형식에 대한 asset에 포함된 메타데이터 항목의 배열 로드
func loadMetadata(for: AVMetadataFormat, completionHandler: ([AVMetadataItem]?, (any Error)?) -> Void)
```

- asset의 지원 여부를 확인하는 속성
```swift
// asset에 재생 가능한 콘텐츠가 포함되어 있는지 여부
static var isPlayable: AVAsyncProperty<Root, Bool>
// asset을 내보내기 할 수 있는지 여부
static var isExportable: AVAsyncProperty<Root, Bool>
// asset 리더를 사용해 미디어 데이터를 추출할 수 있는지 여부
static var isReadable: AVAsyncProperty<Root, Bool>
// asset을 미디어 구성에서 사용할 수 있는지 여부
static var isComposable: AVAsyncProperty<Root, Bool>
// asset이 AirPlay Video와 호환되는지 여부
static var isCompatibleWithAirPlayVideo: AVAsyncProperty<Root, Bool>
// 사진 앨범에 asset을 쓸 수 있는지 여부
static var isCompatibleWithSavedPhotosAlbum: AVAsyncProperty<Root, Bool>
```


### 📍 AVPlayer

플레이어의 재생 동작을 제어하기 위한 인터페이스를 제공하는 객체

``` swift
class AVPlayer: NSObject
```

플레이어는 미디어 Asset의 전반적인 playback(녹은, 녹화, 재생)을 관리하는 컨트롤러 객체이다.
즉, AVPlayerQuickTime 동영상, MP3 오디오 파일 등의 로컬 및 원격 파일 기반 미디어 및 HLS를 사용해 제공되는 미디어까지 재생할때 사용되는 플레이어이다.

해당 플레이어를 사용하면 한 번에 하나의 미디어 Asset을 재생한다.
그렇기 때문에 동일 해당 플레이어 인스턴스를 재사용할 수 있지만 한 번에 하나의 미디어만 재생 관리한다.

	한 번에 하나의 미디어 데이터만 재생할 수 있다고 언급했는데, 
	그렇다면 여러 미디어 데이터를 순서대로 재생하고 싶을때는 어떻게 해야하나?
	-> AVQueuePlayer 클래스를 사용해야 한다.

AVPlayer 자체는 비시각적인 객체로 화면에 그대로 표시할 수는 없다.
그렇기 때문에 화면에 표시하기 위해서 AVKit이나 AVPlayerLayer 방식을 통해 시각적으로 표현할 수 있다.


> ❓ HLS란?
> 앞서 언급된 것처럼 HTTP Live Streaming을 의미하며 그 역할을 아래와 같다.
> HTTP 프로토콜을 사용해 비디오 스트리밍을 제공하는 기술이다. 비디오를 작은 세그먼트로 나눠 전송하고, 네트워크 환경에 따라 최적화된 품질의 비디오를 제공하는 적응형 비트레이트 스트리밍을 지원한다.
> 주로 라이브 및 주문형 비디오 스트리밍에 사용된다.
> 
> ⁉️ 적응형 비트레이트 스트리밍(ABR)이란?
> 네트워크 상태에 따라서 최적의 비디오 품질을 선택해 끊김 없는 재생을 지원하는 스트리밍 방식이다.


##### AVPlayer 주요 메서드

- 생성 및 설정
```swift
init(url: URL)
init(playerItem: AVPlayerItem?) // 미디어를 동적으로 다루기 위해서 권장되는 방식
init()
```

- 재생 항목 관리
```swift
// 현재 재생중인 아이템
var currentItem: AVPlayerItem?
// 새 AVPlayerItem으로 교체 지원
func replaceCurrentItem(with: AVPlayerItem?)
```

- 플레이어 상태 확인
```swift
// 현재 재생 가능 상태 확인
var status: AVPlayer.Status // .readyToPlay .failed 등
var error: Error?
```

- 재생 제어
```swift
func play()
func pause()
// 현재 재생 속도
var rate: Float
// 재생 시작 시점의 기본 속도
var defaultRate: Float
```

- 재생 시간 추적
```swift
// 현재 위치 반환
func currentTime() -> CMTime

// 일정 간격마다 콜백 실행 -> UI 업데이트 등에 사용
func addPeriodicTimeObserver(
  forInterval: CMTime,
  queue: DispatchQueue?,
  using: @escaping (CMTime) -> Void
) -> Any

// 특정 시간 도달 시 콜백 실행
func addBoundaryTimeObserver(
  forTimes: [NSValue],
  queue: DispatchQueue?,
  using: @escaping () -> Void
) -> Any

// 관찰 중지
func removeTimeObserver(Any)
```

- 특정 시간 또는 날짜에 해당하는 부분으로 이동
```swift
// 특정 시간 찾고 완료시 알림
func seek(to: CMTime, completionHandler: @escaping (Bool) -> Void)
// 허용치 정도의 정확도로 지정된 시간을 찾고 완료시 알림
func seek(to: CMTime, toleranceBefore: CMTime, toleranceAfter: CMTime, completionHandler: @escaping (Bool) -> Void)
```

- 절전 및 백그라운드 설정
```swift
// 재생 중 화면 꺼짐 방지 여부
var preventsDisplaySleepDuringVideoPlayback: Bool
// 재생 중 앱 백그라운드 방지 여부
var preventsAutomaticBackgoundingDuringVideoPlayback: Bool
```



### 📍 AVPlayerItem

재생 중 Asset의 시간 및 상태 정보를 모델링하는 객체

```swift
class AVPlayerItem: NSObject
```

쉽게 말해서 시간과 그에 따른 현재 미디어의 상태 정보를 가지고 있는 객체라고 볼 수 있다.
AVAsset은 총 재생 시간이나 생성 날짜처럼 그 자체로 미디어가 가진 모든 정보가 포함되어 있다면, AVPlayerItem은 현재까지 재생된 시간 등 시간 경과에 따른 현재 상태 정보를 담고 있다.

##### AVPlayerItem 주요 메서드

- 생성
```swift
init(asset: AVAsset)
// 특정 키를 사용해 자동으로 로드할 플레이어 아이템 생성
init(asset: AVAsset, automaticallyLoadedAssetKeys: [String]?)
init(url: URL)
```

- 에셋 정보
```swift
// 플레이어 아이템에 연결된 AVAsset 정보
var asset: AVAsset
// 플레이어 아이템 트랙 객체 배열
var tracks: [AVPlayerItemTrack]
```

- 상태
```swift
var status: AVPlayerItem.Status // .unknown .readyToPlay .failed
var error: Error?
```

- 재생 준비 상태 및 버퍼
```swift
// 재생 지속 가능 여부
var isPlaybackLikelyToKeepUp: Bool
// 버퍼 비어있는지 여부
var isPlaybackBufferEmpty: Bool
// 버퍼 가득차있는지 여부
var isPlaybackBufferFull: Bool
```

- 재생 시간 추적
```swift
var duration: CMTime
func currentTime() -> CMTime
func currentDate() -> Date?
```

- 플레이어 아이템 출력 관리
```swift
// 플레이어 아이템과 관련된 출력의 배열
var outputs: [AVPlayerItemOutput]
// 저장된 플레이어 아이템 출력 객체를 추가
func add(AVPlayerItemOutput)
// 지정된 플레이어 아이템 출력 객체를 제거
func remove(AVPlayerItemOutput)
```


### 📍 AVKit

AVFoundation을 가지고 UI를 구현하기 위해서는 AVFoundation과 low-level 수준의 깊은 지식이 필요하다. 관련 UI 요소들이 UIKit 밑단에 있어서 표준화된 UI를 제공하지 않기 때문이다.

AVKit은 AVFoundation 위에 존재하고 미디어 플레이어 Interface를 쉽게 제공하는 SDK이다.
AVKit 안에 있는 AVPlayerViewController 클래스는 AVFoundation에 직접적인 접근을 제공하기 때문에 미디어 재생에 관련된 다양한 기능 구현이 가능하다. (macOS 환경에서는 AVPlayerView 클래스 사용)

보통 재생 UI를 커스텀하지 않고 주어진 UI를 그대로 사용하고 싶은 경우에 AVKit을 사용하며, 커스텀하고 싶은 경우에는 AVPlayerLayer를 사용하게 된다.


### 📍 AVPlayerLayer

동영상을 재생시킬 수 있는 layer로 CALayer의 서브클래스로, player 객체의 시각적 내용을 표시하는 객체

AVPlayer를 생성자로 하여 AVPlayerLayer 인스턴시를 만들고, 표현하려는 뷰에 layer.addSublayer로 삽입해 사용한다.


> 💡 결과적으로, 재생 클래스들은 아래와 같은 관계를 가진다.
> AVPlayer 객체가 AVPlayerItem을 이용하고, AVPlayerItem이 AVAsset을 사용하는 구조.
> 
> 1. AVAsset : 미디어 파일의 실제 콘텐츠
> 2. AVPlayerItem : AVAsset을 감싼 재생 단위의 객체
> 3. AVPlayer : 실제 재생을 담당
> 4. APPlayerLayer / AVKit : 비디오를 화면에 렌더링


<img src = "AVFoundation_Flow.png" width = "600">


### ▶️ 동영상 재생 예시 코드

- UIKit 기반, AVPlayerLayer 사용해 뷰 적용
```swift
import UIKit
import AVFoundation

class VideoPlayerViewController: UIViewController {
    private var player: AVPlayer!
    private var playerLayer: AVPlayerLayer!

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .black
        
        // 영상 URL 설정
        guard let url = Bundle.main.url(
	        forResource: "sample", 
	        withExtension: "mp4"
		) else { return }

        // AVPlayer 생성
        player = AVPlayer(url: url)

        // AVPlayerLayer 생성 및 설정
        playerLayer = AVPlayerLayer(player: player)
        playerLayer.frame = view.bounds
        playerLayer.videoGravity = .resizeAspect

        // View에 삽입
        view.layer.addSublayer(playerLayer)

        // 재생 시작
        player.play()
    }

    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        playerLayer.frame = view.bounds
    }
}

```


-  SwiftUI 기반, AVKit에서 제공하는 VideoPlayer 사용해 뷰 적용
```swift
import SwiftUI
import AVKit

struct VideoPlayerView: View {
    private let player = AVPlayer(
	    url: Bundle.main.url(
		    forResource: "sample", 
		    withExtension: "mp4"
		)!
    )

    var body: some View {
        VideoPlayer(player: player)
            .onAppear {
                player.play()
            }
            .ignoresSafeArea()
    }
}

```


---

### 📍 AVCaptureSession

입력(카메라, 마이크 등)과 출력(파일, 화면, 버퍼 등)을 연결해주는 중심 객체

```swift
class AVCaptureSession: NSObject
```

AVCaptureSession은 캡처 활동의 허브 역할을 하며, 다양한 입력 장치와 출력 방식을 설정하고 관리한다. 
하나의 세션 안에 여러 입력/출력을 설정할 수 있으며, 이를 바탕으로 실시간 카메라 프리뷰, 녹화, 사진 캡처 등이 가능하다.

##### 주요 메서드 및 프로퍼티

- 세션 구성
```swift
func beginConfiguration()
func commitConfiguration()
```

- 세션 시작/중지
```swift
func startRunning()
func stopRunning()
var isRunning: Bool
```

- 입력 및 출력 추가/제거
```swift
func canAddInput(AVCaptureInput) -> Bool
func addInput(AVCaptureInput)
func removeInput(AVCaptureInput)

func canAddOutput(AVCaptureOutput) -> Bool
func addOutput(AVCaptureOutput)
func removeOutput(AVCaptureOutput)
```


### 📍 AVCaptureDevice

카메라나 마이크와 같은 실제 물리 장치를 제어하는 객체

```swift
class AVCaptureDevice
```

AVCaptureDevice는 사용자가 연결하는 AVCaptureSession을 캡처하기 위한 미디어 데이터를 제공한다. 
개별 장치는 특정 유형의 미디어 스트림을 하나 이상 제공할 수 있다.
AVCaptureDevice 인스턴스를 직접 생성하는 대신 AVCaptureDevice.DiscoverySessiondefault( : for: position: )을 사용하거나 메서드를 호출해 인스턴스를 가져와야한다.
##### 주요 메서드 및 프로퍼티

- 디바이스 찾기
```swift
AVCaptureDevice.default(for: .video)
AVCaptureDevice.default(for: .audio)
// 특정 검색 기준과 일치하는 캡션 장치를 찾는 객체
AVCaptureDevice.DiscoverySession
```

- 설정 제어
```swift
func lockForConfiguration() throws
func unlockForConfiguration()
var isFocusModeSupported: Bool
var isTorchAvailable: Bool
var torchMode: AVCaptureDevice.TorchMode
```


### 📍 AVCapturePhotoOutput

정지 이미지(사진) 캡처에 사용하는 출력 클래스

```swift
class AVCapturePhotoOutput
```

AVCapturePhotoOutput은 정적인 사진 관련 캡처 워크플로우를 위한 인터페이스를 제공한다.
기본적인 정지 이미지 캡처 이외에도 사진 출력 기능은 RAW 포맷 캡처, 라이브 포토 등을 지원한다.
RAW 포맷 DNG 파일, JPEG 파일 등 다양한 포맷과 코덱으로 촬영한 사진을 출력할 수 있다

- 특정 캡처에 대한 기능 및 설정 선택 객체 생성
```swift
class AVCapturePhotoSettings
```

- 사진 캡처 요청
```swift
func capturePhoto(with: AVCapturePhotoSettings, delegate: AVCapturePhotoCaptureDelegate)
```


### 📍 AVCaptureMovieFileOutput

동영상 파일로 저장하는 캡처 출력 클래스

```swift
class AVCaptureMovieFileOutput: AVCaptureFileOutput
```

- 녹화 시작/중지
```swift
func startRecording(to: URL, recordingDelegate: AVCaptureFileOutputRecordingDelegate)
func stopRecording()
```


### 📍 AVCaptureVideoDataOutput / AVCaptureAudioDataOutput

실시간 비디오/오디오 프레임을 delegate를 통해 처리하는 출력 객체

- output에 대한 설정을 포함하는 딕셔너리 - 표준 키 값과 상수를 사용해 비디오 처리 설정 구성
```swift
var videoSettings: [String : Any]!
```

- 프레임 Delegate 설정
```swift
videoOutput.setSampleBufferDelegate(_:queue:)
audioOutput.setSampleBufferDelegate(_:queue:)
```

- 델리게이트
```swift
func captureOutput(_: AVCaptureOutput, didOutput: CMSampleBuffer, from: AVCaptureConnection)
```


### ▶️ 캡처 기능 예시 코드

- UIKit 기반
```swift
import UIKit
import AVFoundation

class CameraViewController: UIViewController {
	// 캡처 세션 생성
    let session = AVCaptureSession()
    let previewLayer = AVCaptureVideoPreviewLayer()

    override func viewDidLoad() {
        super.viewDidLoad()

		// 세션 설정
        guard let camera = AVCaptureDevice.default(for: .video),
              let input = try? AVCaptureDeviceInput(device: camera)
        else { return }

        session.addInput(input)

		// 미리보기 레이어 뷰 설정
        previewLayer.session = session
        previewLayer.frame = view.bounds
        view.layer.addSublayer(previewLayer)

        session.startRunning()
    }
}
```

- SwiftUI 기반
```swift
import SwiftUI
import AVFoundation

struct CameraPreviewView: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        let vc = UIViewController()
        let session = AVCaptureSession()

        guard let camera = AVCaptureDevice.default(for: .video),
              let input = try? AVCaptureDeviceInput(device: camera)
        else { return vc }

        session.addInput(input)

        let preview = AVCaptureVideoPreviewLayer(session: session)
        preview.frame = UIScreen.main.bounds
        preview.videoGravity = .resizeAspectFill

        vc.view.layer.addSublayer(preview)
        session.startRunning()

        return vc
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {}
}
```



---

### 📍 AVAssetExportSession

AVAsset을 새로운 파일로 내보내기 위한 클래스

```swift
class AVAssetExportSession: NSObject
```

영상 자르기, 변환, 리사이징, 포맷 변경 등 다양한 후처리에 사용한다.

- Export 세션 생성
```swift
AVAssetExportSession(asset: AVAsset, presetName: String)
```

- 편집 범위 설정
```swift
var timeRange: CMTimeRange
```

- 출력 설정
```swift
var outputURL: URL
var outputFileType: AVFileType
```

- 내보내기 시작
```swift
func exportAsynchronously(completionHandler: @escaping () -> Void)
var status: AVAssetExportSession.Status
var error: Error?
```


### 📍 AVMutableComposition / AVMutableCompositionTrack

여러 개의 AVAsset을 하나로 결합(합치기)하거나 편집(자르기)할 수 있는 구성 클래스

```swift
class AVMutableComposition: AVComposition
class AVMutableCompositionTrack: AVCompositionTrack
```


- 구성 트랙 추가
```swift
func addMutableTrack(withMediaType: AVMediaType, preferredTrackID: CMPersistentTrackID) -> AVMutableCompositionTrack?
```

- 트랙에 삽입
```swift
func insertTimeRange(CMTimeRange, of: AVAssetTrack, at: CMTime) throws
```


### 📍 AVVideoComposition

필터, 오버레이, 트랜지션 등의 시각적 효과를 적용할 수 있는 클래스

```swift
class AVMutableVideoComposition: AVVideoComposition
```

- 타임라인 기반의 커스텀 효과 적용
```swift
videoComposition.animationTool
videoComposition.customVideoCompositorClass
```


### ▶️ 영상 편집 예시 코드

- UIKit 기반
```swift
import UIKit
import AVFoundation

class TrimViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        let button = UIButton(type: .system)
        button.setTitle("Trim Video", for: .normal)
        button.addTarget(self, action: #selector(trimVideoAction), for: .touchUpInside)

        button.frame = CGRect(x: 100, y: 200, width: 200, height: 50)
        view.addSubview(button)
    }

    @objc func trimVideoAction() {
        guard let inputURL = Bundle.main.url(forResource: "sample", withExtension: "mp4") else { return }

        let start = CMTime(seconds: 2, preferredTimescale: 600)
        let end = CMTime(seconds: 5, preferredTimescale: 600)

        trimVideo(sourceURL: inputURL, start: start, end: end) { outputURL in
            if let url = outputURL {
                print("Trimmed video saved at: \(url)")
            } else {
                print("Failed to trim video")
            }
        }
    }
}
```

- SwiftUI 기반
```swift
import SwiftUI
import AVFoundation

struct VideoTrimView: View {
    var body: some View {
        Button("Trim Video") {
            guard let inputURL = Bundle.main.url(forResource: "sample", withExtension: "mp4") else { return }
            let start = CMTime(seconds: 2, preferredTimescale: 600)
            let end = CMTime(seconds: 5, preferredTimescale: 600)

            trimVideo(sourceURL: inputURL, start: start, end: end) { outputURL in
                if let url = outputURL {
                    print("Trimmed video saved at: \(url)")
                }
            }
        }
    }
}
```

```swift
import AVFoundation

// 지정한 시작과 끝 시간을 기준으로 트리밍 기능을 동작하는 함수
func trimVideo(sourceURL: URL, start: CMTime, end: CMTime, completion: @escaping (URL?) -> Void) {
	// 원본 영상 로드
    let asset = AVAsset(url: sourceURL)
    let outputURL = FileManager.default.temporaryDirectory.appendingPathComponent("trimmed.mov")

	// export 세션 생성
    guard let export = AVAssetExportSession(asset: asset, presetName: AVAssetExportPresetHighestQuality) else {
        completion(nil)
        return
    }

	// 출력 위치와 포맷 등 설정
    export.outputURL = outputURL
    export.outputFileType = .mov
    export.timeRange = CMTimeRangeFromTimeToTime(start: start, end: end)
    
    export.exportAsynchronously {
	    // 작업 완료 후 결과 전달
        completion(export.status == .completed ? outputURL : nil)
    }
}
```



---

### 📍 AVAudioRecorder

오디오 데이터를 파일에 **녹음**하는 객체

```swift
class AVAudioRecorder: NSObject
```

시스템의 입력 장치를 통해 오디오 녹음, 사용자가 중지하거나 지정된 시간이 될 때까지 녹음, 녹음 일시 정지 및 재개, 녹음 수준 측정 데이터에 액세스를 가능하게 한다. 만약 녹음된 오디오에 신호 처리를 적용하거나 조금 더 고급화된 녹음 기능을 사용하고 싶다면 AVAudioEngine를 사용할 수 있다.

##### AVAudioRecorder 주요 메서드

- 오디오 파일 생성 및 녹음을 위한 시스템 준비
```swift
func prepareToRecord() -> Bool
```

- 오디오 녹음 시작 및 재개
```swift
func record() -> Bool
```

- 특정 시간부터 오디오 녹음 시작
```swift
func record(atTime: TimeInterval) -> Bool
```

- 표시된 시간 동안 오디오 녹음
```swift
func record(forDuration: TimeInterval) -> Bool
```

- 표시된 기간 동안 특정 시간부터 오디오 녹음
```swift\
func record(atTime: TimeInterval, forDuration: TimeInterval) -> Bool
```

- 오디오 녹음 일시 중지
```swift
func pause()
```

- 오디오 녹음 중지
```swift
func stop()
```

- 녹음된 오디오 파일 삭제
```swift
func deleteRecording() -> Bool
```

- 녹음 중 여부 판별을 위한 변수
```swift
var isRecording: Bool
```


### 📍 AVAudioPlayer
파일이나 버퍼에서 오디오 데이터를 **재생**하는 객체

```swift
class AVAudioPlayer: NSObject
```

파일이나 버퍼에서 원하는 기간의 오디오를 재생하고, 재생되는 오디오의 볼륨/패닝/속도 및 반복 동작 등의 제어하며, 재생 수준 측정 데이터에 액세스하고 여러 플레이어의 재생을 동기화하여 여러 사운드를 동시에 재생할 수 있다. 녹음과 마찬가지로 스트리밍 재생 등의 고급 기능을 사용하려면 AVAudioEngine을 사용할 수 있다.

보통 실시간으로 스트리밍하는 오디오는 AVPlayer를 이용하며, 직접 레코딩을 통해 녹음한 로컬 파일의 재생들은 AVAudioPlayer를 이용할 수 있다.

##### AVAudioPlayer 주요 메서드

- 오디오 재생을 위해 플레이어 준비
```swift
func prepareToPlay() -> Bool
```

- 오디오 비동기적으로 재생
```swift
func play() -> Bool
```

- 오디로를 지정된 지점에서 시작해 비동기적 재생
```swift
func play(atTime: TimeInterval) -> Bool
```

- 오디오 재생 일시 정지
```swift
func pause()
```

- 오디오 재생 중지
```swift
func stop()
```

- 오디오가 현재 재생되고 있는지에 대한 여부 변수
```swift
var isPlaying: Bool
```


### ▶️ 오디오 녹음 및 재생 예시 코드

- UIKit 기반
```swift
import UIKit
import AVFoundation

class AudioViewController: UIViewController {
    var recorder: AVAudioRecorder!
    var player: AVAudioPlayer!

	// 녹음 및 재생에 사용할 파일 경로
    let url = FileManager.default
        .temporaryDirectory
        .appendingPathComponent("record.m4a")

    override func viewDidLoad() {
        super.viewDidLoad()

		// 녹음 설정
        let settings = [
            AVFormatIDKey: kAudioFormatMPEG4AAC,
            AVSampleRateKey: 12000,
            AVNumberOfChannelsKey: 1,
            AVEncoderAudioQualityKey: AVAudioQuality.high.rawValue
        ]

		// 오디오 세션 설정
        try? AVAudioSession.sharedInstance().setCategory(.playAndRecord)
        try? AVAudioSession.sharedInstance().setActive(true)

        recorder = try? AVAudioRecorder(url: url, settings: settings)
    }

    func startRecording() {
        recorder.record()
    }

    func stopRecording() {
        recorder.stop()
    }

    func playRecording() {
        player = try? AVAudioPlayer(contentsOf: url)
        player.play()
    }
}
```

- SwiftUI 기반
```swift
import SwiftUI
import AVFoundation

struct AudioRecorderView: View {
    @State private var recorder: AVAudioRecorder?
    @State private var player: AVAudioPlayer?

    let url = FileManager.default
        .temporaryDirectory
        .appendingPathComponent("record.m4a")

    var body: some View {
        VStack(spacing: 20) {
            Button("Start Recording") {
                let settings = [
                    AVFormatIDKey: kAudioFormatMPEG4AAC,
                    AVSampleRateKey: 12000,
                    AVNumberOfChannelsKey: 1,
                    AVEncoderAudioQualityKey: AVAudioQuality.high.rawValue
                ]
                try? AVAudioSession.sharedInstance().setCategory(.playAndRecord)
                try? AVAudioSession.sharedInstance().setActive(true)
                recorder = try? AVAudioRecorder(url: url, settings: settings)
                recorder?.record()
            }

            Button("Stop") {
                recorder?.stop()
            }

            Button("Play") {
                player = try? AVAudioPlayer(contentsOf: url)
                player?.play()
            }
        }
        .padding()
    }
}
```



---

### 📍 권한 설정

동영상과 오디오 관련 기능을 사용할 때, 카메라 및 마이크 접근 권한 요청이 반드시 필요하다.
아래와 같은 방식으로 권한 요청할 수 있다.

- Info.plist 설정 추가
```xml
<key>NSCameraUsageDescription</key>
<string>이 앱은 동영상 촬영을 위해 카메라 접근 권한이 필요합니다.</string>

<key>NSMicrophoneUsageDescription</key>
<string>이 앱은 동영상 녹음 시 마이크 접근 권한이 필요합니다.</string>
```

- 권한 요청 코드
```swift
// 카메라 접근 권한 요청
AVCaptureDevice.requestAccess(for: .video) { granted in
    if granted {
        print("카메라 접근 허용됨")
    } else {
        print("카메라 접근 거부됨")
    }
}

// 마이크 접근 권한 요청
AVCaptureDevice.requestAccess(for: .audio) { granted in
    if granted {
        print("마이크 접근 허용됨")
    } else {
        print("마이크 접근 거부됨")
    }
}
```




## Keywords
---
* AVSpeechSynthesizer
* AVSpeechUtterance
* AVSpeechSynthesisVoice

## References
---
* [Apple Developer Docs: AVFoundation](https://developer.apple.com/documentation/avfoundation)
* [Apple Developer Docs: Audio and Video - AVFoundation](https://developer.apple.com/av-foundation/)
* [Apple Developer Documentation Archive: AVFoundation](https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40010188-CH1-SW3)
* [Blog - AVFoundation](https://green1229.tistory.com/400)