>[!question]
>GQ1. Liquid Glass가 뭘까?
>GQ2. material이랑 어떻게 다를까?

## 개념
- Apple이 WWDC25에서 공개한 새로운 디자인 언어
- **유리의 광학적 특성**과 **유동적인 느낌**을 결합한 새로운 동적 소재 (dynamic material)
- **명확함과 입체감이 중요한 핵심 인터페이스 컴포넌트**(ex: cards, toolbars, menus)에 적용하는 것을 권장한다
- 앱의 성능을 유지하기 위해 너무 광범위한 사용은 지양한다 (ex: 배경 전체)
- [[glassEffect]] modifier를 통해 적용할 수 있다

## 기존 Material과의 차이점

|                         | 기존 Material (`.ultraThinMaterial` 등)         | Liquid Glass (`glassEffect`)                                                    |
| ----------------------- | -------------------------------------------- | ------------------------------------------------------------------------------- |
| **렌더링 엔진**              | Core Animation + Core Image blur layer       | Metal 기반 custom shader                                                          |
| **구성 layer**            | 단일 blur layer + vibrancy + transparency mask | blur layer + tint layer + reflection layer + interactive layer (shader에서 모두 합성) |

> [!question] Metal shder
> Metal은 애플의 GPU 전용 그래픽/연산 엔진이다
> GPU가 화면 그리기, 물리 연산, 3D 그래픽, 고급 효과를 초고속으로 처리한다
> 
> Metal shader는 Metal에서 실행되는 GPU용 프로그램으로
> Swift나 Objective-C로는 못하는 GPU 직접 렌더링을 담당한다
> 
> glassEffect는 Metal shader로 **blur layer**, **reflection layer**, **tint 색 layer**, 
> **interactive highlight layer**를 하나의 pass에서 실시간으로 합성한다


## References
- https://developer.apple.com/documentation/technologyoverviews/liquid-glass
- https://developer.apple.com/design/human-interface-guidelines/materials
- https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass