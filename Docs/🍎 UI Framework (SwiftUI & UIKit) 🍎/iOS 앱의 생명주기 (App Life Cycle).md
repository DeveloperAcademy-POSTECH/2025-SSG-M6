
## 앱 생명주기란(App Life Cycle): 

앱의 최초 실행부터 앱이 완전히 종료되기까지 시스템이 발생시키는 event에 의해 앱의 상태가 전환되는 일련의 과정을 뜻한다. 각 시점마다 운영체제가 자동으로 호출하는 메서드가 존재하며, 이러한 메서드 실행의 과정을 앱 생명주기라고 한다. 

이 앱 생명주기는 AppDelegate와 SceneDelegate에 의해 관리된다. 

![[AppDelegate와SceneDelegate 파일.png]]


## 앱의 생명주기를 알아야하는 이유: 

#### 1. **리소스 관리 (메모리, 배터리 등)**

- 백그라운드에 있을 때 네트워크 요청이나 애니메이션을 멈추지 않으면 배터리와 리소스를 낭비함.
    ( 예 : `sceneDidEnterBackground(_:)`에서 GPS, 타이머 등을 중단. )
    

#### 2. **데이터 보존과 복원**

- 앱이 백그라운드로 전환되거나 종료될 때 사용자 데이터를 저장해야 함.
    ( 예 : 입력 중이던 내용을 `UserDefaults`에 저장하고, 재진입 시 복원. )
    

#### 3. **UI 상태 유지**

- 앱을 중단했다가 돌아왔을 때도 사용자는 같은 상태를 기대함.
    
- 생명주기를 따라 뷰의 상태를 저장하고 복원하는 로직이 필요함.
    

#### 4. **네트워크 연결 관리**

- 백그라운드에서 연결을 끊거나, 포그라운드로 돌아왔을 때 재연결해야 함.
    ( 예 : 채팅 앱은 앱이 다시 활성화될 때 메시지를 새로 받아와야 함. )
    

#### 5. **알림 처리**

- 앱이 실행 중일 때와 종료되어 있을 때 알림 처리 방식이 다름.
    
- 생명주기에 따라 알림 처리 분기 필요.
    

#### 6. **외부 이벤트 대응**

- 전화 수신, 알림, 앱 전환 등의 상황에서 적절히 반응해야 사용자 경험이 좋아짐.
    ( 예 : 전화가 오면 영상 통화를 일시 중지하고, 종료 후 다시 재개. )



![[앱생명주기 .png]]



### NotRunning : 

앱이 아직 실행되지 않았거나, 완전히 종료되어 동작하지 않는 상태이다.

### Foreground - Inactive : 

앱이 실행 중이지만 사용자로부터 이벤트를 받을수 없는 상태이다.  
( 예 : 멀티태스킹 윈도우로 진입하거나 앱 실행중 전화, 알림 등에 의해 앱을 사용할 수 없게 되는 경우 )

### Foreground - Active : 

앱이 실제 실행중이고 사용자 이벤트를 받아서 상호작용할 수 있는 상태이며,  바로 Active 상태는 될 수 없고 Inactive상태를 거쳐서 Active 상태가 된다. 

### Background - Running : 

홈 화면으로 나가거나 다른 앱으로 전환되어 앱의 하면이 내려갔지만 계속 작동하는 상태이며, 애플은 보안등의 이유로 Backgroud영역에서 앱의 작동을 허용하지 않는다 일부 앱만 작동이 가능하다.
( 예:  음악 앱을 닫아도 재생한 음악이 계속 실행되는 경우, 사용자가 앱을  사용하지 않는 동안 서버와 데이터를 동기화하거나 타이머가 실행되어야 하는 등의 작업을 이 상태에서 할 수 있음 )

### Background - Suspended :

앱을 다시 실행 했을 때 최근 작업을 빠르게 로드하기 위해서 메모리에 관련 데이터만 저장되어 있는 상태이다. 앱이 Background 상태에 진입했을 때 다른 작업을 하지 않으면 Suspended 상태로 진입하게 된다. (보통 2~3초)  Suspended 상태의 앱들은 iOS의 메모리가 부족해지면 가장 먼저 메모리에서 해제된다. 따라서 앱을 종료시킨 적이 없더라도 앱을 다시 실행하려고 하면 처음부터 다시 실행되는 경우도 있다. 


![[ChatGPT Image May 20, 2025 at 07_13_29 PM.png]]

## iOS 13버전 혹은 그 이후 버전:  Scene 사용

iOS13에서 처음으로 iPadOS가 등장했다.  이전에 아이패드는 iOS를 사용해서  아이폰, 아이패드 유사하게 동작했다. 개선된 아이패드에 적합한 기능을 위해(아이패드 기능 중 splitview - 앱 2개를 동시에 띄울 수 있음, 멀티윈도우) iPadOS를 만들었는데, spitview만해도 active인 상태가 너무 많고,  UI의 생명주기가 다양해지면서 이걸 관리해줄 객체가 필요해졌다.  

그래서 UI의 상태만을 관리하는 SceneDelegate가 생겼고, 같은 앱을 여러 개 실행할 수도 있게 되었다. 이전에 AppDelegate에서 담당하던 background, foreground 등의 UI 생명주기를 SceneDelegate에서 담당하고, SceneDelegate에서 Scene이 새롭게 생성되고 종료되는 트리거를 AppDelegate에게 알려줌으로써 AppDelegate가 앱의 생성과 종료 시점을 통제할 수 있다.


