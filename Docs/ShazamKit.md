>[!question]
>GQ0. **ShazamKit**이 뭔데 ! + HIG
>GQ1. **ShazamKit**은 어떤 식으로 음원을 인식할까?
>GQ2. SHSession과 SHSignature의 역할은 무엇일까?


![[ShazamKit.png]]

## ShazamKit이 뭔데 !

<font color="palevioletred">Shazamkit</font>은 오디오 샘플을 <font color="palevioletred">Shazamkit</font> 카탈로그 또는 사용자 설정 오디오 카탈로그와 일치 시켜 오디오 인식을 지원합니다. 

![[ShazamKitPlatform.png]]
다 지원이 됩니다. 🫳🏻 🫳🏻

#### Shazamkit의 기능 

- 현재 재생 중인 음악 장르에 맞는 그래픽으로 경험 향상하기
- 오디오와 동기화되는 자막 또는 수어를 제공하여 청각 장애를 지닌 사람들이 미디어 콘텐츠에 접근할 수 있도록 하기

#### ShazamKit HIG 🔆

1. <font color="palevioletred">Shazamkit</font>은 마이크 접근 권한을 요구하므로, **명확한 목적 설명 필수**
    ( ex : "이 앱은 주변 음악을 인식하기 위해 마이크를 사용합니다." )
    
2. **인식 상태에 대한 명확한 피드백** -> 인식 중, 인식 중지, 완료의 상태를 시각적 또는 음성적으로 알려야 함 
    - 인식 중: 웨이브 애니메이션, 로딩 인디케이터
    - 인식 성공: 음악 정보 표시 + 앨범 커버
    - 인식 실패: “음악을 인식할 수 없었습니다”와 같은 안내문)
      
3. **음악 재생과 권한 관리 분리** -> <font color="palevioletred">Shazamkit</font>은 음악 인식까지만 제공하고, 실제 재생은 Apple Music API 등 별도 처리 해야함.
   
4. **비인식 상황도 고려**
   - 주변 소음이 심할 경우, 오류 메세지를 <font color="orange">! 공격적이지 않게 !</font> "다시 시도하기." 정도로 유도하기



## ShazamKit은 어떤 식으로 음원을 인식할까?

<font color="palevioletred">Shazamkit</font>은 음원의 고유한 '음향 시그니처'를 생성하여 이를 [[데이터베이스 (Database)]]내 시그니처들과 비교해 일치 항목을 찾습니다.

#### 1. 오디오 캡처

마이크 입력이나 앱 내 재생 중인 오디오에서 짧은 오디오 샘플을 캡처합니다.


#### 2. 스펙트로그램 변환 (Spectrogram Analysis)

캡처된 오디오 신호는 먼저 FFT (Fast Fourier Transform)를 통해 시간 - 주파수 도메인으로 변환됩니다. 이를 통해 시간 축과 주파수 축이 있는 스펙트로그램이 생성됩니다.

(지피티로 만들었더니 못생겼어요 죄송)
t →
f ↓   ┌───────────────▶
      │ █      █         
      │   █     █
      │    █       

##### 곡 전체를 듣지 않아도 되는 이유는?

예를 들어, 2초 지점에서 300Hz, 2.3초 지점에서 450Hz가 강한 피크로 잡혔다면,  ShazamKit은 (AnchorFreq: 300, TargetFreq: 450, Δt: 0.3초) 이런 해시 키로 구성하는데, 

이런 해시 키들은 해당 곡에서만 볼 수 있는 고유한 조합이며, 짧은 클립만 있어도 여러 조합이 존재하므로 곡을 인식할 수 있습니다.


#### 3. 오디오 지문 생성

<font color="palevioletred">Shazamkit</font>은 이 피크들을 조합해 특정 방식으로 해시 (Hash)를 만듭니다. 강한 피크를 기준으로 잡고, 그 주변에 있는 다른 피크들과 상대적 시간차와 주파수차를 조합합니다.

노이즈는 낮은 에너지이거나 불규칙하기 때문에 ShazamKit은 높은 에너지를 가진 주파수 피크만 사용하므로 배경 소음이나 잡음은 자연스럽게 무시되고, 주요 정보만 시그니처에 반영됩니다.


#### 4. 쿼리와 기준 데이터 비교

쿼리에서 생성된 해시 키들을 데이터 베이스 내의 기준 해시들과 비교합니다. 다수의 해시가 일치하고, 그 시간 간격이 일관성 있게 나열되어 있다면, 특정 시점에서 녹음이 시작된 것이라고 판단합니다.

쿼리 해시들과 기준 해시들이 여러 개 일치하고, 그 시간 간격이 일정한 패턴을 보이면 ShazamKit은 이를 일치로 간주합니다!

#### 5. 결과 반환

곡이 일치하면 
- 곡 제목
- 아티스트
- 앨범 정보
- 일치 시작 시간
등을 반환합니다 !


## SHSession과 SHSignature의 역할은 무엇일까?

**SHSession : SHSignature를 활용해 음원 매칭을 수행하는 클래스**
- 오디오의 고유한 주파스 패턴을 기반으로 생성된 '오디오 지문'으로, 특정 음원을 식별할 수 있는 데이터입니다.

**SHSignature : 오디오 고유의 특징이 담긴 오디오 지문**
- SHSignature을 입력으로 받아, ShazamKit 라이브러리 또는 서버에서 일치하는 음원을 검색하고 매칭 결과를 반환하는 역할을 합니다.





## Keywords
+ 파생된 키워드들을 작성

## References
- [ShazamKit_Hig](https://developer.apple.com/kr/design/human-interface-guidelines/shazamkit)
- [Document 📃](https://developer.apple.com/documentation/shazamkit/)