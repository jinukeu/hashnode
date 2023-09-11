---
title: "[Android] Intents and Intent Filters"
datePublished: Mon Sep 26 2022 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmeoyc4j000909jl1k1094hl
slug: android-intents-and-intent-filters
tags: android

---

> 안드로이드 공식 문서를 번역하고 내용을 조금 변경하거나 내용을 추가한 게시글입니다. 잘못된 내용이 있을 수 있습니다. [참고한 공식 문서 바로가기](https://developer.android.com/guide/components/intents-filters)

  

Intent는 메시징 객체로, 다른 앱 구성 요소로부터 작업을 요청하는 데 사용할 수 있습니다. 인텐트가 구성 요소 사이의 통신을 촉진하는 데는 여러 가지 방식이 있지만 기본적인 사용 사례는 크게 세 가지로 나눌 수 있습니다.

* 액티비티 시작  
    Activity는 앱 안의 단일 화면을 나타냅니다. Activity의 새 인스턴스를 시작하려면 Intent를 startActivity()로 전달하면 됩니다. Intent는 시작할 액티비티를 설명하고 모든 필수 데이터를 담습니다.  
    액티비티가 완료되었을 때 결과를 수신하려면, startActivityForResult()를 호출합니다. 액티비티는 해당 결과를 이 액티비티의 onActivityResult() 콜백에서 별도의 Intent 객체로 수신합니다. 자세한 내용은 액티비티 가이드를 참조하세요.
    
* 서비스 시작  
    Service는 사용자 인터페이스 없이 백그라운드에서 작업을 수행하는 구성 요소입니다. Android 5.0(API 레벨 21) 이상부터는 JobScheduler로 서비스를 시작할 수 있습니다. JobScheduler에 대한 자세한 내용은 [API-reference documentation](https://developer.android.com/reference/android/app/job/JobScheduler?hl=ko)을 참조하세요.  
    Android 5.0(API 레벨 21) 이하 버전은 Service 클래스의 메서드를 사용하면 서비스를 시작할 수 있습니다. 서비스를 시작하여 일회성 작업을 수행하도록 하려면(예: 파일 다운로드) Intent를 startService()에 전달하면 됩니다. Intent는 시작할 서비스를 설명하고 모든 필수 데이터를 담고 있습니다.  
    서비스가 클라이언트-서버 인터페이스로 디자인된 경우, 다른 구성 요소로부터 서비스에 바인딩하려면 Intent를 bindService()에 전달하면 됩니다. 자세한 내용은 (서비스)\[[https://developer.android.com/guide/components/services?hl=ko](https://developer.android.com/guide/components/services?hl=ko)\] 가이드를 참조하세요.
    
* 브로드캐스트 전달  
    브로드캐스트는 모든 앱이 수신할 수 있는 메시지입니다. 시스템은 시스템이 부팅될 때 또는 기기가 충전을 시작할 때 등 시스템 이벤트에 대한 다양한 브로드캐스트를 전달합니다. Intent를 sendBroadcast() 또는 sendOrderedBroadcast()에 전달하면 다른 앱에 브로드캐스트를 전달할 수 있습니다.
    

이 페이지의 나머지에서는 인텐트의 작동 원리와 사용 방법을 설명합니다. 관련 정보는 (다른 앱과의 상호작용)\[[https://developer.android.com/training/basics/intents?hl=ko](https://developer.android.com/training/basics/intents?hl=ko)\] 및 (콘텐츠 공유)\[[https://developer.android.com/training/sharing?hl=ko\]를](https://developer.android.com/training/sharing?hl=ko]를) 참조하세요.

  

## 인텐트 유형

인텐트에는 두 가지 유형이 있습니다.

* **명시적 인텐트(Explicit intents)**는 인텐트를 충족하는 애플리케이션이 무엇인지 지정합니다. 이를 위해 대상 앱의 패키지 이름 또는 완전히 자격을 갖춘 구성 요소 클래스 이름을 제공합니다. 명시적 인텐트는 일반적으로 앱 안에서 구성 요소를 시작할 때 씁니다. 시작하고자 하는 액티비티 또는 서비스의 클래스 이름을 알고 있기 때문입니다. 예를 들어, 사용자 작업에 응답하여 새로운 액티비티를 시작하거나 백그라운드에서 파일을 다운로드하기 위해 서비스를 시작하는 것 등이 여기에 해당됩니다.
    
* **암시적 인텐트(Implicit intents)**는 특정 구성 요소의 이름을 대지 않지만, 그 대신 수행할 일반적인 작업을 선언하여 다른 앱의 구성 요소가 이를 처리할 수 있도록 해줍니다. 예를 들어 사용자에게 지도에 있는 한 위치를 표시하고자 하는 경우, 암시적 인텐트를 사용하여 해당 기능을 갖춘 다른 앱이 지정된 위치를 지도에 표시하도록 요청할 수 있습니다.
    

그림 1은 액티비티를 시작할 때 인텐트를 사용하는 법을 나타낸 것입니다. Intent 객체가 특정 액티비티 구성 요소를 명시적으로 지정하면 시스템이 해당 구성 요소를 즉시 시작합니다.

![그림 1](https://developer.android.com/static/images/components/intent-filters_2x.png?hl=ko align="left")

그림 1. 암시적(Implicit) 인텐트가 시스템을 통해 전달되어 다른 액티비티를 시작하는 방법. \[1\] 액티비티 A가 작업 설명이 있는 Intent를 생성하여 이를 startActivity()에 전달합니다. \[2\] Android 시스템이 모든 앱에서 해당 인텐트와 일치하는 인텐트 필터를 검색합니다. 일치하는 항목을 찾으면, \[3\] 시스템이 해당 액티비티의 onCreate() 메서드를 호출하여 일치하는 액티비티(액티비티 B)를 시작하고 Intent를 전달합니다.

암시적 인텐트를 사용하면 Android 시스템에서 시작할 적절한 구성 요소를 찾습니다. 이때 인텐트의 내용을 기기에 있는 다른 여러 앱의 매니페스트 파일에서 선언된 인텐트 필터와 비교하는 방법을 사용합니다. 해당 인텐트와 일치하는 인텐트 필터가 있으면 시스템에서 해당 구성 요소를 시작하고 Intent 객체를 전달합니다. 호환되는 인텐트 필터가 여러 개인 경우, 시스템에서 대화상자를 표시하여 사용자가 어느 앱을 사용할지 직접 선택할 수 있게 합니다.

인텐트 필터란 앱의 매니페스트 파일에 들어 있는 표현으로, 해당 구성 요소가 수신하고자 하는 인텐트의 유형을 나타냅니다. 예를 들어 액티비티에 대한 인텐트 필터를 선언하면 다른 여러 앱이 특정한 종류의 인텐트를 가지고 액티비티를 직접 시작할 수 있습니다. 마찬가지로 액티비티에 대한 인텐트 필터를 선언하지 않는 경우, 이것은 명시적인 인텐트로만 시작할 수 있습니다.

> 주의 : 앱의 보안을 보장하려면 Service를 시작할 때에는 항상 명시적 인텐트만 사용하고 서비스에 대한 인텐트 필터는 선언하지 마세요. 암시적 인텐트를 사용하여 서비스를 시작하면 보안 위험을 초래합니다. 인텐트에 어느 서비스가 응답할지 확신할 수 없고, 사용자는 어느 서비스가 시작되는지 볼 수 없기 때문입니다. Android 5.0(API 레벨 21)부터 시스템은 개발자가 암시적 인텐트로 bindService()를 호출하면 예외를 발생시킵니다.

  

## 인텐트 빌드

Intent 객체에는 Android 시스템이 어느 구성 요소를 시작할지 판별하는 데 사용하는 정보가 담겨 있습니다(예를 들어 정확한 구성 요소 이름 또는 인텐트를 수신해야 하는 구성 요소 카테고리 등). 또한 수신자 구성 요소가 작업을 적절히 수행하기 위해 사용할 정보(예: 수행할 작업 및 조치를 취할 데이터 위치 등)도 이 안에 담겨 있습니다.

Intent에 포함된 기본 사항은 다음과 같습니다.

### 구성 요소 이름

시작할 구성 요소의 이름입니다. 이것은 선택 항목이지만, 인텐트를 명시적인 것으로 만들어주는 중요한 정보입니다. 다시 말해 이 인텐트는 구성 요소 이름이 정의한 앱 구성 요소에만 전달되어야 한다는 뜻입니다. 구성 요소 이름이 없으면 해당 인텐트는 암시적이며, 인텐트를 수신해야 하는 구성 요소는 다른 인텐트 정보를 기반으로 시스템이 결정합니다(예를 들어 작업, 데이터 및 카테고리. 아래 설명 참조). 앱에서 특정한 구성 요소를 시작해야 하는 경우에는 구성 요소 이름을 지정해야 합니다.

> 참고 : Service를 시작하는 경우, 항상 구성 요소 이름을 지정해야 합니다. 그렇지 않으면 인텐트에 어느 서비스가 응답할지 확신할 수 없고, 사용자도 어느 서비스가 시작되는지 볼 수 없게 됩니다.

Intent의 필드는 ComponentName 객체로, 이것을 지정하려면 대상 구성 요소의 완전히 정규화된 클래스 이름(앱의 패키지 이름 포함)을 사용해야 합니다. 예를 들어 com.example.ExampleActivity를 쓰세요. 구성 요소 이름을 설정하려면 setComponent(), setClass(), setClassName()을 사용하거나 또는 Intent 생성자를 사용합니다.

### 작업

수행할 일반적인 작업을 나타내는 문자열입니다(예: 보기 또는 선택).

브로드캐스트 인텐트의 경우, 이것은 이미 실행되어 보고되고 있는 작업입니다. 이 작업은 대체로 나머지 인텐트의 구조를 결정합니다. 특히, 데이터와 엑스트라에 포함되는 정보가 결정됩니다.

본인의 앱 내에 있는 인텐트가 사용할 작업(또는 다른 앱이 사용하여 본인의 앱 안의 구성 요소를 호출하게 할 작업)을 직접 지정할 수도 있지만, 보통은 Intent 클래스나 다른 프레임워크 클래스가 정의한 작업 상수를 지정해야 합니다. 다음은 액티비티를 시작하는 데 쓰이는 보편적인 작업입니다.

#### ACTION\_VIEW

이 작업은 액티비티가 사용자에게 표시할 수 있는 어떤 정보를 가지고 있을 때 startActivity()가 있는 인텐트에서 사용합니다. 예를 들어 갤러리 앱에서 볼 사진이나 지도 앱에서 볼 주소 등이 이에 해당됩니다.

#### ACTION\_SEND

공유 인텐트라고도 하며, 사용자가 다른 앱을 통해 공유할 수 있는 데이터를 가지고 있을 때 startActivity()가 있는 인텐트에서 사용해야 합니다. 예를 들어 이메일 앱 또는 소셜 공유 앱 등이 이에 해당됩니다. 일반적인 작업을 정의하는 상수에 대한 자세한 내용은 [Intent](https://developer.android.com/reference/android/content/Intent?hl=ko) 클래스 참조 문서를 확인하세요. 다른 작업은 Android 프레임워크의 다른 곳에서 정의됩니다. 예를 들어 시스템의 설정 앱에서 특정 화면을 여는 작업의 경우 Settings에서 정의됩니다.

인텐트에 대한 작업을 지정하려면 setAction() 또는 Intent 생성자를 사용하면 됩니다.

나름의 작업을 직접 정의하는 경우, 앱의 패키지 이름을 접두어로 포함시켜야 합니다. 이는 다음 예시에서 확인할 수 있습니다.

```kotlin
const val ACTION_TIMETRAVEL = "com.example.action.TIMETRAVEL"
```

### 데이터

작업을 수행할 데이터 및/또는 해당 데이터의 MIME 유형을 참조하는 URI(Uri 객체)입니다. 제공된 데이터의 유형을 나타내는 것은 일반적으로 인텐트의 작업입니다. 예를 들어 인텐트가 ACTION\_EDIT인 경우, 데이터에 편집할 문서의 URI가 들어 있어야 합니다. 인텐트를 생성할 때는 URI 외에 데이터 유형(데이터의 MIME 유형)을 지정하는 것이 중요한 경우가 많습니다. 예를 들어 이미지를 표시할 수 있는 액티비티라면 오디오 파일은 재생하지 못할 것입니다(URI 형식은 비슷할지라도). 데이터의 MIME 유형을 지정하면 Android 시스템이 인텐트를 수신하기 가장 좋은 구성 요소를 찾는 데 도움이 됩니다. 다만 때로는 MIME 유형을 URI를 통해 추론할 수 있습니다. 특히 데이터가 content: URI일 때 추론하기 쉽습니다. content: URI는 데이터가 기기에 위치하고 ContentProvider가 제어한다는 것을 의미하며, 따라서 데이터 MIME 유형이 시스템에 표시됩니다.

데이터 URI만 설정하려면 setData()를 호출하세요. MIME 유형만 설정하려면 setType()을 호출하세요. 필요한 경우, setDataAndType()을 사용하여 두 가지를 모두 명시적으로 설정할 수 있습니다.

> 주의: URI와 MIME 유형을 둘 다 설정하고자 하는 경우, setData() 및 setType()을 호출하면 안 됩니다. 이 둘은 서로의 값을 무효화하기 때문입니다. URI와 MIME 유형을 둘 모두 설정하려면 항상 setDataAndType()을 사용하세요.

### 카테고리

인텐트를 처리해야 하는 구성 요소의 종류에 관한 추가 정보를 담은 문자열입니다. 인텐트 안에는 카테고리 설명이 얼마든지 들어 있을 수 있지만, 대부분의 인텐트에는 카테고리가 없어도 됩니다. 다음은 몇 가지 보편적인 카테고리입니다.

#### CATEGORY\_BROWSABLE

대상 액티비티가 웹브라우저를 통해 시작되도록 허용하고 이미지, 이메일 메시지 등의 링크로 참조된 데이터를 표시하게 합니다.

#### CATEGORY\_LAUNCHER

이 액티비티가 작업의 최초 액티비티이며, 시스템의 애플리케이션 시작 관리자에 목록으로 게재됩니다. 카테고리의 전체 목록을 보려면 Intent 클래스 설명을 참조하세요.

카테고리는 addCategory()로 지정할 수 있습니다.

위에 나열된 이러한 속성(구성 요소 이름, 작업, 데이터 및 카테고리)은 인텐트를 정의하는 특성을 나타냅니다. Android 시스템은 이와 같은 속성을 읽어 어느 앱 구성 요소를 시작해야 할지 확인할 수 있습니다. 그러나 인텐트는 앱 구성 요소로 확인되는 방법에 영향을 미치지 않는 추가 정보도 담고 있을 수 있습니다. 인텐트가 제공할 수 있는 기타 정보는 다음과 같습니다.

### 엑스트라

요청된 작업을 수행하는 데 필요한 추가 정보가 담긴 키-값 쌍입니다. 몇몇 작업이 특정한 종류의 데이터 URI를 사용하는 것과 마찬가지로, 몇몇 작업은 특정한 엑스트라도 사용합니다. 다양한 putExtra() 메서드로 엑스트라 데이터를 추가할 수 있습니다. 각 메서드는 키 이름과 값, 이렇게 두 개의 매개변수를 취합니다. 또한 모든 엑스트라 데이터를 포함한 Bundle 객체를 만든 다음 Bundle을 Intent에 putExtras()로 삽입할 수도 있습니다.

예를 들어 ACTION\_SEND로 이메일을 전송할 인텐트를 생성하는 경우 받는 사람 수신자를 지정할 때 EXTRA\_EMAIL 키를 사용한 다음, 제목은 EXTRA\_SUBJECT 키로 지정하면 됩니다.

Intent 클래스는 표준화된 데이터 유형에 대해 여러 가지 EXTRA\_\* 상수를 지정합니다. 나름의 엑스트라 키를 선언해야 하는 경우(본인의 앱이 수신할 인텐트에 대하여), 앱의 패키지 이름을 접두어로 포함해야 합니다.

```kotlin
const val EXTRA_GIGAWATTS = "com.example.EXTRA_GIGAWATTS"
```

> 주의 : 다른 앱이 수신할 것으로 생각하는 인텐트를 전송할 때는 Parcelable 또는 Serializable 데이터를 사용하지 마세요. 앱이 Bundle 객체에 있는 데이터에 액세스하려고 시도했지만 파슬링되거나 직렬화된 클래스에 액세스하지 못하면 시스템이 RuntimeException을 발생시킵니다.

### 플래그

플래그는 Intent 클래스에서 정의되고, 인텐트에 대한 메타데이터와 같은 기능을 합니다. 이런 플래그는 Android 시스템에 액티비티를 시작할 방법에 대한 지침을 줄 수도 있고(예를 들어 액티비티가 어느 작업에 소속되어야 하는지 등) 액티비티를 시작한 다음에 어떻게 처리해야 하는지도 알려줄 수 있습니다(예를 들어 해당 액티비티가 최근 액티비티 목록에 소속되는지 여부). 자세한 내용은 [setFlags()](https://developer.android.com/reference/android/content/Intent#setFlags(int)) 메서드를 참조하세요.

## 명시적 인텐트 예시

명시적 인텐트는 앱 내의 특정 액티비티나 서비스 등 특정한 앱 구성 요소를 시작하는 데 사용하는 인텐트입니다. 명시적 인텐트를 생성하려면 Intent 객체에 대한 구성 요소 이름을 정의합니다. 다른 인텐트 속성은 모두 선택 사항입니다.

예를 들어 앱 안에 DownloadService라는 서비스를 구축했다고 가정해보겠습니다. 이 서비스는 웹상에서 파일을 다운로드하도록 설계되었습니다. 이 서비스를 시작하려면 다음과 같은 코드를 사용합니다.

```kotlin
// Executed in an Activity, so 'this' is the Context
// The fileUrl is a string URL, such as "http://www.example.com/image.png"
val downloadIntent = Intent(this, DownloadService::class.java).apply {
    data = Uri.parse(fileUrl)
}
startService(downloadIntent)
```

Intent(Context, Class) 생성자가 앱에 Context를 제공하고 구성 요소에 Class 객체를 제공합니다. 이처럼 이 인텐트는 앱 내의 DownloadService 클래스를 명시적으로 시작합니다.

서비스를 빌드하고 시작하는 방법에 대한 자세한 내용은 (서비스)\[[https://developer.android.com/guide/components/services](https://developer.android.com/guide/components/services)\] 가이드를 참조하세요.

## 암시적 인텐트 예시

암시적 인텐트는 작업을 지정하여 기기에서 해당 작업을 수행할 수 있는 모든 앱을 호출할 수 있도록 합니다. 본인의 앱은 작업을 수행할 수 없지만 다른 앱은 그 작업을 수행할 수 있는 가능성이 있고, 사용자가 어떤 앱을 사용할지 선택하기를 원할 경우에 암시적 인텐트가 유용합니다.

예를 들어 사용자가 다른 사람들과 공유했으면 하는 콘텐츠를 가지고 있는 경우, ACTION\_SEND 작업이 있는 인텐트를 생성한 다음 공유할 콘텐츠를 지정하는 엑스트라를 추가하면 됩니다. 해당 인텐트로 startActivity()를 호출하면 사용자가 어느 앱을 통해 콘텐츠를 공유할지 선택할 수 있습니다.

```kotlin
// Create the text message with a string.
val sendIntent = Intent().apply {
    action = Intent.ACTION_SEND
    putExtra(Intent.EXTRA_TEXT, textMessage)
    type = "text/plain"
}

// Try to invoke the intent.
try {
    startActivity(sendIntent)
} catch (e: ActivityNotFoundException) {
    // Define what your app should do if no activity can handle the intent.
}
```

startActivity()를 호출하면 시스템이 설치된 앱을 모두 살펴보고 이런 종류의 인텐트를 처리할 수 있는 앱이 어느 것인지 알아봅니다(ACTION\_SEND 작업이 있는 인텐트이며 "텍스트/일반" 데이터가 담긴 것). 이것을 처리할 수 있는 앱이 하나뿐이면, 해당 앱이 즉시 열리고 이 앱에 인텐트가 주어집니다. 인텐트를 허용하는 액티비티가 여러 개인 경우, 시스템은 그림 2에서와같이 대화상자를 표시하여 사용자에게 앱을 선택하게 합니다.

다른 앱을 시작하는 방법에 대한 자세한 내용은 [사용자를 다른 앱에 보내기](https://developer.android.com/training/basics/intents/sending)를 참조하세요.

![그림 2](https://developer.android.com/static/images/training/basics/intent-chooser.png align="left")

그림 2

### 앱 선택기 강제 적용

암시적 인텐트에 응답하는 앱이 두 개 이상인 경우, 사용자가 어느 앱을 사용할지 선택하고 이 작업에 대한 기본 앱으로 지정할 수 있습니다. 이는 사용자가 이제부터 계속 같은 앱을 사용할 것으로 추정되는 작업을 수행할 때 좋습니다. 예를 들어 웹 페이지를 열 때가 이에 해당합니다(대다수 사용자가 웹브라우저는 하나만 사용하는 것을 선호합니다).

그러나 인텐트에 응답할 수 있는 앱이 여러 개이고 사용자가 매번 다른 앱을 사용하기를 원할 수도 있는 경우라면, 선택기 대화상자를 명시적으로 표시해야 합니다. 선택기 대화상자는 사용자가 작업에 어떤 앱을 사용할지 매번 물어봅니다(사용자는 작업에 사용할 기본 앱을 선택할 수 없습니다). 예를 들어 앱이 ACTION\_SEND 작업으로 "공유"를 수행하는 경우, 사용자는 각자의 현재 상황에 따라 여러 가지 다른 앱을 사용하여 공유하기를 원할 수도 있습니다. 따라서 항상 선택기 대화상자를 사용해야 합니다(그림 2 참조).

선택기를 표시하려면 createChooser()를 사용하여 Intent를 만들고 startActivity()에 전달합니다. 이 예시는 createChooser() 메서드에 전달된 인텐트에 응답하는 앱의 목록을 대화상자에 표시하며, 제공된 텍스트를 대화상자 제목으로 사용합니다.

```kotlin
val sendIntent = Intent(Intent.ACTION_SEND)
...

// Always use string resources for UI text.
// This says something like "Share this photo with"
val title: String = resources.getString(R.string.chooser_title)
// Create intent to show the chooser dialog
val chooser: Intent = Intent.createChooser(sendIntent, title)

// Verify the original intent will resolve to at least one activity
if (sendIntent.resolveActivity(packageManager) != null) {
    startActivity(chooser)
}
```

## 안전하지 않은 인텐트 시작 감지

당신의 앱은 앱 내부 구성 요소를 이동하거나 다른 앱 대신 작업을 수행하기 위해 인텐트를 실행할 수 있습니다. 플랫폼 보안을 향상시키기 위해, 안드로이드 12(API level 31) 이상부터는 만약 당신의 앱이 안전하지 않은 인텐트 실행을 수행한다면 경고를 해주는 디버깅 기능을 제공합니다. 예를 들어 다른 인텐트에서 extra로 전달된 인텐트를 호출하는 안전하지 않은 *nested 인텐트*를 당신의 앱이 실행할 수 있습니다.

만약 앱이 다음 두 개의 작업을 수행한다면, 시스템은 안전하지않은 인텐트 실행을 김자하고 StrictMode 위반을 발생시킵니다.

1. 앱이 전달된 인텐트의 extras로부터의 nested intent를 unparcels했을 때
    
2. 앱이 즉시 nested 인텐트를 사용하여 앱 구성 요소를 시작할 때, 예를 들어 startActivity(), startService() 또는 bindService()로 인텐트를 전달할 때
    

이런 상황을 감지하고 앱을 변화시키는 방법은 (Android Nesting Intents)\[[https://medium.com/androiddevelopers/android-nesting-intents-e472fafc1933\]를](https://medium.com/androiddevelopers/android-nesting-intents-e472fafc1933]를) 읽어보세요.

  

### 안전하지 않은 인텐트 실행 확인

앱에서 안전하지 않은 인텐트를 확인하기 위해, VmPolicy를 설정할 때 detextUnsafeIntentLaunch()를 호출하세요. 다음 코드 스니펫에서 확인할 수 있습니다. 만약 앱이 StrictMode violation을 감지한다면 잠재적으로 민감한 정보를 보호하기 위해 앱 실행을 멈추고 싶을 것이다.

> 노트 : 만약 VmPolicy 정의에서 안드로이드 12를 타겟팅한다면 detectUnsafeIntentLaunch() 메서드는 자동으로 실행됩니다.

```kotlin
fun onCreate() {
    StrictMode.setVmPolicy(VmPolicy.Builder()
        // Other StrictMode checks that you've previously added.
        // ...
        .detectUnsafeIntentLaunch()
        .penaltyLog()
        // Consider also adding penaltyDeath()
        .build())
}
```

### 인텐트를 더 책임감 있게 사용

안전하지 않은 인텐트 실행과 StrictMode 위반을 방지하기 위해 다음 best practices를 따르세요.

**인텐트안에서 필수적인 extras만 복사하고 어느 필요한 sanitation과 validation을 수행하세요.** 앱은 아마 extras를 하나의 인텐트에서 새로운 구성 요소를 시작하기 위한 다른 인텐트로 복사할 것입니다. 이는 앱이 putExtras(Intent) 또는 putExtras(Bundle)를 호출할 때 발생합니다. 만약 이러한 작업을 앱이 수행한다면 수신하는 구성 요소가 예상하는 extras만 복사하세요. 만약 다른 인텐트가 (복사본을 전달 받는) export하지 않은 구성 요소를 실행하는 경우, 구성 요소를 실행하는 인텐트로 복사하기 전에 extras를 검사하고 유효성을 검사하세요.

**앱 컴포넌트를 불필요하게 export 하지마세요.** 예를 들어, internal nested intent를 사용하여 앱 컴포넌트를 시작하고 싶다면, 해당 컴포넌트의 `android:exported` attribute를 false로 설정하세요.

**nested intent 대신 PendingIntent를 사용하세요.** 그렇게하면 다른 앱이 Intent를 포함한 PendingIntent를 뜯어볼 때(unparcels), 다른 앱이 당신 앱의 identity를 사용하여 PendingIntent를 사용할 수 있습니다.

그림 2는 어떻게 시스템이 제어권을 당신의 앱(client)에서 다른 앱(service)으로 넘기고 당신의 앱으로 돌아오는 것을 보여줍니다.

1. 당신의 앱은 다른 앱에서 activity를 실행하는 intent를 만듭니다. intent 안에서, PendingIntent 객체를 extra로 추가합니다. 이 pending intent는 당신의 앱안에서 component를 실행합니다.; 이 component는 exproted되지 않았습니다.
    
2. 당신 앱의 intent를 받을 때까지, 다른 앱은 nested PendingIntent 객체를 추출합니다.
    
3. 다른 앱은 PendingIntent 객체의 send() 메서드를 실행합니다.
    
4. 당신의 앱이 제어권을 다시 갖게되면, 시스템은 당신 앱의 context를 사용하여 pending intent를 실행합니다.
    

![](https://developer.android.com/static/images/guide/components/nested-pending-intent.svg align="left")

## 암시적 인텐트 수신

앱이 수신할 수 있는 암시적 인텐트가 어느 것인지 알리려면, `<intent-filter>` 요소를 사용하여 각 앱 구성 요소에 대해 하나 이상의 인텐트 필터를 매니페스트 파일에 선언합니다. 각 인텐트 필터는 인텐트의 작업, 데이터 및 카테고리를 기반으로 어느 유형의 인텐트를 수락하는지 지정합니다. 시스템은 인텐트가 인텐트 필터 중 하나를 통과한 경우에만 암시적 인텐트를 앱 구성 요소에 전달합니다.

> 참고 : 명시적 인텐트는 항상 자신의 대상에게 전달되며, 이는 구성 요소가 어떤 인텐트 필터를 선언하든 무관합니다.

앱 구성 요소는 자신이 수행할 수 있는 각각의 고유한 작업에 대하여 별도의 필터를 선언해야 합니다. 예를 들어 이미지 갤러리 앱에 있는 어떤 액티비티에 두 개의 필터가 있을 수 있습니다. 한 필터는 이미지를 보고, 다른 필터는 이미지를 편집하기 위한 것입니다. 액티비티가 시작되면, Intent를 검사한 다음 Intent에 있는 정보를 근거로 어떻게 동작할 것인지 결정합니다(편집기 제어 항목을 표시할 것인지 말 것인지 등).

각 인텐트 필터는 앱의 매니페스트 파일에 있는 `<intent-filter>`요소에서 정의하고, 이는 대응되는 앱 구성 요소에서 중첩됩니다(예: `<activity>` 요소).

`<intent-filter>`요소를 포함한 각각의 앱 component 안에서, `android:exported` 값을 명백하게 설정해야합니다. 이 attribute는 앱 컴포넌트가 다른 앱에 접근할 수 있는지를 나타냅니다. 몇몇 상황에서, `LAUNCHER` 카테고리를 포함된 intent filters를 갖고 있는 activities에서 이 attribute를 true로 하는 것은 유용하다. 반면에 이 attribute를 false로 하는게 더 안전하긴하다.

> 주의 : 당신의 앱 내의 activity, service 또는 broadcast receiver가 intent filters를 사용하지만 명백하게 android:export 값을 설정하지 않은 경우, Android 12 이상의 디바이스에서 당신의 앱은 설치되지 않는다.

`<intent-filter>` 내부에서는 다음과 같은 세 가지 요소 중 하나 이상을 사용하여 허용할 인텐트 유형을 지정할 수 있습니다.

`<action>` name 특성에서 허용된 인텐트 작업을 선언합니다. 이 값은 어떤 작업의 리터럴 문자열 값이어야 하며, 클래스 상수가 아닙니다. `<data>` 허용된 데이터 유형을 선언합니다. 이때 데이터 URI(scheme, host, port, path)와 MIME 유형의 여러 가지 측면을 나타내는 하나 이상의 특성을 사용합니다. `<category>` name 특성에서 허용된 인텐트 카테고리를 선언합니다. 이 값은 어떤 작업의 리터럴 문자열 값이어야 하며, 클래스 상수가 아닙니다.

> 참고 : 암시적 인텐트를 수신하려면 CATEGORY\_DEFAULT 카테고리를 인텐트 필터에 포함해야 합니다. startActivity() 및 startActivityForResult() 메서드는 마치 CATEGORY\_DEFAULT 범주를 선언한 것처럼 모든 인텐트를 취급합니다. 이 카테고리를 인텐트 필터에서 선언하지 않으면 액티비티에 어떤 암시적 인텐트도 확인되지 않습니다.

예를 들어 데이터 유형이 텍스트인 경우 ACTION\_SEND 인텐트를 수신할 인텐트 필터가 있는 액티비티 선언은 다음과 같습니다.

```kotlin
<activity android:name="ShareActivity" android:exported="false">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```

`<action>`, `<data>` 또는 `<category`\&gt;의 인스턴스를 두 개 이상 포함하는 필터를 생성할 수 있습니다. 이렇게 하는 경우, 구성 요소가 그러한 필터 요소의 모든 조합을 처리할 수 있는지 확인해야 합니다.

여러 가지 종류의 인텐트를 처리하고자 하되 특정 조합의 작업, 데이터, 카테고리 유형으로만 한정하고자 할 때는 여러 가지 인텐트 필터를 생성해야 합니다.

암시적 인텐트는 인텐트를 세 가지 요소와 각각 비교하여 필터를 테스트합니다. 인텐트가 구성 요소에 전달되려면 세 가지 테스트를 모두 통과해야 합니다. 그중 하나라도 통과하지 못하면 Android 시스템에서 해당 인텐트를 구성 요소에 전달하지 않습니다. 그러나 구성 요소에는 여러 개의 인텐트 필터가 있을 수도 있으므로, 구성 요소의 필터 중 하나를 통과하지 않는 인텐트도 다른 필터를 통하면 성공할 수 있습니다. 시스템이 인텐트를 확인하는 방법에 대한 자세한 내용은 인텐트 확인에 대해 다룬 아래의 섹션에 제공되어 있습니다.

> 주의 : 인텐트 필터를 사용하면 다른 앱이 여러분의 구성 요소를 시작할 위험이 있습니다. 인텐트 필터는 구성 요소가 특정한 종류의 암시적 인텐트에만 응답하도록 제한하기는 하지만, 다른 개발자가 여러분의 구성 요소 이름을 알아내서 명시적 인텐트를 사용할 경우 다른 앱이 여러분의 구성 요소를 시작할 수도 있습니다. 자신의 앱만 구성 요소를 시작하는 것이 중요할 경우 매니페스트에 인텐트 필터를 선언하지 마세요. 그 대신 해당 구성 요소에 대해 exported 특성을 "false"로 설정하세요.

마찬가지로 의도치 않게 다른 앱의 Service를 실행하는 일을 피하려면, 항상 명시적 인텐트를 사용하여 본인의 서비스를 시작하세요.

> 참고 : 모든 액티비티는 매니페스트 파일에서 인텐트 필터를 선언해야 합니다. 그러나 Broadcast Receiver의 필터는 registerReceiver()를 호출하여 동적으로 등록할 수 있습니다. 그런 다음 unregisterReceiver()를 사용하여 해당 수신기를 등록 해제할 수 있습니다. 이렇게 하면 앱이 실행되는 동안에 정해진 기간 동안만 특정한 브로드캐스트에 대해 수신 대기할 수 있습니다.

### 필터 예시

몇 가지 인텐트 필터 동작 보여드리기 위해 소셜 공유 앱의 매니페스트 파일 예시를 준비했습니다.

```kotlin
<activity android:name="MainActivity" android:exported="true">
    <!-- This activity is the main entry, should appear in app launcher -->
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>

<activity android:name="ShareActivity" android:exported="false">
    <!-- This activity handles "SEND" actions with text data -->
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
    <!-- This activity also handles "SEND" and "SEND_MULTIPLE" with media data -->
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <action android:name="android.intent.action.SEND_MULTIPLE"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="application/vnd.google.panorama360+jpg"/>
        <data android:mimeType="image/*"/>
        <data android:mimeType="video/*"/>
    </intent-filter>
</activity>
```

첫 번째 액티비티인 MainActivity는 앱의 주요 진입 지점입니다. 즉 이것은 사용자가 시작 관리자 아이콘을 사용하여 앱을 처음 시작할 때 열리는 액티비티입니다.

ACTION\_MAIN 작업은 이것이 주요 진입 지점이며 어느 인텐트 데이터도 기대하지 않는다는 것을 나타냅니다. CATEGORY\_LAUNCHER 카테고리는 이 액티비티의 아이콘이 시스템의 앱 시작 관리자에 배치되어야 한다는 것을 나타냅니다. 요소가 아이콘을 icon으로 지정하지 않은 경우, 시스템은 요소로부터 가져온 아이콘을 사용합니다. 이들 두 가지가 짝을 이루어야 액티비티가 앱 시작 관리자에 나타날 수 있습니다.

두 번째 액티비티인 ShareActivity는 텍스트와 미디어 콘텐츠 공유를 용이하게 할 목적으로 만들어졌습니다. 사용자가 MainActivity에서 이 액티비티로 이동하여 진입할 수도 있지만, 두 가지 인텐트 필터 중 하나와 일치하는 암시적 인텐트를 발생시키는 다른 앱에서 ShareActivity에 직접 진입할 수도 있습니다.

> 참고 : MIME 유형, 즉 application/vnd.google.panorama360+jpg는 파노라마 사진을 지정하는 특수 데이터 유형으로, Google Panorama API로 처리할 수 있습니다.

## intent와 다른 앱의 intent filters를 일치시키세요

만약 다른 앱이 Android 13 (API level 33) 이상을 target 한다면, 당신 앱의 intent가 다른 앱의 `<intent-filter>`의 actions와 categories가 일치하는 경우에만 당신 앱의 intent를 실행할 수 있다.

마찬가지로 만약 당신의 앱을 Android 13 이상으로 업데이트 한다면, 외부 앱으로부터 유래되는 모든 intent들은 당신 앱의 export된 component로 전달된다. 오직 해당 intent가 당신의 앱이 선언한 `<intent-filter>`의 actions과 categories가 일치할 때만 말이다. 이 행동은 intent를 전송하는 앱의 target SDK Version과 상관없이 발생한다.

intent matching이 강제되지 않은 예외들:

* 어떠한 intent filters가 선언되지 않은 components로 전달되는 intent들
    
* 같은 앱에서 유래된 Intent
    
* system으로부터 유래된 intent; intent들은 "system UID"(uid=1000)으로부터 보내진다. System 앱은 `system_server`와 `android:sharedUserId`가 `android.uid.system`으로 설정된 앱들을 포함하고 있다.
    
* root로부터 전달된 Intent들
    

[인텐트 일치](https://developer.android.com/guide/components/intents-filters#imatch)에서 더 공부하세요.

## pending intent 사용

PendingIntent 객체는 Intent 객체 주변을 감싸는 래퍼입니다. PendingIntent의 기본 목적은 외부 애플리케이션에 권한을 허가하여 안에 들어 있는 Intent를 마치 본인 앱의 자체 프로세스에서 실행하는 것처럼 사용하게 하는 것입니다.

보류 인텐트의 주요 사용 사례는 다음과 같습니다.

* 사용자가 여러분의 알림으로 어떤 작업을 수행할 때 인텐트가 실행되도록 선언합니다(Android 시스템의 가 NotificationManagerIntent를 실행합니다).
    
* 사용자가 여러분의 앱 위젯으로 어떤 작업을 수행할 때 인텐트가 실행되도록 선언합니다(메인 화면 앱이 Intent를 실행합니다).
    
* 향후 지정된 시간에 인텐트가 실행되도록 선언합니다(Android 시스템의 AlarmManager가 Intent를 실행합니다).
    

각 Intent 객체는 특정한 유형의 앱 구성 요소(Activity, Service 또는 BroadcastReceiver)가 처리하도록 설계되어 있으므로, PendingIntent도 같은 고려 사항을 생각해서 생성해야 합니다. 보류 인텐트를 사용하는 경우, 여러분의 앱은 startActivity()와 같은 호출이 있는 앱을 실행하지 않게 됩니다. 대신 PendingIntent를 생성할 때 원래 의도한 구성 요소 유형을 선언해야 합니다. 이때 각각의 생성자 메서드를 호출하는 방법을 씁니다.

* Activity를 시작하는 Intent의 경우, PendingIntent.getActivity()
    
* Service를 시작하는 Intent의 경우, PendingIntent.getService()
    
* BroadcastReceiver를 시작하는 Intent의 경우, PendingIntent.getBroadcast()
    

여러분의 앱이 다른 앱에서 보류 인텐트를 수신하지 않는 한, 위의 PendingIntent를 생성하는 PendingIntent 메서드만 있으면 될 것입니다.

각 메서드는 현재 앱의 Context, 감싸고자 하는 Intent, 인텐트의 적절한 사용 방식을 나타내는 하나 이상의 플래그(예: 인텐트를 한 번 이상 사용할 수 있는지 여부) 등을 취합니다.

보류 인텐트 사용에 관한 자세한 내용은 각 해당되는 사용 사례에 대한 문서를 참조하세요. 예를 들어 Notifications 및 App Widgets API 가이드가 있습니다.

## mutability 명시

만약 앱이 안드로이드 12 이상을 target한다면, 반드시 앱이 만든 각각의 PendingIntent 객체의 mutability를 명시해야한다. 주어진 PendingIntent 객체가 mutable 또는 immutable한지 선언하기 위해, `PendingIntent.FLAG_MUTABLE` 또는 `PendingIntent.FLAG_IMMUTABLE`를 각각 사용해라.

만약 앱이 둘 중 하나의 mutability flag를 설정하지 않은 PendingIntent를 만들려고 시도하면, 시스템은 IllegalArgumentException를 발생시키고 다음 메세지를 Logcat에 표시합니다.

PACKAGE\_NAME: Targeting S+ (version 31 and above) requires that one of  
FLAG\_IMMUTABLE or FLAG\_MUTABLE be specified when creating a PendingIntent.

Strongly consider using FLAG\_IMMUTABLE, only use FLAG\_MUTABLE if  
some functionality depends on the PendingIntent being mutable, e.g. if  
it needs to be used with inline replies or bubbles.

### 가능한 경우 언제든지 변경할 수 없는 PendingIntent 만들기

대부분의 경우, 다음 코드 스니펫처럼, 앱은 immutable PendingIntent 객체를 만들어야합니다. 만약 PendingIntent 객체가 immutable하다면 다른 앱은 intent를 호출한 결과를 조절하기 위해 intent를 수정할 수 없습니다.

```kotlin
val pendingIntent = PendingIntent.getActivity(applicationContext,
        REQUEST_CODE, intent,
        /* flags */ PendingIntent.FLAG_IMMUTABLE)
```

그러나, 특정한 use case는 대신 mutable PendingIntent를 요구합니다.

* [알림에서 바로 답장](https://developer.android.com/training/notify-user/build-notification#reply-action)을 지원할 때. 바로 답장은 답장과 연관된 PendingIntent 객체 안에서 clip data에 대한 변화를 요구합니다. 보통, 이러한 변화를 `FILL_IN_CLIP_DATA`를 flag로서 fillIn() 메서드로 전달함으로써 제공합니다.
    
* Android Auto framework와 연관된 알림의 경우, CarAppExtender 인스턴스를 사용하는 경우
    
* PendingIntent의 인스턴스를 사용하는 [bubbles](https://developer.android.com/guide/topics/ui/bubbles)안에 대화를 배치하는 경우. mutable PendingIntent 객체는 시스템이 올바른 flags를 적용하는 것을 허용합니다. (예: `FLAG_ACTIVITY_MULTIPLE_TASK`, `FLAG_ACTIVITY_NEW_DOCUMENT`)
    
* `requestLocationUpdates()` 또는 유사한 API를 호출하여 디바이스의 위치 정보를 요구할 때. mutable PendingIntent 객체는 시스템이 location lifecycle event를 나타내는 intent extras를 추가하는 것을 허용해줍니다. 이러한 이벤트들은 위치 변화와 provider 사용 가능 여부가 포함된다.
    
* AlarmManager를 사용하여 알람을 관리할 때. mutable PendingIntent 객체는 시스템이 `EXTRA_ALARM_COUNT` intent extra를 추가하는 것을 허용해줍니다. 이러한 extra는 실행된 반복되는 알람의 횟수를 나타냅니다. 이 extra를 포함함으로써 intent는 확실하게 반복되는 알람이 몇번 실행되었는지(예: 앱이 sleep 일때) 앱에게 알릴 수 있습니다.
    

만약 mutable PendingIntent 객체를 만든다면, explict intent를 사용하고 ComponentName을 채워넣을 것을 권고합니다. 그렇게 하면, 다른 앱이 PendingIntent를 실행하고 제어권이 다시 당신의 앱으로 넘어올 때마다, 당신 앱의 같은 component가 항상 실행됩니다.

### pending intents 안에서 explict intent를 사용하라

다른 앱이 당신의 앱의 pending intents를 사용할 수 있는 방법을 더 잘 정의하려면 항상 pending intent를 explict intent로 감싸세요. 이 모범 사례를 따르려면 다음을 수행하십시오.

1. 설정된 base intent의 action, package, 그리고 component 필드를 확인하세요.
    
2. pending intents를 생성하기 위해 Android 6.0 (API level 23)에서 추가된 `FLAG_IMMUTABLE`를 사용하세요. 이 flag는 PendingIntent를 수신한 앱이 unpopluated 프로퍼티로 채워지는 것을 방지해줍니다. 만약 앱의 minSdkVersion이 22 이하라면, 다음 코드를 통해 safety하고 compatibility를 동시에 제공할 수 있다.
    

```kotlin
if (Build.VERSION.SDK_INT >= 23) {
  // Create a PendingIntent using FLAG_IMMUTABLE.
} else {
  // Existing code that creates a PendingIntent.
}
```

## 인텐트 확인

시스템이 액티비티를 시작하라는 암시적 인텐트를 수신하면, 시스템은 해당 인텐트에 대한 최선의 액티비티를 검색합니다. 이때 다음과 같은 세 가지 측면을 근거로 인텐트를 인텐트 필터에 비교합니다.

* 작업.
    
* 데이터(URI와 데이터 유형 둘 다).
    
* 카테고리. 다음 섹션에서는 인텐트 필터가 앱 매니페스트 파일에서 어떻게 선언되었는지에 따라 인텐트가 적절한 구성 요소에 어떻게 매칭되는지 설명합니다.
    

### 작업 테스트

허용된 인텐트 작업을 나타내기 위해 인텐트 필터는 0개 이상의 `<action>` 요소를 선언할 수 있습니다.

```kotlin
<intent-filter>
    <action android:name="android.intent.action.EDIT" />
    <action android:name="android.intent.action.VIEW" />
    ...
</intent-filter>
```

이 필터를 통과하려면 Intent에 지정된 작업이 필터에 나열된 작업 중 하나와 일치해야만 합니다.

필터에 나열된 작업이 없는 경우, 인텐트가 일치될 대상이 아무것도 없으므로 모든 인텐트가 테스트에 실패합니다. 그러나 Intent가 작업을 지정하지 않는 경우, 필터에 최소한 한 개 이상의 작업이 들어 있지 않다면 인텐트가 테스트를 통과하게 됩니다.

### 카테고리 테스트

허용된 인텐트 카테고리를 나타내기 위해 인텐트 필터는 0개 이상의 `<category>` 요소를 선언할 수 있습니다.

```kotlin
<intent-filter>
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    ...
</intent-filter>
```

인텐트가 카테고리 테스트를 통과하려면 Intent 내의 모든 카테고리가 필터 내의 카테고리에 일치해야 합니다. 그 역은 반드시 성립하지 않아도 됩니다. 인텐트 필터가 Intent에서 지정된 것보다 더 많은 카테고리를 선언할 수 있지만, 그래도 Intent는 통과합니다. 그러므로 카테고리가 없는 인텐트라면 필터에 어떤 카테고리가 선언되어 있든 이 테스트를 항상 통과하는 것이 맞습니다.

> 참고 : Android는 startActivity()와 startActivityForResult()에 전달된 모든 암시적 인텐트에 CATEGORY\_DEFAULT 카테고리를 자동으로 적용합니다. 따라서 액티비티가 암시적 인텐트를 수신하기를 원하는 경우, 그 인텐트 필터 내에 "android.intent.category.DEFAULT"에 대한 카테고리가 반드시 포함되어 있어야 합니다(이전의 `<intent-filter>` 예시 참조).

### 데이터 테스트

허용된 인텐트 데이터를 나타내기 위해 인텐트 필터는 0개 이상의 `<data>` 요소를 선언할 수 있습니다.

```kotlin
<intent-filter>
    <data android:mimeType="video/mpeg" android:scheme="http" ... />
    <data android:mimeType="audio/mpeg" android:scheme="http" ... />
    ...
</intent-filter>
```

각 `<data>` 요소는 URI 구조와 데이터 유형(MIME 미디어 유형)을 나타낼 수 있습니다. URI의 각 부분은 별도의 특성(scheme, host, port, path)으로 구성됩니다.

`<scheme>://<host>:<port>/<path>`

다음 예시는 이 특성에 사용 가능한 값을 보여줍니다.

`content://com.example.project:200/folder/subfolder/etc`

이 URI에서 구성표는 content, 호스트는 com.example.project, 포트는 200, 경로는 folder/subfolder/etc입니다.

이와 같은 특성은 각각 `<data>` 요소에서 선택 항목이지만, 서로 선형 종속성이 있습니다.

* 구성표가 지정되지 않으면 호스트가 무시됩니다.
    
* 호스트가 지정되지 않으면 포트가 무시됩니다.
    
* 구성표와 호스트가 둘 다 지정되지 않으면 경로가 무시됩니다.
    

인텐트 안의 URI가 필터 안의 URI 사양에 비교되는 경우, 이것은 필터 내에 포함된 URI의 몇몇 부분에만 비교되는 것입니다. 예를 들면 다음과 같습니다.

* 필터가 구성표만 지정하는 경우, 해당 구성표가 있는 모든 URI가 필터와 일치합니다.
    
* 필터가 구성표와 권한은 지정하지만 경로는 지정하지 않는 경우, 같은 구성표와 권한이 있는 모든 URI가 경로와 관계없이 필터를 통과합니다.
    
* 필터가 구성표, 권한과 경로를 지정하는 경우 같은 구성표, 권한과 경로가 있는 URI만 해당 필터를 통과합니다.
    

> 참고 : 경로 사양에는 경로 이름이 부분적으로만 일치하도록 요구하기 위해 와일드카드 별표(\*)가 들어 있을 수 있습니다.

데이터 테스트는 인텐트 안의 URI와 MIME 유형 둘 모두를 필터 안에서 지정된 MIME 유형과 비교합니다. 규칙은 다음과 같습니다.

1. URI도 MIME 유형도 들어 있지 않은 인텐트는 필터가 URI나 MIME 유형을 전혀 지정하지 않은 경우에만 테스트를 통과합니다.
    
2. URI는 들어 있지만 MIME 유형은 없는 인텐트(URI로부터는 명시적이지도 않고 추론할 수도 없음)는 그 URI가 필터의 URI 형식과 일치하고, 필터가 인텐트와 마찬가지로 MIME 유형을 지정하지 않는 경우에만 테스트를 통과합니다.
    
3. MIME 유형은 들어 있지만 URI는 없는 인텐트는 해당 필터가 같은 MIME 유형을 나열하지만 URI 형식은 지정하지 않은 경우에만 테스트를 통과합니다.
    
4. URI와 MIME 유형이 둘 다 들어 있는 인텐트(URI로부터는 명시적이지도 않고 추론할 수도 없음)는 해당 유형이 필터 내에 나열된 유형과 일치하는 경우에만 MIME 유형 부분의 테스트를 통과합니다. 이 인텐트는 URI가 필터 내의 URI와 일치하는 경우나, content: 또는 file: URI가 있고 필터가 URI를 지정하지 않은 경우에 URI 부분의 테스트를 통과합니다. 달리 말하면, 필터가 MIME 유형만 나열하는 경우, 구성 요소가 content: 및 file: 데이터를 지원하는 것으로 간주됩니다.
    

> 참고 : 인텐트가 URI 또는 MIME 유형을 지정하는 경우, `<intent-filter>`에 `<data>` 요소가 없으면 데이터 테스트를 통과하지 못합니다.

이 마지막 규칙인 규칙 (d)는 구성 요소가 파일 또는 콘텐츠 제공자로부터 로컬 데이터를 가져올 수 있다는 기대를 반영한 것입니다. 따라서 이런 필터는 데이터 유형만 나열할 수 있고, content: 및 file: 구성표를 명시적으로 지명하지 않아도 됩니다. 다음 예시는 `<data>` 요소가 구성 요소가 콘텐츠 제공자에서 이미지 데이터를 가져와 표시할 수 있다고 Android에 알리는 일반적인 사례를 보여줍니다.

```kotlin
<intent-filter>
    <data android:mimeType="image/*" />
    ...
</intent-filter>
```

데이터 유형은 지정하지만 URI는 지정하지 않는 필터가 가장 보편적일 것입니다. 그 이유는 대부분의 사용 가능한 데이터를 콘텐츠 제공자가 제공하기 때문입니다.

다른 보편적인 구성을 예로 들자면 구성표와 데이터 유형을 가진 필터가 있겠습니다. 예를 들어 다음과 같은 `<data>` 요소는 구성 요소가 작업을 수행하기 위해 네트워크에서 비디오 데이터를 검색할 수 있다고 Android에 알립니다.

```kotlin
<intent-filter>
    <data android:scheme="http" android:mimeType="video/*" />
    ...
</intent-filter>
```

### 인텐트 일치

인텐트를 인텐트 필터에 비교해 일치시키면 활성화할 대상 구성 요소를 찾아낼 수 있을 뿐만 아니라, 기기에 있는 일련의 구성 요소에 대해 무언가 알아낼 수도 있습니다. 예를 들어 홈 앱이 앱 시작 관리자를 채우려면 ACTION\_MAIN 작업과 CATEGORY\_LAUNCHER 카테고리를 지정하는 인텐트 필터를 가진 액티비티를 모두 찾아야 합니다. 인텐트의 작업과 카테고리가 필터와 일치해야 매칭이 성공합니다. 이 내용은 IntentFilter 클래스에 대한 문서에서 설명합니다.

애플리케이션은 홈 앱과 같은 방식으로 인텐트 매칭을 사용합니다. PackageManager에 있는 query...() 메서드 집합은 특정 인텐트를 허용하는 구성 요소를 모두 반환하며, 이와 유사한 일련의 resolve...() 메서드는 한 인텐트에 응답하는 데 가장 좋은 구성 요소를 판별합니다. 예를 들어 queryIntentActivities()는 인텐트를 인수로 전달할 수 있는 모든 액티비티의 목록을 반환하고 queryIntentServices()는 서비스와 관련하여 그와 유사한 목록을 반환합니다. 어느 메서드도 구성 요소를 활성화하지는 않습니다. 단지 응답할 수 있는 것을 나열하기만 합니다. 이와 유사한 메서드인 queryBroadcastReceivers()도 있는데, 이것은 Broadcast Receiver용입니다.