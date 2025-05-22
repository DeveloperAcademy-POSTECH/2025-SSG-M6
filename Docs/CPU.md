>[!question]
>GQ1. CPU의 역할이 뭔가요?
>GQ2. RAM, 저장장치와 어떤 상호작용을 하나요?

## Description
CPU = 아이폰 & 맥 속 “두뇌 칩”

Swift 코드를 받아 “계산하고, 결정하고, 지시”합니다.

화면에 글자를 그릴까? 네트워크로 데이터를 받을까? 👉 전부 CPU가 판단 후 다른 하드웨어([[RAM]]·[[GPU]]·[[SSD]])에 업무를 분배합니다.

## 주요 기능

| **상황**                 | **CPU가 하는 일**         | **체감하는 결과**              |
| ---------------------- | --------------------- | ------------------------ |
| for _ in 0..<10_000 루프 | 덧셈·곱셈 명령어를 초당 수억 번 실행 | 프레임 드롭** or **부드러운 애니메이션 |
| Task { await fetch() } | 스레드 스케줄링·컨텍스트 스위칭     | 메인 스레드 안 멈춤              |
| @State 값 변경            | 새 UI 트리 diff 계산       | 즉각 화면 업데이트               |
| RealityKit/Metal       | 그리기 명령을 GPU로 전달       | 발열·배터리 소모 변화             |

## 코드 예시
#### 상황 1. 루프가 UI를 끊는다 vs 안 끊는다

```
// ❌ 메인 스레드 독점 → 프레임 드롭
for _ in 0..<10_000 {
    heavyMath()          // 덧셈·곱셈 등 CPU 바쁨
}

// ✅ 백그라운드에서 계산
Task.detached {          // 다른 스레드
    for _ in 0..<10_000 { heavyMath() }
}
```

핵심: 메인(UI) 스레드에 과부하를 주면 화면이 끊긴다.

---

#### 상황 2. 네트워크 fetch를 기다리는 동안 UI 멈추지 않기

```
struct ContentView: View {
    @State private var text = "Loading…"

    var body: some View {
        Text(text)
            .task {                       // 자동 비동기
                text = try await fetchQuote()
            }
    }
}

func fetchQuote() async throws -> String {
    let url = URL(string: "https://api.quotable.io/random")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(Quote.self, from: data).content
}
```

_핵심_: await가 있는 동안 CPU는 다른 일 하러 갔다가, 데이터 오면 다시 이어서 실행.

---

#### **상황 3.** @State값 변경 → 새 UI 트리 diff 계산

```
struct CounterView: View {
    @State private var n = 0

    var body: some View {
        Button("Tap: \(n)") { n += 1 }     // 상태 변경
            .animation(.default, value: n) // diff 결과 애니메이션
    }
}
```

_핵심_: 값이 바뀌면 SwiftUI가 **CPU로 diff 계산** 후, 바뀐 부분만 그리라고 GPU에 지시.

---

#### 상황 4. RealityKit/Metal로 GPU에 일 넘기기

```
import RealityKit
import SwiftUI

struct ARViewContainer: UIViewRepresentable {
    func makeUIView(context: Context) -> ARView {
        let arView = ARView(frame: .zero)
        let box = ModelEntity(mesh: .generateBox(size: 0.1))
        let anchor = AnchorEntity(world: .zero)
        anchor.addChild(box)
        arView.scene.anchors.append(anchor)   // ✅ CPU→GPU draw call
        return arView
    }
    func updateUIView(_ uiView: ARView, context: Context) {}
}
```

_핵심_: RealityKit가 **CPU에서 그리기 명령**을 만들어 GPU에 던짐 → 발열·배터리는 GPU와 CPU가 나눠서 먹음.

---

> **요약 한 줄**

> CPU는 “나눌 수 있는 일은 스레드로, 무거운 픽셀 일은 [[GPU]]로”. 나눠 쓰면 끊김이 사라진다. 😎

## Keywords
- [[Thread]]
- [[Task]]
- [[ARC (Automatic Reference Counting)]]

## References
- https://developer.apple.com/documentation/uikit/uiview
- https://developer.apple.com/documentation/xcode/diagnosing-performance-issues-early
- https://developer.apple.com/documentation/realitykit/modifying-realitykit-rendering-using-custom-materials