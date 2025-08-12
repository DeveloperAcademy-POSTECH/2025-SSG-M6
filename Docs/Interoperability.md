### Interoperability란 ?

Swift 언어가 다른 프로그래밍 언어- 프레임워크를 연결해서 쓰는것을 말합니다. 
애플에서는 브리징(Bridging), Interop(Interoperability)라고 부릅니다.

Bridging : 서로 다른 언어를 연결하는 개념

Interoperability : 언어 프레임워크 간 상호 운용성


## 1. Swift 에서 Objective-C 코드 사용하기 

	-Objective-C 파일 만들기
	
```Swift
	// MyObjcClass.h 헤더파일 만들기
	#import <Foundation/Foundation.h>
	
	@interface MyObjcClass : NSObject
	- (NSString *)sayHelloWithName:(NSString *)name;
	@end

	// MyObjcClass.m 파일 만들기
	#import "MyObjcClass.h"

	@implementation MyObjcClass
	- (NSString *)sayHelloWithName:(NSString *)name {
        return [NSString stringWithFormat:@"Hello, %@ from Objective-C!", name];
	}
	@end
```
	
	-Bridging Header 만들기
	
	Xcode에서 Swift 파일과 Objective-C 파일이 같이 있으면 **처음 만들 때 자동으로 "Bridging Header" 생성 여부를 물어봄** → `Yes` 선택

	또는 

	수동 생성 시: `YourProjectName-Bridging-Header.h` 파일 만들고  
	Build Settings → `Swift Compiler - General` → **Objective-C Bridging Header** 경로에 등록

```Swift
	// YourProjectName-Bridging-Header.h
	#import "MyObjcClass.h"
```

	-Swift에서 호출
	
```Swift
	import UIKit

	class ViewController: UIViewController {

	  override func viewDidLoad() {
		super.viewDidLoad()

        let objcInstance = MyObjcClass()
        let result = objcInstance.sayHello(withName: "Gus")
        print(result) // Optional("Hello, Gus from Objective-C!")
	    }
	}
```

	변환 규칙 요약
	- 메서드 이름 첫 부분 ➡️ Swift의 함수명
	- 콜론(:) 앞 단어  ➡️  파라미터 라벨로 변환 (단, 첫 번째는 상황에 따라 _ 로 생략)
	- With, And 같은 연결어 ➡️  라벨에서 자동으로 제거됨
	- 타입 변환 : Objective-C 타입이 Swift 기본 타입으로 매핑됨 (NSString  ➡️  String, NSInteger ➡️  Int 등)

| Objective-C 메서드 선언                                             | Swift에서 보이는 메서드                                      | 설명                                             |
| -------------------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------- |
| `- (void)doSomething;`                                         | `func doSomething()`                                 | 파라미터가 없으면 이름 그대로                               |
| `- (void)doSomethingWithName:(NSString *)name;`                | `func doSomething(withName name: String)`            | `WithName:`의 `With`는 제거되고, `Name`이 파라미터 라벨로 변환 |
| `- (void)updateValue:(NSInteger)value;`                        | `func updateValue(_ value: Int)`                     | 파라미터 이름이 첫 번째 단어 바로 뒤에 오면 라벨 생략 (`_`)          |
| `- (void)updateValue:(NSInteger)value forKey:(NSString *)key;` | `func updateValue(_ value: Int, forKey key: String)` | 첫 번째는 `_`로, 두 번째부터는 라벨 유지                      |

## 2. Objective-C 에서 Swift 코드 사용하기 

	-Swift 클래스 만들기
	`@objc` 붙이면 Objective-C에서 접근 가능
	`NSObject` 상속 필요 (순수 Swift 타입은 Objective-C가 모름)

```swift
	// MySwiftClass.swift
	import Foundation

	@objc class MySwiftClass: NSObject {
	    @objc func sayHi(to name: String) -> String {
	        return "Hi, \(name) from Swift!"
	    }
	}
```

	-자동 생성 헤더 사용
		Swift 코드를 Objective-C에서 쓰려면 **자동 생성된 헤더**를 import
		Xcode가 자동으로 만듦, 직접 파일이 보이진 않음.  Build된 시점에만 존재.
		
```swift
		#import "YourProjectName-Swift.h"
```
	
	빌드 후, Xcode의 `DerivedData` 폴더 안에 가면 실제 `YourProjectName-Swift.h` 파일이 생성되어 있습니다.

