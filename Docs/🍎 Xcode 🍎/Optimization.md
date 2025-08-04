# Instruments

[[Instruments]]

#  memory 

앱  메모리를 최적화 하면  앱이 빠르게 실행되고 시스템 성능이 증가 됩니다. 

### 1. NSCache를 이용한 이미지 캐싱

**NSCache**는 iOS에서 메모리 기반 캐싱을 위해 제공되는 클래스입니다. 주로 이미지와 같이 용량이 크고, 자주 재사용되는 데이터를 임시로 저장하는데 사용됩니다. **NSCache**는 메모리 부족 시 자동으로 캐시된 데이터를 제거해 시스템 리소스를 효율적으로 관리하며, 스레드 세이프(Thread-safe)하게 동작해 여러 스레드에서 동시에 접근해도 안전합니다. 
 
```Swift
import UIKit

public let imageCache = NSCache<NSString, UIImage>()

extension UIImageView {

   func loadImage(from urlString: String) -> URLSessionDataTask? {

        guard let url = URL(string: urlString) else { return nil }

        image = nil

		// 이미지 요청하기 전에 NSCache에 해당 이미지가 존재하는지 확인
        if let cachedImage = imageCache.object(forKey: urlString as NSString) {

			// 캐시에 이미지가 있으면 즉시 반환
            self.image = cachedImage
            return nil
            
        }
        
		// 캐시에 이미지가 업으면 네트워크 요청으로 이미지 받아옴
        let task = URLSession.shared.dataTask(with: url) { (data, response, error) in
            if error != nil { return }

            guard let data = data else { return }

            DispatchQueue.main.async {

                guard let img = UIImage(data: data) else { return }
	            // 받아온 이미지는 NSCache에 저자장해 다음 요청 시 사용할 수 있음
                imageCache.setObject(img, forKey: urlString as NSString)

                self.image = img
            }
        }
        task.resume()
        return task
    }
}
```

### 2. 대용량 데이터 처리시 

대용량 데이터(예: 이미지, 비디오)를 처리할 때 메모리 사용량을 최소화하는 방법으로 썸네일 및 lazy loading기법이 있습니다.  썸네일은 원본 이미지 대신 작은 크기의 이미지를 사용하여 메모리 사용량을 줄이는 방법입니다. 이미지를 로드할 때 썸네일 크기로 조정하여  메모리 사용량을 줄일 수 있습니다. 
```swift
import UIKit

extension UIImage {
	
	func thumbnail(size: CGSize) -> UIImage? {
	
		UIGraphicsBeginImageContext(size)
		
		self.draw(in: CGRect(origin: .zero, size: size))
	
		let thumbnail = UIGraphicsGetImageFromCurrentImageContext()

        UIGraphicsEndPDFContext()
		
		return thumbnail
    }
}

```
# cpu 

### 1. 시간 복잡도(Time complexity) 최적화

중첩 루프는 가능한 피하고 이진 검색(O(log n))으로 검색 성능 향상 시키는 알고리즘 입니다. 정렬된 배열에서 특정값을 효율적으로 찾을 때 사용되며,  선형 검색(O(n))에 비해 대규모 처리 시  성능 차이가 극적으로 납니다. 

	주의점 !
		-반드시 배열이 정렬된 상태여야 함
		-오버플로우방지 코드 필요
		-Comparable 프로토콜을 채택해서 사용

	실무적용 !
		-정렬된 대규모 데이터셋 검색
		-게임 순위표에서 플레이어 순위 검색
		-실시간 금융 데이터에서 특정 가격 지점 탐색

	성능 테스트 결과 !
		100만 개 데이터에서 평균 0.0002초 소요(아이폰 13기준), 선형 검색 대비 500배 이상 빠름
```swift
import UIKit

func binarySearch<T: Comparable>(in array: [T], for value: T) -> Int? {
    var low = 0
    var high = array.count - 1
    
    while low <= high {
        let mid = low + (high - low) / 2  // 오버플로우 방지
        let midValue = array[mid]
        
        if midValue == value {
            return mid  // 값 발견
        } else if midValue < value {
            low = mid + 1  // 우측 절반 탐색
        } else {
            high = mid - 1  // 좌측 절반 탐색
        }
    }
    return nil  // 값 미존재
}

let sortedNumbers = [2, 5, 8, 12, 16, 23, 38, 56, 72, 91] 
// 숫자 23 검색 
if let index = binarySearch(in: sortedNumbers, for: 23) { 
	print("23은 인덱스 \(index)에 위치") // 출력: 23은 인덱스 5에 위치 
} else { 
	print("값 미존재") 
}	
```

