>[!question]
>GQ1. **core haptics**는 어떻게 작동할까?
>GQ2. 데시벨 입력을 실시간으로 어떻게 측정하고, 특정 기준에 따라 분류할 수 있을까?
>GQ3. 햅틱 피드백을 동적으로 변경하려면 어떤 API를 사용해야 할까?

## Core Haptics

<font color="orange">Core Haptics</font>는 iOS 앱의 햅틱 피드백을 사용자 정의된 패턴으로 구성하고 재생할 수 있게 해주는 프레임워크입니다. 
앱에 맞춤형 햅틱 및 오디오 피드백을 추가할 수 있게 해주는 기능을 제공해줍니다. 
햅틱을 사용하면 사용자의 신체적 반응을 유도할 수 있으며, 주의를 끌거나 특정 동작을 강조하는 데 효과적입니다. 😎

스위치, 슬라이더, 피커 등 일부 시스템 제공 UI 요소는 기본적으로 사용자 상호작용 시 햅틱 피드백을 제공합니다. <font color="orange">Core Haptics</font>를 사용하면 이러한 기본 피드백을 넘어서, 더욱 정밀하게 구성된 햅틱 패턴을 직접 만들고 재생할 수 있습니다. 
## core haptics는 어떻게 작동할까?

#### 작동 흐름
![[CoreHapticsFlow.jpg]]
#### CHHapticEngine (햅틱 엔진)

Core Haptics의 중심 객체로, 하드웨어에 접근하여 햅틱을 재생
앱에서 한 번만 생성 후 재사용
thread-safe 하며, 여러 개의 엔진도 독립적으로 운영이 가능

#### CHHapticPattern (햅틱 패턴)

하나 이상의 이벤트와 파라미터로 구성된 진동 패턴
JSON이나 Swift 딕셔너리로도 정의 가능

앱은 <font color="teal">CHHapticEvent</font>라고 불리는 기본 구성 요소로 햅틱 패턴을 만들 수 있습니다.
- 순간적인(transient) 이벤트는 스위치를 토글할 때처럼 기본 구성 요소로 햅틱 패턴을 만들 수 있음 !
- 지속적인(continuous) 이벤트는 벨소리처럼 진동이나 사운드가 일정 시간 유지되는 경우입미다.
이 두 유형의 이벤트를 개별적으로 사용할 수도 있고, 조합해서 정교한 패턴을 만들 수도 있어요.
![[CHHapticPattern.png]]

#### CHHapticEvent (햅틱 이벤트)

하나의 진동 / 소리 **이벤트 단위**
<font color="teal">eventType</font> : 진동인지 소리인지
<font color="teal">time</font> : 언제 시작할지
<font color="teal">duration</font> : 지속 시간
<font color="teal">parameters</font> : 강도(intensity), 날카로움(sharpness)


#### CHHapticPatternPlayer (패턴 재생기)

패턴을 받아 재생하는 플레이어 객체

#### Handler
**resetHandler** : 엔진 오류나 미디어 서버 실패 시 호출됨 -> 재시작 필요
**stoppedHandler** : 외부 요인으로 엔진이 멈췄을 때 호출


## 데시벨 입력을 실시간으로 어떻게 측정하고, 특정 기준에 따라 분류할 수 있을까? -> 🙆🏻‍♀️

1. 마이크 입력에서 데시벨 실시간 측정

```swift
self.audioPlayer?.updateMeters()
let db = self.audioPlayer?.averagePower(forChannel: 0) ?? -100
```

2. 측정값을 기준으로 비교하여 상태 분류

```swift
switch db {
case -20...(-15):
intensity = 0.3
sharpness = 0.8
    ...
case -15...(-10):
    intensity = 0.5
    ...
case -10...(-5):
    intensity = 0.7
    ...
case -5...0:
    intensity = 1.0
    ...
default:
    return
}
```

3. 이후 햅틱이나 피드백에 활용 가능

```swift
self.playHapticByDB(db)  // <- 데시벨을 전달해 햅틱 피드백을 연결한다.
```


## 햅틱 피드백을 동적으로 적용하려면 어떤 API를 사용해야 할까? -> CHHapticAdvancedPatternPlayer + .hapticContinuous 이벤트 + sendParameters(_:atTime:) 


## References
- [📑 Documentation](https://developer.apple.com/documentation/corehaptics)