```shell
		~/Library/Developer/Xcode/DerivedData/<프로젝트명>-<랜덤문자>/Build/Intermediates.noindex/<...>/YourProjectName-Swift.h
```


	-Objective-C에서 호출
	
```swift
	// main.m
	#import <Foundation/Foundation.h>
	#import "YourProjectName-Swift.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        MySwiftClass *obj = [[MySwiftClass alloc] init];
        NSLog(@"%@", [obj sayHiTo:@"Gus"]); // Hi, Gus from Swift!
    }
    return 0;
}
```

## 3. SwiftUI에서  Objective-C 코드 사용하기 

1. Objective-C 파일들 (.h, .m) 프로젝트에 추가
2. Bridging Header에 헤더 파일들 import
3. SwiftUI에서 바로 사용!

- `NSInteger` → `Int`
- `NSString` → `String`
- `BOOL` → `Bool`

```swift
// Calculator.h
#import <Foundation/Foundation.h>

@interface** Calculator : NSObject

+ (NSInteger)add:(NSInteger)a to:(NSInteger)b;

+ (NSInteger)multiply:(NSInteger)a by:(NSInteger)b;

+ (NSString *)formatNumber:(NSInteger)number;

@end
```

```swift
// Calculator.m
#import <Foundation/Foundation.h>
#import "Calculator.h"

@implementation Calculator

+ (NSInteger)add:(NSInteger)a to:(NSInteger)b {
    return a + b;
} 

+ (NSInteger)multiply:(NSInteger)a by:(NSInteger)b {
    return a * b;
}

  
+ (NSString *)formatNumber:(NSInteger)number {

    if (number >= 1000) {
        return [NSString stringWithFormat:@"%ld,000+", (long)(number / 1000)];
    }
    return [NSString stringWithFormat:@"%ld", (long)number];
}


@end
```
```swift
// 
import SwiftUI

struct ContentView: View {

    @State private var count = 0
    

    var body: some View {

        VStack(spacing: 30) {
            // Calculator 섹션
            VStack(spacing: 15) {

                Text("Calculator")
                    .font(.title2)
                    .bold()

                Text("Count: \(Calculator.formatNumber(count))")
                    .font(.largeTitle)

                HStack(spacing: 20) {
                    Button("Add 1") {
                        // Objective-C 메서드 사용
                        count = Calculator.add(count, to: 1)
                    }
                    .buttonStyle(.borderedProminent)

                    Button("Multiply by 2") {
                        // Objective-C 메서드 사용
                        count = Calculator.multiply(count, by: 2)
                    }
                    .buttonStyle(.bordered)

                    Button("Reset") {
                        count = 0
                    }
                    .buttonStyle(.bordered)

                }
            }
            .padding()
            .background(Color.blue.opacity(0.1))
            .cornerRadius(10)

        }
    }
}
```
![[SwiftUIToObjective-C.png|400x800]]

## 4. Objective-C에서 SwiftUI 코드 사용하기 

- SwiftUI 뷰를 `UIHostingController`로 감싸 UIKit 뷰 컨트롤러로 만들어서 사용 가능.
- Objective-C에서 Swift 코드 접근을 위해 브릿지 헤더와 `@objc` 노출 필요.
- SwiftUI 뷰는 기본적으로 `@objc` 노출 불가하지만, 이를 `UIHostingController`로 감싸서 사용 가능.

```swift
// ContentView.swift
import SwiftUI
struct ContentView: View {

    @State private var count = 0

    var body: some View {

        VStack {
            Text("Count: \(count)")
                .font(.largeTitle)
                .padding()

            Button("Increment") {
                count += 1
            }
            .padding()
            .background(Color.blue)
            .foregroundColor(.white)
            .cornerRadius(8)

        }

    }

}
```

```swift
// 2. Objective-C에서 사용할 수 있는 Swift 래퍼 클래스 (SwiftUIWrapper.swift)
import UIKit import SwiftUI @objc public class SwiftUIWrapper: NSObject { 

	// SwiftUI View를 UIViewController로 래핑하는 메서드 
	@objc public static func createContentViewController() -> UIViewController { 
		let contentView = ContentView() 
		let hostingController = UIHostingController(rootView: contentView)
		return hostingController 
	} 
	
	// 매개변수를 받는 SwiftUI View 래퍼 예제 
	@objc public static func createCounterViewController(initialCount: Int) -> UIViewController { 
		let contentView = CounterView(initialCount: initialCount) 
		let hostingController = UIHostingController(rootView: contentView)
		return hostingController 
		} 
	} 
	
	// 매개변수를 받는 SwiftUI View 예제 
	struct CounterView: View { 
		@State private var count: Int 
		
		init(initialCount: Int) { 
			self._count = State(initialValue: initialCount) 
		} 
		
		var body: some View { 
			VStack { 
				Text("Counter: \(count)")
					.font(.title)
					.padding() 
				
				HStack { 
					Button("-") { 
						count -= 1
					}
					.padding()
					.background(Color.red)
					.foregroundColor(.white)
					.cornerRadius(8) 
					
					Button("+") { 
						count += 1 
					}
					.padding()
					.background(Color.green)
					.foregroundColor(.white)
					.cornerRadius(8) 
				} 
			} 
			.padding() 
		} 
	}
```