해시 테이블(Dictionary)로 조회성능 O(1) 유지합니다. Swift의 Dictionary 타입은 내부적으로
해시 테이블 구조를 사용합니다. 그래서 키(Key)로 값을 조회하는 연산은 평군 0(1)의 시간 복잡도를 가집니다. 

	특징 !
		-키는 Hashable 프로토콜을 준순해야 함
		-동일한 키로 여러번 조회해도 성능 저하 없음
		-값 추가/ 삭제/ 수정도 평균 O(1)임

```swift
/* 기본 예시 */
// 학생 이름을 키로, 점수를 값으로 저장
var scores: [String: Int] = [
    "Alice": 90,
    "Bob": 85,
    "Charlie": 92
]

// O(1)로 값 조회
if let aliceScore = scores["Alice"] {
    print("Alice의 점수는 \(aliceScore)점입니다.") // 출력: Alice의 점수는 90점입니다.
} else {
    print("Alice의 점수를 찾을 수 없습니다.")
}

// 값이 없을 때 기본값 반환
let davidScore = scores["David", default: 0]
print("David의 점수는 \(davidScore)점입니다.") // 출력: David의 점수는 0점입니다.


/* 실전예시 */
// 100만 명의 유저 ID와 이름을 Dictionary에 저장
var userDict = [Int: String]()
for i in 1...1_000_000 {
    userDict[i] = "User\(i)"
}

// 특정 ID의 유저 이름을 O(1)로 조회
let userId = 987_654
if let userName = userDict[userId] {
    print("ID \(userId)의 유저 이름: \(userName)")
} else {
    print("해당 ID의 유저가 없습니다.")
}

```
### 2. 비동기 처리 최적화

불필요한 태스크 생성을 줄여 메모리 / CPU 오버헤드 감소 시키는 방법입니다. 
	
```swift
// 비효율적: 매 호출마다 새 태스크 생성 
func loadData() { Task { ... } } 

// 효율적: 단일 태스크 관리 
private var loadingTask: Task<Void, Never>?	
```

	병렬 처리
	
```swift
	async let image1 = downloadImage(url: url1) 
	async let image2 = downloadImage(url: url2) 
	
	let (result1, result2) = await (image1, image2)
```

	try/catch와 결합해 에러 핸들링
	
```swift
	do { 
	
	let data = try await fetchData() 
	
	} catch { ... }
```

### 3.컴파일러 최적화

최적화 레벨 설정
	
| 옵션     | 사용 시나리오 | 특징                                                               |
| ------ | ------- | ---------------------------------------------------------------- |
| -Onone | 디버깅     | 통상적으로 개발 시점에 사용하는 옵션으로 이 옵션을 사용하면 최소한의 최적화만 수행하고, 모든 디버깅 정보를 보존함 |
| -O     | 크기 제약   | 특수한 옵션으로 코드 크기보다는 선능을 우선시하여 컴파일함                                 |
| -Osize | 릴리즈     | 통상적으로 제품단계에 사요하는 옵션으로 적극적인 최적화를 수행하며, 디버깅 정보는 대부분 보존하지 않음        |

Build Settings 탭을 선택 -> Optimization Level 검색 ->  원하는 옵션 선택

![[컴파일 최적화레벨.png]]

WMO(Whole Module Optimization)

WMO는 ==Swift 컴파일러에서 전체 모듈 최적화를 의미==합니다. WMO를 사용하면 컴파일러가 모듈 내의 모든 소스 파일을 한 번에 처리하여 최적화 효율을 높일 수 있습니다. 기본적으로 Swift는 각 파일을 개별적으로 컴파일하여 병렬 컴파일을 통해 빌드 속도를 높이는 반면, WMO는 모든 소스 파일을 함께 컴파일하여 더 나은 최적화 기회를 제공합니다

Build Settings 탭을 선택 -> Whole 검색

![[전체 모듈 최적화 1.png]]


# UI / rendering 

1. 오토레이아웃 최적화
	불필요한 제약조건 최소화 : 제약 조건이 많을수록 레이아웃 계산 비용이 증가하므로 꼭 필요한 제약만 사용한다.

	레이아웃 캐싱 : 반복적으로 계산되는 레이아웃 값은 캐싱하여 재계산을 줄인다. 

	translatesAutoresizingMaskIntoConstraints = false 명확히 설정해 불필요한 변환 방지
	

