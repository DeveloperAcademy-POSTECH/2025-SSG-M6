>[!question]
>GQ1. Realm은 어떤 구조 덕분에 빠르다고 평가받을까?
>GQ2. Realm은 어떤 작업에서 유리할까?
>GQ3. Realm의 단점이나 제한점은 무엇일까?

## Description
---
Realm이 빠르다는 평가를 받는 구조적 근거를 알아보고, 실제로 SwiftData, [[Core Data]], SQLite와 어떤 작업에서 성능 차이를 보이는지 실험을 통해 확인한 내용을 정리한 것이다. 

초기에는 "Realm이 빠르다는 건 어떤 의미일까?"라는 질문에서 출발해, 읽기/쓰기/삭제/조회 등 다양한 작업 중 Realm이 특히 **쓰기 성능에서 유리하다는 점**에 주목했다.

Realm의 빠른 쓰기 성능을 만들어내는 핵심 구조로는 `트랜잭션 기반 처리`, `제로-카피 아키텍처`, 그리고 `메모리 매핑(Memory-Mapped File)`이 있다. 이 요소들은 직렬화 비용과 디스크 I/O 횟수를 줄여주며, 특히 수천~수만 개의 객체를 반복 저장하는 상황에서 SwiftData 대비 안정적인 속도를 유지할 수 있도록 한다.

## 1. Realm의 성능이 좋은 이유
---
#### 1. 트랜잭션 기반 처리 
Realm의 쓰기 성능에서 가장 핵심적인 요소는 트랜잭션 처리 구조다. 트랜잭션 시작과 끝 사이에 수행되는 모든 변경들은 메모리 안의 버퍼에 임시 저장되며, 최종적으로 한 번에 디스크에 반영된다.

이 구조 덕분에 수십수백 개의 객체도 단 한 번의 디스크 I/O로 저장할 수 있고, 반복적인 `save()` 호출이 필요한 CoreData/SwiftData보다 훨씬 빠르게 일괄 저장이 가능하다.

>[!info] 트랜잭션 중간에 앱이 꺼지면 어떻게 되는가?
>트랜잭션 처리 구조란 하나의 작업 단위로 묶어서 전부 성공하거나, 전부 실패시키는 처리 방식으로 데이터 일관성을 지키기 위한 모든 DB 시스템의 기본이다.
>
>예를 들어 100개의 데이터를 저장할 때,
>- 트랜잭션이 없다면 → 50개 저장되고 앱이 꺼지면 나머지 50개는 사라진다
>- 트랜잭션이 있다면 → 100개를 다 성공해야 저장, 중간에 끊기면 아예 저장 안 된다
>
>Realm은 트랜잭션이 완료되기 전에는 실제 파일에 반영하지 않는다. 이로 인해 앱이 중간에 종료되더라도, **트랜잭션 단위로  롤백**되어 이전 상태로 유지된다. 
>또한 내부적으로 자체 트랜잭션 로그 구조를 사용하여, **write-ahead log (WAL)** 과 유사한 원리로 작동한다. 이를 통해 예기치 않은 종료에도 정합성과 복구가 가능한 구조를 제공한다.

>[!question] Realm은 WAL을 직접 사용하진 않는다!
>Realm은 전통적인 WAL을 채택한 것은 아니지만, **비슷한 철학을 반영한 트랜잭션 기반 복구 시스템**을 갖추고 있다.
>여기서 **트랜잭션 기반 복구 시스템**이란, 변경 사항을 바로 반영하지 않고 **로그를 통해 작업을 관리한 뒤, 커밋 여부에 따라 적용 여부를 결정하는 방식**을 말한다. 이 시스템의 대표적인 구현 방식 중 하나가 바로 **WAL(Write-Ahead Logging)**이다.
>
>* Realm에서는 realm.write {} 블록 내에서 수행된 변경사항이 즉시 디스크에 반영되지 않고, 먼저 트랜잭션 로그 형태로 기록된다.
>* 이 로그는 트랜잭션이 성공적으로 커밋되기 전까지 메인 데이터에 적용되지 않는다.
>* 앱이 중간에 종료되더라도, 커밋되지 않은 내용은 반영되지 않는다.
>* 다음 실행 시 마지막 커밋된 안전한 상태로 복구된다.
>
>따라서 Realm은 전통적인 WAL을 채택한 것은 아니지만, 비슷한 철학을 반영한 트랜잭션 기반 복구 시스템을 갖추고 있다. (따로 이름이 있진 않다!)

