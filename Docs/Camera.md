>[!question]
>GQ1. Swift에서 Camera를 사용하는 방식, 구조가 궁금해요.
## Description
- Swift, iOS에서 카메라 디바이스를 호출하고 처리, UI에 보여주는 과정이 궁금해요.

## 주요 기능
### 카메라 Device 호출
```swift
AVCaptureDevice.DeviceType.builtInWideAngleCamera       // 기본 카메라 (전/후방)
AVCaptureDevice.DeviceType.builtInUltraWideCamera       // 울트라 와이드 카메라 (후방)
AVCaptureDevice.DeviceType.builtInTelephotoCamera       // 망원 카메라 (후방)
AVCaptureDevice.DeviceType.builtInDualCamera            // 듀얼 카메라 (후방)
AVCaptureDevice.DeviceType.builtInTripleCamera          // 트리플 카메라 (후방)
AVCaptureDevice.DeviceType.builtInDualWideCamera        // 듀얼 와이드 (후방)
AVCaptureDevice.DeviceType.builtInTrueDepthCamera       // TrueDepth 카메라 (전면, Face ID용)
```
##### Capture Session 활성화
캡쳐 세션이 뭐냐면, 디바이스에서 신호를 받아 앱 내에 데이터로 보내는 세션인데요.
###### **1.** Input 설정 (카메라, 마이크 등)
어떤 장치를 입력으로 쓸지
여러 장치를 동시에 사용하는 **멀티캠 구성**도 가능 (iOS 13+)
```
let deviceInput = try AVCaptureDeviceInput(device: device)
session.addInput(deviceInput)
```

###### **2. Output 설정 (사진, 영상, 실시간 처리)
- AVCapturePhotoOutput: 사진 촬영용
- AVCaptureMovieFileOutput: 파일 저장용
- AVCaptureVideoDataOutput: 실시간 영상 처리용 (ML 등)

```
let photoOutput = AVCapturePhotoOutput()
session.addOutput(photoOutput)
```

이런 식으로 할 수 있구요.

###### **3.** Session Preset (해상도 프리셋)
해상도 및 품질 수준을 정의. 세션 성능에 직접 영향.
```
session.sessionPreset = .high // 또는 .photo, .medium, .low 등
```

대표 프리셋:
```
.photo            // 사진 품질 우선 (4096x2160까지 가능)
.high             // 고화질 비디오 (1920x1080)
.medium           // 보통 품질 (640x480)
.low              // 저화질 (352x288)
.inputPriority    // iOS 15+ 입력 장치가 우선
```

해상도 등등의 세팅도 미리 가능합니다.

### 사진찍기 기능
카메라관리 객체에 AVCapturePhotoCaptureDelegate 프로토콜을 위임하고, photoOutput 함수를 사용합니다.

## 코드 예시
```swift
import SwiftUI
import AVFoundation

// MARK: - 1. 메인 앱 진입점
@main
struct SimpleCameraApp: App {
    var body: some Scene {
        WindowGroup {
            CameraView()
        }
    }
}

// MARK: - 2. 카메라 뷰 (메인 화면)
struct CameraView: View {
    @State private var camera = CameraController()
    
    var body: some View {
        ZStack {
            // 2-1. 카메라 프리뷰 화면
            CameraPreviewView(session: camera.session)
                .ignoresSafeArea()
            
            // 2-2. 촬영 버튼
            VStack {
                Spacer()
                Button("📸 사진 촬영") {
                    camera.takePhoto()
                }
                .font(.title2)
                .padding()
                .background(Color.white)
                .cornerRadius(50)
                .padding(.bottom, 50)
            }
        }
        .onAppear {
            camera.start()
        }
    }
}

// MARK: - 3. 카메라 컨트롤러 (카메라 로직 관리)
@MainActor
@Observable
class CameraController: NSObject {
    
    // 3-1. 카메라 세션 (카메라와 연결하는 통로)
    let session = AVCaptureSession()
    
    // 3-2. 사진 출력 객체
    private let photoOutput = AVCapturePhotoOutput()
    
    override init() {
        super.init()
        setupCamera()
    }
    
    // 3-3. 카메라 설정 함수
    private func setupCamera() {
        // 3-3-1. 카메라 디바이스 가져오기
        guard let camera = AVCaptureDevice.default(for: .video) else {
            print("❌ 카메라를 찾을 수 없습니다")
            return
        }
        
        // 3-3-2. 카메라 입력 만들기
        guard let input = try? AVCaptureDeviceInput(device: camera) else {
            print("❌ 카메라 입력을 만들 수 없습니다")
            return
        }
        
        // 3-3-3. 세션에 입력과 출력 연결
        if session.canAddInput(input) {
            session.addInput(input)
            print("✅ 카메라 입력 연결 완료")
        }
        
        if session.canAddOutput(photoOutput) {
            session.addOutput(photoOutput)
            print("✅ 사진 출력 연결 완료")
        }
    }
    
    // 3-4. 카메라 시작 함수
    func start() {
        // 3-4-1. 카메라 권한 요청
        AVCaptureDevice.requestAccess(for: .video) { allowed in
            if allowed {
                DispatchQueue.main.async {
                    self.session.startRunning()
                    print("✅ 카메라 시작됨")
                }
            } else {
                print("❌ 카메라 권한이 거부되었습니다")
            }
        }
    }
    
    // 3-5. 사진 촬영 함수
    func takePhoto() {
        let settings = AVCapturePhotoSettings()
        photoOutput.capturePhoto(with: settings, delegate: self)
        print("📸 사진 촬영 중...")
    }
}

// MARK: - 4. 사진 촬영 완료 처리
extension CameraController: AVCapturePhotoCaptureDelegate {
    
    // 4-1. 사진 촬영이 완료되면 호출되는 함수
    func photoOutput(_ output: AVCapturePhotoOutput,
                    didFinishProcessingPhoto photo: AVCapturePhoto,
                    error: Error?) {
        
        if let error = error {
            print("❌ 사진 촬영 실패: \(error)")
            return
        }
        
        // 4-2. 사진 데이터를 이미지로 변환
        guard let imageData = photo.fileDataRepresentation(),
              let image = UIImage(data: imageData) else {
            print("❌ 이미지 변환 실패")
            return
        }
        
        // 4-3. 사진첩에 저장
        UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil)
        print("✅ 사진이 저장되었습니다!")
    }
}

// MARK: - 5. 카메라 화면 표시 (UIKit을 SwiftUI에서 사용)
struct CameraPreviewView: UIViewRepresentable {
    let session: AVCaptureSession
    
    // 5-1. UIKit 뷰 만들기
    func makeUIView(context: Context) -> UIView {
        let view = UIView()
        
        // 5-2. 카메라 화면을 보여주는 레이어 만들기
        let previewLayer = AVCaptureVideoPreviewLayer(session: session)
        previewLayer.videoGravity = .resizeAspectFill
        
        view.layer.addSublayer(previewLayer)
        
        // 5-3. 화면 크기에 맞게 조정
        DispatchQueue.main.async {
            previewLayer.frame = view.bounds
        }
        
        return view
    }
    
    // 5-4. 뷰 업데이트 (필요시)
    func updateUIView(_ uiView: UIView, context: Context) {
        if let layer = uiView.layer.sublayers?.first as? AVCaptureVideoPreviewLayer {
            layer.frame = uiView.bounds
        }
    }
}
```

## Keywords
+ [[AVFoundation]]

## References
- https://developer.apple.com/documentation/avfoundation/avcam-building-a-camera-app