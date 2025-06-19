### 앱 인텐트(App Intent)란?

WWDC 2022에서 AppIntents Framework로 소개 앱이 제공하는 주요 기능을 인텐트(intent)라는 형태로 정의하면, 사용자가 직접 실행하지 않아도 시스템 곳곳에서 그 기능을 바로 쓸수 있게 해주는 일종의 명령어 입니다.

예: 할일 추가, 음악 재생, 운동시작, 노트 열기

"Siri야, 오늘 할 일 추가해줘 " -> 할 일  앱의 인텐트가 동작해서 바로 할 일이 추가됩.


![[donate intents.jpg]]
앱에서 특정 인텐트에 해당하는동작이 발생하면 ==donate()== 메서드를 통해 iOS Framework에 알려줄 수 있습니다.

### 어떻게 쓸수 있나?

![[appIntent.png.webp]]
- Siri 음성 명령
- Spotlight 검색
- 단축어(Shortcuts) 앱
- 위젯, 액션 버튼, 제어 센터등의 다양한 시스템

### 인텐트, 엔티티, 쿼리 구조

- 인텐트(Intent) : 앱이 수행할 수 있는 동작 (동사 역할)
- 엔티티(Entity) : 앱이 다루는 데이터 (명사 역할 : 할 일, 메모, 연락처 등), Parameter를 정적, 동적으로 받을 수 있음
- 쿼리(Query) : 엔티티를 검색하거나 필터리 하는 기능 


```swift

// 엔티티
// 명소 데이터 정의
import AppIntents

// AppEntity 프로토콜을 사용하여 Siri/Spotlight/shortcuts 등에서 Landmark 데이터를 인식할 수 있게 함
struct Landmark: AppEntity, Identifiable {
	var id: String
	var name: String
	var description : String

	static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "Landmark")

	var displayRepresentation: DisplayRepresentaiton {
		DisplayRepresentation(titile: "\(name)", subtitle: "\(description)")
	}
}
```

```swift

// 쿼리
// 이름이나 설명에 특정 단어가 포함된 명소 찾기
// 사용자가 Siri나 Spotlight에서 "부산"이라 검색하면, 이름이나 설명에 "부산"이 포함된 명소만 추려서 보여줄수 있습니다. 
struct LandmarkQuery: EntityQuery {
	func entities(matching string: String) async throws -> [Landmark] {
		let allLandmarks = [
		{id: 0, name: "부산", description: "Gus가 다녀온 다이빙 풀장" }, 
		{id: 1, name: "홍대", description: "Gus가 사는 곳"}, 
		{id: 2, name: "포항", description: "Gus가 공부하는 곳"}] // 앱의 모든 명소 데이터 
		return allLandmarks.filter {$0.name.contains(string) || $0.description.contains(string)}
	}

	func entities(for identifiers: [String]) async throws -> [Landmark] {
		let allLandmarks = [
		{id: 0, name: "부산", description: "Gus가 다녀온 다이빙 풀장" }, 
		{id: 1, name: "홍대", description: "Gus가 사는 곳"}, 
		{id: 2, name: "포항", description: "Gus가 공부하는 곳"}]
		return allLandmarks.filter { identifiers.contains($0.id) }
	}
	
	func suggestedEntities() async throws -> [Landmark] { 
		// 예시: 사용자가 자주 방문한 명소 추천 
		return ... // 추천 명소 리스트 
	}
}

```
```swift

// 인텐트
// 특정 명소의 상세 정보 보기
// 사용자가 Siri에 "랜드마크 부산 타워 보여줘"라고 말하면, `LandmarkQuery`가 "부산 타워"를 찾아서, 해당 명소의 상세 정보를 보여주는 인텐트가 실행됩니다
struct ShowLandmarkDetailIntent: AppIntent { 
	static var title: LocalizedStringResource = "Show Landmark Detail" 
	
	@Parameter(title: "Landmark") 
	var landmark: Landmark 
	
	static var parameterSummary: some ParameterSummary { 
		Summary("Show details for \(\.$landmark)") 
	} 
	
	func perform() async throws -> some IntentResult { 
		// landmark의 상세 화면으로 이동 
		return .result() 
		} 
	}
```