```swift
// ViewController.h
#import <UIKit/UIKit.h>

@interface ViewController : UIViewController

@end

// ViewController.m
#import "ViewController.h"
#import "YourProject-Swift.h" // Swift 브리징 헤더

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 방법 1: SwiftUI View를 현재 뷰컨트롤러에 추가
    [self embedSwiftUIView];
    
    // 방법 2: SwiftUI View를 새 화면으로 present
    // [self presentSwiftUIView];
}

// SwiftUI View를 현재 뷰컨트롤러에 임베드하는 방법
- (void)embedSwiftUIView {
    // SwiftUI View를 UIViewController로 래핑
    UIViewController *swiftUIController = [SwiftUIWrapper createContentViewController];
    
    // 자식 뷰컨트롤러로 추가
    [self addChildViewController:swiftUIController];
    swiftUIController.view.frame = self.view.bounds;
    swiftUIController.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
    
    [self.view addSubview:swiftUIController.view];
    [swiftUIController didMoveToParentViewController:self];
}

// SwiftUI View를 새 화면으로 present하는 방법
- (void)presentSwiftUIView {
    UIViewController *swiftUIController = [SwiftUIWrapper createCounterViewController:10];
    swiftUIController.modalPresentationStyle = UIModalPresentationPageSheet;
    
    [self presentViewController:swiftUIController animated:YES completion:nil];
}

// 버튼 액션 예제
- (IBAction)showSwiftUIView:(id)sender {
    [self presentSwiftUIView];
}

// Navigation Controller를 사용하는 경우
- (void)pushSwiftUIView {
    UIViewController *swiftUIController = [SwiftUIWrapper createContentViewController];
    swiftUIController.title = @"SwiftUI View";
    
    [self.navigationController pushViewController:swiftUIController animated:YES];
}

@end
```

![[Objective-CToSwiftUI.png|400x800]]

## 5.  Swift UIKit코드를 SwiftUI에서 사용하기 

UIView 방식 (UIViewRepresentable)
- **UICounterLabel**: UILabel을 래핑해서 카운트 숫자 표시
- **UICounterButton**: UIButton을 래핑해서 증가/감소 버튼

UIViewController 방식 (UIViewControllerRepresentable)
- **CounterViewController**: 완전한 UIKit 뷰컨트롤러
- UILabel, UIButton, UIStackView 등을 사용해 레이아웃 구성
- `@Binding`으로 SwiftUI와 데이터 동기화