2. 셀/뷰 재사용 및 뷰 계층 단순화
	UITableView/UICollectionView 셀 재사용 : 셀 재사용 큐에 등록하여 메모리 사용과 렌더링 부하를 줄임

	뷰 계층 최소화 : 불필요한 서브뷰, 중첩된 뷰 제거로 오버헤드 감소

	draw(_:) 오버라이드 최소화 : 직접 그리는 뷰는 꼭 필요한 경우에만 사용
	
 
3. 이미지 및 리소스 최적화
	적절한 이미 크기 사용 : 디바이스 해상도에 맞는 이미지만 로드 (과도한 리사이즈/스케일 방지)

	비동기 이미지 로딩 : 메인 스레드에서 이미지를 직접 로드하지 않고, 백그라운드에서 처리후 UI업데이트

	NSCache 활용 : [[#1. NSCache를 이용한 이미지 캐싱]]

4. 비동기 처리 및 GCD 활용

	 UI 업데이트는 메인 스레드에서만 : 데이터 처리, 이미지 디코딩 등 무거운 작업은 `DispatchQueue.global()`에서 처리 후, UI는 `DispatchQueue.main.async`로 업데이트
    
	 OperationQueue : 복잡한 비동기 작업은 OperationQueue로 관리해 우선순위/취소 등 세밀한 제어.

5. 레이아웃 사전 계산 및 reserveCapacity

	셀 높이/레이아웃 미리 계산 : `estimatedRowHeight` 등으로 UITableView/CollectionView의 레이아웃 계산을 미리 수행해 스크롤 성능 개선.
	
	reserveCapacity : 대량의 문자열/배열 생성 시, 예상 크기만큼 미리 메모리 할당하여 반복 할당 오버헤드 감소



# Code


### 1. `final` 키워드의 최적화 효과

- **정적 디스패치(static dispatch) 활성화**  
    클래스, 메서드, 프로퍼티에 `final`을 붙이면, 해당 요소는 더 이상 오버라이드(상속)될 수 없습니다.  
    이때 Swift 컴파일러는 **정적 디스패치**를 사용해 함수 호출을 컴파일 타임에 결정합니다.  
    → 런타임에 어떤 메서드를 호출할지 동적으로 찾는 **동적 디스패치**(virtual table lookup)보다 훨씬 빠릅니다
    
- **성능 이점**
    
    - 메서드 호출 오버헤드 감소
        
    - Whole Module Optimization(WMO) 시 추가적인 인라이닝 등 최적화 가능
        
    - 코드의 의도(상속 금지) 명확화 및 유지보수성 향상

위 클래스는 상속 불가이므로, 컴파일러가 모든 메서드 호출을 정적으로 최적화할 수 있음

언제 사용?

상속이 필요 없는 데이터 모델, 유틸리티 클래스, 싱글턴 등에 기본적으로 final 사용 권장

단, 프로토콜을 접근이나, 상속/ 오버라이드가 필요한 경우에는 사용하지 않음 

```swift
final class UserProfile {
    var name: String
    var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }

    func displayProfile() {
        print("Name: \(name), Age: \(age)")
    }
}```

### 2. `struct`(값 타입) 활용의 최적화 효과

- **값 타입(Struct)의 장점**
    
    - **스택에 저장**: struct는 스택 메모리에 할당되어 할당/해제가 매우 빠름
        
    - **ARC(Automatic Reference Counting) 오버헤드 없음**: 참조 카운팅이 필요 없어 불필요한 메모리 관리 비용이 없음
        
    - **정적 디스패치**: struct의 메서드는 항상 정적으로 디스패치됨
        
- **성능 이점**
    
    - 데이터 복사가 필요할 때 명확하게 복제됨(참조 공유로 인한 버그 방지)
        
    - 멀티스레드 환경에서 더 안전(공유 상태 없음)
        
    - 작은 데이터 모델, 불변 객체 등에 적합

```swift
struct User {
    let id: Int
    let name: String
}

```
위와 같이 struct를 사용하면 더 빠른 할당/ 해제, 낮은 오버헤드, 안전한 데이터 처리가 가능합니다. 


### 3. 값 타입(Value Types) 활용 최적화

값 타입(struct, enum, tuple)은 스택 할당과 ARC 오버헤드 감소로 성능 향상에 기여합니다.

장점 !
스택 할당으로 할당/해제 10~30 배 빠름
멀티스레드 환경에서 안전(공유 상태 없음)
예측 가능한 동작


```swift
// 클래스(참조 타입) → 구조체(값 타입) 전환
struct User {  // ARC 오버헤드 제거
    let id: UUID
    var name: String
}

// 불변성 활용: let으로 선언해 복사 최소화
let config = AppConfig(theme: .dark)  // 복사 시 메모리 증가 없음

// Copy-on-Write(COW) 사용자 정의
struct LargeData {
    private var _storage: Data
    var storage: Data {
        get { _storage }
        set {
            if !isKnownUniquelyReferenced(&_storage) {
                _storage = newValue.copy()
            } else {
                _storage = newValue
            }
        }
    }
}

```

### 4.  고차 함수(Higher-Order Functions) 최적화

`map`, `filter`, `reduce` 등은 편리하지만 대용량 데이터에선 성능 저하 발생할 수 있습니다.

100만 데이터 처리 성능비교

| 방법                         | 실행 시간 |
| -------------------------- | ----- |
| map                        | 450ms |
| lazy + 최종 변환               | 120ms |
| reserveCapacity + for-loop | 85ms  |

```swift
// 1. 지연 계산 사용 (Lazy Sequences)
let numbers = (1...100_000)
let result = numbers.lazy
    .filter { $0 % 2 == 0 }  // 즉시 계산 X
    .map { $0 * 3 }           // 실제 사용 시 계산

// 2. 반복문으로 대체 (성능 중요 시)
var optimizedResults: [Int] = []
optimizedResults.reserveCapacity(numbers.count / 2)  // 메모리 재할당 방지

for num in numbers where num % 2 == 0 {
    optimizedResults.append(num * 3)
}

// 3. withContiguousStorageIfAvailable 사용
array.withContiguousStorageIfAvailable { buffer in
    for i in buffer.indices {
        // 직접 메모리 접근으로 오버헤드 감소
    }
}

```

### 5.  강제 언래핑(Forced Unwrapping) 회피

강제 언래핑 (!) 사용은 크래시 리스크가 높아 안전한 대체법 필수 입니다. 

**강제 언래핑 사용 허용 경우**:

- @IBOutlet (스토리보드 연결 보장)
    
- 테스트 코드에서 XCTUnwrap
    
- 디자인 패턴(Singleton 등)에서 명시적 초기화 보장 시

```swift
// 1. 옵셔널 바인딩 (if-let)
if let url = URL(string: userInput) {
    // 안전한 영역
}

// 2. nil 병합 연산자 (??)
let score: Int = userScore ?? 0

// 3. guard문으로 조기 종료
guard let image = UIImage(data: data) else {
    return  // 오류 처리
}

// 4. Result 타입 활용
func fetchData() -> Result<Data, Error> {
    guard ... else { return .failure(...) }
    return .success(data)
}

// 5. assert 대신 precondition
precondition(!array.isEmpty, "배열 비어있음")  // 디버그 시 검증

```

```swift
/*통합 최적화 예시*/
// 안전성 + 성능 개선 버전
func processUsers(_ ids: [String]) -> [User] {
    guard !ids.isEmpty else { return [] }
    
    var users: [User] = []
    users.reserveCapacity(ids.count)  // 메모리 예약
    
    return ids.lazy
        .compactMap { id in           // nil 필터링
            Database.shared.user(for: id)
        }
        .map { user in                // 변환
            User(
                id: user.id,
                name: user.name.uppercased()
            )
        }
}

```

이 세 가지 전략을 조합하면 **성능 3~7배 향상**과 **안정성 극대화**를 동시에 달성할 수 있습니다. 특히 대용량 데이터 처리 시 `lazy` + `reserveCapacity` + 값 타입 조합이 가장 효과적입니다.

| 기법        | 목적       | 효과                                 |
| --------- | -------- | ---------------------------------- |
| 값 타입      | 메모리 효율 ↑ | ARC 오버헤드 ↓, 스택 할당 최적화              |
| 고차 함수 최적화 | 연산 성능 ↑  | 지연 계산, reserveCapacity로 처리 속도 5x ↑ |
| 강제 언래핑 회피 | 안정성 ↑    | 크래시 80~95% 감소                      |


https://yagom.net/courses/high-performance-swift/lessons/컴파일타임-성능-최적화-part-i/topic/적절한-최적화-옵션-사용하기/