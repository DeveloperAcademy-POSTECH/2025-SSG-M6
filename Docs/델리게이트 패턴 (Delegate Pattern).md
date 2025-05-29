>[!question]
>GQ1. Delegate 패턴이 뭘까?
>GQ2. Delegate 패턴은 왜 필요할까?

## Description
- **Delegate 패턴**이란 **클래스** 또는 **구조체**가 **자신의 역할이나 책임을 다른 타입의 인스턴스에게 넘겨주거나, 위임**할 수 있게 해주는 디자인 패턴이다.
- **Delegate**: 대리인

> [! SSG에서의 예시]
> **Lumi(Delegator)**: SSG 스터디 책임자, 팀장들에게 스터디 관리 책임을 넘겨줌
> **각 팀별 팀장(Delegate)**: 각 팀의 스터디를 관리함
> **위임된 책임**: 스터디 관리 업무
> 
> ➡️ **책임이 분산**되고 **역할이 명확**해져 효율적인 스터디 관리가 가능해짐

## 주요 기능
#### ✨ 공통 구조
- `SomeDelegate`는 **역할(기능)** 을 약속하는 **프로토콜**
- `A`는 일을 직접 하지 않고, **delegate에게 위임**
- `A`는 `delegate?.didSomething()`으로 **대신 행동해줄 대리인(delegate)을 호출**
- `B`는 `SomeDelegate`를 채택하고, **실제로 일을 하는 객체**
- 외부에서 `a.delegate = b`로 **A에게 "B가 대신 일할 거야"라고 연결**
- `a.doWork()`을 호출하면, A는 B에게 일을 시키고 → B가 `didSomething()`을 실행
```swift
protocol SomeDelegate: AnyObject {
    func didSomething()
}

// A: delegator, 일을 위임
class A {
    weak var delegate: SomeDelegate?
    func doWork() {
        delegate?.didSomething()
    }
}

// B: delegate, 일을 대신 함
class B(Delegate): SomeDelegate {
    func didSomething() {
        print("B가 대신 일함!")
    }
}

let a = A()
let b = B()
a.delegate = b
a.doWork()  // 출력: B가 대신 일함!
```
#### 1️⃣ `UITableViewDelegate` + `UITableViewDataSource` 예제
- **UITableViewDelegate**, **UITableViewDataSource는** UIKit에서 제공하는 내장 프로토콜
```swift
import UIKit

class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
	let tableView = UITableView()
	let learners = ["Jam", "Mini", "Gus", "Frank", "Berry", "Glowny", "Jay"]

	override func viewDidLoad() {
		super.viewDidLoad()
        
        // 테이블 뷰 셋업
        tableView.frame = view.bounds
        view.addSubview(tableView)
        
        tableView.delegate = self       // 행동 위임
        tableView.dataSource = self     // 데이터 공급
        
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
	}
	
    // ✅ DataSource 필수 메서드
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return learners.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = learners[indexPath.row]
        return cell
    }

    // ✅ Delegate 선택 메서드
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        print("👉 Selected Learner: \(learners[indexPath.row])")
    }
}
```

#### 2️⃣ 셀 안의 버튼 눌렀을 때 알리기 (뷰 내부 액션 전달) 예제
- **커스텀 셀 내부**에서 “⭐️” 버튼을 눌렀을 때, 어떤 셀이 눌렸는지 **부모 뷰컨트롤러에 알림**
- **FavoriteCellDelegate라는 프로토콜을 만들어 사용**함
##### FavoriteCellDelegate 프로토콜 정의
```swift
protocol FavoriteCellDelegate: AnyObject {
    func didTapFavoriteButton(in cell: UITableViewCell)
}
```

##### FavoriteCell 커스텀 셀
```swift
class FavoriteCell: UITableViewCell {

    // ✅ delegate: B역할, 일을 대신 해줄 객체
    weak var delegate: FavoriteCellDelegate?

    private let favoriteButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("⭐️", for: .normal)
        return button
    }()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
        setupLayout()  // 버튼 UI 배치
        // favoriteButtonTapped() 연결
        favoriteButton.addTarget(self, action: #selector(favoriteButtonTapped), for: .touchUpInside)
    }

    private func setupLayout() {
	    //UI 코드 생략
    }

    // ✅ 버튼이 눌리면 delegate에게 알려줌
    // 이 시점에 ViewController(B)의 didTapFavoriteButton 메서드가 호출됨
    @objc func favoriteButtonTapped() {
        delegate?.didTapFavoriteButton(in: self)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

##### ViewController
```swift
class ViewController: UIViewController, UITableViewDataSource, FavoriteCellDelegate {

