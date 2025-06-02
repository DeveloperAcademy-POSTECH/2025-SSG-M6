## 1.  UIKit에서 프리뷰 사용하기

- UIKit 기반 앱에서 UI를 구성한 뒤 확인을 하려면 앱을 빌드하고 확인해야 할 뷰까지 타고 들어가야 합니다.
- 또한, 뷰의 구성이 복잡하거나 Constraint가 복잡해지면 뷰가 어떻게 출력될지 직접 확인하지 않고서는 예상하기 어렵습니다.
- 이때, SwiftUI 프리뷰 기능을 활용하면 손쉽게 뷰를 확인할 수 있습니다.
- 예를 들어서, 아래와 같은 뷰가 있을 때

```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()

        let label = UILabel()
        label.text = "Label"
        label.backgroundColor = .yellow
        label.translatesAutoresizingMaskIntoConstraints = false

        self.view.addSubview(label)

        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: self.view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: self.view.centerYAnchor),
            label.widthAnchor.constraint(equalToConstant: 100),
            label.heightAnchor.constraint(equalToConstant: 50),
        ])
    }
}
```

- SwiftUI를 import 해주고 프리뷰 관련 코드를 입력해 주면, UIKit 프로젝트에서 뷰가 수정되어도 바로바로 확인이 가능합니다.
- (프리뷰 코드는 Debug 빌드에서만 사용할 수 있도록 전처리해 주시면 더욱 좋습니다.)

```Swift
import UIKit
import SwiftUI

class ViewController: UIViewController {
  ...
}

#if DEBUG
struct ViewControllerRepresentation: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> ViewController {
        return ViewController()
    }

    func updateUIViewController(_ uiViewController: ViewController, context: Context) {

    }
}


struct ViewController_Previews: PreviewProvider {
    static var previews: some View {
        ViewControllerRepresentation()
    }
}
#endif
```

![[swift에서preview사용 2.png]]


## 2. Environment Overrides

- 앱을 만들 때 라이트 모드 & 다크 모드에서 화면이 잘 출력되는지 확인해야 합니다. 또한, 텍스트 크기가 커졌을 때 앱을 정상적으로 이용할 수 있는지도 중요하게 고려해야 합니다.
- Environment Overrides는 앱 빌드 후에 위 설정들을 동적으로 쉽게 바꿀 수 있게 해줍니다.

![[Environment Overrides.png]]


## 3.  View Hierarchy Debug


뷰를 계층별로 분리해서 입체적으로 분석할게 도와주는 기능

![[view hierachy 디버깅.png]]

View Hierachy는 일시정지 후에 기능을 사용할수 있는데 일시정지 없이 사용하는 도구들이있음
https://lookin.work (무료)
https://revealapp.com/free-trial/ (유료)