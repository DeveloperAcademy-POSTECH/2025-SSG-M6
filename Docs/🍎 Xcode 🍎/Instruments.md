# Instruments란?


iOS, macOS, watchOS, tvOS 애플리케이션의 성능 분석 및 디버깅을 위한 도구 입니다. 
이를 활용하여 앱의 CPU 사용량, 메모리 누수, 네트워크 활동 등을 분석하고 최적화 할 수 있습니다.

![[Instruments.png]]
## Instruments Workflow

1. 원하는 Instruments 및 설정을 포함하는 trace document 설정
2. 프로파일링 할 기기와 앱을 선택
3. 앱 프로파일링
4. 프로파일링 중에 캡쳐된 데이터를 분석
5. 소스코드의 문제 해결


![[Instruments Workflow.png]]


## Instruments 실행 방법

1. Xcode  ➡️  File Open Developer  ➡️  Tool Instruments   

2. Xcode ➡️ Product ➡️ Profile

3. Taget Process list에서 측정하고 싶은 기기나 에뮬레이터를 선택

![[ActivityMonitorControl.png]]


- Activity Monitor

	CPU, Memory, Disk, Network 중 하나를 선택해서 관련 통계를 분석 할수 있습니다.
	
![[Graph Display.png]]


	옆의 CPU Total Load는 현재 CPU를 얼마나 사용하고 있는지 시각적으로 확인 
	가능합니다. 


![[cpu total load.png]]

	하단에는 현재 측정한 기기에 어떤 프로세스들이 CPU를 할당 받아 실행중인지 또 프로세스 
	내에서 몇 개의 스레드가 동작 하는지 확인 가능합니다. 만약 현재 실행 중인 앱에 대해서만
	정보를 알고 싶다면 맨 하단의 Detail Filter 부분에 현재 실행 중인 앱의 이름을 입력하면 
	됩니다. 

![[Live Processes.png]]


- Time Profiler

	어떤 작업이 처리에 오래 걸리는지, 현재 CPU 사용량에서 각각의 스레드가 점유하는 정도 등을 알아 볼수 있습니다. 주로 메인스레드의 [[Hang]] 해결하는데 많이 사용합니다. 

	CPU Usage는 CPU 사용량 Thermal State는 현재 온도 상태를 알수 있습니다. 

![[Time Profiler.png]]

	하단의 정보를 통해 현재 어떤 스레드가 얼마나 CPU 리소스를 사용하는지 확인할 수 있습니다.

![[profile.png]]

	만약 system library 관련된 처리를 숨기고 싶다면 하단 Call Tree 버튼을 눌러 
	Hide System Library 체크를 하면 됩니다.

![[call tree.png]]



- Allocations

	프로그램에서 현재 얼마나 많은 메모리가 할당되어 사용되고 있는지 확인할 수 있습니다. 

	하단의 Allocation Summary를 통해서는 메모리 할당에 대한 상세한 정보를 확인할 수 있고 
	Graph을 체크하면 메모를 비교해서 볼수 있습니다. 

![[Allocations.png]]

-  Leaks

	메모리 누수를 확인하는 용도로 사용합니다.  
	초록색 체크 마크는 메모리 누수가 없음을 뜻하고 빨간색 체크 X 마크는 메모리 누수가
	있음을 뜻합니다. 

	![[Leak.png]]

	빨간색 X마크를 눌러보면 어떤 누수가 일어나고 있는지 상세히 확인 가능합니다. 
	Cycles & Roots를 변경하면 순환 참조 발생 유무를 확인 할 수 있습니다.

![[순한참조 여부확인.png]]
