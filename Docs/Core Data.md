>[!question]
>GQ1. Core Data란 무엇일까? DB와 DBMS랑 어떤 점에서 다르다고 하는거지?
>GQ2. SwiftUI에서는 Core Data를 사용하기 위해 어떤 작업이 필요한거지?

## Description

- Core Data는 Apple의 데이터 관리 프레임워크입니다.
- 여기서 말하는 "데이터 관리 프레임워크"라는 것은 Core Data가 데이터베이스 그 자체가 아닌 / iOS 앱에 존재하는 데이터 객체의 상태를 정의, 관리하고, 이를 영속적 (Persistence)으로 저장할 수 있도록 돕는 기술이라는 것을 의미하죠.

>[!question] Core Data와 DB 혹은 DBMS 사이의 관계 이해하기
>
>- **Core Data** : Apple이 제공하는 객체 상태 관리 (Object Graph Management) 프레임워크
>- **Database (DB)** : 데이터를 구조화해서 저장하는 파일 (예를 들어, SQLite 파일)
>- **DBMS (Database Management System)** : DB를 생성, 조작, 관리할 수 있는 소프트웨어 시스템 (예를 들어, SQLite, PostgreSQL, MySQL 등)
>  
>  즉, Core Data는 DB가 아니라 DB의 기능을 수행할 수 있는 역할을 갖고 있는 프레임워크.
>  Core Data는 내부적으로 DB (SQLite)를 사용합니다. 하지만 Swift 수준에서 SQL을 직접 다룰 일은 없죠!


- Core Data는 제공하는 핵심 기능은 아래와 같은 것들입니다.
	- **Persistence (영속성)** 
	  : 앱이 꺼진 이후, 혹은 기기가 재시작되더라도 데이터가 사라지지 않고 남아있습니다. 
	  객체를 생성, 수정한 후 `context.save()`를 호출하면 그 객체가 영구 저장소 (SQLite, Binary)에 기록되어 앱이 꺼져도 유지될 수 있습니다. 데이터베이스를 직접 관리하지 않는 방법으로요!
	  
	- **Undo and redo of individual and batched changes (실행 취소)** 
	  : Core Data의 실행 취소 관리자는 변경 사항을 추적하고 있기 때문에, 개별적 혹은 그룹으로 롤백 (앱의 실행 취소 기능)을 쉽게 구현할 수 있습니다.
	  
	- **Background data tasks (백그라운드 데이터 작업)** 
	  : UI를 방해할 수 있는 데이터 작업을 백그라운드에서 수행합니다. 그런 다음 결과를 캐시하거나 저장하여 서버 왕복 시간을 줄일 수 있습니다.
	  
	- **View synchronization (뷰 동기화)** 
	  : UIKit의 UITableView, UICollectionView에 대한 DataSource를 제공하여 뷰와 데이터를 동기화된 상태로 유지하는데 기여할 수 있습니다.
	  
	- **Versioning and migration (버전 관리 및 마이그레이션)** 
	  : 데이터 모델 버전을 관리하고 사용자 데이터를 마이그레이션하는 메커니즘이 포함되어 있습니다.



## 핵심 개념 (Core Data Stack)

+ ##### NSPersistentContainer (📦 Core Data의 가장 큰 구조 )
  : Core Data의 주요 컴포넌트를 캡슐화하여 포함하고 관리하는 영구 컨테이너
  Core Data의 구성요소들은 직접 초기화해줄 것 없이 / 앱이 시작될 때 자동으로 초기화되어 Core Data의 표준 진입점 (entry point)이 됩니다.

그렇다면 Persistent Container가 담고 있는 하위 Component들이 무엇인지 알아볼까요?

- ##### Model (NSManagedObjectModel, .xcdatamodeld 파일)
  : Core Data의 데이터 모델 (Scema)에 해당하는 개념입니다.
  엔티티(Entity), 속성(Attribute), 관계(Relationship) 등의 구조를 정의할 수 있죠. (어떤 데이터를 저장하고? 그 데이터는 어떻게 서로 연결되어있지?를 정의하는 공간)

- ##### Context (NSManagedObjectContext)
  : Core Data의 임시 작업 공간입니다.
  데이터의 생성, 삭제, 수정 등 CRUD 작업을 주로 수행하면서 변경사항을 계속 추적하고 있죠. 
  실제로 `context.save()`가 호출되기 전까지는 데이터베이스에 반영되지 않고, 임시적으로 context에만 남아있는 셈입니다.