### Shortcuts Action

- AppIntent를 구현하면 Shortcuts에서 Action으로 제공됩니다.
- 기본적인 형태는 이름, 동작부정도 구현하고 ==perform()== 메소드로 결과를 리턴하도록 되어 있습니다.


### Parameterized Action


- 단순히 고정된 동작을 수행하는 것이 아닌, Action에 대한 세부 설정을 할수 있습니다.
	예 : 단순 카메라 화면 띄우기 아닌 특정 촬영모드로 카메라 촬영화면 띄우기


### Home Screen Widget

- Widget 중에서 Configurable Widget이 있는데 , 여기에 ==AppIntentConfiguration, AppIntent, Entity==를 사용합니다.

### Control Center 

- ==ControlWidget==를 상속받아 만들고 여기에 AppIntent를 활용합니다.

### Spotlight, Siri

- ==AppShortcutsProvider==로 ==AppShortcut==을 제공하면 됨. AppShortcut은 인텐트를 지정해서알려주고, 이름, 아이콘, Siri를 위한 발화 인식 형태 리스트를 같이 전달하면 됩니다.
- 참고 :  https://developer.apple.com/videos/play/wwdc2024/10133/


```swift
import AppIntents

// App Intent를 사용하려면 최소한 title과 perform 메소드를 사용 해야함 
NavigateIntent: AppIntent {

	// title을 unique 해야함, title이 인텐트의 이름임
	static let title: LocalizedStringResource = "Navigate to Landmarks"

	// (새로운 기능) 인텐트는 기본적으로 foreground에서 동작하지 않음
	// 그러나 네비게이션은 foreground에서 동작해야하기 때문에 
	// IntentModes 사용하여 인텐트 사용전에 앱이 열리도록 처리
	static let suppoertedModes: IntentModes = .foreground 

	// 인텐트 파라미터로 사용하기 위해 @Parameter를 추가
	@Parameter var navigationOption; NavigationOption

	// perform은 인텐트 로직을 넣어야 한다. 
	// 여기서는 Landmarks view를 여는 로직을 넣음
	// navigation은 반드시 메인 스레드에서 사용해야기 때문에 @MainActor 사용
	@MainActor
	func perform() async throws -> some IntentResult {
		Navigator.shared.navigate(to: .landmarks)
		return .result() // 인텐트 결과 반환
	} 
}
```

```swift
enum NavigationOption: String, AppEnum {
	case landmarks
	case map
	case collecitons

	//  커스텀 타입의 UI 표현을 설명하는 타입
	static let typeDisplayRepresentation: TypeDisplayRepresentation = "Navigation Option"

	// enum의 각 case를 설명하기 위해 사용 
	static let caseDisplayRepresentations: [NavigationOption: DisplayRepresentation] = [
		.landmarks : "Landmarks",
		.map : "Map",
		.collections : "Collections" 
	]
}
// typeDisplayRepresentation와 caseDisplayRepresentationssms 컴파일시 이정보를 사용하기 위해 반드시 상수 값이여야함
```


```swift
import AppIntents

NavigateIntent: AppIntent {

	static let title: LocalizedStringResource = "Navigate to Section" // 수정

	static let suppoertedModes: IntentModes = .foreground 

	// 인텐트 파라미터로 사용하기 위해 @Parameter를 추가
	@Parameter var navigationOption; NavigationOption // 추가

	@MainActor
	func perform() async throws -> some IntentResult {
		// resoleved navigationOption 사용 
		Navigator.shared.navigate(to: .navigationOption) // 수정
		return .result() // 인텐트 결과 반환
	} 
}
```

<table>  
<tr>  
<td><img src="shortcut.png" width="200"></td> <td><img src="랜드마크 선택.png" width="200"></td> <td><img src="맵이동.png" width="200"></td> </tr>  
</table>

![[네비게이션심볼넣기.png]]

![[파라미터요약 실제 구동화면.png]]

![[파라미터 실제 구동화면.png]]

![[파라미터 스포트라이트 구동화면.png]]


