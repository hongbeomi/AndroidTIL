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
  
  - `MAIN` - 기본 진입점으로 사용됨 (intent에서 다른 정보를 요구하지 않음) 
  
  - `VIEW` - 사용자에게 데이터를 표시함
  
  - `DEFAULT` - `VIEW`와 동의어

- `category` : 인텐트 종류에 대한 내용을 정의합니다. 인텐트 빌드 시엔 잘 쓰이지 않고, 필터에서 선언될 때 자주 사용됩니다.
  
  - `LAUNCHER` - 최상위 진입점으로 사용됨.
  
  - `DEFAULT` - 암시적 인텐트에 의해 사용. 카테고리가 설정되지 않은 암시적 인텐트와 자동으로 일치하게 됨.

- `data` : 데이터에 포함된 URI 정보에 대한 필터를 지정할 수 있음. (ex: `host`, `scheme` 값 등)



## Pending Intent

**외부 앱에 권한을 허가하여 안에 들어 있는 `Intent` 객체를 마치 본인 앱 자체 프로스세스에서 실행되는 것처럼 사용하게 하는 것**을 목적으로 하는 객체입니다. 주요 사용 사례는 아래와 같습니다.

- 사용자가 알림을 통해 작업을 실행할 때 실행할 인텐트를 선언합니다. (Android 시스템의 `NotificationManager`가 `Intent`를 실행함)

- 사용자가 위젯으로 작업을 실행할 때 실행할 인텐트를 선언합니다. (메인 화면 앱이 `Intent`를 실행함)

- 향후 지정된 시간에 인텐트가 실행되도록 선언합니다. (`AlarmManager`가 `Intent`를 실행함)

`PendingIntent`도 `Intent`와 마찬가지로 동일한 고려 사항을 생각하여 생성해야 합니다. `getActivity`, `getService`, `getBroadcast`와 같은 `PendingIntent`의 함수를 사용하여 `PendingIntent` 객체를 생성할 수 있습니다.

> Android 12 이상을 타겟팅 하는 경우 `PendingIntent` 객체의 변경 가능 여부를 지정해야 합니다. `PendingIntent.FLAG_MUTABLE`  또는 `PendingIntent.FLAG_IMMUTABLE` 플래그를 각각 사용하여 변경 불가능 여부를 선업합니다.



# Fragment

# 

# RecyclerView



# Save State

## ViewModel

> Question
> 
> ViewModel은 어떻게 액티비티가 파괴되도 유지되나요?

## Save state API



# Thread vs Process



# Dependency Injection

## Koin

## Dagger Hilt



# Architecture

## MVC

## MVP

## MVVM

## MVI



# Network

## Retrofit2

## OkHttp3

## Gson



# Image

## Glide

## Coil



# UI Optimization

## Like 처리

## 중복된 API 호출 문제



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