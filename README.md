# AndroidTIL

안드로이드 정리 공간



# Fundamental

### Application File

- **aab** : 런타임에 필요하지 않은 메타데이터 같은 값들을 포함한 앱 내용물이 포함되어 있고 기기에 직접적으로 설치할 수 없습니다. 게시 형식의 파일이라서 apk 생성 및 서명을 나중으로 미루게 됩니다.

- **apk** : 안드로이드 패키지로서, 런타임에 필요한 앱의 내용물이 포함되어 있기에 기기에서 설치할 수 있는 파일 타입입니다.



## Component (Activity, Service, Content Provider, Broadcast Receiver)

### Activity

유저와 **상호 작용하기 위한 진입점**이며 앱이 UI를 그리는 창을 제공합니다. 여러 액티비티가 상호작용하여 애플리케이션을 구성하며 각각의 액티비티는 독립적으로 운용됩니다.



**Activity Lifecycle Callback Method**

![Android Activity Lifecycle - javatpoint](https://images.javatpoint.com/images/androidimages/Android-Activity-Lifecycle.png)

- `onCreate` : 첫 번째로 시작될 때 실행됩니다. **초기화 작업을 실행**하는데 용이합니다. 이 함수는 반드시 오버라이드하여 사용해야 합니다. `savedInstanceState`라는 파라미터를 수신하는데, 이는 이전 상태가 저장된 `Bundle` 객체입니다.

- `onStart` : 액티비티가 **사용자에게 보여질 때 호출**됩니다. `onRestart`에서 재호출되거나 기타 이유로 재호출 될 수 있는 함수입니다. 매우 빠른 속도로 실행됩니다.

- `onResume` : 사용자가 **액티비티와 상호작용 할 수 있게 되는 타이밍에 호출**됩니다. 사용자가 현재 액티비티로 다시 돌아올 때도 호출될 수 있습니다. (전화를 받고 오는 등..) `onPause`와 쌍으로 리소스를 등록하거나 해제하는데 용이합니다.

- `onPause` : 액티비티가 **조금이라도 가려지거나 포커스를 잃는 경우**(멀티 윈도우 화면 시)에 호출됩니다. 언젠가 다시 시작할 작업을 일시중지하는 작업을 수행하기에 용이합니다. 전화가 오거나, 시스템 다이얼로그 때문에 화면이 가려진다거나 하는 등의 이벤트 발생 시 호출됩니다. **아주 잠깐 실행되기에 무엇을 저장하는 작업을 두기엔 부적합**합니다.

- `onStop` : 액티비티가 사용자에게 **완전히 보여지지 않을 때 호출**됩니다. 액티비티가 백그라운드로 진입하는 경우처럼 사용자가 액티비티와 직접적으로 상호작용 할 수 없게 됩니다. **필요하지 않은 리소스를 해제하거나 정리하기에 용이**합니다. (애니메이션 중지, 무거운 리소스 해제 등등..)

- `onDestroy` : 액티비티가 사용자에 의해 종료되거나 기기 회전 등 화면 구성의 이벤트가 발생하여 액티비티가 **파괴되는 경우 호출**됩니다. 이전까지 **정리되지 않는 리소스가 있다면 여기서 정리**해야 합니다. 그렇지 않으면 메모리 누수의 위험이 존재합니다. 이후 액티비티는 메모리에서 완전히 소멸합니다.



> Question
> 
> A 액티비티에서 B 액티비티로 진입했다가 다시 A로 진입하는 경우 Lifecycle은 어떻게 될까요?
> 
> Answer
> 
>  A : `onCreate` -> A : `onStart` -> A : `onResume` -> 전환 이벤트 발생 -> A : `onPause` -> B : `onCreate` -> B : `onStart` -> B : `onResume` -> A : `onStop` -> A : `onSaveInstanceState` -> 다시 A로 전환 이벤트 발생 및 B 액티비티 종료 -> B : `onPause` -> A : `onRestart` -> A : `onStart` -> A : `onResume` -> B : `onStop` -> B : `onDestroy`



### Service

서비스는 백그라운드에서 실행되는 컴포넌트입니다. 크게 두 가지로 나뉘고 세분화하면 세 가지로 나뉩니다.

- **Started Service**
  
  - **foreground service** : 어느 정도 사용자에게 **보여지는 작업**을 수행합니다. (ex : 상단바의 컨트롤러) 반드시 알림(노티피케이션)이 지속적으로 표시해야 합니다.
  
  - **background service** : 사용자에게 **보여지지 않는 작업**을 수행합니다. 앱이 API 26 이상을 타겟팅한다면 시스템 자원 낭비 차원에서 실행이 제한되는데, [`WorkManager`](https://developer.android.com/topic/libraries/architecture/workmanager)를 사용하는 것이 권장된다.

- **Bound Service** : **앱 컴포넌트가 바인딩 된 형태의 서비스**입니다. 바인딩된 컴포넌트와 클라이언트-서버 구조처럼 상호작용하는 것이 가능합니다. 
  
  ![](https://velog.velcdn.com/images%2Fhaero_kim%2Fpost%2F7dc04ab6-0179-4047-980f-662c1df8db1a%2F0_rbbPsYYkjekH6LYV.png)

> 세 가지 서비스 구조가 분리된 것이 아닌, 혼합하여 사용할 수 있으며 기본적으로 서비스를 생성하면 호스팅된 프로세스의 기본 스레드에서 동작하게 되는데 블로킹 우려가 있는 작업(음악 재생, 파일 다운로드)을 실행하려면 별도의 스레드를 생성하여 작업을 수행해야 합니다. 

> Question
> 
> 서비스를 사용하면서 라이프 사이클 관리는 어떻게 해야할까?
> 
> Answer
> 
> 개발자가 자체적으로 생명주기 콜백 메소드에 맞춰서 작업을 해도 되지만, **LifecycleService**를 상속받는 Service를 구현하여 쉽게 작업할 수 있습니다.



### Broadcast Receiver

안드로이드의 **시스템 이벤트나 기타 앱에서 특정 이벤트가 발생했을 때 이를 수신**하는 컴포넌트입니다. 앱에서 특정 브로드캐스트를 수신하는 리시버를 등록하면 시스템은 해당 브로드캐스트가 발생할 때마다 해당 앱에 브로드캐스트를 자동으로 라우팅해주는데, 이 동작은 옵저버 패턴으로 구현되어 있습니다.

> 앱에서 브로드캐스트 이벤트를 발생시킬 수도 있고 다른 앱에서 이를 수신받게 할 수도 있습니다.



### Component Provider

다른 앱들과 **특정 데이터를 공유할 수 있도록 해주는 컴포넌트**입니다. 데이터 캡슐화, 데이터 보안처리 등 다양한 필수 메커니즘이 구현되어 있기 때문에 쉽게 데이터에 안전하게 접근하고 수정할 수 있습니다. 

![콘텐츠 제공자와 기타 구성요소 간의 관계](https://developer.android.com/static/guide/topics/providers/images/content-provider-tech-stack.png?hl=ko)

추상화가 잘 되어 있기 때문에, **타 앱의 데이터 액세스 권한 의존성과 상관없이 앱 데이터베이스를 구현**할 수 있으며 **다른 데이터베이스로 이관해도 타 앱에 영향이 없습니다**. 데이터 접근에 대한 **세분화된 제어 기능을 제공**하며 (타 앱에서만 데이터에 접근이 불가하도록) **여러 데이터 저장소에 액세스하기 위한 세부 정보를 추상화**할 수 있습니다. `CursorLoader` 객체를 사용하여 비동기식 쿼리를 실행하고 앱의 UI에서 결과를 반환합니다.



# Context

앱의 상태를 지니고 있으며, **액티비티나 애플리케이션의 정보를 얻을 수 있는 객체**입니다. `Activity`와 `Application` 클래스가 이 클래스를 상속받고 있으며, Application Context와 Activity Context로 나눌 수 있습니다.

- **Application Context** : 애플리케이션 라이프사이클과 묶여있는 Context이며, 오랫 동안 지속되거나 전역적인 부분에서 사용되어야 할 경우 접근하여 사용하기에 적합합니다.

- **Activity Context** : 액티비티 라이프사이클과 묶여있는 Context이며 토스트, 다이얼로그처럼 액티비티보다 라이프사이클 주기가 짧은 컴포넌트에서 사용하기에 적합합니다. 

> Application Context는 Activity Context가 지원하는 모든 것을 지원하지 않기 때문에, Application Context를 GUI 같은 이벤트들에 사용하게 된다면 크래시가 발생할 확률이 높으며 앱 프로세스가 살아있는 동안 계속 남아있기 때문에 적절한 시기에 메모리에서 할당해주는 것이 중요합니다.



# Intent

인텐트는 일종의 **메시지 객체**입니다. 관심있는 컴포넌트에 데이터를 보관하여 전달할 수 있습니다. 액티비티를 시작하거나, 서비스를 시작하거나, 브로드캐스트를 전송하는 등 다양한 이벤트에 사용할 수 있습니다. 

- **명시적 인텐트** : **컴포넌트 이름을 지정하여 사용하는 방식**입니다. 일반적으로 앱 내부에서 특정 컴포넌트를 실행할 때 사용됩니다. 파일을 다운로드하는 서비스를 시작하거나 액티비티를 시작하는 등의 이벤트에 사용되는 인텐트가 명시적 인텐트에 해당합니다.

- **암시적 인텐트** : 특정 컴포넌트의 이름을 지정하지 않고 대신에 **실행해야하는 일반적인 작업을 선언하는 방식**입니다. 암시적 인텐트를 사용하면 Android 시스템에서 적절한 컴포넌트를 찾아서 실행하는데, 이 때 여러 앱의 매니페스트 파일에 선언된 인텐트 필터와 비교하여 컴포넌트를 찾아냅니다. 호환되는 필터가 여러 개인 경우 다이얼로그가 표시되어 사용자가 직접 선택할 수 있도록 합니다.
  
  <img src="https://developer.android.com/static/images/components/intent-filters_2x.png?hl=ko" title="" alt="" width="513">

인텐트는 컴포넌트 이름, 액션, 데이터 (URI 객체), 카테고리, 엑스트라 (키-값 쌍의 번들 데이터), 플래그 등의 값을 보관하여 전달할 수 있습니다.



## Intent Filter

**인텐트를 수신할 때 구별할 수 있는 필터**입니다. 매니페스트 파일에서 특정 컴포넌트를 선언할 때, `<intent-filter>` 태그를 사용하여 선언할 수 있습니다. 사용할 수 있는 값들은 아래와 같습니다.

- `action` : 수행할 작업에 대한 내용을 정의합니다. 대표적으로 사용되는 액션들은 아래와 같습니다.
  
  - `MAIN` - 기본 진입점으로 사용됨 (intent에서 다른 정보를 요구하지 않음) 
  
  - `VIEW` - 사용자에게 데이터를 표시함
  
  - `DEFAULT` - `VIEW`와 동의어

- `category` : 인텐트 종류에 대한 내용을 정의합니다. 인텐트 빌드 시엔 잘 쓰이지 않고, 필터에서 선언될 때 자주 사용됩니다.
  
  - `LAUNCHER` - 최상위 진입점으로 사용됨.
  
  - `DEFAULT` - 암시적 인텐트에 의해 사용됨. 카테고리가 설정되지 않은 암시적 인텐트와 자동으로 일치하게 됨.

- `data` : 데이터에 포함된 URI 정보에 대한 필터를 지정할 수 있음. (ex: `host`, `scheme` 값 등)



## Pending Intent

**외부 앱에 권한을 허가하여 안에 들어 있는 `Intent` 객체를 마치 본인 앱 자체 프로스세스에서 실행되는 것처럼 사용하게 하는 것**을 목적으로 하는 객체입니다. 주요 사용 사례는 아래와 같습니다.

- 사용자가 알림을 통해 작업을 실행할 때 실행할 인텐트를 선언합니다. (Android 시스템의 `NotificationManager`가 `Intent`를 실행함)

- 사용자가 위젯으로 작업을 실행할 때 실행할 인텐트를 선언합니다. (메인 화면 앱이 `Intent`를 실행함)

- 향후 지정된 시간에 인텐트가 실행되도록 선언합니다. (`AlarmManager`가 `Intent`를 실행함)

`PendingIntent`도 `Intent`와 마찬가지로 동일한 고려 사항을 생각하여 생성해야 합니다. `getActivity`, `getService`, `getBroadcast`와 같은 `PendingIntent`의 함수를 사용하여 `PendingIntent` 객체를 생성할 수 있습니다.

> Android 12 이상을 타겟팅 하는 경우 `PendingIntent` 객체의 변경 가능 여부를 지정해야 합니다. `PendingIntent.FLAG_MUTABLE`  또는 `PendingIntent.FLAG_IMMUTABLE` 플래그를 각각 사용하여 변경 불가능 여부를 선업합니다.



# Fragment

액티비티 내에서 사용되는 재사용 가능한 UI 부분을 표현하는데 사용되는 객체입니다. 다양한 화면 크기에 반응하는 앱을 만들어야 한다면, 화면 크기에 따라 프래그먼트를 다르게 배치하여 표현이 가능합니다. 자체 수명주기가 존재하고 자체적으로 입력 이벤트를 처리할 수 있습니다. 

프래그먼트는 `FragmentManager`에 의해 관리되며 `FragmentManager`는 프래그먼트가 어떤 상태여야 하는지 확인 한 후 다음 상태로 전환하는 일을 담당합니다. 개발자는 `onAttach`, `onDetach` 콜백에서 `FragmentManager`에 프래그먼트가 추가되거나 삭제되었을 때 처리되어야 하는 작업을 작성할 수 있습니다.

프래그먼트의 수명주기는 프래그먼트 자체의 수명주기와 보여지는 뷰의 수명주기로 나눌 수 있고 아래와 같은 메소드들이 호출됩니다. 

`Lifecycle` 상태는 `State` 열거형으로 표현됩니다. (`INITIALIZED`, `CREATED`, `STARTED`, `RESUMED`, `DESTROYED`)

**Fragment Lifecycle Callback Method**

<img src="https://developer.android.com/static/images/guide/fragments/fragment-view-lifecycle.png?hl=ko" title="" alt="프래그먼트 수명 주기 상태와 이 상태가 프래그먼트의 수명 주기 콜백 및 프래그먼트의 뷰 수명 주기와 갖는 관계" width="510">

- `onCreate` : 프래그먼트가 `FragmentManager`에 추가되고 `onAttach`가 호출된 직후의 상태입니다. 프래그먼트 자체와 연결된 저장 상태를 복원하는데 적절합니다. (인수로 `savedInstanceState`가 존재합니다)

- `onCreateView` : 해당 함수를 재정의하여 `View`를 생성할 수 있습니다. (프래그먼트 생성자에 `@LayoutId`를 넘겨서도 가능함) 

- `onViewCreated` : 이 위치에서는 `View`의 초기 상태를 설정하고 `View`를 업데이트 하는 콜백을 받는 `LiveData` 인스턴스를 관찰하거나 `RecyclerView`, `ViewPager2`의 어댑터를 지정하기에 적합합니다.

- `onViewStateRestored` : 이전 뷰의 상태가 복원되고 뷰의 `Lifecycle`이 `CREATED` 상태로 전환됩니다. 여기에서 프래그먼트의 뷰와 관련된 상태를 복원하기에 적합합니다. (`savedInstanceState`을 파라미터로 수신함)

- `onStart` : 수명주기 인식 컴포넌트를 연결하기에 적합합니다. 이 상태와 연결하면 프래그먼트의 뷰를 사용할 수 있고 하위 `FragmentManager`에서 `FragmentTransaction`을 안전하게 실행할 수 있습니다. 

- `onResume` : 모든 애니메이션 효과와 트랜지션 효과가 완료되어 프래그먼트가 사용자와 상호작용할 수 있는 상태입니다. 만약 `RESUMED` 상태가 아닌 프래그먼트가 뷰에 포커스를 수동으로 지정하는 등의 작업을 처리하려고 시도해서는 안 됩니다.

- `onPause` : 프래그먼트가 포커스를 잃기 시작하면 프래그먼트의 뷰의 `Lifecycle`은 다시 `STARTED` 상태로 전환되고 관찰자에게 `ON_PAUSE` 이벤트를 보내며, 프래그먼트의 `onPause` 콜백이 호출됩니다.

- `onStop` : 더 이상 프래그먼트가 표시되지 않는 경우 `Lifecycle`은 `CREATED`로 전환되고 관찰자에게 `ON_STOP` 이벤트를 보냅니다. 하위 `FragmentManager`에서 `FragmentTransaction`을 안전하게 실행할 수 있는 마지막 지점입니다. **이 때 `onStop`과 `onSaveInstanceState`의 순서는 API 수준에 따라 순서가 다릅니다**. 안전하게 상태가 저장된 시점을 이용하려면 `onSaveInstanceState` 이후를 참조해야 합니다.
  
  ![onStop() 및 onSaveInstanceState()의 호출 순서 차이](https://developer.android.com/static/images/guide/fragments/stop-save-order.png?hl=ko)

- `onSaveInstanceState` : 상태를 저장할 수 있는 콜백이며 `outState` 파라미터에 상태를 저장할 수 있습니다.

- `onDestroyView` : 프래그먼트의 뷰가 윈도우에서 분리되면 뷰의 `Lifecycle`이 `DESTROYED` 상태에 진입되고 `ON_DESTROY` 이벤트를 관찰자에 보냅니다. 이 시점에서 뷰의 수명주기는 끝나게 됩니다. **이 때 프래그먼트 뷰의 모든 참조를 제거해야 합니다**.

- `onDestroy` : 프래그먼트가 삭제되거나 `FragmentManager`가 소멸되면 프래그먼트의 `Lifecycle`이 `DESTROYED` 상태로 전환되고 `ON_DESTROY` 이벤트를 관찰자에게 전송합니다. 

- 

> Question
> 
> 기본 생성자에 인자를 넣어서 생성하지 않는 이유는 무엇인가요?
> 
> Answer
> 
> 인자를 생성자에 넣어서 생성하는 경우 Fragment는 재생성 될 때 기본 생성자로만 생성되기에  인자를 더 이상 알 수 없기 때문에 이슈가 발생합니다. 이런 이유로 Fragment 생성 시에는 기본 생성자를 사용하여 생성해야 하며 argument를 통해 데이터를 전달하는 것이 권장됩니다.



> Question
> 
> 프래그먼트에서 LifecycleOwner를 this로 지정하는 것과 ViewLifecycleOwner로 지정하는 것의 차이는 무엇인가요?
> 
> Answer
> 
> 프래그먼트의 수명주기와 프래그먼트에 붙어있는 뷰의 수명주기가 다릅니다. 프래그먼트는 액티비티와 다르게 onDestroy가 호출되지 않은 상태에서 onCreateView가 여러 번 호출될 수 있습니다. 그러므로 뷰의 UI를 업데이트 하는 경우, 프래그먼트의 Lifecycle을 사용하게 된다면 메모리 누수를 유발할 수 있기 때문에 뷰의 Lifecycle을 이용하는 것이 좋습니다.



# RecyclerView

- [RecyclerView Deep Dive 참조](https://medium.com/hongbeomi-dev/recyclerview-deep-dive-with-google-i-o-2016-21e0895819d2)

> Question
> 
> 리사이클러뷰의 아이템 뷰가 모두 동일한 크기를 가질 때 성능 향상을 이룰 수 있는 방법이 있을까요?
> 
> Answer
> 
> `setHasFixedSize` 함수를 사용하면 리사이클러뷰가 아이템이 추가되거나 제거될 때 불필요한 레이아웃 계산을 건너뛸 수 있습니다. 

# Save State

안드로이드 앱 개발에서 액티비티가 소멸한 뒤 신속하게 화면의 UI 상태를 저장하고 복원하는 것은 중요합니다. 아래와 같은 방법으로 상태를 저장할 수 있습니다.

- ViewModel 객체 내에 위치

- **Compose** : `rememberSaveable` 사용

- **View** : `onSaveInstanceState` API 사용

- **ViewModel** : `SavedStateHandle` 객체 사용

- 로컬 저장소 사용 : (`SharedPreference`, `DataStore`, `Room`...등등)



### Activity에서 데이터 복원

아래 그림은 액티비티 라이프사이클에 따라 상태 저장/복원 콜백이 호출되는 순서를 나타냅니다.

<img src="https://pluu.github.io/assets/img/blog/2020/0208-savedsate/04.png" title="" alt="" width="488">

`Activity`에서는 `onSaveInstanceState`에서 상태를 저장하고, `onCreate(savedInstanceState: Bundle?)` 함수를 사용하거나 `onRestoreInstanceState` 함수를 사용하여 상태를 복원할 수 있습니다.

> `onRestoreInstanceState`는 `onStart` 이후에 호출되고, 재생성되어 시작될 때만 호출됩니다.





상태를 저장하기 위한 방법들의 옵션은 아래와 같은 기준에 따라 다릅니다.

![](/Users/hongbeom/Library/Application%20Support/CleanShot/media/media_5nojApuj4L/CleanShot%202024-11-13%20at%2016.04.28@2x.png)

`ViewModel` 내부에 저장된 상태는 저장된 인스턴스 상태(Save Instance State)와 달리 시스템에 의해 종료될 때는 폐기됩니다. 시스템에 의한 프로세스 종료 후 재시작 시 상태를 유지하고 싶다면 

[SavedStateHandle](https://developer.android.com/topic/libraries/architecture/viewmodel/viewmodel-savedstate?_gl=1*k4lah3*_up*MQ..*_ga*MTI1MjE3MTM0NC4xNzMxNDgxMzA5*_ga_6HH9YJMN9M*MTczMTQ4MTMwOS4xLjAuMTczMTQ4MTMwOS4wLjAuMTYxNDA5Njk4Nw..) API를 활용해야 합니다. 이 API는 키-값 쌍으로 구성되어 사용할 수 있으며 UI 로직이 아닌 **비즈니스 로직**을 저장하기에 적합합니다. **UI 로직**은 `onSaveInstanceState` API나 `rememberSaveable` API를 사용하는 것이 적합합니다.

> Question
> 
> API를 사용했다면, 상태는 어떤 원리로 액티비티가 파괴되도 유지되는 걸까요? 
> 
> 



## ViewModel

뷰모델은 액티비티가 `onCreate`를 최초로 호출할 때 생성되며 액티비티가 최종적으로 파괴되는 순간에 소멸합니다. 액티비티와 라이프사이클을 비교해본다면 아래와 같습니다. 

![](https://2.bp.blogspot.com/-yDA6lQPeUM0/WjMEoM8_qsI/AAAAAAAABrU/aSrk1ePRyugp6Mna8mSPlq5K-4Moz9EcACLcBGAs/s1600/image1.png)

뷰모델은 액티비티의 화면 구성 이벤트에도 상태를 안전하게 보관할 수 있는 장소입니다. 하지만 시스템에 의한 프로세스 종료 시 상태는 유지되지 않기에, `SavedStateHandle` API를 사용하여 이를 해소할 수 있는데 뷰모델이 생성될 때 `SavedStateViewModelFactory`를 통해 Saved State Handle 정보가 뷰모델로 전달됩니다. (`ComponentActivity`에서 `SavedStateViewModelFactory` 객체를 Lazy한 변수로 사용하고 있습니다.)

액티비티와 프래그먼트는 각각 `ComponentActivity`, `Fragment` 클래스에서 `SavedStateViewModelFactory`를 통해 `ViewModel`을 생성하는 `Factory` 객체를 가지고 있는데, 여기서 파라미터로 각각 `intent`의 `extra`, `argument`를 넘겨줌으로서 `SavedStateHandle`의 기본값이 지정되게 됩니다.



> Question
> 
> ViewModel은 어떻게 액티비티가 파괴되도 유지되나요?
> 
> Answer
> 
> `ViewModelStoreOwner`라는 인터페이스가 `ViewModelStore`를 구성 변경 중에도 유지하고 만약 파괴될 경우 `ViewModelStore`의 `clear`를 호출하는 역할을 담당하는데, 이 `ViewModelStoreOwner`를 구현하는 클래스가 `ComponentActivity`와 `Fragment`입니다.  액티비티의 경우 뷰모델 객체를 관리하는 `ViewModelStore`를 생성할 때 `NonConfigurationInstances`라는 객체에서 참조하거나 스스로 생성하는데, 이 `NonConfigurationInstances`가 구성 변경시 이전 `ViewModelStore`의 인스턴스를 보유하고 있기에 구성 변경 시 해당 인스턴스를 액티비티/프래그먼트에 다시 연결해줌으로써 `ViewModel`이 유지됩니다. (아래 코드 참고)
> 
> ```kotlin
> final override fun onRetainNonConfigurationInstance(): Any? {
>     val custom = onRetainCustomNonConfigurationInstance()
>     var viewModelStore = _viewModelStore
>     // 현재 액티비티 인스턴스가 처음 호출될 경우
>     if (viewModelStore == null) {
>         val nc = lastNonConfigurationInstance as NonConfigurationInstances?
>             if (nc != null) {
>                 // 이전의 뷰모델 스토어 객체 연
>                 viewModelStore = nc.viewModelStore
>             }
>         }
>     if (viewModelStore == null && custom == null) {
>         return null
>     }
>     val nci = NonConfigurationInstances()
>     nci.custom = custom
>     nci.viewModelStore = viewModelStore
>     return nci
> }
> ```
> 
> `Fragment`의 경우 실질적으로는 `FragmentManager`가 `ViewModelStore`를 관리하고 있으며, 내부적으로 해시맵 객체를 통해 `Fragment` 의 이름을 key로 두고 `ViewModelStore`를 value으로 보관하고 있습니다. 이 때 액티비티와 비슷하게 액티비티 재구성 이벤트 전반에 걸쳐 보존되는 `FragmentManagerNonConfig` 객체를 이용하여 `ViewModelStore`를 보존시켜줍니다.



> Question
> 
> viewModelScope를 사용한다면 이 스코프는 어떻게 구성되어 있고, 왜 그렇게 구성되어 있으까요?
> 
> Answer
> 
> `CloseableCoroutineScope` 타입의 scope를 사용하는데 이 객체는 `Dispatchers.Main.immediate` 디스패처와 `SupervisorJob`이 합쳐진 `CoroutineContext`를 사용하고 있습니다. **`ViewModel`의 `onCleared`가 호출될 때 실행되고 있는 job을 종료하기 위해 `Closeable` 스코프로 구성**되어있으며, 일반 `Main`이 아닌 `Main.immediate` 디스패처가 사용된 이유는 **이미 올바른 컨텍스트에 있는 경우에 추가 재디스패치 없이 즉시 코루틴을 실행하는 디스패처를 반환**하여 사용하기 위함입니다. 코루틴은 Context에 맞는 스레드로 적재하는 과정이 필요한데, 이 과정을 생략하고 즉각적으로 작업을 실행하기 위해 해당 디스패처를 사용합니다. 또한 **`SupervisorJob` 내의 자식 코루틴들은 서로에게 영향을 받지 않기에 여러 개의 자식 코루틴들 중 하나가 취소되어도 다른 코루틴에 영향을 주지 않으며, `SupervisorJob`에도 영향을 미치지 않습니다**. 이런 이유로 `SupervisorJob`을 내부적으로 사용하고 있습니다.



# Thread vs Process

**Process**는 메모리에 올라가서 실행되고 있는 작업의 단위를 뜻합니다. 반면 **Thread**는 프로세스 내에서 실행되는 흐름의 단위를 뜻합니다. 

각각의 프로세스는 별도의 주소 공간에서 실행되며, 프로세스는 타 프로세스의 변수나 자료구조에 접근 할 수 없습니다. 프로세스는 타 프로세스의 자원에 접근하려면 IPC(inter process communication) 방식을 사용해야 합니다. (ex : 파일, 소켓 등을 이용한 통신 방법)

스레드는 하나의 프로세스 내에서 동작되는 여러 실행의 흐름이기에 프로세스 내의 자원들에 접근하고 스레드끼리 공유할 수 있습니다. 



> Question
> 
> 안드로이드 개발 시 멀티쓰레드 환경에서 안전하게 접근을 처리할 수 있는 방법들은 어떤 것이 있을까요? 
> 
> Answer
> 
> - `synchronized` : 메서드나 블록을 감싸서 사용하면 **단일 스레드만 접근하도록 할 수 있습니다.** 중첩으로 사용할 경우 데드락을 마주할 수 있기 때문에 주의하여야 합니다. 
> 
> - `volatail` : **변수에 해당 어노테이션을 붙여서 사용하면 변수의 값이 메인 메모리에만 저장되어 타 쓰레드 환경에서 메인 메모리의 값을 참조**하므로 변수 값 불일치 문제를 해결할 수 있게 됩니다. 다만 기본적으로 변수는 CPU 캐시에 저장되기 때문에 이 경우 CPU 캐시가 아닌 메인 메모리의 값을 참조하게 되어 성능이 떨어질 수 있습니다.
> 
> - `지역 변수 활용하기` : 아래 코드와 같이 전역 변수를 각각 다른 스레드가 참조하는 구조에서 동기화를 보장하기 위해서는 문제가 발생할 수 있는 변수를 지역 변수로 캐스팅하여 사용하여 해결할 수 있습니다.
>   
>   ```kotlin
>   private var str: String? = null
>   
>   // 메인 스레드에서 실행되는 함수
>   fun startProcess1(value : String) {
>       this.str = value
>   }
>   
>   // 타 스레드에서 실행되는 함수
>   fun startProcess2() {
>       if (str == null) {
>           return
>       }
>       str.XXX
>   }
>   
>   --------------------------------------------
>   
>   // 지역 변수를 활용하여 해결
>   fun startProcess2() {
>       val localStr = str
>       if (localStr == null) {
>           return
>       }
>       localStr.XXX
>   }
>   ```



# Dependency Injection

종속성 주입은 특정 클래스가 다른 클래스가 필요할 경우 직접 종속성을 가져오는 대신, 인스턴스를 주입 받는 것을 의미합니다. 코드의 재사용성, 리팩토링, 테스트에 용이성을 가집니다. 안드로이드에서 사용되는 DI 라이브러리는 크게 `Koin`과 `Dagger`로 나뉩니다.

## Koin

`Koin`은 코틀린 DSL을 사용하고 런타임에서 의존성 주입을 제공하는 라이브러리 입니다. 어노테이션 프로세싱을 사용하여 빌드 시 별도의 파일을 생성하지 않지만 리플렉션을 사용하고 인스턴스를 런타임에 주입해주기 때문에 런타임 시 크래시를 마주할 수 있습니다. Service Locator 패턴을 사용합니다. 

(`Koin`의 축소판으로 구현한 [SimpleKoin](https://github.com/hongbeomi/SimpleKoin) 코드 참고)



## Dagger Hilt

힐트는 코인과 다르게 컴파일 시점에 주입과 관련된 스텁 코드를 생성하고 오류가 있는 경우 컴파일 시점에 표시합니다. 어노테이션을 통해 사용할 수 있으며 의존 관계 파악이 쉽습니다.

사용할 수 있는 스코프와 컴포넌트는 아래 그림과 같습니다.

<img title="" src="https://velog.velcdn.com/images/leeyjwinter/post/e6acd118-6e64-4d65-ae4e-b0daced88c46/image.png" alt="안드로이드 Hilt 4] - Component와 Scope" width="592">



> Question
> 
> Hilt 사용 시 동일한 타입의 객체를 구분해서 제공해야 하는 경우 어떻게 처리할 수 있을까요?
> 
> Answer
> 
> 특정 이름으로 `@Qualifier` 어노테이션을 작성하고 제공하는 곳에 해당 이름으로 어노테이션을 달아서 사용하여 처리할 수 있습니다.



# Architecture

## MVC

model, view, controller로 이루어져 있으며 **model**은 데이터, 비즈니스 로직들을 합쳐 부릅니다. (나머지 아키텍쳐도 동일합니다.) **view**는 컨트롤러에 의해 UI 갱신이 이루어집니다. **controller**는 사용자의 이벤트를 받고 그에 따라 model을 변경하며 view를 다룹니다(activity...). 컨트롤러는 뷰와 강하게 결합되고 시간이 지남에 따라 많은 코드가 쌓이게 됩니다.



## MVP

mvc와 다르게 **view**는(xml, activity...) 사용자의 이벤트를 입력받으며 presenter를 참조합니다. **presenter**는 view로 부터 이벤트를 전달받아 model을 처리하고 다시 view를 업데이트 합니다. 또한 view와 1:1 관계를 가지고 있습니다. view와 model의 비즈니스 로직이 의존하지 않게 되지만, view와 presenter가 강하게 결합됩니다. (물론 인터페이스로 의존성을 처리해볼 수 있습니다.) 또한 view마다 큰 의미가 없는 presenter가 추가될 수 있습니다.

## 

## MVVM

mvvm에서 **view**는 viewmodel을 알고 있습니다. **viewmodel**은 view를 모르며, view로 부터 사용자 이벤트를 전달 받고 model을 업데이트 합니다. viewmodel이 모델을 업데이트하고 외부로 노출할 상태를 정의하면, view가 적절한 방법으로 해당 상태를 가져갑니다. (observable, 데이터 바인딩 등..) 이 아키텍쳐는 view가 model의 비즈니스 로직에 의존하지 않으며 view와 viewmodel이 강하게 결합되지 않습니다. 하지만 역시 viewmodel이 시간이 지날수록 복잡해진다는 단점이 있습니다.



## MVI

mvi에서 **model**은 앱의 상태를 나타냅니다. 기존의 아키텍쳐에서 많은 수의 입출력 이벤트 처리는 백그라운드 처리가 많아질 경우 문제를 일으킬 수 있으며 비즈니스 로직과 뷰가 다른 상태를 가질 경우 동기화 처리가 쉽지 않아집니다. model이 자체적으로 앱의 상태를 나타낸다면 이런 이슈를 해결할 수 있습니다. 또한 변경이 불가능하여 단일 상태를 유지할 수 있도록 하는 것에 목표를 두고 있습니다. **intent**는 앱의 상태를 변화시킬 액션을 의미합니다. mvi는 **데이터가 단방향으로 순환**되고, view가 일관성있는 상태를 가질 수 있다는 장점이 있습니다. 



# Network

## Retrofit2

## OkHttp3

## Gson



# Image

## Glide

## Coil



# UI Optimization

- Like 처리

- 중복된 API 호출 문제 해결



# Kotlin

## Sealed class vs Enum



## Lazy vs lateinit



## Serialize vs Parcelable



## Coroutine

- CPS

> Question
> 
> 스레드와 차이점이 뭘까요?



## Flow

### Hot Flow

- StateFlow

- SharedFlow

### Cold Flow



## LiveData

> Question
> 
> LiveData 대신 Flow를 쓴다면 이유가 무엇인가요?



# Compose



## Composition



## Recomposition



## Performance



## LazyXXX

> Question
> 
> key, contentType은 무엇이고 각각 지정하지 않으면 어떤 값이 사용되나요?

> Question
> 
> 어떤 원리로 아이템들이 재사용되는 걸까요