```swift
// UIComponent.swift
import SwiftUI 
import UIKit 
// MARK: - 1. UILabel을 SwiftUI에서 사용하기 (UIViewRepresentable) 
struct UICounterLabel: UIViewRepresentable { 
	let count: Int 
	
	func makeUIView(context: Context) -> UILabel { 
		let label = UILabel() 
		label.textAlignment = .center 
		label.font = UIFont.boldSystemFont(ofSize: 48) 
		label.textColor = .systemBlue label.backgroundColor = UIColor.systemGray6 
		label.layer.cornerRadius = 20 
		label.clipsToBounds = true 
		return label 
	} 
	
	func updateUIView(_ uiView: UILabel, context: Context) { 
		uiView.text = "\(count)" 
	} 
} 

// MARK: - 2. UIButton을 SwiftUI에서 사용하기 (UIViewRepresentable) 
struct UICounterButton: UIViewRepresentable { 
	let title: String 
	let backgroundColor: UIColor 
	let action: () -> Void 
	
	func makeUIView(context: Context) -> UIButton { 
		let button = UIButton(type: .system) 
		button.titleLabel?.font = UIFont.boldSystemFont(ofSize: 24)
		button.setTitleColor(.white, for: .normal) 
		button.layer.cornerRadius = 25 
		button.addTarget( 
			context.coordinator, 
			action: #selector(Coordinator.buttonTapped), 
			for: .touchUpInside 
		) 
		return button 
	} 
	
	func updateUIView(_ uiView: UIButton, context: Context) { 
		uiView.setTitle(title, for: .normal) 
		uiView.backgroundColor = backgroundColor 
	} 
	
	func makeCoordinator() -> Coordinator { 
		Coordinator(action: action) 
		} 
	
	class Coordinator: NSObject { 
		let action: () -> Void 
		
		init(action: @escaping () -> Void) { 
			self.action = action 
		} 
		
		@objc func buttonTapped() { 
			action() 
		} 
	} 
} 

// MARK: - 3. UIViewController를 SwiftUI에서 사용하기 (UIViewControllerRepresentable) 

class CounterViewController: UIViewController { 
	var count: Int = 0 { 
		didSet { 
			updateCountLabel() 
		} 
	} 
	
	var onCountChanged: ((Int) -> Void)? 
	
	private let countLabel = UILabel() 
	private let incrementButton = UIButton(type: .system) 
	private let decrementButton = UIButton(type: .system)
	
	override func viewDidLoad() { 
		super.viewDidLoad() 
		setupUI() 
	} 
	
	private func setupUI() { 
		view.backgroundColor = .systemBackground 
		
		// 카운트 라벨 설정 
		countLabel.textAlignment = .center 
		countLabel.font = UIFont.boldSystemFont(ofSize: 64) 
		countLabel.textColor = .systemIndigo 
		countLabel.text = "\(count)" 
		
		// 증가 버튼 설정 
		incrementButton.setTitle("+ 증가", for: .normal) 
		incrementButton.titleLabel?.font = UIFont.boldSystemFont(ofSize: 20) 
		incrementButton.backgroundColor = .systemGreen 
		incrementButton.setTitleColor(.white, for: .normal) 
		incrementButton.layer.cornerRadius = 15 
		incrementButton.addTarget(self, action: #selector(incrementTapped), for: .touchUpInside) 
		
		// 감소 버튼 설정 decrementButton.setTitle("- 감소", for: .normal) 
		decrementButton.titleLabel?.font = UIFont.boldSystemFont(ofSize: 20) 
		decrementButton.backgroundColor = .systemRed 
		decrementButton.setTitleColor(.white, for: .normal) 
		decrementButton.layer.cornerRadius = 15 
		decrementButton.addTarget(self, action: #selector(decrementTapped), for: .touchUpInside) 
		
		// 스택뷰로 버튼 배치 
		let buttonStack = UIStackView(arrangedSubviews: [decrementButton, incrementButton]) 
		buttonStack.axis = .horizontal 
		buttonStack.spacing = 20 
		buttonStack.distribution = .fillEqually 
		
		let mainStack = UIStackView(arrangedSubviews: [countLabel, buttonStack]) 
		mainStack.axis = .vertical 
		mainStack.spacing = 40 
		mainStack.alignment = .center 
		view.addSubview(mainStack) 
		
		// Auto Layout 설정 
		mainStack.translatesAutoresizingMaskIntoConstraints = false 
		incrementButton.translatesAutoresizingMaskIntoConstraints = false 
		decrementButton.translatesAutoresizingMaskIntoConstraints = false 
		
		NSLayoutConstraint.activate([ 
			mainStack.centerXAnchor.constraint(equalTo: view.centerXAnchor), 
			mainStack.centerYAnchor.constraint(equalTo: view.centerYAnchor), 
			incrementButton.widthAnchor.constraint(equalToConstant: 120), 
			incrementButton.heightAnchor.constraint(equalToConstant: 50), 
			decrementButton.widthAnchor.constraint(equalToConstant: 120), 
			decrementButton.heightAnchor.constraint(equalToConstant: 50) 
		]) 
	} 
	
	@objc private func incrementTapped() { 
		count += 1 
		onCountChanged?(count) 
	} 
	
	@objc private func decrementTapped() { 
		count -= 1 
		onCountChanged?(count) 
	} 
	
	private func updateCountLabel() { 
		countLabel.text = "\(count)" 
	}
} 

struct UICounterViewController: UIViewControllerRepresentable { 
	@Binding var count: Int 
	
	func makeUIViewController(context: Context) -> CounterViewController { 
		let controller = CounterViewController() 
		controller.count = count 
		controller.onCountChanged = { newCount in 
			count = newCount 
		} 
		return controller 
	} 
	
	func updateUIViewController(_ uiViewController: CounterViewController, context: Context) { 
		uiViewController.count = count 
	} 
}
```

