
![[HealthKit.png]]

## 개요
---
HealthKit은 iPhone, Apple Watch, visionOS 등에서 **사용자의 건강·피트니스 데이터를 중앙 저장소**로 관리하는 프레임워크입니다. 권한을 받은 앱만 접근 가능하며, **데이터 읽기·쓰기** 기능을 제공합니다.

또한 *CareKit, ResearchKit, SensorKit* 등 다양한 프레임워크와 연계해 개인 건강 관리, 연구 참여, 실시간 워크아웃 기능까지 확장할 수 있습니다.


## 주요 기능
---
HealthKit은 아래와 같이 넓은 범위의 건강 데이터를 나눕니다.

| 카테고리                      | 제공 데이터                        |
| ------------------------- | ----------------------------- |
| Activity (활동)             | 걸음 수, 이동 거리, 계단 오르기, 운동 세션 요약 |
| Body Measurements (신체 정보) | 키, 체중, 체지방률, 체질량지수            |
| Cycle Tracking (생리 주기)    | 생리 정보, 기초 체온                  |
| Hearing (청각)              | 이어폰 및 환경 소음 노출, 청력 검사 결과      |
| Heart (심장)                | 심박, 심박변이, ECG                 |
| Medications (약물)          | 복약 기록, 약물 정보                  |
| Mental Wellbeing (정신 건강)  | 우울/불안 지표, 감정 기록               |
| Mobility (이동성)            | 이동 관련 메트릭 (워킹, 러닝, 운동)        |
| Nutrition (영양)            | 수분, 칼로리, 비타민, 미네랄 섭취          |
| Respiratory (호흡)          | 호흡수, 산소포화도, 흡입기 사용            |
| Sleep (호흡)                | 수면 상태, 잠든 시간, 수면 단계           |
| Symptoms (증상)             | 두통, 복통 등 증상 기록                |
| Vitals (활력 징후)            | 혈압, 체온, 혈당 등                  |
| Other Data (기타)           | 사용자 특성 정보 (성별, 혈액형, ...)      |

>[!question] 데이터 제공 방식
>* HealthKit은 **"저장된 데이터"**만 기본 제공
>* 실시간 데이터가 필요한 경우에는 `HKLiveWorkOutBuilder` 사용 (운동 세션 중에만 작동)
>	* 심박수, 칼로리 등 운동과 관련된 정보들만 스트리밍 수신 가능

>[!question] HealthKit의 데이터 저장 위치
>* 모든 데이터는 사용자의 로컬 디바이스에 저장됨
>* 일반 앱처럼 파일 경로나 저장소 접근 X
>* 대신 HKHealthStore API로만 접근 가능
>* 저장 위치는 - **Sandboxed 보호 영역** (iOS 내부 보안 구역)
>* 기기별로 독립적으로 저장 → iPhone 두 대를 같은 Apple ID로 로그인했어도 동기화 X

>[!question] 보안 처리 방식
>- 모든 데이터는 **암호화 + Secure Enclave**에 의해 보호
>→ Secure Enclave란? iOS 내 **독립된 하드웨어 보안 모듈**로, Apple도 접근 불가
>- 디바이스가 잠금 상태면 접근 차단됨
>- Face ID / Touch ID / 비밀번호로만 복호화 가능
>- Apple도 복호화 못함


## 코드 예시
---
1. 권한 요청
	* 읽기와 쓰기 각각 명시적으로 요청해야함 (read, toShare)
	* 사용자가 거부하면 접근 불가. 부분 승인도 가능함
	* HIG에서 **최소한의 데이터만 요청하고, 기능을 실행할 때 권한을 요청하는 방식**을 권장
	
```Swift
import HealthKit

let healthStore = HKHealthStore()
let readTypes: Set = [
    HKObjectType.quantityType(forIdentifier: .stepCount)!,
    HKObjectType.categoryType(forIdentifier: .sleepAnalysis)!
]

let writeTypes: Set = [
    HKObjectType.quantityType(forIdentifier: .stepCount)!,
    HKObjectType.categoryType(forIdentifier: .sleepAnalysis)!
]

healthStore.requestAuthorization(toShare: writeTypes, read: readTypes) { success, error in
    if success {
        print("읽기+쓰기 권한 허용됨")
    } else {
		print("권한 오류: \(error?.localizedDescription ?? "알 수 없는 오류")")
    }
}
```

