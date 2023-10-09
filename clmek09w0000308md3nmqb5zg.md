---
title: "[Android] Activity"
datePublished: Thu Sep 01 2022 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmek09w0000308md3nmqb5zg
slug: android-activity
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/W9f-PrkRqk0/upload/0b1f079fcb9d560f2fdabe0a4618f329.jpeg
tags: android

---

> 안드로이드 공식 문서를 번역하고 내용을 조금 변경하거나 내용을 추가한 게시글입니다. 잘못된 내용이 있을 수 있습니다. [참고한 공식 문서 바로가기](https://developer.android.com/guide/components/activities/intro-activities)

`Activity` 클래스는 Android 앱의 중요한 컴포넌트입니다. `Activity`가 실행되고 합쳐지는 방법은 플랫폼 어플리케이션 모델의 근본적인 부분입니다. `main()` 메서드에서 앱이 실행되는 프로그램과는 달리, 안드로이드 시스템은 `Activity` 인스턴스 안에서 특정한 생명주기에 맞는 callback 메서드를 실행함으로써 코드를 시작합니다.

  

이 문서는 Activities 개념을 소개하고 어떻게 동작하는지에 대한 가벼운 가이드를 제공합니다.

  

## The concept of activities

모바일 앱 환경은 데스크톱 앱 환경과 달리 사용자와 앱 상호작용이 항상 동일한 위치에서 시작되지 않습니다. 대신 사용자는 앱을 다른 화면에서 시작할 수 있습니다. 예를 들어 홈 화면에서 이메일 앱을 열면 이메일 목록이 표시될 수 있고 반대로 SNS 앱을 사용하고 있는 상태에서 이메일 앱을 실행하면 이메일을 작성 화면으로 바로 이동할 수 있습니다.

  

`Activity` 클래스는 이 패러다임이 가능하도록 설계되었습니다. A앱이 B앱을 작동시켰을 때, A앱은 B앱의 전체를 호출하지 않고 B앱의 activity를 호출합니다. 이런 방식으로 activity는 앱과 사용자의 상호작용을 위한 진입점 역할을 합니다. activity는 Activity 클래스의 서브클래스로 구현됩니다.

  

activity는 UI를 그리는 window를 제공합니다. window는 일반적으로 화면 전체를 채우지만 화면보다 작고 다른 window 위에 떠 있을 수 있습니다. 일반적으로 하나의 activity는 하나의 화면을 구현합니다. 예를 들어 activity 중 하나는 환경설정 화면을 다른 activity는 사진 선택 화면을 구현할 수 있습니다.

  

대부분의 앱은 여러 화면을 포함하고 있습니다. 즉, 여러 activity로 구성됩니다. 일반적으로 *기본 activity*가 지정되며 기본 activity는 사용자가 앱을 실행할 때 표시되는 첫 번째 화면입니다. 각 activity는 또 다른 activity를 실행할 수 있습니다. 예를 들어 기본 activity는 받은 편지함을 표시하고 이메일 작성 및 이메일 상세보기 화면(activity)을 실행할 수 있습니다.

  

activity는 앱의 일관된 사용자 환경을 형성하기 위해 함께 작동하지만, 각 activity는 다른 activity에 느슨하게 결합됩니다. 일반적으로 activity 간에는 최소한의 종속성만 있습니다. 실제로 activity는 흔히 다른 앱의 activity를 시작할 수 있습니다. 예를 들어, 브라우저 앱은 SNS 앱의 공유 activity를 실행할 수 있습니다.

  

activity를 사용하려면 앱의 manifest에 activity 관련 정보를 등록하고 activity 생명 주기를 적절히 관리해야합니다. 이 문서의 나머지 부분에는 이러한 주제를 소개합니다.

  

## manifest 구성

manifest에 activity 및 관련된 특정 속성을 선언해야 앱에서 activity를 사용할 수 있습니다.

  

### Activity 선언

Activity를 선언하려면 manifest 파일을 열고 `<activity>` 요소 `<application>` 요소의 하위 요소로 추가해야 합니다. 예를 들면 다음과 같습니다.

```kotlin
    <manifest ... >
      <application ... >
          <activity android:name=".ExampleActivity" />
          ...
      </application ... >
      ...
    </manifest >
```

이 요소의 유일한 필수 속성은 activity의 클래스 이름을 지정하는 android:name입니다. 또한 라벨, 아이콘 또는 UI 테마와 같은 activity 특성을 정의하는 속성도 추가할 수 있습니다. 이러한 속성 및 기타 속성에 관한 자세한 내용은 `<activity>` 요소 참조 문서에서 확인할 수 있습니다.

  

> 참고: 앱을 게시한 후에는 활동 이름을 변경해서는 안 됩니다. 활동 이름을 변경하면 앱 바로가기와 같은 일부 기능이 작동하지 않을 수 있습니다. 게시 후 피해야 할 변경사항에 관한 자세한 내용은 변경할 수 없는 사항을 참조하세요.

  

### 인텐트 필터 선언

인텐트 필터는 Android 플랫폼의 매우 강력한 기능입니다. 인턴트 필터는 *명시적* 요청 뿐만 아니라 *암시적* 요청을 기반으로도 activity를 실행하는 기능을 제공합니다. 예를 들어 명시적 요청은 'Gmail 앱에서 이메일 보내기 activity를 시작'하도록 시스템에 지시할 수 있습니다. 이와 반대로 암시적 요청은 '작업을 할 수 있는 activity으로 이메일 보내기 화면을 시작'하도록 시스템에 지시합니다. 시스템 UI 에서 사용자에게 작업을 실행할 때 어떤 앱을 사용할지 묻는 메시지가 표시되면 바로 인텐트 필터가 작동한 것입니다.

  

`<activity>`요소에서 `<intent-filter>` 속성을 선언함으로써 이 기능을 활용할 수 있습니다. 이 요소의 정의에는 `<activity>`요소와 선택적으로 `<category>`요소 또는 `<data>` 요소가 포함됩니다. 이러한 요소를 결합하여 activity가 응답할 수 있는 인텐트 유형을 지정할 수 있습니다. 예를 들어 다음 코드 스니펫은 텍스트 데이터를 전송하고 다른 activity들의 요청을 수신하는 activity를 구성하는 방법을 보여줍니다.

```kotlin
    <activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
        <intent-filter>
            <action android:name="android.intent.action.SEND" />
            <category android:name="android.intent.category.DEFAULT" />
            <data android:mimeType="text/plain" />
        </intent-filter>
    </activity>
```

  

예시에서 `<action>` 요소는 activity가 데이터를 전송하도록 지정합니다. `<category>` 요소를 `DEFAULT`로 선언하면 activity가 실행 요청을 수신할 수 있습니다. `<data>` 요소는 activity가 전송할 수 있는 데이터 유형을 지정합니다. 다음 코드 스니펫은 위에서 묘사된 activity를 어떻게 호출하는지 알려줍니다.

  

```kotlin
val sendIntent = Intent().apply {
    action = Intent.ACTION_SEND
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, textMessage)
}
startActivity(sendIntent)
```

앱이 독립적인 상태를 유지하도록 만들고 다른 앱이 앱 activity를 활성화하지 못하게 하려면 인텐트 필터를 사용하지 않으면 됩니다. 다른 애플리케이션에서 사용하지 못하게 하려는 activity에는 인텐트 필터가 없어야 하며, 개발자는 명시적 인텐트를 사용하여 activity를 직접 시작할 수 있습니다. activity가 인텐트에 어떻게 응답할 수 있는지에 관한 자세한 내용은 [인텐트 및 인텐트 필터](https://developer.android.com/guide/components/intents-filters)를 참조하세요.

  

### 권한 선언

manifest의 `<activity>` 태그를 사용하여 특정 activity를 시작할 수 있는 앱을 제어할 수 있습니다. 부모 activity와 자식 activity가 모두 각 manifest에서 동일한 권한을 가지고 있지 않다면 부모 activity가 자식 activity를 실행할 수 없습니다. 부모 activity에 `<uses-permission>` 요소가 선언되었다면 각 자식 activity는 매칭되는 `<uses-permission>`을 가지고 있어야합니다.

  

예를 들어, 당신의 앱이 소셜 미디어 게시글을 공유하기 위해 `소셜앱`을 사용하고 싶다면 `소셜앱` 자체에서 permission 정의해야 합니다.

```kotlin
<manifest>
<activity android:name="...."
   android:permission=”com.google.socialapp.permission.SHARE_POST”

/>
```

다음과 같이 당신의 앱은 소셜앱의 manifest와 permission 일치해야 합니다.

```kotlin
<manifest>
   <uses-permission android:name="com.google.socialapp.permission.SHARE_POST" />
</manifest>
```

  

권한 및 보안에 관한 전반적인 내용은 [보안 및 권한](https://developer.android.com/guide/topics/security/security)을 참조하세요

  

## Activity 수명 주기 관리

Activity는 수명 주기 전체 기간에 걸쳐 여러 상태를 가집니다. 상태 간 전환을 처리하는 데 일련의 콜백을 사용할 수 있습니다. 다음 섹션에서는 이러한 콜백을 소개합니다.

### onCreate()

시스템이 Activity를 생성할 때 실행되는 onCreate()를 구현해야 합니다. 구현 시 activity의 필수 구성요소를 초기화애햐 합니다. 예를 들어 앱은 여기에서 뷰를 생성하고 데이터를 목록에 결합해야 합니다. onCreate()에서 setContentView()를 호출하여 activity의 사용자 인터페이스를 위한 레이아웃을 정의해야 하며 이 작업이 가장 중요합니다.

onCreate()가 완료되면 다음 콜백은 onStart()입니다.

  

### onStart()

onCreate()가 종료되면 activity는 *Started* 상태로 전환되고 activity가 사용자에게 표시됩니다. 이 콜백에는 activity가 foreground로 나와 interactive되기 위한 최종 준비에 준하는 작업이 포함됩니다.

  

### onResume()

activity가 사용자와 상호작용을 시작하기 직전에 시스템은 이 콜백을 호출합니다. 이 시점에서 activity는 activity 스택의 맨 위에 있으며 모든 사용자 입력을 포착합니다. 앱의 핵심 기능은 대부분 onResume() 메서드로 구현됩니다.

  

### onPause()

activity가 포커스를 잃고 `paused` 상태로 전환될 때 시스템은 onPause()를 호출합니다. 예를 들어 이 상태는 사용자가 뒤로가기 또는 최근 버튼을 누를 때 발생합니다. 시스템이 activity에서 onPause()를 호출할 때 이는 활동이 여전히 부분적으로 표시되지만 대체로 사용자가 activity를 떠나고 있으며 activity가 조만간 `Stopped` 또는 `Resumed` 상태로 전환됨을 나타냅니다.

사용자가 UI 업데이트를 기다리고 있다면 `Paused` 상태의 activity는 계속 UI를 업데이트할 수 있습니다. 이러한 activity의 예는 내비게이션 지도 화면 또는 미디어 플레이어 재생을 표시하는 activity가 포함됩니다. 이러한 activity는 포커스를 잃더라도 사용자는 UI가 계속 업데이트될 것으로 예상합니다.

애플리케이션 또는 사용자 데이터를 저장하거나 네트워크를 호출하거나 데이터베이스 트랜잭션을 실행하는데 onPause()를 사용하면 **안됩니다** 데이터 저장에 관한 자세한 내용은 [activity 상태 저장 및 복원](https://developer.android.com/guide/components/activities/activity-lifecycle#saras)을 참조하세요.

onPause()가 실행을 완료하면 다음 콜백은 activity가 `Pause` 상태로 전환된 후 발생하는 상황에 따라 onStop()또는 onResume()입니다.

  

### onStop()

activity가 사용자에게 더 이상 표시되지 않을 때 시스템은 onStop()을 호출합니다. 이는 activity가 제거 중이거나 새 activity가 시작 중이거나 기존 activity가 `Resume` 상태로 전환 중이고 중지된 activity를 다루고 있기 때문에 발생할 수 있습니다. 이 모든 상태에서 중지된 activity는 더 이상 표시되지 않습니다.

시스템이 호출하는 다음 콜백은 activity가 사용자와 상호작용하기 위해 다시 시작되면 onRestart()이며 이 activity가 완전히 종료되면 onDestroy()입니다.

  

### onRestart()

`Stopped` 상태의 activity가 다시 시작되려고 할 때 시스템은 이 콜백을 호출합니다. onRestart()는 activity가 중지된 시간 부터 activity 상태를 복원합니다.

이 콜백 뒤에 항상 onStart()가 옵니다.

  

### onDestroy()

시스템은 activity가 제거되기 전에 이 콜백을 호출합니다.

이 콜백은 activity가 수신하는 마지막 콜백입니다. onDestroy()는 일반적으로 activity 또는 activity가 포함된 프로세스가 제거될 때 activity의 모든 리소스를 해제하도록 구현됩니다.

이 섹션에서는 이 주제에 관한 소개만 제공합니다. activity 수명 주기 및 관련 콜백에 관한 자세한 논의는 [activity 수명 주기](https://developer.android.com/guide/components/activities/activity-lifecycle)를 참고하세요.