    let tableView = UITableView()
    let learners = ["Jam", "Mini", "Gus", "Frank", "Berry", "Golwny", "Jay"]

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        setupTableView()
    }

    private func setupTableView() {
        view.addSubview(tableView)

        tableView.snp.makeConstraints {
            $0.edges.equalToSuperview()
        }

        tableView.dataSource = self

        tableView.register(FavoriteCell.self, forCellReuseIdentifier: "FavoriteCell")

        tableView.rowHeight = 60
    }

    // 셀 개수 리턴
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return learners.count
    }

    // 셀 구성
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "FavoriteCell", for: indexPath) as! FavoriteCell

        cell.textLabel?.text = learners[indexPath.row]

        // ✅ delegate 지정!
        // 여기서 ViewController(self)가 FavoriteCell의 delegate가 됨
        // 이걸 지정해야만 셀 내부에서 delegate?.didTapFavoriteButton() 호출 시 ViewController의 메서드가 실행됨
        cell.delegate = self

        return cell
    }

    // ✅ 실제로 "일을 대신 해주는" 메서드 (delegate가 호출하는 메서드)
    func didTapFavoriteButton(in cell: UITableViewCell) {
        if let indexPath = tableView.indexPath(for: cell) {
            print("찜 버튼 눌림: \(indexPath.row)번 셀")
        }
    }
}
```

### 전체 코드(실행 가능)
```swift

import UIKit
import SnapKit

// MARK: - Protocol
protocol FavoriteCellDelegate: AnyObject {
    func didTapFavoriteButton(in cell: UITableViewCell)
}

// MARK: - Custom Cell
class FavoriteCell: UITableViewCell {
    weak var delegate: FavoriteCellDelegate?

    private let favoriteButton: UIButton = {
        let button = UIButton(type: .system)
        button.setTitle("⭐️", for: .normal)
        return button
    }()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        setupLayout()
        favoriteButton.addTarget(self, action: #selector(favoriteButtonTapped), for: .touchUpInside)
    }

    private func setupLayout() {
        contentView.addSubview(favoriteButton)

        favoriteButton.snp.makeConstraints {
            $0.centerY.equalToSuperview()
            $0.trailing.equalToSuperview().inset(16)
        }

        textLabel?.snp.makeConstraints {
            $0.leading.equalToSuperview().inset(16)
            $0.centerY.equalToSuperview()
            $0.trailing.lessThanOrEqualTo(favoriteButton.snp.leading).offset(-8)
        }
    }

    @objc func favoriteButtonTapped() {
        delegate?.didTapFavoriteButton(in: self)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

// MARK: - ViewController
class ViewController: UIViewController, UITableViewDataSource, FavoriteCellDelegate {

    let tableView = UITableView()
    let learners = ["Jam", "Mini", "Gus", "Frank", "Berry", "Glowny", "Jay"]

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .white
        
        setupTableView()
    }

    private func setupTableView() {
        view.addSubview(tableView)

        tableView.snp.makeConstraints {
            $0.edges.equalToSuperview()
        }

        tableView.dataSource = self
        tableView.register(FavoriteCell.self, forCellReuseIdentifier: "FavoriteCell")
        tableView.rowHeight = 60
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return learners.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "FavoriteCell", for: indexPath) as! FavoriteCell
        cell.textLabel?.text = learners[indexPath.row]
        cell.delegate = self
        return cell
    }

    func didTapFavoriteButton(in cell: UITableViewCell) {
        if let indexPath = tableView.indexPath(for: cell) {
            print("찜 버튼 눌림: \(indexPath.row)번 셀")
        }
    }
}

```
## SwiftUI에서는?
> SwiftUI에서는 delegate 패턴 대신 아래 네 가지 방식으로 대체한다

| 역할         | SwiftUI                           | UIKit                                                 |
| ---------- | --------------------------------- | ----------------------------------------------------- |
| 양방향 데이터 전달 | `@Binding`                        | `delegate`, `target-action`                           |
| 단방향 이벤트 전달 | 클로저(`onTap`, `onSubmit` 등)        | `target-action`, `delegate`                           |
| 상태 변화 알림   | `@Published`+ `@ObservableObject` | `KVO`, `NotificationCenter`, `delegate`, `Combine/Rx` |
| 전역 상태 공유   | `@EnvironmentObject`              | `Singleton`, `AppDelegate`, `DI`                      |

> [!SwiftUI에서 delegate 패텬이 필요 없는 이유]
> SwiftUI는 **“이벤트가 발생하면 그에 따라 상태를 바꾸고, 상태에 따라 UI를 그린다”**  
> 라는 흐름 자체가 명확하고 자동화돼 있어서  
> delegate처럼 행동을 위임하고 응답받는 구조가 거의 필요 없다

## References
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/#Delegation