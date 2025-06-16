>[!question]
>GQ1. 어떤 파라미터가 있을까?
>GQ2. 내부는 어떻게 이루어져있을까?

## `glassEffect(_:in:isEnabled:)`

> SwiftUI 뷰에 Liquid Glass 효과를 적용하는 modifier
```swift
func glassEffect(
	_ glass: Glass = .regular,
	in shape: some Shape = .capsule,
	isEnabled: Bool = true
) -> some View
```

- `glass`: 사용할 `Glass` 스타일 (기본값 `.regular`)
- `shape`: glassEffect의 형태를 정의 (기본값 `.capsule`)
- `isEnabled`: glassEffect를 활성화할지 여부 (기본값 `true`)

### `Glass` 타입
```swift
@available(iOS 26.0, macOS 26.0, tvOS 26.0, watchOS 26.0, *)

public struct Glass : Equatable, Sendable {

public static var regular: Glass { get }

public func tint(_ color: Color?) -> Glass

public func interactive(_ isEnabled: Bool = true) -> Glass

public static func == (a: Glass, b: Glass) -> Bool

}
```

- `Glass.regular`: 기본 Liquid Glass preset 제공
- `tint`: Glass에 tint 색 입힌 새 Glass 반환
- `interactive`: 상호작용 가능 상태의 새 Glass 반환


### 간단 예제

- TabView UI 변화 확인용
```swift
//  MainTabView.swift

import SwiftUI

struct MainTabView: View {
    var body: some View {
        TabView {
            FirstView()
                .tabItem {
                    Label("First", systemImage: "house")
                }
            
            SecondView()
                .tabItem {
                    Label("Second", systemImage: "gear")
                }
        }
    }
}

#Preview {
    MainTabView()
}
```

- glassEffect modifier 확인용
```swift
//  FirstView.swift

import SwiftUI

enum Topping: String, CaseIterable, Identifiable {
    case nuts, cookies, blueberries
    var id: Self { self }
}

struct FirstView: View {
    
    @State private var selectedTopping: Topping = .nuts
    
    var body: some View {
        
        NavigationStack {
            VStack {
                Picker("Topping", selection: $selectedTopping) {
                    ForEach(Topping.allCases) { topping in
                        Text(topping.rawValue.capitalized)
                    }
                }
                .pickerStyle(.segmented)
                .padding()
                
                GlassEffectContainer(spacing: 100) {
                    Text("Default Text")
                        .padding()
                        .glassEffect(.regular.interactive())
                    
                    Text("Color Text")
                        .padding()
                        .glassEffect(Glass.regular.tint(.purple.opacity(0.9)).interactive(), isEnabled: true)
                }
                
                
                Button(action: {}){
                    Text("Text Color Button")
                        .tint(.black)
                        .padding()
                        .glassEffect(Glass.regular.tint(.cyan.opacity(0.9)).interactive(), isEnabled: true)
                }
    
                Button(action: {}){
                    Text("Text Button without interactive")
                        .tint(.black)
                        .padding()
                        .glassEffect(.regular)
                }
                
                Button(action: {}){
                    Text("Style Button")
                        .padding()
                        
                }
                .buttonStyle(.glass)
            }
            .frame(maxWidth: .infinity, maxHeight: .infinity)
            .background(LinearGradient(colors: [.blue, .purple, .pink], startPoint: .top, endPoint: .bottom))
            
            .navigationTitle("Liguid Glass")
            .toolbar {
                ToolbarItem{
                    Button {
                        print("Home tapped")
                    } label: {
                        Image(systemName: "star.fill")
                    }
                    
                }
                
                ToolbarSpacer(.flexible)
                
                ToolbarItem{
                    Button {
                        print("Home tapped")
                    } label: {
                        Image(systemName: "house.fill")
                    }
                    
                }
            }
            
        }
    }
    
    
}

#Preview {
    FirstView()
}
```

- TabView 확인용
```swift

//  SecondView.swift
import SwiftUI

struct SecondView: View {
    var body: some View {
        Text("Second!")
    }
}

#Preview {
    SecondView()
}
```