### WWDC 2025에서 주요 변화는?

![[인텐트 통합기능.png]]


### 1. Interactive snippets


<table>    
<td><img src="interactive스니핏1.png" width="200"></td> <td><img src="interactive스니핏2.png" width="200"></td>  
</table>

Interactive snippets는 App Intent를 기반으로 맞춤형 정보를 표시하는 dynamic view들입니다. 예를 들어 정원 가꾸기 앱에서 스프링클러를 켜는 것을 제안하거나 음식 주문 앱에서 주문 전에 주문을 요청할 수 있습니다.
==즉 앱을 켜지 않아도 앱의 핵심 동작을 실행 및 확인할 수 있습니다.==

이런 기능은 'SnippetIntent' protocol을 사용하면 됩니다. 
- (새로 추가) Result snipped
- (새로 추가) Confirmation snippet 



![[스니핏 인텐트.png]]

1. App Intent 
2. ↓  Return result  (필요시 매번 스니핏을 리프레쉬함)
3. Parameterized Result Snipped Intent
4. ↓ populate parameters  (스니핏 인텐트로 전달 앱의 엔티티들이 쿼리에 의해 패치됨) 
5. Result Snippet Intent
6. ↓ ( 스니핏 인텐트의 perform 메소드를 실행함 )
7. SwiftUi view

```swift
// 2. 스니핏 인텐트 반환
struct ClosestLandmarkIntent: AppIntent {

	func perform() async throws -> some ReturnsValue<LandmarkEntity> 
	& ShowsSnippetIntent { // 추가
		let landmark = await self.findClosestLandmark()

		return .result (
			value: landmark
			snippetIntent: LandmarksnippetIntent(landmark: landmark) // 추가
		)
	}

}

```

```swift
// 5. 스니핏 인텐트 빌드
struct LandmarkSnippetIntent: SnippetIntent {
	static let title: LocalizedStringResource = "Landmark Snippet"

	@Parameter var landmark: LandmarkEntity // view 렌더링에 필요한 데이터를 넘겨줄 변수
	@Dependency var modelData: ModelData // AppDependencies의 접근 권한
	
	func perform() async throws -> some IntentResult 
	& ShowsSnippetView { // 추가

		let isFavorite = await modelData.isFavorite(landmark) // 상태 변환 확인 변수

		return .result(
			view: LandmarkView(landmark: landamark, isFavorite:isFavorite)
		)
	} 
}

```
```swift
// 7. 버튼들로 인텐트 연결
struct LandmarkView: View {
	var body: some View {

		Button(
			intent: UpdateFavoritesIntent(landmark: landmark, isFavorite: !isFavorite)
		){
		}

		Button(
			intent: FindTicketsIntent(landmark: landmark)
		){
		}
	}
}

```
![[스니핏 확인 요청1.png]]
![[스니핏 확인 요청2.png]]
8. 티켓을 찾기
9. 확인 스니핏 요청 

```swift
// 9. 확인 스니핏 요청
struct FindTicketsIntent: AppIntent {
	func perform() async throws -> some IntentResult & ShowSnippetIntent {
		let searchRequest = awatit searchEngine.createRequest(landmarkEntity: landmark)

	try await requestConfirmation( // 스니핏이 취소되면 에러를 던짐
		actionName: .search
		snippetIntent: TicketRequestSnippetIntent(searchRequest: searchRequest)
	)
	
	}
}
```


![[스니핏 확인  요청 3.png]]
10. 파라미터 스니핏 인텐트 확인 (업데이트 사이클) 
11.  스니핏 인텐트 확인

```swift
// 10. 파라미터로 엔티티 사용
// 검색 요청을 AppEntity로 모델링함
struct TicketRequestSnippetIntent: SnippetIntent {

	@Parameter var searchRequest: SearchRequestEntity

	func perform() async throws -> some IntentResult & ShowSnippetIntent {
		let view = TicketRequestView(searchRequest: searchRequest)
	
		return .result(view: view)
	}
}

```

```swift
// 스니핏 업데이트

struct performRequest(request: SearchRequestEntity) async throws {

	TicketResultSnippetIntent.reload()
	
}

```

