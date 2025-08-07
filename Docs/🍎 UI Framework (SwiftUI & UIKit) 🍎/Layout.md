>[!question]
>GQ1. V, H, ZStack의 아버지를 아십니까? Layout protocol

## Description
- SwiftUI의 Layout 프로토콜은 **커스텀 레이아웃 컨테이너**를 만들기 위한 도구입니다. iOS 16부터 도입된 이 프로토콜은 HStack·VStack 같은 기본 스택으로는 구현하기 어려운 배치(예: 플로우 레이아웃, 일자 배치)를 만들 때 사용합니다. 개발자는 레이아웃이 어떤 공간을 요구하는지 계산하고 그 공간 안에서 자식 뷰들을 어떻게 배치할지 명시적으로 제어할 수 있습니다.

## 주요 기능
+ **두 개의 필수 메서드**:
    - sizeThatFits(proposal:subviews:cache:) – 부모가 제안하는 크기(proposal)와 자식 뷰의 이상적 크기를 바탕으로 레이아웃이 차지해야 할 전체 크기를 계산합니다 . 이 메서드는 다양한 proposal로 여러 번 호출될 수 있습니다.
        
    - placeSubviews(in:proposal:subviews:cache:) – 위에서 계산된 영역(bounds) 안에서 각 자식 뷰를 배치합니다. subview.place(at:anchor:proposal:)를 호출해 좌표와 anchor, 제안 크기를 지정합니다.
    
- **Subview 크기 측정**: subview.sizeThatFits(.unspecified)를 통해 이상적인 크기를 얻을 수 있습니다. .zero는 최소 크기, .infinity는 최대 크기를 요청합니다.
    
- **다중 호출**: 레이아웃 엔진은 여러 proposal을 시험하기 위해 sizeThatFits를 반복 호출할 수 있습니다. macOS에서는 .zero proposal도 요청해 최소 크기를 계산합니다.
    
- **캐싱(optional)**: 반복적인 계산을 줄이기 위해 Cache 타입과 makeCache/updateCache 메서드를 구현할 수 있습니다. 캐시 안에 자식 뷰의 크기나 위치 정보를 저장해 성능을 향상시킵니다.
    
- **layoutProperties(optional)**: 스택 방향(stackOrientation)을 변경해 Spacer의 확장 방향 등을 제어할 수 있습니다.

## 코드 예시
+ 공간이 부족하면 자동으로 줄을 바꾸는 가로 래핑 레이아웃
```swift
struct WrapLayout: Layout {
    func sizeThatFits(proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) -> CGSize {
        var width: CGFloat = 0
        var height: CGFloat = 0
        var rowWidth: CGFloat = 0
        var rowHeight: CGFloat = 0
        let maxWidth = proposal.width ?? .infinity

        for subview in subviews {
            let size = subview.sizeThatFits(.unspecified)  // 자식의 이상적 크기
            if rowWidth + size.width > maxWidth {
                width = max(width, rowWidth)
                height += rowHeight
                rowWidth = 0
                rowHeight = 0
            }
            rowWidth += size.width
            rowHeight = max(rowHeight, size.height)
        }
        width = max(width, rowWidth)
        height += rowHeight
        return CGSize(width: width, height: height)
    }

    func placeSubviews(in bounds: CGRect, proposal: ProposedViewSize, subviews: Subviews, cache: inout ()) {
        var x: CGFloat = bounds.minX
        var y: CGFloat = bounds.minY
        var rowHeight: CGFloat = 0
        let maxWidth = bounds.width

        for subview in subviews {
            let size = subview.sizeThatFits(.unspecified)
            if x + size.width > maxWidth {
                x = bounds.minX
                y += rowHeight
                rowHeight = 0
            }
            subview.place(
                at: CGPoint(x: x, y: y),
                proposal: ProposedViewSize(width: size.width, height: size.height)
            )
            x += size.width
            rowHeight = max(rowHeight, size.height)
        }
    }
}

/// 사용 예시 – 래핑 레이아웃을 이용해 텍스트를 배치
struct ContentView: View {
    var body: some View {
        WrapLayout {
            ForEach(0..<20) { index in
                Text("Item \(index)")
                    .padding(8)
                    .background(Color.blue.opacity(0.3))
                    .cornerRadius(8)
            }
        }
        .padding()
    }
}
```

## Keywords
+ size

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)