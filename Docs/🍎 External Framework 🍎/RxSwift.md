RxSwift는 ReactiveX 패턴을 기반으로 한 라이브러리로, [[비동기 (Asynchronous)]] 이벤트 스트림을 쉽게 관리할 수 있게 해줍니다. ReactiveX 패턴은 관찰 가능한 시퀀스(Observable Sequence)와 옵서버(Observer)를 통해 데이터를 처리합니다. RxSwift는 다양한 연산자(Operators)를 제공하여 데이터 스트림을 필터링, 변환, 결합할 수 있습니다.

#### RxSwift 구성요소
1. **Observable**: 이벤트를 발행하는 스트림
2. **Observer**: Observable로부터 발행된 이벤트를 구독하는 객체
3. **Subject**: Observable과 Observer의 역할을 동시에 수행하는 객체
4. **Schedulers**: 코드가 실행되는 스레드를 제어하는 데 사용