### 2. New system intergrations


- Image search 
	말그대로 이미지로 검색가능한 기능으로, search pannel이 나타나고 해당 이미지 선택하면 이미지로 인한 검색 결과가 나타납니다. 
	
<table>    
<td><img src="이미지 서치 패널.png" width="200"></td> <td><img src="이미지 검색결과.png" width="200"></td>  
</table>

	 구현 방법은 IntentValueQeury 프로토콜을 사용하여 Image Search를 쿼리로 실행하고 
	 SemanticContentDescriptor타입으로 입력하고 App Entity를 배열 
	 형태로 반환하면 Image Search가 App Entity를 화면에 표시합니다. 사용자가 반환된 결과인 
	 App Entity 선택하면 선택한 App Entity를 OpenIntent로 보냅니다. OpenIntent가 있지
	 않으면 앱에서 Image Search의 결과를 절대로 볼수 없습니다.
	 
![[image search과정.png]]

```swift
// Image search 응답
// 쿼리는 아래 구조체로 만들어서 실행
struct LandmarkIntentValueQuery: IntentValueQeury {
	
	@Dependency var modelData: ModelData

	func value(for input: SemanticContentDescriptor) async throws -> [LandmarkEntity] { // 반드시 SemanticContentDescriptor 타입으로 입력해야함
	
		guard let pixelBuffer: CVReadOnlyPixelBuffer = input.pixelBuffer else { // 선택한 영역을 pixel로 넘겨줌
			return [] // 매치된 entitiy 결과는 반드시 배열로 반환
		}
		
		let landmarks = try await modelData.searchLandmarks(matching: pixelBuffer)

		return landmarks
	}
}


// AppEntity 열기
// 사용자가 탭한 결과를 보여주는 OpenIntent 프로토콜을 준수하는 인텐트 구현
// OpenIntent는 Image search 뿐만아니라 Spotlight도 지원함
struct OpenLandmarkIntent: OpenIntent { 
	static var title:LocalizedStringResource = "랜드마크 열기"

	@Parameter(title: "랜드마크")
	var target: LandmarkEntity // 반드시 결과와 타입이 같아야 함

	func perform() async throws -> some IntentResult {
		...
	}
}
```

- Onscreen Entities
	NSUserActivities 사용하여 앱에 엔티티를 화면 콘텐츠와 연결하여 현재 앱에 표시 할수 있습니다. 예를 들어 지도 앱에서 폭포를 보던중에 Siri로 가까이 있는 호수를 찾아달라고 물어보면 폭포와 연관된 호수를 찾아줍니다. 그리고 스크린샷으로 chatGPT에게 보내고 결과의 응답이 화면으로 표시 됩니다.  

```swift
// AppEntity를 View에 연결

struct LandmarkDetailView: View {

	let landmark: LandmarkEntity

	var body: some View {
		Group{/*..*/}
		.userActivity("com.landmarks.ViewingLandmark") { activity in
			activity.title = "Viewing \(landmark.name)" 
			activity.appEntityIdentifier = EntityIdentifier(for: landmark) // EntityIdentifier로 activity 연결
		}
	}
}

// chatGPT가 이해할수 있게 LandmarkEntity를 Data type로 converting 예를 들면 pdf
// AppEntity를 pdf로 converting 하기
import CoreTransferable
import PDFkit

extension LandmarkEntity: Transferable { 
	static var transferRepresentation: some TransferRepresentation {
		DataRepresentation(exportedContentType: .pdf) { landmark in
			// Create PDF data...
			return data
		}
	}
}


```
- Spotlight
	Spotlight가 검색 필터링 환경을 주도할 수 있도록 IndexedEntity로 확인하고 Spotlight로 넘깁니다.
	
```swift
// 프로퍼티를 spotlight key로 연결
struct LandmarkEntity: IndexedEntity {
	// ...
	@Property(indexingkey: \.displayName)
	var name: String

	@Property(customIndexingKey: /*...*/)
	var continent: String 
}
```

	엔티티로 화면 콘텐츠에  Annotate를 달면 해당 엔티티가 View에 표시될 때 제안에서 우선 순위가 
	지정됩니다.

	PredictableIntent 실행하면 시스템이 학습하고 기본적으로 사용자의 이전 행동을 바탕으로 
	intent와 prameter을 제안하여 제공합니다. 

	value에 따라 맞춤형 설명도 제공 가능합니다. 

### 3.User experience refinements

- undo

	인텐트를 이전으로 다시 복구되는 기능으로  UndoableIntnet protocol을 사용하여 코드를 구현하면 됩니다. 

```swift
// 인텐트 undo 가능하게 하기

struct DeleteCollectionIntent: UndoableIntent {

// 삭제 확인

// undomanager로 undo 액션을 등록
await undoManager?.registerUndo(withTarget: modelData) { modelData in
	// collection 복구
}

// collection 삭제 

}

```

- Multiple Choice

	사용자에게 여러 선택 옵션을 제공하기 위한 API 입니다.


```swift
struct DeleteCollectionIntent: UndoableIntent {

	func perform() async throws -> some IntentResult & ReturnsValue<CollectionEntity?> {
		let archive = Option(title: "Archive", style: .default)
		let delete = Option(title: "Delete", style: .destructive)

	let resultChoice = try await requestChoice (
		between: [.cancel, archive, delete],
		dialog: "\(collection.name) Archive 또는 Delete 둘중 어떤거 선택할래?",
		view: collectionSnippetView(collection)
	
	)

	// 여기서 사용자가 선택한 옵션을 switch문으로
	switch resultChoice {
		case archive : // archive 선택시 로직
		case delete : // delete 선택시 로직
		default: // 아무것도 안함
	}
	}

}

```
- Supported Modes

	foregrouding 상태에서 인텐트를 좀더 멋지게 제어하는 방법입니다.

```swift
	struct GetCrowdStatusIntent: AppIntent {
		
		static let supportedModes: IntentModes = [.backgroud, .foreground]

		func perform() async throws -> some ReturnValue<Int> & ProvidesDialog {
			// 인텐트는 foreground 에서만
			if systemContext.currentMode == .foreground {
				await navigator.navigateToCrowdStatus(landmark)
			}
		}
		
	}

```

background 모드와  3가지 foreground 모드를 제공합니다.

Foreground
- Immediate :  인텐트 실행전에  시스템에게 앱을 foreground 하라고 지시 (즉시 실행)
- Dynamic: 인텐트가 앱의 실행여부를 결정하도록 허용함 (앱을 실행하지 않는다면 이걸 선택)
- deferred: 인텐트가 앱을 결국 foreground될것이라  표시  (즉시 실행은 아님)

위 2가지 모드는 continueInForeground 메서드를 사용하면 앱을 forward 할때 정확하게 제어가 가능합니다.


```swift
	struct GetCrowdStatusIntent: AppIntent {
		
		static let supportedModes: IntentModes = [.backgroud, .foreground(.dynamic)]

		func perform() async throws -> some ReturnValue<Int> & ProvidesDialog {
			guard await modelData.isOpen(landmark) else {/*일찍 종료*/}

			if systemContext.currentMode.canContinueInForeground {
				do {
					try await continueInForeground(alwaysconfirm: false)
					await navigator.navigateToCrowdStatus(landmark)
				} catch {
					//  사용자 또는 시스템 앱열기 거부 할경우~
				}
			}
		}
		
	}

```

### 4.  Convenience APIs


- View Control

	앱 인텐트가 UI navigation을 사용하려면 View들 구동하는 모든 앱상태에 access가 가능해야 하는데,  전역으로 사용하는 AppDependencie 또는 singleton과 같은 shared objects로 해결했습니다. 그러나 새로운 View Control API를 사용하면 그딴 코드는 이제 날려도 됩니다.
	
	onAppIntentExecution는 앱이 foreground 전에 실행, 인텐트 파라미터로만 읽고, 모든 활성화된 
	뷰들의 각각 이벤트를 받을수 있습니다.

