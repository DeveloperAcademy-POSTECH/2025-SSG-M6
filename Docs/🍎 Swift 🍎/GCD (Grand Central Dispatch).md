>[!question] 
>GQ1. Grand Central Dispatch(GCD)의 주요 개념과 사용 방법은 무엇인가요?
>GQ2. OperationQueue와 DispatchQueue의 차이점은 무엇인가요?


## Description
---
#### GCD

Grand Central Dispatch는 Apple이 제공하는 저수준의 동시성 API로, 멀티스레딩을 좀 더 쉽게 다룰 수 있게 동시 코드 실행을 지원 해준다. GCD는 DispatchQueue를 통해 [[비동기 (Asynchronous)]]나 동기 작업을 큐에 등록하고, 시스템이 적절한 스레드에서 해당 작업들을 실행하도록 관리해준다.

- DispatchQueue
	- 작업을 실행할 큐를 의미한다. 
	- 직렬 큐 (Serial Queue)와 동시 큐 (Concurrent Queue)로 나눈다. 즉, 한 번에 하나의 작업만을 순차적으로 수행하거나 여러 작업을 동시적으로 수행한다.
	- DispatchQueue에 등록된 작업은 시스템에서 관리하는 스레드 풀에서 실행된다.


DispatchQueue는 Main, Global, Custom 3가지로 나뉜다.

- Main Queue
	- 직렬 큐의 대표적인 예시로 단일 스레드이다.
	- Main Thread에서 실행되는 DispatchQueue 이다.
	- UI와 관련된 모든 작업은 반드시 메인 큐에서 처리되어야 한다.
- Global Queue
	- Main Thread가 아닌 Thread에서 실행되는 DispatchQueue 이다.
	- 우선순위가 다른 system-provided concurrent queues이다.
	- QoS를 설정할 수 있다.
- Custom Queue
	- Custom으로 생성하는 Queue를 말한다.
	- 기본은 직렬이지만, 동시로 설정할 수도 있다.
	- QoS를 설정할 수 있다.


- sync / async
	- 각각 작업을 큐에 넣는 방식들을 의미한다.
	- sync
		- 동기로 작업을 등록하는 방식으로, 해당 작업이 완료될 때까지 현재 스레드를 차단한다.
		- Sync 작업은 메인 스레드에서 주의해서 사용해야 한다.
		- 메인 스레드에서 Sync 작업을 하게되면 UI가 응답하지 못해 데드락 문제가 발생할 수 있다.
	- async
		- 비동기로 작업을 등록하는 방식으로, 스레드를 따로 차단하지 않고 실행한다.
		- 주로 네트워크 호출, 파일 다운로드, 계산량이 많은 작업 등을 처리할때 사용한다.


> 💡 여기서 잠깐, 멀티 스레드란?
> 
> 간단히 말해, 하나의 프로세스 내에서 여러 스레드가 동시에 작업을 수행하는 것을 의미한다.
> 하나의 프로세스 내에 여러 개의 스레드가 존재하고, 각각의 스레드들이 프로세스의 자원을 공유하지만 실행은 독립적으로 이뤄지는 구조이다.

> 💡 여기서 잠깐, 스레드 풀이란?
> 
> 간단히 말해, 작업을 처리하기 위해 미리 생성해 둔 스레드들의 집합을 의미한다.
> 새로운 작업이 발생할 때마다 스레드를 생성하고 파괴하는 오버헤드를 줄이기 위해서 사용되며, 시스템의 자원을 효율적으로 관리하고 작업 부하에 따라 스레드를 재사용하여 성능을 최적화한다.
> DispatchQueue에 등록된 작업들은 내부적으로 스레드 풀에서 관리되는 스레드에 의해 실행된다.
> 


작업의 중요도에 따라 QoS(Quality of Service)를 설정할 수 있으며, 시스템을 이를 기반으로 Concurrent하게 작업을 처리하면서 작업 실행 우선순위를 결정한다. 아래를 높은 우선순위부터 순서대로 QoS를 정리한 내용이다.

- .userInteractive
	- UI 반응과 관련되어 사용자와 직접 상호작용하는 작업으로 가장 높은 우선순위를 갖는다.
- .userInitiated
	- 즉시 필요하지만, 사용자의 입력을 기다릴 수 있는 작업이다.
	- 사용자가 직접 시작한 작업을 말한다.
- .default
	- 일반적인 작업의 기본 우선순위로, QoS 따로 지정하지 않은 경우 사용된다.
- .utility
	- 사용자가 의도했지만, 지속적으로 관찰하지는 않는 작업이다.
	- progress bar, 다운로드 실행 등 꽤나 오래 걸리는 작업이다.
- .background
	- 백업이나 데이터 정리, 네트워크 동작 등 사용자와 관계없이 백그라운드에서 실행되는 작업이다.
- .unspecified
	- QoS 정보가 없는 작업으로, 시스템이 적절한 우선순위를 결정한다.


앞서 DispatchQueue는 작업을 실행할 큐를 의미하며, 직렬/동시 큐로 나뉜다고 말했다. 그렇다면 많이들 들어봤을 OperationQueue는 무엇일까?

간단히 말해서, OperationQueue는 GCD보다 추상화 수준이 높은 동시성 API를 말한다. 내부적으로는 동일하게 GCD를 사용하지만, 작업을 Operatiton 객체 단위로 다루기 때문에 작업을 더 세밀하게 제어할 수 있다. 

- OperationQueue
	- cancel()를 통해 작업 취소를 지원gksek.
	- addDependency()를 통해 작업 간 의존성을 지원한다. 
	- queuePriority를 통해 우선순위의 세부 조정이 가능하다. 
	- maxConcurrentOperationCount를 통해서 관리하는 스레드의 수를 제어할 수 있다.

반면에, 앞서 말했던 DispatchQueue는 클로저 기반으로 작업을 단순하게 처리하는 데 적합하다. 상대적으로 빠르고 가볍다는 특징을 갖기 때문에, 단순한 비동기 작업에서는 DispatchQueue가 적합하며, 복잡한 작업 흐름이나 취소, 혹은 의존성 관리가 필요한 경우에는 OperationQueue가 더 적합하다.

