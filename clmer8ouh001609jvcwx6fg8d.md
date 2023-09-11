---
title: "[Android] Fragment, Context, Intent"
datePublished: Mon Jan 16 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmer8ouh001609jvcwx6fg8d
slug: android-fragment-context-intent
tags: android

---

# 프래그먼트(Fragment)

## 정의

> * 앱 UI의 재사용 가능한 부분을 나타낸다.
>     
> * 자체 layout을 정의하고 관리한다.
>     
> * 자체 lifecycle을 가지고 있으며 input event 역시 직접 처리한다.
>     

**Fragment는 혼자서는 존재할 수는 없으며 Activity 또는 다른 Fragment에 의해 *hosted*되어야한다.**

Fragment의 view 계층은 host view 계층의 부분이 되거나 host view 계층에 attach 된다.

## 모듈성

프래그먼트는 UI를 분할하여 액티비티의 UI에 모듈성과 재사용성을 도입한다.

## 장점

![](https://developer.android.com/static/images/guide/fragments/fragment-screen-sizes.png align="left")

* Fragment를 사용하여 반응형 웹사이트처럼 구현할 수 있다.
    
* 액티비티의 UI를 더 쉽게 수정할 수 있게된다. 액티비티가 `STARTED` 수명 주기 상태 이상에 있는 동안 프래그먼트를 추가/교체/삭제할 수 있다. 이러한 기록을 액티비티 백 스택에 보관할 수 있으며, 변경사항을 취소할 수도 있다.
    
* 동일한 프래그먼트 클래스의 여러 인스턴스를 사용할 수 있다. ***이 점에 유의하여 자체 UI 관리에 필요한 로직만 프래그먼트에 제공해야 한다. 한 프래그먼트에 의존하거나 다른 프래그먼트에서 프래그먼트를 조작하지 않아야 한다.***
    

## Fragment Lifecycle(프래그먼트 생명 주기)

[참고 자료](https://readystory.tistory.com/199)

[API 28 이상부터 onSavedInstanceState() 함수와 onStop()함수의 호출 순서가 달라진 이유](https://stackoverflow.com/questions/73363275/what-are-the-reasons-behind-the-change-in-the-order-of-onsaveinstancestate-and-o)

## 주의 사항

[프래그먼트를 인자와 함께 생성할 때 newInstance()를 사용하는 이유](https://black-jin0427.tistory.com/250)

# Context

[참고 자료 1](https://hee96-story.tistory.com/68)  
[참고 자료 2](https://www.charlezz.com/?p=1080)  
[참고 자료 3](https://amitshekhar.me/blog/context-in-android-application)  
[참고 자료 4](https://roomedia.tistory.com/entry/Android-Context%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)  
[참고 자료 5](https://velog.io/@sery270/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-Context)

# Intent

## 정의

> Intent는 메시징 객체로, 다른 앱 구성요소(액티비티, 서비스, 브로드캐스트 리시버, 콘텐츠 제공자)로부터 작업을 요청하는 데 사용할 수 있습니다. -&gt; Component를 실행하기 위해 시스템에 넘기는 정보

## 주요 사용 사례

### 액티비티 시작

액티비티의 새 객체를 시작하려면 Intent를 startActivity()로 전달하면 됩니다. Intent는 시작할 액티비티를 설명하고 모든 필수 데이터를 담습니다.

### 서비스 시작

Android 5.0 (API 21) 이전 버전에서 Intent를 startService()에 전달하여 서비스를 시작할 수 있습니다. Intent는 시작할 서비스를 설명하고 모든 필수 데이터를 담고 있습니다.

Android 5.0 (API 21) 이상부터는 JobScheduler로도 서비스를 시작할 수 있습니다.

서비스가 클라이언트-서버 인터페이스로 디자인된 경우, 다른 구성 요소로부터 서비스에 바인딩하려면 Intent를 bindService()에 전달하면 됩니다.

### 브로드캐스트 전달

Intent를 sendBroadcase() 또는 sendOrderedBrocast()에 전달하여 다른 앱에 브로드캐스트를 전달할 수 있습니다.

## 인텐트 타입

### 명시적 인텐트

> 인텐트에 클래스 객체나 컴포넌트 이름을 지정하여 호출될 대상을 정확히 알 수 있는 인텐트

앱 내의 특정 액티비티나 서비스 등 특정한 앱 구성 요소를 시작하는데 사용하는 인텐트.

### 암시적 인텐트

> 호출될 대상의 속성들을 지정했지만 호출될 대상이 달리질 수 있는 인텐트

작업을 지정하여 기기에서 해당 작업을 수행할 수 있는 모든 앱을 호출할 수 있도록 합니다.

ex) 메세지 공유하기 버튼을 누르면 메세지를 공유할 수 있는 모든 앱(카톡, 인스타 등)을 호출합니다. 이때 사용자는 어느 앱을 사용할 지 선택해야합니다.

## PendingIntent (보류 인텐트)

[참고 자료 1](https://velog.io/@haero_kim/Android-PendingIntent-%EA%B0%9C%EB%85%90-%EC%9D%B5%ED%9E%88%EA%B8%B0)  
[참고 자료 2](https://bb-library.tistory.com/267)

앱이 Android 12를 타겟팅하는 경우 앱에서 만드는 각 PendingIntent 객체의 변경 가능 여부()를 지정해야 합니다. 이 추가 요구사항은 앱의 보안을 강화합니다.

FLAG\_IMMUTABLE or FLAG\_MUTABLE 적용

```kotlin
PendingIntent mPendingIntent = PendingIntent.getBroadcast(this, 0, alarmIntent, PendingIntent.FLAG_CANCEL_CURRENT | PendingIntent.FLAG_IMMUTABLE);
```