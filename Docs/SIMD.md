>[!question]
>GQ1. SIMD는 어떻게 사용할까?
>GQ2. AR 작업과 SIMD는 왜 잘 어울릴까?
>GQ3. 애니메이션 이용 시, SIMD를 사용하면 얼마나 더 빠를까?

## SIMD(Single Instruction, Multiple Data)가 뭔데!

SIMD는 하나의 명령어로 여러 데이터를 동시에 처리할 수 있게 해주는 CPU 기능입니다. 
일반적으로 CPU는 한 번에 하나의 데이터만 처리하지만, SIMD는 **하나의 명령으로 여러 개의 데이터를 병렬로 처리**할 수 있게 해줍니다. 예를 들어, 벡터 덧셈처럼 반복적인 연산을 CPU가 병렬적으로 처리할 수 있도록 합니다.

**일반 방식**
```swift
let a = [1, 2, 3, 4]
let b = [5, 6, 7, 8]
var result = [Int](repeating: 0, count: 4)

for i in 0..<4 {
    result[i] = a[i] + b[i]  // 하나씩 더함
}
```

**SIMD 방식**
```swift
let a = SIMD4<Int>(1, 2, 3, 4)
let b = SIMD4<Int>(5, 6, 7, 8)
let result = a + b  // 한 줄로 동시에 4개 더함!
```
SIMD는 내부적으로 4개를 한꺼번에 처리함

### 왜 빠를까?

CPU 내부에는 다음과 같은 **SIMD 명령어 집합**이 존재합니다:
- **SSE / AVX (Intel/AMD)**
- **NEON (ARM, iPhone 포함)**
    
이 명령어들은 한 번에 4개, 8개, 심지어 16개의 숫자를 병렬로 처리할 수 있게 설계되어 있습니다.
Swift의 SIMD 타입을 쓰면 이 명령어들을 **자동으로 활용**하게 됩니다.

### Swift에서 제공하는 SIMD 타입들

`SIMD2<Float> : 2D 벡터`
`SIMD3<Float> : 3D 벡터`
`SIMD4<Float> : 4D 벡터`
`SIMD16<UInt8> :16개의 UInt8 병렬 처리`
`simd_float4x4 : 4X4 행렬` 


## AR과 SIMD가 왜 잘 어울릴까?

AR 시스템은 실시간으로 3D를 계산하고 렌더링을 해야 하는데, 이를 위해 하는 연산들이 벡터와 행렬 연산이고 ARKit / RealityKit은 이미 SIMD 타입을 기본 값으로 사용하고 있습니다.

```swift
import SwiftUI
import simd

struct ContentView: View {
    @State private var positions: [CGPoint] = Array(repeating: .zero, count: 10_000)
    @State private var useSIMD = false
    @State private var angle: Float = 0.0
    @State private var frameDurations: [Double] = []
    @State private var statsText: String = ""

    let radius: Float = 100
    let timer = Timer.publish(every: 1.0 / 60.0, on: .main, in: .common).autoconnect()

    var body: some View {
        VStack(spacing: 16) {
            Toggle("Use SIMD", isOn: $useSIMD)
                .padding(.horizontal)

            Text(statsText)
                .font(.caption)
                .multilineTextAlignment(.leading)

            ZStack {
                ForEach(0..<positions.count, id: \.self) { i in
                    Circle()
                        .fill(useSIMD ? .orange : .blue)
                        .frame(width: 2, height: 2)
                        .position(positions[i])
                }
            }
            .frame(width: 400, height: 400)
            .background(Color.black.opacity(0.05))
            .clipShape(RoundedRectangle(cornerRadius: 10))
        }
        .onReceive(timer) { _ in
            let start = CACurrentMediaTime()
            angle += 0.01
            
            if useSIMD {
                updateWithSIMD4()
            } else {
                updateWithLoop()
            }

            let frameTime = CACurrentMediaTime() - start
            record(frameTime)
        }
    }

    func updateWithLoop() {
        let center = CGPoint(x: 200, y: 200)

        for i in 0..<positions.count {
            let theta = angle + Float(i) * 0.001
            let x = center.x + CGFloat(radius * cos(theta))
            let y = center.y + CGFloat(radius * sin(theta))
            positions[i] = CGPoint(x: x, y: y)
        }
    }

    func updateWithSIMD4() {
        let center = SIMD2<Float>(200, 200)
        var newPositions = positions
        var i = 0
        
        while i + 3 < positions.count {
            let theta0 = angle + Float(i) * 0.001
            let theta1 = angle + Float(i+1) * 0.001
            let theta2 = angle + Float(i+2) * 0.001
            let theta3 = angle + Float(i+3) * 0.001

            let x = SIMD4<Float>(cos(theta0), cos(theta1), cos(theta2), cos(theta3))
            let y = SIMD4<Float>(sin(theta0), sin(theta1), sin(theta2), sin(theta3))
            let resultX = x * radius + center.x
            let resultY = y * radius + center.y

            newPositions[i]   = CGPoint(x: CGFloat(resultX[0]), y: CGFloat(resultY[0]))
            newPositions[i+1] = CGPoint(x: CGFloat(resultX[1]), y: CGFloat(resultY[1]))
            newPositions[i+2] = CGPoint(x: CGFloat(resultX[2]), y: CGFloat(resultY[2]))
            newPositions[i+3] = CGPoint(x: CGFloat(resultX[3]), y: CGFloat(resultY[3]))

            i += 4
        }
        positions = newPositions
    }
    
    func record(_ frameTime: Double) {
        frameDurations.append(frameTime)
        
        if frameDurations.count > 60 {
            frameDurations.removeFirst()
        }

  

        let avg = frameDurations.reduce(0, +) / Double(frameDurations.count)
        let min = frameDurations.min() ?? 0
        let max = frameDurations.max() ?? 0
        let fps = 1.0 / avg

        statsText = String(format: """
        Frames: %d
        Avg Frame Time: %.2f ms
        Min: %.2f ms | Max: %.2f ms
        Estimated FPS: %.1f
        """, frameDurations.count, avg * 1000, min * 1000, max * 1000, fps)
    }
```

![[SIMD_test.png]]
오 ,. 확실히 빠르다

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)