>[!question] WAL(Write-Ahead Logging)은 어떤 구조인가?
>WAL은 전통적인 관계형 데이터베이스(RDB) 시스템에서 흔히 사용되는 트랜잭션 로그 구조이다. 대표적으로 SQLite, PostgreSQL, MySQL 등이 WAL 구조를 사용한다. 
>
>과거에는 `DB 반영 → 로그 작성` 순으로 진행해서 DB 변경하고 시스템이 다운되어버리면 데이터는 수정되었는데 로그 기록은 남지 않아 복구가 불가능했다. 
>WAL 방식은 `로그 작성 → DB 반영` 순이어서 요청이 들어오면 로그를 먼저 작성하기 때문에 중간에 꺼져도 WAL의 기록을 보고 다시 적용(redo)하거나 무시(rollback)할 수 있다. 만약에 로그를 쓰다가 꺼진다면 해당 트랜잭션은 커밋되지 않은 상태로 간주하고 복구 시 무시(rollback)하므로 안전성이 보장된다.

>[!info] SwiftData는 이걸 어떻게 처리할까?
>SwiftData는 modelContext라는 작업 공간에 변경 사항이 축적되고, 저장은 필요 시점에 save()로 커밋하는 구조다. 다만 SwiftData는 “개발자가 꼭 save()를 명시적으로 하지 않아도 된다”는 점을 특징으로 내세운다. 이 말은 곧, SwiftData가 내부적으로 감지해 자동으로 `save()`를 호출하는 타이밍을 결정한다는 뜻이다.
>
>SwiftData에서 자동 저장이 일어나는 상황은 다음과 같다. (사용자가 입력을 끝냈다고 판단되는 시점)
>- 사용자가 뒤로가기 했을 때
>- 텍스트 필드와 같은 곳에 입력을 마쳤을 때
>
>자동 저장이 보장되지 않는 상황도 있다. 
>- 입력 중 앱이 종료될 때
>- 포커스를 옮기지 않았을 때
>- Navigation 없이 단일 화면 앱일 때 (뒤로 가기 없이)
>- @Bindable 없이 수동으로 모델을 변경 했을 때 (modelContext 사용해서 수동 저장해야함!)
>- View의 상태가 SwiftData 모델과 분리되어 로컬 상태로 사용할 때

#### 2. Zero-Copy 아키텍처
일반적인 데이터베이스는 객체를 디스크에 저장할 때, 먼저 해당 객체를 저장 가능한 형태인 바이트 스트림으로 변환한다. 이 과정을 직렬화(Serialization)라고 한다. 직렬화를 거친 데이터는 디스크에 저장되거나, 네트워크로 전송될 수 있다. 하지만 이 과정에서 CPU 연산과 메모리 복사 비용이 수반되고, 객체 수가 많거나 모델 구조가 복잡할수록 저장과 불러오기 성능에 부담이 커진다.

Realm은 이 직렬화 과정을 생략한다. 즉, _객체 → 변환 → 저장_ 이 아니라 _객체 → 직접 저장_ 의 방식으로 동작한다. 이를 통해 CPU 사용량과 메모리 소비를 줄이고, 쓰기/읽기 성능을 향상시킬 수 있다.

일반적인 SQLite나 Core Data에서는 Swift 객체를 INSERT INTO 구문으로 저장하기 위해 내부적으로 아래와 같이 SQL로 변환된 쿼리를 실행해야한다. 이 과정에서 Swift 객체는 **명시적으로 문자열 또는 정수 값으로 추출되어 SQL에 맞는 타입으로 변환**된다. 이 또한 직렬화의 일종이다.

```
let query = "INSERT INTO todos (title, is_done) VALUES (?, ?)"
```

