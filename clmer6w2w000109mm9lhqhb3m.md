---
title: "[Android] 4대 컴포넌트"
datePublished: Mon Jan 09 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmer6w2w000109mm9lhqhb3m
slug: android-4
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/xUUZcpQlqpM/upload/5814cb73f8aea88ba217ffab449866b9.jpeg
tags: android

---

# 액티비티(Activity)

## 정의

> 안드로이드 시스템의 구성 요소 중 하나로 사용자와 상호작용하는 UI를 제공함과 동시에 진입점 역할도 수행.

### 진입점 역할이란?

예를 들어 ...

* 홈 화면에서 이메일 앱 실행 -&gt; 이메일 목록을 표시하는 액티비티 표시
    
* SNS에서 이메일 앱 실행 -&gt; 이메일을 작성하는 액티비티 표시
    

상황에 따라 여러 진입점을 가질 수 있다. 진입점 설정은 인텐트 및 인텐트 필터를 통해 할 수 있다.

## 단점

[레퍼런스](https://charlezz.medium.com/activity-vs-fragment-%EB%AC%B4%EC%97%87%EC%9D%84-%EC%84%A0%ED%83%9D%ED%95%B4%EC%95%BC-%ED%95%A0%EA%B9%8C-56ce7fa2bfc4)

### 퍼포먼스 관점

액티비티는 프래그먼트에 비해 상대적으로 무겁다. 액티비티의 경우 안드로이드 시스템에서 제공하는 액션 바, 액션 메뉴 등도 관리하고 앱의 전체 화면을 제어한다. 반면 프래그먼트는 작은 UI요소를 구성할 수 있기 때문이다. (프래그먼트에서 액션 바, 액션 메뉴를 사용할 수 있는 방법은 존재하나, 부모 액티비티에 접근해야함)

### 데이터 접근하기

액티비티간 데이터를 공유하는 가장 일반적인 방법은 인텐트(Intent)를 사용하는 방법 밖에 없다. (싱글톤 사용 및 애플리케이션 내 객체 공유 등의 특수한 경우는 고려하지 않는다.)  
**\-&gt; why? 액티비티는 다른 프로세스에서 실행하는 것을 염두하고 설계 되었기 때문에 메모리 영역을 공유하지 않기 때문. (진입점)**  
프래그먼트를 사용하는 경우 프래그먼트 간 데이터 공유는 액티비티내에서 자유롭게 이루어진다.

## 생명주기

![](https://developer.android.com/guide/components/images/activity_lifecycle.png align="left")

### onCreate()

시스템이 Activity를 생성할 때 실행되며 필수적으로 구현해야 한다. Activity의 전체 수명 주기 동안 한 번만 발생해야하는 기본 시작 로직을 실행한다. (데이터를 목록에 바인딩, Activity와 ViewModel 연결 등) 해당 메서드는 Activity의 이전 저장 상태가 포함된 Bundle 객체인 `savedInstanceState` 매개변수를 수신한다. (처음 생성된 Activity인 경우 Bundle 객체의 값은 null이다.)

Activity의 수명 주기와 연결된 수명 주기 인식 구성요소가 있다면 이 구성요소는 ON\_CREAT 이벤트를 수신한다. 따라서 @OnLifecycleEvent라는 주석이 있는 메서드가 호출되고, 수명 주기 인식 구성요소는 *Created* 상태에 필요한 모든 설정 코드를 실행할 수 있게 됩니다.

onCreate() 메서드가 실행을 완료하면 Activity는 *Started* 상태에 진입합니다. 그리고 시스템은 onStart()와 onResume() 메서드를 연달아 호출합니다.

### onStart()

onStart() 호출은 두 가지 경우로 나뉘어 실행되는데 처음 생성 이후 호출될 때와 onRestart() 콜백을 수신 후 재시작될 때 호출됩니다. onStart에서는 Activity가 사용자에게 표시되고, 포그라운드로 이동할 준비를 하게 됩니다. 해당 콜백은 빠르게 완료가 되고 onResume 콜백을 호출하게 됩니다.

Activity가 *Started* 상태로 전환하면 이 Activity의 수명 주기와 연결된 모든 수명 주기 인식 구성요소는 ON\_START 이벤트를 수신합니다.

### onResume()

onResume이 호출된 이후에만 Activity가 포그라운드 상태가 되었고 사용자와 상호작용을 할 수 있는 곳입니다. 실제로 setContentView 결과가 보여지는 부분입니다. 재시작 이후 백그라운드에 있던 Activity가 포그라운드에 상태가 되고 모든 화면에 해당 Acitivty가 가득 찬 모습이어야 합니다. 전화가 오거나 , 홈버튼을 눌러 해당 Activity의 화면이 꺼지는 이벤트가 발생하기 전까지 onResume 상태에서 머무르게 됩니다.

화면이 꺼지는 이벤트가 발생하면 Activity는 *Paused* 상태에 들어가고 onPause()가 호출됩니다. 다시 재개가 되면 onResume 콜백이 호출하게 됩니다. onResume()은 onPause()에서 해제했던 리소스들을 다시 초기화 하는 코드가 있어야 합니다. 즉, onResume()과 onPause()는 대칭적으로 리소스/데이터 초기화와 해제를 해야 합니다.

### onPause()

Activity가 포그라운드에서 백그라운드로 바뀌는 시점으로 화면의 일부가 가려진 상태일 때 onPause 콜백이 호출됩니다. Activity가 이 상태에 들어가는 이유는 여러 가지가 있습니다.

* 전화가 오거나, 사용자가 다른 Activity로 이동하거나, 기기 화면이 꺼지는 이벤트
    
* Android 7.0(API 수준 24) 이상에서는 여러 앱이 멀티 윈도우 모드에서 실행됩니다. 언제든지 그중 하나의 앱(창)만 포커스를 가질 수 있기 때문에 시스템이 그 외에 모든 다른 앱을 일시중지시킵니다.
    
* 새로운 반투명 Activity(예: 대화상자)이 열립니다. Activity이 여전히 부분적으로 보이지만 포커스 상태가 아닌 경우에는 일시중지됨 상태로 유지됩니다.
    

일시중지된 활동은 멀티 윈도우 모드에서 여전히 완전히 보이는 상태일 수 있습니다. 그러므로 멀티 윈도우 모드를 더욱 잘 지원하기 위해 UI 관련 리소스와 작업을 완전히 해제하거나 조정할 때는 onPause() 대신 onStop()을 사용하는 것이 좋습니다.

onPause는 아주 잠깐 실행되므로 현재 화면을 구성하는 데이터를 저장하거나, 네트워크 호출 또는 DB 트랙잭션과 같은 시간 소요가 있는 작업들을 실행해서는 안됩니다.

LifecycleObserver가 ON\_PAUSE 이벤트에 대응합니다.

### onStop()

Activity가 이벤트로 인해 새로운 화면이 나타나서 전체 화면을 가리게 되면 onStop 이 호출되게 됩니다. Activity이 중단됨 상태로 전환하면 이 Activity의 수명 주기와 연결된 모든 수명 주기 인식 구성요소는 ON\_STOP 이벤트를 수신합니다.

onStop()메서드에서는 필요하지 않은 리소스를 해제하거나 조정해야 합니다. 예를 들어 애니메이션을 일시 중지하거나, 세밀한 위치 업데이트에서 대략적인 위치 업데이트로 전환할 수 있습니다. onPause() 대신 onStop()를 사용하면 사용자가 멀티 윈도우 모드에서 Activity를 보고있더라고 UI 작업이 계속 진행됩니다.

또한 onStop()을 사용하여 CPU를 비교적 많이 소모하는 종료 작업을 실행해야 합니다. 예를 들어 정보를 데이터베이스에 저장할 적절한 시기를 찾지 못했다면 onStop() 상태일 때 저장할 수 있습니다.

> Activity가 STOP되면 시스템은 해당 Activity이 포함된 프로세스를 소멸시킬 수 있습니다(시스템이 메모리를 복구해야 하는 경우). Activity이 중단된 동안 시스템이 프로세스를 소멸시키더라도 Bundle(키-값 쌍의 blob)에 있는 View 객체(예: EditText 위젯의 텍스트) 상태가 그대로 유지되고, 사용자가 이 Activity으로 돌아오면 이를 복원합니다. 사용자가 돌아올 때의 Activity 복원에 관한 자세한 내용은 [Activity 상태 저장 및 복원](https://developer.android.com/guide/components/activities/activity-lifecycle#saras)을 참조하세요.

Activity은 정지됨 상태에서 다시 시작되어 사용자와 상호작용하거나, 실행을 종료하고 사라집니다. Activity이 다시 시작되면 시스템은 onRestart()를 호출합니다. Activity가 실행을 종료하면 시스템은 onDestroy()를 호출합니다. 다음 섹션에서는 onDestroy() 콜백에 관해 설명합니다.

### onDestroy

onDestroy()는 Activity이 소멸되기 전에 호출됩니다. 시스템은 다음 중 하나에 해당할 때 이 콜백을 호출합니다.

1. (사용자가 Activity을 완전히 닫거나 Activity에서 finish()가 호출되어) Activity이 종료되는 경우
    
2. 구성 변경(예: 기기 회전 또는 멀티 윈도우 모드)으로 인해 시스템이 일시적으로 Activity을 소멸시키는 경우
    

Activity이 소멸됨 상태로 전환하면 이 Activity의 수명 주기와 연결된 모든 수명 주기 인식 구성요소는 ON\_DESTROY 이벤트를 수신합니다. 여기서 수명 주기 구성요소는 활동이 소멸되기 전에 필요한 것을 정리할 수 있습니다.

Activity에 소멸되는 이유를 결정하는 로직을 입력하는 대신 ViewModel 객체를 사용하여 Activity의 관련 뷰 데이터를 포함해야 합니다. Activity이 구성 변경으로 인해 다시 생성될 경우 ViewModel은 그대로 보존되어 다음 활동 인스턴스에 전달되므로 추가 작업이 필요하지 않습니다. Activity가 다시 생성되지 않을 경우 ViewModel은 onCleared() 메서드를 호출하여 Activity가 소멸되기 전에 모든 데이터를 정리해야 합니다.

이와 같은 두 가지 시나리오는 isFinishing() 메서드로 구분할 수 있습니다.

Activity가 종료되는 경우 onDestroy()는 Activity가 수신하는 마지막 수명 주기 콜백이 됩니다. 구성 변경으로 인해 onDestroy()가 호출되는 경우 시스템이 즉시 새 Activity 인스턴스를 생성한 다음, 새로운 구성에서 그 새로운 인스턴스에 관해 onCreate()를 호출합니다.

onDestroy() 콜백은 이전의 콜백에서 아직 해제되지 않은 모든 리소스(예: onStop())를 해제해야 합니다.

# 서비스(Service)

예전에 정리해놓은 자료 참고 [https://koownij.github.io/android/Service-Basic/](https://koownij.github.io/android/Service-Basic/)

## 정의

> Service는 백그라운드에서 오래 실행되는 작업을 수행할 수 있는 컴포넌트  
> 한번 시작하면, 사용자가 다른 application으로 전환해도 service는 계속 실행될 수 있음

***주의 : Service는 Hosting Process의 Main Thread에서 실행됨. ANR을 피하기 위해 blocking operations를 별개의 thread에서 작동시켜야함***

## 포그라운드

예를 들면 오디오 트랙을 재생하는 사용자에게 잘 보이는 작업을 수행합니다. 포그라운드 서비스는 사용자들에게 서비스가 실행 중임을 알리기 위해 반드시 Notification을 표시해야합니다.

## 백그라운드

백그라운드 서비스는 저장 공간 최적화와 같은 유저에게 직접적으로 알려지지 않는 작업을 수행합니다.

*만약 API level 26 이상을 타겟팅한다면, 시스템은 앱이 포그라운드에 있지 않을 때 백그라운드 서비스 제한을 강요합니다. 예를 들어, 대부분의 상황에서 백그라운드에서 위치 정보에 접근할 수 없습니다. 대신 WorkManager를 사용하여 작업을 스케쥴링 해야합니다.*

> UI가 없기 때문에 사용자는 앱에서 실행 중인 서비스 유형과 사용 중인 리소스를 인식하지 못합니다. 이는 보안과 성능 모두에 영향을 미치기 때문.

### Android 12 (API LEVEL 31) 변경점

Android 12를 타겟팅하는 앱은 몇 가지 특수한 사례를 제외하고 백그라운드에서 실행되는 동안 더 이상 포그라운드 서비스를 시작할 수 없습니다.

[https://developer.android.com/about/versions/12/foreground-services?hl=ko#cases-fgs-background-starts-allowed](https://developer.android.com/about/versions/12/foreground-services?hl=ko#cases-fgs-background-starts-allowed)

# 방송 수신자 (BroadCast Receiver)

안드로이드 어플리케이션은 [Publish-Subscribe 패턴](https://velog.io/@minsuk/Publish-Subscribe-%ED%8C%A8%ED%84%B4%EC%95%8C%EB%A6%BC)과 유사하게 안드로이드 시스템 또는 다른 안드로이드 어플리케이션으로부터 브로드캐스트 메시지를 받습니다.

## 정의

> 안드로이드 시스템 또는 다른 안드로이드 어플리케이션이 보낸 브로드캐스트 메세지를 받아주는 안드로이드 컴포넌트

안드로이드 시스템이 보내는 브로드캐스트 종류

* 시스템 부팅
    
* 메세지 수신
    
* 기기 충전 시작
    
* 비행기 모드 전환
    

## 브로드캐스트 수신 방법

manifest에 선언된 수신자 및 컨텍스트에 등록된 수신자를 통해 브로드캐스트를 수신할 수 있습니다.

### manifest에 선언된 수신자

manifest에 broadcast receiver를 선언하면 브로드캐스트가 전송될 때 앱이 아직 실행 중이 아니라면 시스템에서 앱을 실행합니다.

> 참고: 앱이 API 레벨 26 이상을 타겟팅하면 manifest를 사용하여 암시적 브로드캐스트(앱을 구체적으로 타겟팅하지 않는 브로드캐스트)의 수신자를 선언할 수 없습니다. 단, 제한이 면제되는 몇 가지 암시적 브로드캐스트는 예외입니다. 대부분의 상황에서 예약된 작업 을 대신 사용할 수 있습니다.

### context에 등록된 수신자

컨텍스트에 등록된 수신자는 등록 컨텍스트가 유효한 동안 브로드캐스트를 수신합니다. 예를 들어 Activity 컨텍스트 내에 등록하면 Activity가 제거되지 않는 한 브로드캐스트를 수신합니다. Application 컨텍스트에 등록하면 앱이 실행되는 동안 브로드캐스트를 수신합니다.

수신자의 등록 및 등록 취소 위치에 유의해야 합니다. 예를 들어 Activity의 컨텍스트를 사용하여 onCreate(Bundle)에 수신자를 등록했으면 Activity 컨텍스트에서 수신자가 유출되지 않도록 onDestroy()에서 수신자를 등록 취소해야 합니다. onResume()에 수신자를 등록했으면 onPause()에서 수신자를 등록 취소하여 수신자가 여러 번 등록되지 않도록 해야 합니다(일시중지되었을 때 브로드캐스트를 수신하지 않으려면). 이렇게 하면 불필요한 시스템 오버헤드를 줄일 수 있습니다. 그리고 onSaveInstanceState(Bundle)에서 등록을 취소해서는 안 됩니다. 사용자가 기록 스택으로 되돌아가면 이 메서드가 호출되지 않기 때문입니다.

### 프로세스 상태에 미치는 영향

BroadcastReceiver의 상태(실행 중인지 아닌지 여부)는 포함된 프로세스의 상태에 영향을 주며, 결과적으로 시스템에 의해 종료될 가능성에 영향을 줄 수 있습니다. 예를 들어 프로세스가 수신자를 실행할 때(즉, 현재 onReceive() 메서드의 코드를 실행 중일 때) 이는 포그라운드 프로세스로 간주됩니다. 메모리 부족이 심한 상황을 제외하고 시스템은 프로세스를 계속 실행합니다.

하지만 코드가 onReceive()에서 반환되면 BroadcastReceiver는 더 이상 활성 상태가 아닙니다. 수신자의 호스트 프로세스는 프로세스 내에서 실행 중인 다른 앱 구성요소만큼만 중요합니다. 이 프로세스가 manifest에 선언된 수신자만 호스팅한다면(사용자가 최근에 상호작용한 적이 없거나 상호작용하지 않은 앱의 일반적인 사례) onReceive()에서 반환 시 시스템은 이 프로세스를 우선순위가 낮은 프로세스로 간주하여 더 중요한 다른 프로세스에서 리소스를 사용할 수 있도록 이 프로세스를 종료할 수 있습니다.

이러한 이유로 broadcast receiver에서 장기 실행 백그라운드 스레드를 시작해서는 안 됩니다. onReceive() 이후 시스템은 언제든지 프로세스를 종료하여 메모리를 회수할 수 있으며 이 과정에서 프로세스에서 생성되어 실행 중인 스레드를 종료할 수 있습니다. 이 문제를 방지하려면 goAsync()를 호출하거나(백그라운드 스레드의 브로드캐스트를 처리하는 데 시간이 좀 더 필요한 상황에서) JobScheduler를 사용하여 수신자의 JobService를 예약해야 합니다. 그러면 시스템은 프로세스가 계속 활성 작업을 실행하고 있음을 알게 됩니다. 자세한 내용은 [프로세스 및 애플리케이션 수명 주기](https://developer.android.com/guide/topics/processes/process-lifecycle?hl=ko)를 참조하세요.

# 콘텐트 제공자 (Content Provider)

[참고 1](https://layers7.tistory.com/50)  
[자료 1](https://velog.io/@alsgk721/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-14-ContentProvider)

## 정의

> ContentProvider란 안드로이드 응용 프로그램을 구성하는 컴포넌트 중 하나로서 데이터를 제공하는 역할을 하며 응용 프로그램끼리 데이터를 공유하는 유일한 방법이다.  
> \-&gt; A라는 앱과 B라는 앱 사이에 데이터를 공유