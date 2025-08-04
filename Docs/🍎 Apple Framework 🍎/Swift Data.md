> [!question]  
> GQ1. SwiftData는 어떻게 사용될까? 
> GQ2. 2025 WWDC에서 발표된 SwiftData 변화에는 뭐가 있을까?


## Description

---

WWDC 2023에서 처음 소개된 Swift 기반의 Persistence Data 저장용 프레임워크이다. 선언적이고 간단한 코드로 데이터 관리가 가능해져 앱에서 데이터를 저장하고 관리하는 것을 더 쉽게 만들었다.

SwiftData가 등장하기 전에는, [[Core Data]]라는 프레임워크가 존재했다. 기존의 CoreData는 오랜 시간동안 Apple 생태계의 표준 Persistence Data 도구였지만, 아래와 같은 몇가지 단점이 있다.

- NSmanagedObject, NSFetchRequest 등 복잡한 객체 지향 설계
- Xcode의 .xcdatamodeld 파일을 따로 관리해야 하는 불편함과 사용 시작 시점의 공수 발생
- SwiftUI와의 연동이 매끄럽지 않음
- Swift 언어의 강력한 기능(타입 안전성, 프로토콜 지향 설계 등)을 활용하기 어려움

SwiftData는 이런 문제들을 해결하며, SwiftUI 중심 앱 개발에서 보다 자연스럽고 선언적인 데이터 흐름을 가능하게 해준다. 즉, 아래와 같은 특징들을 가지며, .xcdatamodeld 없이도 모델 정의가 가능하다는 간편함과 간단하게 CRUD 구현이 가능하다는 장점이 있다.

- 선언형 (Declarative)
    - SwiftData는 Swift의 최신 문법을 활용해 코드가 어떻게 무엇을 수행하는지를 명확하게 작성할 수 있다.
- 영속성 (Persistence)
    - SQLite 기반의 로컬 저장소를 활용해, 앱을 종료하거나 기기를 재시작해도 데이터가 유지되어 관리할 수 있다
- SwiftUI와의 호환성
    - SwiftUI와의 통합을 바탕으로 설계되어 데이터의 상태 변화가 UI에 실시간 반영되는 바인딩이나 쿼리 사용이 간단하다


### 구조

#### @Model

SwiftData에서의 데이터 모델은 클래스로 정의하며, @Model 속성을 붙여 Schema를 정의한다. 이런 모델 클래스는 속성, 관계, 제약조건을 포함하는 엔티티 형태가 되며, @Model은 해당 클래스가 SwiftData의 Persistence 대상임을 나타낸다.

- @Model은 내부적으로 Core Data의 기능을 래핑한다
- 클래스만 지원하며, 구조체는 지원하지 않는다
- Identifiable 프로토콜을 자동 채택한다

```swift
@Model
class Task {
    var title: String
    var isDone: Bool
    var createdAt: Date
}
```

#### ModelContext

데이터를 추가, 삭제, 저장하는 등 조작하는 데 사용되는 객체이다. SwiftUI 뷰 안에서는 @Environment(\.modelContext) 형태로 주입받아 사용한다.

→ Core Data에서의 NSManagedObjectContext에 해당한다

```swift
@Environment(\\.modelContext) private var context

let task = Task(title: "할 일")
context.insert(task)
```

#### @Query

특정 모델의 데이터를 불러오는 역할을 한다. @FetchRequest의 SwiftData 버전이다. 필터링, 정렬, 연관 관계 쿼리를 지원한다.

- 자동으로 상태 변화를 감지해 SwiftUI 뷰를 갱신한다
- 간단한 정렬 및 필터링 문법을 제공한다

```swift
@Query var tasks: [Task]

// 조건 추가하는 경우 -> sort, filter 가능
@Query(sort: \\Task.createdAt, order: .reverse)
private var tasks: [Task]
```

#### modelContainer

앱 전체에서 사용할 수 있는 모델 컨테이너를 생성한다. 모델 컨테이너는 모델 컨텍스트와 DB 사이의 브로커 역할을 한다. 주어진 스키마를 통해 DB를 생성하고, 디스크 간 읽기 및 쓰기를 관리한다. 직접적으로 데이터를 쓰거나 수정, 삭제 하지는 않으며, 모델 컨텍스트가 수행하고 컨테이너는 해당 내용을 전달하기만 한다. SwiftUI 앱에서는 .modelContainer(for: ) 형태를 통해서 모델을 지정한다.

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Task.self)
    }
}
```



## 코드 예시

---

앞서 SwiftData에 대해서 설명하기 위해 만든 Task 모델을 이용해 TODO 앱을 예시로 사용 방법을 설명한다.

```swift
@Model
class Task {
    var title: String
    var isDone: Bool
    var createdAt: Date = .now

    init(title: String, isDone: Bool = false) {
        self.title = title
        self.isDone = isDone
    }
}
```

```swift
struct ContentView: View {
    @Environment(\\.modelContext) private var context
    @Query(sort: \\Task.createdAt, order: .reverse) private var tasks: [Task]
    @State private var input: String = ""

    var body: some View {
        VStack {
            TextField("할 일 입력", text: $input)
                .textFieldStyle(.roundedBorder)
            Button("추가") {
                let newTask = Task(title: input)
                context.insert(newTask)
                input = ""
            }

            List {
                ForEach(tasks) { task in
                    HStack {
                        Text(task.title)
                    }
                }
                .onDelete { indexSet in
                    for index in indexSet {
                        context.delete(tasks[index])
                    }
                }
            }
        }
        .padding()
    }
}
```



## WWDC25에서의 SwiftData 변화 (iOS 26+)

---

#### 1. 클래스 상속 지원

- 기존에는 SwiftData 모델은 상속을 지원하지 않았음
- 이제 상속 구조를 정의하고 저장/쿼리 가능
    - 공통 속성/기능을 상위 클래스에 정의하고, 세부적인 속성은 하위 클래스에서 확장

```swift
@Model
class Task {
    var title: String
    var isDone: Bool
    var createdAt: Date
}