>[!question] Realm은 디스크 저장이나 네트워크 전송이 불가한가요?
>
> 그렇지 않다! 
> 
> 전통적인 데이터베이스에서는 객체를 디스크에 저장하거나 네트워크로 전송하기 위해 
> `객체 → JSON, XML 등의 포맷으로 변환 → 저장/전송` 과정을 거친다. 이때 데이터를 복사하고 변환하는 연산이 필요하다.
> 
> Realm은 위와 같은 범용 직렬화 포맷을 사용하지 않는다. 대신 Realm 전용 바이너리 파일 포맷을 사용하여, Swift 객체의 메모리를 그대로 반영하여 저장할 수 있도록 설계되어있다. (직접 참조) 이러한 구조 덕분에 메모리 포인터 기반으로 빠르게 읽고 쓸 수 있으며, 직렬화와 역직렬화 과정에서 발생하는 비용을 줄일 수 있다.
> 
> 엄밀히 말하면 Realm 역시 저장을 위한 변환 과정, 즉 직렬화를 수행한다. 다만, 이는 일반적인 범용 포맷을 사용하는 것이 아닐 뿐!
> 
>네트워크 전송의 경우는 Realm 객체로는 곧바로 네트워크 전송이 불가능하다. 그래서 JSON으로 변환하는 과정을 거쳐야한다. 
>
>이 구조는 Realm이 스토리지 최적화에는 강점이 있지만, 통신 계층과의 연결에는 명시적인 변환이 필요하다는 점을 의미한다. 


## 2. Realm의 단점 및 제한점
---
1. Realm 파일은 Realm에서만 쓸 수 있다.
    - JSON, SQLite처럼 다른 시스템과 직접 공유 불가
    - 백엔드와의 데이터 연동이 필요할 경우 불리함
2. SwiftUI에서 코딩 복잡도가 증가한다
    - @Model, @Query 같이 간단한 바인딩 구조가 없고 따로 관리해줘야한다.
3. 비동기/스레드 동시성 관리가 번거롭다.
    - Realm은 “이 객체는 지금 스레드에서만 써!”라고 기억함
    - 다른 스레드(ex. 비동기 코드 안)에서 쓰면 앱이 크래시
    - SwiftData는 내부에서 이걸 자동으로 해결해주기 때문에, 신경 안 써도 됨
4. 단순한 앱에서는 오히려 과하다.
    - Realm은 고성능이지만, 초기화 비용, 앱 크기 증가 있음
    - 데이터량이 적은 앱에서는 SwiftData가 더 간단하고 효율적

## 코드 예시
---
```Swift
import RealmSwift

class RPerson: Object { // Realm용 모델
    @Persisted var name: String
    @Persisted var age: Int
}
```

```Swift
import SwiftData

@Model
class SPerson {
    var name: String
    var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}
```

```Swift
// Realm 실험 코드
let realm = try! Realm()
let people = (0..<1000).map {
    let p = RPerson()
    p.name = "User \($0)"
    p.age = Int.random(in: 20...40)
    return p
}

try! realm.write {
    realm.add(people)
}

// SwiftData 실험 코드
for i in 0..<1000 {
    let person = SPerson(name: "User \(i)", age: Int.random(in: 20...40))
    modelContext.insert(person)
}

try? modelContext.save()
```

## 실험 요약
---
📌 Realm은 대량 저장/삭제/반복 쓰기에서 실제 성능 차이를 보여줌
* 데이터 1000개 → SwiftData가 더 빠른 결과 (지연 저장으로 인한 착시 가능성 있음)
* 데이터 10000개 이상 → SwiftData는 context 처리 속도 급감, Realm은 성능 유지

## Keywords
---
* 트랜잭션
* Zero-Copy

## References
---
- [Realm 공식 문서 – Realm by MongoDB](https://www.mongodb.com/docs/realm/sdk/swift/)
- [SwiftData vs Realm: Performance Comparison – Emerge Tools](https://www.emergetools.com/blog/posts/swiftdata-vs-realm-performance-comparison)
- [Realm vs SQLite (2025) – Orangesoft](https://orangesoft.co/blog/realm-vs-sqlite)
- [CoreData vs Realm: iOS DB 선택 가이드 – Agilie](https://agilie.com/blog/coredata-vs-realm-what-to-choose-as-a-database-for-ios-apps)
- [Realm 기본 개념 및 예제 – UKSeung 블로그](https://ukseung2.tistory.com/entry/Swift-Realm-%EA%B8%B0%EB%B3%B8-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EA%B0%84%EB%8B%A8-%EC%98%88%EC%A0%9C)
- [SwiftUI + Realm 사용기 – 이택 블로그](https://leetaek.tistory.com/15)
- [Realm이란? Swift 적용 가이드 – odinios 블로그](https://odinios.tistory.com/7)
- [Realm으로 메모장 만들기 – velog simh3077](https://velog.io/@simh3077/realm%EC%9C%BC%EB%A1%9C-%EA%B0%84%EB%8B%A8%ED%95%9C-%EB%A9%94%EB%AA%A8%EC%9E%A5-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0)