```swift
// targetContentProvidingIntent

extension OpenLandmarkIntent: TargetContentProvidingIntent {

}


// 인텐트 실행으로 응답 받는 뷰
struct LandmarksNavigationStack: View {

	@State var path: [Landmark] = []

	var body: some View {
		NavigationStack(path: $path) {}
		.onAppIntentExecution(OpenLandmarkIntent.self) { intent in // 제어하기 위한 앱 인텐트 타입
			self.path.append(intent.landmark) // 인텐트 파라미터를 참조
		}
	}
}


// 앱 인텐트 에서 UI 코드 제거
struct OpenLandmarkIntent: OpenIntent {
 
	static let title: LocalizedStringResource = "랜드마크 열기"


	@Parameter(title: "랜드마크", requestValueDialog: "랜드마크 어디인데?")
	var target: LandmarkEntity

	@Dependency var navigator: Navigator // 이제 필요 없음

	@MainActor
	func perform() async throws -> some IntentResult {
		navigator.navigate(to: landmark)
	return .result()
	} // 이제 필요 없음
}

extension OpenLandmarkIntent: TargetContentProvidingIntent {}

```
	handlesExternalEvents로 화면에 인텐트 활성화 조건을 설정 할 수 있습니다.
```swift

// 예시 1 Scene activation condition
@main
struct AppIntentsTravelTrackerApp: App {
	var body: some Scene {
		WindowGroup {}

		WindowGroup {}
		.handlesExternalEvents(matching: [ // 배열로 매칭
			OpenLandmarkIntent.persistentIdentifier // 만들어둔 content identifier
		])
	}
}

// 예시 2 View activation condition
struct LandmarksNavigationStack: View {
	var body: some View {
		 NavigationStack(path: $path) {}
		 .handlesExternalEvents(
			 perferring: [],
			 allowing: !isEditing ? [OpenLandmarkIntent.persistentIdentifier] : []
		 )
	} 
}
```
	uikit을 사용하는 경우에는
	UISceneAppIntent 로 앱 인텐트를 확인
	uiScene과 performNavigation(forScene:) scenes을 제어
	AppIntentSceneDelegate로 Scene delegate 확인
	func scene(willPerformAppIntent:) 이벤트 제어
	UISceneActivationConditions로 조건 설정 하면 됩니다.



- Computed Property 매크로

	Property를 AppEntity에 값을 저장하지 않고 사용하기위해 사용합니다.

```swift

// Computed Property 사용전 
struct SettingEntity: UniqueAppEntity {

	@Property
	var defaultplace: PlaceDescriptor

	init() {
		self.defaultPlace = UserDefaults.standard.defaultPlace
	}
}


// Computed Property 사용후 
struct SettingEntity: UniqueAppEntity {

	@ComputedProperty
	var defaultplace: PlaceDescriptor{
		UserDefaults.standard.defaultPlace  // getter를 바로 접근
	}

	init() {
		
	}
}

```

- Deferred Property 매크로

	낮은 비용으로 AppEntity를 인스턴스화합니다/

```swift

// Deferred Property 사용전 
struct LandmarkEntity: IndexedEntity {

	@Property
	var crowdStatus: Int // 서버와 통신후 fetching 되는 Property
	
}

// Deferred Property 사용후
struct LandmarkEntity: IndexedEntity {

	@DeferredProperty
	var crowdStatus: Int { // 시스템 기능이 명시적인 요청이 있을때만 fetching
		get async throws { // LandmarkEntity 생성되어 전달될때는 동작 안함
			await modelData.getCrowdStatus(self)
		}
	}
	
}
```

- Swift Package
	 Swift Package과 static library로 사용 가능합니다. 

![[swift package.png]]


```swift
// 프레임워크 또는 동적 라이브러리
public struct LandmarkskitPackage: AppIntentsPackage {}


// App target
struct LandmarksPackage: AppIntentsPackage {
	static var includedPackages: [any AppIntentsPackage.Type] {
		[LandmarksKitPackage.self]
	}
}

```

https://dev-repository.com/?p=613
https://developer.apple.com/videos/play/wwdc2025/244/?time=167
https://developer.apple.com/videos/play/wwdc2025/275/