@Model
class LongTask: Task {
    var period: Int
}
```

- Task 쿼리에 LongTask 포함 가능
- 타입 기반 필터링 가능
    - Query(filter: Task.self is LongTesk.self)
- 상속 모델을 사용할 때는 .modelContainer(for:)에 하위 클래스 포함시켜야 함
    - .modelContainer(for: [Task.self, LongTask.self])


> ✅ 언제 상속을 써야 할까?
> 
> - 모델들이 자연스러운 계층 관계를 가지고 있을 때
> - 상위 타입과 "is-a 관계" 를 형성할 때
> - 공통 속성만 공유하기 위해 상속을 사용하면 잘못된 설계임 → 공통 속성은 차라리 프로토콜로 관리해야 함



> ✅ Query 방식에 따른 상속 사용 여부
> 
> - Deep Search : 상위 타입(Task) 기준으로 전체를 쿼리
> - Shallow Search : 하위 타입(LongTask)만 별도로 쿼리
>   
>   Shallow Search의 경우에는 상속을 사용하지 않는 일반적인 방식이 적합하며, Deep Search 나 둘 모두를 사용하는 경우에는 상속을 사용하는 모델 구조가 더 적합하다.


#### 2. 마이그레이션 전략 명시 가능

SwiftData는 앱의 모델 구조 변경 시 데이터 유실 없이 마이그레이션할 수 있게 해준다. 이를 위해 버전 스키마(versioned schema)와 마이그레이션 단계(migration stage)를 정의한다.

|iOS 버전|주요 변경 사항|스키마 버전|
|---|---|---|
|iOS 17|초기 Trip 모델 도입|v1|
|개선된 iOS 17|이름 고유성 추가, 속성명 변경|v2|
|iOS 18|unique, index, 삭제 후 식별 가능성 추가|v3|
|iOS 26|상속 클래스 도입|v4|

- 다양한 스키마 버전을 선언하고 마이그레이션 순서를 정의할 수 있음
- MigrationStage를 이용해 버전별 전략 작성 가능함

#### 3. 성능 최적화를 위한 fetch 개선

- FetchDescriptor에 세부 옵션 추가 → 마이그레이션 시 불필요한 데이터 로딩 방지 위함
    - propertiesToFetch : 가져올 속성 지정
    - relationshipsToPrefetch : 관계형 데이터 미리 로딩
    - fetchLimit : 위젯과 같은 경량 작업에서 사용

```swift
static let migrateV1toV2 = MigrationStage.custom(
   fromVersion: TaskSchemaV1.self,
   toVersion: TaskSchemaV2.self,
   willMigrate: { context in
      var fetchDesc = FetchDescriptor<TaskSchemaV1.Task>()
      fetchDesc.propertiesToFetch = [\\.title, \\.createdAt]
      fetchDesc.fetchLimit = 100

      let tasks = try? context.fetch(fetchDesc)

      // 마이그레이션 작업

      try? context.save()
   },
   didMigrate: nil
)
```

#### 4. 변경 감지 개선: Observation + Persistent History

- 로컬 변경은 withObservationTracking으로 자동 감지 가능
- 다른 프로세스나 컨텍스트 변경은 자동 갱신되지 않으며, 직접 refetch 필요
- Persistent History API
  
    - 변경 이력 조회
    - 토큰 기반 최신 이력만 챙기기 (sortBy + fetchLimit)
    
    ```swift
    var historyDesc = HistoryDescriptor<DefaultHistoryTransaction>()
    historyDesc.sortBy = [.init(\\.transactionIdentifier, order: .reverse)]
    historyDesc.fetchLimit = 1
    
    let transactions = try context.fetchHistory(historyDesc)
    if let transaction = transactions.last {
        historyToken = transaction.token
    }
    ```
    
    - 관심 모델에 필터링하여 효율적 갱신 가능
    
    ```swift
    let tokenPredicate = #Predicate<DefaultHistoryTransaction> { $0.token > historyToken }
    
    let entityNames = [Task.self]
    let changesPredicate = #Predicate<DefaultHistoryTransaction> {
        $0.changes.contains { change in
            entityNames.contains(change.changedPersistentIdentifier.entityName)
        }
    }
    
    let fullPredicate = #Predicate<DefaultHistoryTransaction> {
        tokenPredicate.evaluate($0) && changesPredicate.evaluate($0)
    }
    
    let desc = HistoryDescriptor<DefaultHistoryTransaction>(predicate: fullPredicate)
    let newTransactions = try context.fetchHistory(desc)
    ```
    

## Keywords

---

- Swift Data
- [[Core Data]]

## References

---

[https://youngkdevlog.tistory.com/67](https://youngkdevlog.tistory.com/67)

[https://mini-min-dev.tistory.com/329](https://mini-min-dev.tistory.com/329)

[https://developer.apple.com/documentation/SwiftData](https://developer.apple.com/documentation/SwiftData)

[https://developer.apple.com/videos/play/wwdc2024/10137/](https://developer.apple.com/videos/play/wwdc2024/10137/)

[https://developer.apple.com/videos/play/wwdc2025/291/](https://developer.apple.com/videos/play/wwdc2025/291/)