- ##### Store Coordinator (NSPersistentStoreCoordinator)
  : 실제로 영구적으로 데이터를 저장하는 **NSPersistentStore**와 연결되어 있는 곳입니다.
  context의 변경사항을 store에 반영하거나, store의 fetch 요청을 수행하는 등의 "모델과 데이터베이스 사이 중재자" 역할을 수행하는 곳입니다.

![[CoreData_Stack.png]]



## SwiftUI에서 Core Data를 사용하는 예시

+ **SwiftUI + Core Data 환경에서 사용하는 PersistenceController** 구조체 생성
+ 해당 구조체가 Core Data의 전체 스택 구조를 구성하고 있고, Core Data를 초기화하는 단일 진입점 (Single Point of Access) 역할을 합니다.
+ [[싱글톤 패턴 (Singleton Pattern)]]으로 해당 Controller 객체에 이후 접근을 수행하게 되는거죠.

```Swift
import CoreData

struct PersistenceController {
    static let shared = PersistenceController()

    let container: NSPersistentContainer

    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "BeyondTheLine_iOS")
        if inMemory {
            container.persistentStoreDescriptions.first!.url = URL(fileURLWithPath: "/dev/null")
        }
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        container.viewContext.automaticallyMergesChangesFromParent = true
    }
}
```

- @main 구조체에서 `.environment(\.managedObjectContext, ...)`로 context (`NSManagedObjectContext`)를 주입합니다.

```Swift
import SwiftUI

@main
struct BeyondTheLine_iOSApp: App {
    let persistenceController = PersistenceController.shared
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, persistenceController.container.viewContext)
        }
    }
}
```

- 엔티티 생성하기 (`.xcdatamodeld` 파일로 설정)

![[CoreData_xcdatamodeld.png]]


- 실제 View에서의 사용방법
	- `viewContext` : @main 구조체에서 정의했던 viewContext 환경객체를 KeyPath로 불러와 사용
	- `items` : `@FetchRequest`와 `FetchResults<NSManagedObjectModel>`를 사용해 Core Data에서 사용하는 데이터 모델을 불러와 바인딩
	- `NSManagedObjectModel(context: viewContext)`로 viewContext에 데이터 추가 -> 이후, `viewContext.save()` 명시적 호출로 영구 저장소에 저장
	- `viewContext.delete`로 viewContext에 데이터 삭제 -> 이후, `viewContext.save()` 명시적 호출로 영구 저장소에 저장


```Swift
import SwiftUI
import CoreData

struct ContentView: View {
	// @main에서 환경 객체로 주입한 Context
    @Environment(\.managedObjectContext) private var viewContext

    @FetchRequest(
        sortDescriptors: [NSSortDescriptor(keyPath: \Item.timestamp, ascending: true)],
        animation: .default
    )
    private var items: FetchedResults<Item>

    var body: some View {
        NavigationView {
            List {
                ForEach(items) { item in
                    NavigationLink {
                        Text("Item at \(item.timestamp!, formatter: itemFormatter)")
                    } label: {
                        Text(item.timestamp!, formatter: itemFormatter)
                    }
                }
                .onDelete(perform: deleteItems)
            }
            .toolbar {
#if os(iOS)
                ToolbarItem(placement: .navigationBarTrailing) {
                    EditButton()
                }
#endif
                ToolbarItem {
                    Button(action: addItem) {
                        Label("Add Item", systemImage: "plus")
                    }
                }
            }
            Text("Select an item")
        }
    }

	/// Create : Item 항목 추가하기
    private func addItem() {
        withAnimation {
            let newItem = Item(context: viewContext)
            newItem.timestamp = Date()
            do {
                try viewContext.save()
            } catch {
                let nsError = error as NSError
                fatalError("Unresolved error \(nsError), \(nsError.userInfo)")
            }
        }
    }

    /// Delete : Item 항목 삭제하기
    private func deleteItems(offsets: IndexSet) {
        withAnimation {
            offsets.map { items[$0] }.forEach(viewContext.delete)
            do {
                try viewContext.save()
            } catch {
                let nsError = error as NSError
                fatalError("Unresolved error \(nsError), \(nsError.userInfo)")
            }
        }
    }
}
```



## References

- [Apple Developer Documentation - Core Data](https://developer.apple.com/documentation/coredata)