## 1-2 BreakPoint

- breakpoint 생성 및 제거

	코드 왼쪽 코드 라인을 클릭하면 breakpoint를 생성할 수 있습니다. 이렇게 생성한 breakpoint를 line breakpoint라고 부릅니다. 코드 라인에 breakpoint를 생성하는 것입니다. 생성한 breakpoint는 드래그해서 밖으로 끄집어내면 breakpoint를 제거할 수있습니다. 

- Edit Breakpoint

	Xcode UI 상에서 Edit Breakpoint 사용할 수 있습니다. 생성한 breakpoint를 더블클릭하면 Edit Breakpoint 창이 뜹니다. 

![[Edit Breakpoint.png]]

마우스 오른쪽 클릭 -> Edit Breakpoint를 눌러도 됩니다. 

![[마우스 오른쪽 버튼 edit breakpoint.png]]

-Name : breakpoint 네이밍을 지정할 수 있습니다. 
-Condition : 특정 조건일 때만 디버깅 실행 (swift 문법으로 작성)
-Ignore : 특정 횟수 조건 이후에 디버깅 실행
-Action : 디버깅 실행전 특정 동작을 수행함 (LLDB 명령어, script, sound) 보통은 코드를 수정하고 재빌드 해서 결과를 확인하지만  재빌드 없이 Action안에 코드로만으로 결과를 확인할 수 있기 때문에 잘 활용하면 유용한 기능임
-Automatically continue after evaluation actions :  설정한 동작은 수행하지만, 디버깅이 실행되지 않습니다. 


- watchpoint 

	Xcode UI 상에서 왼쪽 콘솔 화면에서 트래킹할 변수를 오른쪽 클릭하고 "Watch {변수}"를 클릭해서 설정할 수 있습니다. 


![[UI watch변수.png]]