2. 걸음 수 가져오기

```Swift
func fetchStepCount() {
    // 1. 걸음 수 타입 지정
    guard let stepType = HKQuantityType.quantityType(
        forIdentifier: .stepCount
    ) else { return }

    // 2. 조회 시간 범위 설정
    let startDate = Calendar.current.startOfDay(for: Date())

    // 3. 쿼리 조건 구성
    let predicate = HKQuery.predicateForSamples(
        withStart: startDate,
        end: Date()
    )

    // 4. HKSampleQuery 생성
    let query = HKSampleQuery(
        sampleType: stepType, 
        predicate: predicate, 
        limit: HKObjectQueryNoLimit, 
        sortDescriptors: nil
    ) { _, samples, error in
        guard let quantitySamples = samples as? [HKQuantitySample]
        else { return } 
        let total = quantitySamples.reduce(0.0) { sum, sample in
            sum + sample.quantity.doubleValue(for: .count())
        } 
        print("오늘 걸음 수: \(total)")
    }
    HKHealthStore().execute(query) 
}
```

**데이터 조회할 때 자주 쓰는 옵션들**
* `predicate` (조회 범위 설정)
	* 시간 범위, 샘플 조건 등을 필터링할 수 있음
	* 자주 쓰는 predicate 종류
		- `predicateForSamples(withStart:end:options:)` : 시간 필터링
	    - `predicateForObjects(from:)` : 특정 앱이 기록한 데이터만
	    - `predicateForObjects(withDeviceProperty:)` : 특정 기기에서 기록한 데이터
* `limit` (최대 조회 수 제한)
	* 한번에 가져올 샘플 수 제한
	* `HKObjectQueryNoLimit` 사용 시 전부 가져옴
* `sortDescriptors` (정렬)
	* 조회된 샘플의 정렬 기준 
	* 자주 사용하는 키
		* `HKSampleSortIdentifierStartDate` : 샘플이 시작된 시간 기준 정렬
		* `HKSampleSortIdentifierEndDate` : 샘플이 종료된 시간 기준으로 정렬

```Swift
let sortByDate = NSSortDescriptor(
    key: HKSampleSortIdentifierStartDate, // 시작된 시간 기준
    ascending: false // 시작 시간 기준으로 한 내림차순 정렬
)
```


2. 걸음 수 입력하기

```Swift
func writeStepCount(stepCount: Double) {
    let healthStore = HKHealthStore()

    // 1. 걸음 수 타입 지정
    guard let stepType = HKQuantityType.quantityType(forIdentifier: .stepCount) else {
        print("❌ 걸음 수 타입 생성 실패")
        return
    }
    
    // 2. 수치 + 단위로 Quantity 생성
    let quantity = HKQuantity(unit: .count(), doubleValue: stepCount)

    // 3. 샘플 생성 (측정 시간 범위는 현재 시점 기준 10분)
    let now = Date()
    let tenMinutesAgo = now.addingTimeInterval(-600)
    let sample = HKQuantitySample(
        type: stepType,
        quantity: quantity,
        start: tenMinutesAgo,
        end: now
    )
    
    // 4. 저장 실행
    healthStore.save(sample) { success, error in
        if success {
            print("걸음 수 \(stepCount) 저장 성공")
        } else {
            print("저장 실패: \(error?.localizedDescription ?? "알 수 없는 오류")")
        }
    }
}
```


## References
---
* [Apple Developer - HealthKit](https://developer.apple.com/documentation/healthkit)
* [Apple HIG - HealthKit](https://developer.apple.com/design/human-interface-guidelines/healthkit)
* [Apple - 개인정보 보호: Health](https://www.apple.com/privacy/features/)
