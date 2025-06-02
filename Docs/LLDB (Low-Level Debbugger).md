## 1. LLDB (Low-Level Debbugger)

 - Xcode에 기본적으로 내장되어 있는 Command Line Debug 환경이다.
   LLDB를 통해 실행되고 있는 프로세스의 값을 확인할 수 있고 프로세스 값을 변경하여 제어할 수 있다. 


### 1-1. LLDB 실행 방법

Breakpoint에서 멈추거나, pause버튼을  누르면 실행이 일시정지되면서 Xcode 하단 Debug콘솔에 나타납니다. 

![[breakpoint로 lldb실행.jpg]]

![[pause버튼으로 lldb실행.jpg]]




### 1-2. LLDB 명령어의 문법 구성

```shell
<command> [<subcommand> [<subcommand>...]] <action> [-options [option-value]] [argument [argument...]]
```

- command와 subcommand는 LLDB내 Object의 이름(예. breakpoint, watchpoint, set, list...)입니다.  이 Object는 모두 계층화되어 있어, command에 따라 사용가능한 subcommand 종류가 다릅니다. 

- option의 경우, command 뒤 어느 곳에든 위치가 가능하며, - (하이픈)로 시작합니다. 

- argument에 공백이 포함 되는 경우도 있기 때문에, ""로 묶어 줄수 있습니다. 

```shell
(lldb) command [subcommand] -option "arguments"

```

- help와 apropos 명령어 

help 명령어는 command나 command로 사용가능한 subcommand, option 리스트나 사용방법을 알려줍니다. 
```swift
# lldb에서 제공하는 Command가 궁금하다면,
(lldb) help
```

![[help 명령어 결과.png]]

```swift
# 특정 Command의 Subcommand나, Option이 궁금하다면,
(lldb) help breakpoint
(lldb) help breakpoint set
```

![[help breakpoint 명령어 결과.png]]


apropos 명령어는 원하는 기능 명령어가 있는지 입력한 키워드를 통해서 알수 있습니다. 

```swift
# reference count를 체크할 수 있는 명령어가 있는지 궁금하다면,
(lldb) apropos "reference count"
```

![[apropos 명령어 결과.png]]


- breakpoint 명령어

프로그램에서 문제가 되는 지점을 찾아 멈추고, 여러가지 조건을 바꿔보며 테스트하기 위해 필요합니다. 

```swift
# viewDidLoad를 breakpoint 설정
(lldb) breakpoint set --name viewDidLoad

# 축약형 
(lldb) b -n viewDidLoad
```

```swift
# 정규식을 따르는 모든 함수에 breakpoint 설정
(lldb) breakpoint set  --func-regex '^hello'
(lldb) br s -r '^hello'

# 'breakopint set --func-regex'는 줄여서 'rb'로 사용 가능
(lldb) rb '^hello'
```

```swift
# 파일의 이름과 line번호를 이용해 breakpoint 설정
# 특정 파일의 20번째 line에서 break 
(lldb) br s --file ViewController.swift --line 20
(lldb) br s -f ViewController.swift -l 20
```

```swift
# breakpoint에 원하는 조건 걸기. (option 뒤의 expression이 true인 경우에만 멈춤)
# viewWillAppear 호출시, animated가 true인 경우에만 break
(lldb) breakpoint set --name "viewWillAppear" --condition animated
(lldb) br s -n "viewWillAppear" -c animated
(lldb) br s -l 27 -condition 'helloWorld == "hello World"'
```

```swift
# breakpoint 리스트 확인
(lldb) breakpoint list
(lldb) br list

# 특정 breakpoint 정보 확인
(lldb) br list 1

# breakpoint 전체 삭제
(lldb) breakpoint delete
(lldb) br de

# 특정 breakpoint 삭제
(lldb) br de 1
```

- watchpoint 명령어
변수값이 수정되는 곳에 일일이 breakpoint를 걸지 않고 값이 변경될 때  디버깅이 실행되도록 설정하는 방법입니다. 
```swift

# 변수 이름으로 watchpoint 설정
(lldb) watchpoint set variable gus
(lldb) wa s v my_var

# 메모리 주소에 watchpoint 설정
(lldb) watchpoint set expression -- gus
(lldb) wa s e -- gus
```


- po 명령어

breakpoint를 기점으로 변수에 어떤 값이 저장되어 있는지 알 수 있습니다. 
expression -O --` 의 축약형 (or `expression -object-description --) 여기서 -O 는 object의 설명(debugDescription)을 출력하겠다는 의미. option을 사용하면 반드시 `--` 를 option과 raw input 사이에 넣어야합니다.

```swift
  (lldb) expression -O -- self
  (lldb) po self
```

po가 출력하는 description은 NSObject의 debugDescription입니다.  그래서 해당 프로퍼티를 override 하면 po 결과 customize 해서 사용 가능합니다. 

```swift
    override var debugDescription: String {
        return "Gus의 description \(super.debugDescription)"
    }
```
```shell
(lldb) po self
# Gus의 description <sample.ViewController: 0x7fc66ed08850>
```

 struct 같이 NSObject 상속 안하는거에 커스텀 디버깅 인포 찍고싶다? 하면  `CustomDebugStringConvertible` protocol conform으로 사용하면 됩니다. 

```swift
struct Trip {
 var name: String
 var desinations: [String]
}

extension Trip: CustomDebugStringConvertible {
  var debugDescription: String { "Trip description" }
}
```

하지만 이는 top level description만 변경하기 때문에 substructure까지 변경하고 싶다하면 `CustomReflectable` protocol conform 하면 됩니다.  Objective-C 객체라면 description method를 implement하면 됩니다.

po는 변수만 출력하는 것이 아니라 프롬프트에 컴파일되는 모든 내용이 argument로 전달되어 arbitary expression을 수행할 수 있습니다.


```shell
(lldb) po cruise.name.uppercased() # 대문자 계산
(lldb) po cruise.desinations.sorted()  # 알파벳 정렬 순
```

Swift debugging context에서 `expression [self.view recursiveDescription]` 과 같이 Objective-C 표현을 사용하면 오류가 나는데 이럴때는 Objective-C 실행을 강제하기 위한 `-l` (`-language`) 옵션 존재 합니다. 

기존 po Command에 `-l objc` option을 추가해서 Swift Context안에서도 Objective-C Expression을 사용할 수 있습니다.

```
(lldb) e -l objc -O -- [self.view recursiveDescription]
```

하지만 위의 코드는 에러가 나는데, 이유는 Objective-C 컴파일을 위한 임시 expression context를 컨텍스트를 생성하며 Swift 프레임에서 모든 변수를 상속하지 않기 때문입니다. 

이때 self.view에 backticks 을 추가해주면 되는데 이 의미는 '현재 frame 에서, backtick 안에 있는 것부터 먼저 evaluate 해서 결과에 넣은 후에 나머지를 evaluate 하는 것 ' 명시하는 겁니다. 


```
(lldb) e -l objc -O -- [`self.view` recursiveDescription]
```