```swift
// ContentView.swift
// MARK: - 4. SwiftUI에서 UIKit 컴포넌트들 사용하기 

struct ContentView: View { 
	@State private var swiftUICount = 0 
	@State private var uiKitCount = 0 
	
	var body: some View { 
		ScrollView { 
			VStack(spacing: 40) { 
			
				// SwiftUI 기본 카운터 
				VStack(spacing: 20) { 
					Text("SwiftUI 카운터")
						.font(.title2)
						.fontWeight(.bold) 
						
					Text("\(swiftUICount)")
						.font(.system(size: 48, weight: .bold)) 
						.foregroundColor(.blue) 
						.frame(width: 150, height: 100) 
						.background(Color.gray.opacity(0.1)) 
						.cornerRadius(20) 
					
					HStack(spacing: 20) { 
						Button("- 감소") { 
							swiftUICount -= 1
						} 
						.frame(width: 120, height: 50)
						.background(Color.red) 
						.foregroundColor(.white) 
						.cornerRadius(25) 
						
						Button("+ 증가") { 
							swiftUICount += 1 
						} 
						.frame(width: 120, height: 50) 
						.background(Color.green)
						.foregroundColor(.white) 
						.cornerRadius(25) 
					} 
				} 
				
				Divider() 
				
				// UIKit 컴포넌트를 사용한 카운터 (UIView 방식) 
				VStack(spacing: 20) { 
					Text("UIKit UIView 카운터")
						.font(.title2)
						.fontWeight(.bold) 
					
					UICounterLabel(count: uiKitCount) 
						.frame(width: 150, height: 100)
					
					HStack(spacing: 20) { 
						UICounterButton( 
							title: "- 감소", 
							backgroundColor: .systemRed 
							) { 
								uiKitCount -= 1 
							}
							.frame(width: 120, height: 50)
							
							UICounterButton( 
								title: "+ 증가", 
								backgroundColor: .systemGreen 
							) { 
								uiKitCount += 1 
							}
							.frame(width: 120, height: 50) 
						} 
					} 
					
					Divider() 
					
					// UIKit ViewController를 사용한 카운터 
					VStack(spacing: 20) { 
						Text("UIKit ViewController 카운터")
							.font(.title2) 
							.fontWeight(.bold) 
						
						UICounterViewController(count: $uiKitCount)
						.frame(height: 300)
						.background(Color.gray.opacity(0.05))
						.cornerRadius(20) 
					} 
				} 
				.padding() 
			} 
		} 
	}
```

![[SwiftUIKitToSwiftUI.png|400x800]]
## 6.  SwiftUI코드를 Swift에서 사용하기

- **`UIHostingController`** 사용이 핵심
- SwiftUI View를 UIHostingController로 래핑하면 UIKit에서 사용 가능
- Child View Controller로 추가하거나 직접 루트 뷰컨트롤로 설정

```swift
// SimpleCounterView.swift
import SwiftUI
  
struct SimpleCounterView: View {

    @State private var count = 0

    var body: some View {

        VStack(spacing: 20) {
            Text("카운터: \(count)")
	            .font(.largeTitle)

            HStack {
                Button("-") {
                    count -= 1
                }
                .padding()
                .background(Color.red)
                .foregroundColor(.white)
                .cornerRadius(8)

                Button("+") {
                    count += 1
                }
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
                .cornerRadius(8)

            }
        }
        .padding()

    }
}
```

```swift
// ViewController
import UIKit
import SwiftUI

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground

        setupSwiftUIView()
    }

    private func setupSwiftUIView() {

         // SwiftUI View를 UIHostingController로 래핑
         let swiftUIView = SimpleCounterView()
         let hostingController = UIHostingController(rootView: swiftUIView)

         // Child View Controller로 추가
         addChild(hostingController)
         view.addSubview(hostingController.view)
         hostingController.didMove(toParent: **self**)

         // Auto Layout 설정
         hostingController.view.translatesAutoresizingMaskIntoConstraints = false

         NSLayoutConstraint.activate([
             hostingController.view.centerXAnchor.constraint(equalTo: view.centerXAnchor),
             hostingController.view.centerYAnchor.constraint(equalTo: view.centerYAnchor),
             hostingController.view.widthAnchor.constraint(equalTo: view.widthAnchor, multiplier: 0.8),
             hostingController.view.heightAnchor.constraint(equalToConstant: 200)
         ])
     }
}
```



![[SwiftUIToSwift.png|400x800]]