![[AppDelegate와SceneDelegate관계.png]]



## @AppDelegate

### didFinishLaunchinWithOptions()

앱이 처음 실행될때,  Not Running 상태의 초기 앱을 실행할 때 사용자에게 화면(Scene)이 보일수 있도록 Foreground 영역에 접근하는 메서드이며, 앱화면이 보이기 직전에 호출된다. 


### configurationForConnecting()

새로운 화면(Scene)을 생성할 때 호출, iOS 12 이전에는 하나의 디바이스에서 단일 화면 만 존재 할 수 있어지만 iOS13부터 하니의 다비이스에 여러화면을 가지는 것이 가능해짐


### didDiscardSceneSessions()

화면의 세션이 제거되기 직전에 호출되는 메서드 화면을 영구적으로 메모리에서 해제될 때만 해당 메서드가 호출된다. 


예시코드
```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        print("✅ didFinishLaunchingWithOptions")
        return true
    }

    func application(
        _ application: UIApplication,
        configurationForConnecting connectingSceneSession: UISceneSession,
        options: UIScene.ConnectionOptions
    ) -> UISceneConfiguration {
        print("🛠 configurationForConnecting")
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }

    func application(
        _ application: UIApplication,
        didDiscardSceneSessions sceneSessions: Set<UISceneSession>
    ) {
        print("🗑 didDiscardSceneSessions")
    }
}

```


## @SceneDelegate


### scene()

화면(Scene)이 해당 앱에 추가(연결)될 때 호출되는 메서드이며, 초기 화면을 지정하거나 Window를 설정한다. 

### sceneDidBecomeActive()

화면(Scene)을 InActive에서 Active로 이동시킬 때 호출되는 메서드로, 앱을 처음 실행할 때 사용자가 해당 앱을 100% 조작할 수 있도록 시켜주는 메서드다. 

해당 메서드는 Foreground영역에서만 동작한다.

### sceneWillResignActive()

화면(Scene)을 Active에서 InActive로 이동시킬 때 호출되는 메서드로, 앱 사용중 전화 또는 문자등의 특정 이벤트가 발생하면 해당 앱의 화면은 이 메서드를 통해 Active에서 InActive로 이동한다.

해당 메서드는 Foreground 영역에서만 동작한다.

### sceneDidEnterBackground()

화면(Scene)을 InActive에서 Background영역으로 이동시킬 때 호출되는 메서드다.

### sceneWillEnterForeground()

화면(Scene)을 Background에서 Frreground(InActive)영역으로 이동시킬 때 호출되는 메서드

### sceneDidDisconnect()

화면(Scene)이 해제될 때 호출되는 메서드로 해당 메서드는 App의 화면이 Background영역에 들어간 직후 또는 화면의 세션이 버려질 때 호출되는 메서드이다. (메모리로부터 영구적으로 버려지는건 아니다. 사용자가 해당 화면에 다시 접근할 여지가 있기 때문이다. 완젼한 App종료가 아님.)


예시코드
```swift
// SceneDelegate.swift

import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func sceneDidBecomeActive(_ scene: UIScene) {
        print("🌟 Foreground 진입")
        NotificationCenter.default.post(name: .appDidBecomeActive, object: nil)
    }

    func sceneDidEnterBackground(_ scene: UIScene) {
        print("🌙 Background 진입")
        NotificationCenter.default.post(name: .appDidEnterBackground, object: nil)
    }
}

extension Notification.Name {
    static let appDidBecomeActive = Notification.Name("appDidBecomeActive")
    static let appDidEnterBackground = Notification.Name("appDidEnterBackground")
}

```

```swift
// ViewController.swift

import UIKit

class ViewController: UIViewController {

    let stateLabel = UILabel()

    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        setupObservers()
    }

    func setupUI() {
        view.backgroundColor = .white

        stateLabel.text = "앱 상태: 시작됨"
        stateLabel.textAlignment = .center
        stateLabel.font = .systemFont(ofSize: 24)
        stateLabel.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(stateLabel)

        NSLayoutConstraint.activate([
            stateLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            stateLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }

    func setupObservers() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(appDidBecomeActive),
            name: .appDidBecomeActive,
            object: nil
        )

        NotificationCenter.default.addObserver(
            self,
            selector: #selector(appDidEnterBackground),
            name: .appDidEnterBackground,
            object: nil
        )
    }

    @objc func appDidBecomeActive() {
        updateUI(for: "Foreground", backgroundColor: .systemGreen)
    }

    @objc func appDidEnterBackground() {
        updateUI(for: "Background", backgroundColor: .systemGray)
    }

    func updateUI(for state: String, backgroundColor: UIColor) {
        UIView.animate(withDuration: 0.5) {
            self.view.backgroundColor = backgroundColor
            self.stateLabel.text = "앱 상태: \(state)"
        }
    }

    deinit {
        NotificationCenter.default.removeObserver(self)
    }
}

```

https://developer.apple.com/documentation/uikit/managing-your-app-s-life-cycle
https://clamp-coding.tistory.com/407