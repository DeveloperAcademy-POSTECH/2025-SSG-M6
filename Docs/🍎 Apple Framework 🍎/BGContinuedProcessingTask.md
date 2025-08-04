
iOS는 배터리 절약과 자원 최적화를 위해 앱의 **백그라운드 실행을 엄격히 제한**한다. 하지만 사용자의 기대는 다르다. 예를 들어 "SNS 업로드"나 "파일 내보내기"처럼 사용자가 시작한 작업은 앱이 백그라운드로 가도 끝까지 실행되길 원한다. 이 간극을 해결해주는 핵심이 바로 **Background Tasks 프레임워크**이며, iOS 26에서는 새로운 `BGContinuedProcessingTask`가 도입되었다.


#### 1. 기존 Background Task의 한계와 허용 조건
---
기존 iOS 백그라운드 작업 API들은 대부분 *시스템이 알아서 실행 시점을 결정* 하는 구조였다. 하지만 이 방식은 사용자가 시작한 작업이 중단되거나 실패하는 문제를 유발했다.
이러한 한계를 해결하기 위해 등장한 것이 바로 `BGContinuedProcessingTask`이다.


#### 2. 왜 `BGContinuedProcessingTask`가 등장했을까?
---
- 기존 작업은 대부분 시스템이 알아서 실행 시간을 정했다 → 사용자 주도 작업에는 부적합
- 앱이 백그라운드로 전환되면 중간에 끊기거나, 시스템이 종료시킬 수도 있다
- 사용자 입장에서는 불확실하고 UX가 별로라고 느낄 수 있음

그래서 등장한 해결책이 바로, `BGContinuedProcessingTask` → 사용자가 시작한 작업을, 앱이 백그라운드로 가더라도 끝까지 실행하도록 보장한다.


#### 3. BGContinuedProcessingTask의 주요 특징
---
**✔️ 사용자 중심의 작업 흐름**
- 반드시 **사용자의 명확한 행동**(예: 버튼 탭)으로 시작해야 함
- 앱이 백그라운드로 가도 작업이 시스템에 의해 계속 유지됨

**✔️ 시스템이 UI를 직접 제공**
- iOS 26부터는 시스템이 자동으로 UI를 제공
- 사용자에게 진행 중인 작업의 제목/부제/진행률 표시
- 별도 구현 없이도 통일된 UX 제공 가능

<img src ="BGContinuedProcessingTask_Progress_UI.png" width="300">

**✔️ task.progress로 진행률 연동 가능**
* 진행 상태가 시스템 UI에 반영됨 (취소 버튼도 자동 지원)

```Swift
task.progress.totalUnitCount = 100
task.progress.completedUnitCount = 0
...
task.progress.completedUnitCount += 10
```


#### 4. Apple이 강조한 두 가지 철학
---
**1. Continue Initiated Work**
> 사용자가 시작한 작업은, 앱이 백그라운드로 가더라도 **끊기지 않고 계속되어야 한다.**

예: 사용자가 '내보내기' 버튼을 눌렀다면 → 해당 작업은 앱이 백그라운드로 가도 계속 진행되어야 함

**2. Avoid Unexpected Work**
> 사용자가 몰랐던 작업이, 백그라운드에서 **몰래 진행되는 일은 없어야 한다.**

- 사용자의 명시적인 행동이 없으면 백그라운드 작업은 시작조차 못함
- 시스템은 submit 타이밍, 호출 API, 상태를 감지해 비정상 작동 탐지 가능

#### Reference
---
- Apple Developer Doc: [BGContinuedProcessingTask](https://developer.apple.com/documentation/backgroundtasks/bgcontinuedprocessingtask)
- WWDC25 세션 영상: [WWDC25 Session 227](https://developer.apple.com/videos/play/wwdc2025/227/)




