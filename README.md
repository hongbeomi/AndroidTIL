# AndroidTIL

안드로이드 정리 공간



# Fundamental

### Application File

- aab : 런타임에 필요하지 않은 메타데이터 같은 값들을 포함한 앱 내용물이 포함되어 있고 기기에 직접적으로 설치할 수 없습니다. 게시 형식의 파일이라서 apk 생성 및 서명을 나중으로 미루게 됩니다.

- apk : 안드로이드 패키지로서, 런타임에 필요한 앱의 내용물이 포함되어 있기에 기기에서 설치할 수 있는 파일 타입입니다.

## Component (Activity, Service, Content Provider, Broadcast Receiver)

### Activity

유저와 상호 작용하기 위한 진입점이며 앱이 UI를 그리는 창을 제공합니다. 여러 액티비티가 상호작용하여 애플리케이션을 구성하며 각각의 액티비티는 독립적으로 운용됩니다.



**Activity Lifecycle Callback Method**

![Android Activity Lifecycle - javatpoint](https://images.javatpoint.com/images/androidimages/Android-Activity-Lifecycle.png)

- `onCreate` : 첫 번째로 시작될 때 실행됩니다. 초기화 작업을 실행하는데 용이합니다. 이 함수는 반드시 오버라이드하여 사용해야 합니다. `savedInstanceState`라는 파라미터를 수신하는데, 이는 이전 상태가 저장된 `Bundle` 객체입니다.

- `onStart` : 액티비티가 사용자에게 보여질 때 호출됩니다. `onRestart`에서 재호출되거나 기타 이유로 재호출 될 수 있는 함수입니다. 매우 빠른 속도로 실행됩니다.

- `onResume` : 사용자가 액티비티와 상호작용 할 수 있게 되는 타이밍에 호출됩니다. 사용자가 현재 액티비티로 다시 돌아올 때도 호출될 수 있습니다. (전화를 받고 오는 등..) `onPause`와 쌍으로 리소스를 등록하거나 해제하는데 용이합니다.

- `onPause` : 액티비티가 조금이라도 가려지거나 포커스를 잃는 경우(멀티 윈도우 화면 시)에 호출됩니다. 언젠가 다시 시작할 작업을 일시중지하는 작업을 수행하기에 용이합니다. 전화가 오거나, 시스템 다이얼로그 때문에 화면이 가려진다거나 하는 등의 이벤트 발생 시 호출됩니다. 아주 잠깐 실행되기에 무엇을 저장하는 작업을 두기엔 부적합합니다.

- `onStop` : 액티비티가 사용자에게 완전히 보여지지 않을 때 호출됩니다. 액티비티가 백그라운드로 진입하는 경우처럼 사용자가 액티비티와 직접적으로 상호작용 할 수 없게 됩니다. 필요하지 않은 리소스를 해제하거나 정리하기에 용이합니다. (애니메이션 중지, 무거운 리소스 해제 등등..)

- `onDestroy` : 액티비티가 사용자에 의해 종료되거나 기기 회전 등 화면 구성의 이벤트가 발생하여 액티비티가 파괴되는 경우 호출됩니다. 이전까지 정리되지 않는 리소스가 있다면 여기서 정리해야 합니다. 그렇지 않으면 메모리 누수의 위험이 존재합니다. 이후 액티비티는 메모리에서 완전히 소멸합니다.



> Question
> 
> A 액티비티에서 B 액티비티로 진입했다가 다시 A로 진입하는 경우 Lifecycle은 어떻게 될까요?
> 
> Answer
> 
>  A : `onCreate` -> A : `onStart` -> A : `onResume` -> 전환 이벤트 발생 -> A : `onPause` -> B : `onCreate` -> B : `onStart` -> B : `onResume` -> A : `onStop` -> A : `onSaveInstanceState` -> 다시 A로 전환 이벤트 발생 및 B 액티비티 종료 -> B : `onPause` -> A : `onRestart` -> A : `onStart` -> A : `onResume` -> B : `onStop` -> B : `onDestroy`



### Service

서비스는 백그라운드에서 실행되는 컴포넌트입니다. 크게 두 가지로 나뉘고 세분화하면 세 가지로 나뉩니다.

- Started Service
  
  - foreground service : 어느 정도 사용자에게 보여지는 작업을 수행합니다. (ex : 상단바의 컨트롤러) 반드시 알림(노티피케이션)이 지속적으로 표시해야 합니다.
  
  - background service : 사용자에게 보여지지 않는 작업을 수행합니다. 앱이 API 26 이상을 타겟팅한다면 시스템 자원 낭비 차원에서 실행이 제한되는데, WorkManager를 사용하는 것이 권장된다.

- Bound Service : 앱 컴포넌트가 바인딩 된 형태의 서비스이다. 바인딩된 컴포넌트와 클라이언트-서버 구조처럼 상호작용하는 것이 가능하다. 
  
  ![](https://velog.velcdn.com/images%2Fhaero_kim%2Fpost%2F7dc04ab6-0179-4047-980f-662c1df8db1a%2F0_rbbPsYYkjekH6LYV.png)

> 세 가지 서비스 구조가 분리된 것이 아닌, 혼합하여 사용할 수 있으며 기본적으로 서비스를 생성하면 호스팅된 프로세스의 기본 스레드에서 동작하게 되는데 블로킹 우려가 있는 작업(음악 재생, 파일 다운로드)을 실행하려면 별도의 스레드를 생성하여 작업을 수행해야한다. 

> Question
> 
> 서비스를 사용하면서 라이프 사이클 관리는 어떻게 하는가?
> 
> Answer
> 
> 개발자가 자체적으로 생명주기 콜백 메소드에 맞춰서 작업을 해도 되지만, LifecycleService를 상속받는 Service를 구현하여 쉽게 작업할 수 있다.