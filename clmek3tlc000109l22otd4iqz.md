---
title: "[Android] Service"
datePublished: Sun Sep 11 2022 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmek3tlc000109l22otd4iqz
slug: android-service
tags: android

---

> 안드로이드 공식 문서를 번역하고 내용을 조금 변경하거나 내용을 추가한 게시글입니다. 잘못된 내용이 있을 수 있습니다. [참고한 공식 문서 바로가기](https://developer.android.com/guide/components/services)

`Service`는 백그라운드에서 오래 실행되는 작업을 수행할 수 있는 application component입니다. 사용자 인터페이스를 제공하지 않습니다. 한번 시작하면, 사용자가 다른 application으로 전환해도 service는 계속 실행될 수 있습니다. 추가로 component를 service에 바인딩하여 service와 상호작용할 수 있으며 심지어는 프로세스 간 통신(IPC)도 수행할 수 있습니다. 예를 들어 한 service는 네트워크 트랜잭션을 처리하고, 음악을 재생하고 파일 I/O를 수행하거나 content provider와 상호작용할 수 있으며 이 모든 것을 백그라운드에서 수행할 수 있습니다.

> 주의 : service는 hosting process의 main thread에서 실행됩니다. service는 자신만의 thread를 생성하지 않고 달리 명시하지 않는 한 별개의 process에서 동작하지 않습니다. ANR(Application Not Responding) 에러를 피하기 위해 service안에서 아무 blocking operations를 별개의 thread에서 작동시켜야합니다.

서비스에는 세 가지 유형이 있습니다.

  

### 포그라운드

포그라운드 서비스는 사용자에게 잘 보이는 작업을 수행합니다. 예를 들어 오디오 앱이라면 오디오 트랙을 재생하기 위한 포그라운드 서비스를 사용합니다. 포그라운드 서비스는 반드시 [Notification](https://developer.android.com/develop/ui/views/notifications)을 표시해야합니다. 포그라운드 서비스는 유저가 앱과 상호작용하지 않을 때도 계속 실행됩니다.

포그라운드 서비스를 사용할 때, notification을 반드시 보여줘야합니다. 그래야 사용자들이 서비스가 실행 중임을 알 수 있습니다. 이러한 notification은 서비스가 포그라운드에서 중지 또는 제거되지 않는한 사라지지 않습니다.

[포그라운드 서비스](https://developer.android.com/guide/components/foreground-services) 환경 설정 방법을 배워보세요

> 노트 : WorkManager API는 작업을 스케쥴링하는 유연한 방법을 제공하고 필요한 경우 이런 작업들을 포그라운드 서비스로써 실행할 수 있게 합니다. 많은 경우에 포그라운드 서비스들을 직접적으로 사용하기 위해 WorkManger를 사용합니다.

  

### 백그라운드

백그라운드 서비스는 유저에게 직접적으로 알려지지 않는 작업을 수행합니다. 예를 들어, 저장 공간 최적화를 위한 서비스는 보통 백그라운드 서비스로 실행합니다.

> 노트 : 만약 API lever 26 이상을 타겟팅한다면, 시스템은 앱이 포그라운드에 있지 않을 때 백그라운드 서비스 제한을 강요합니다. 예를 들어, 대부분의 상황에서 백그라운드에서 위치 정보에 접근할 수 없습니다. 대신 WorkManager를 사용하여 작업을 스케쥴링 해야합니다.

  

### 바인드(Bound)

어플리케이션 component가 bindService()를 호출하여 해당 서비스에 바인딩되면 서비스가 바인딩됩니다. 바인딩된 서비스는 클라이언트-서버 인터페이스를 제공하며 component가 서비스와 상호작용하게 합니다. request를 보내고 result를 받고 심지어 이런 작업을 여러 프로세스에 걸쳐 프로세스 간 통신(IPC)으로 수행할 수도 있습니다. 바인딩된 서비스는 또 다른 어플리케이션 component가 이에 바인딩되어 있는 경우에만 실행됩니다. 여러 component가 서비스에 한번에 바인딩될 수 있지만 모든 것이 언바인드되면 서비스는 파괴됩니다.

이 문서에서는 시작된 서비스와 바인딩된 서비스를 전반적으로 별도로 논하지만, 서비스는 양쪽 방식으로 둘 다 작동할 수 있습니다. 즉 시작해서(무한히 실행되도록) 바인드도 허용할 수 있습니다. 이것은 단순히 두어 가지 콜백 메서드를 구현했는지 여부에 좌우되는 문제입니다. onStartCommand()는 component가 서비스를 시작하게 하고 onBind()는 바인드를 허용합니다.

서비스가 시작되었든, 바인딩되었든 아니면 양쪽 모두이든 모든 애플리케이션 component가 해당 서비스를 사용할 수 있으며(심지어 별도의 애플리케이션에서도 사용 가능), 이는 어느 component든 액티비티를 사용할 수 있는 것과 같습니다. 이를 Intent로 시작하면 됩니다. 그러나 매니페스트에서 서비스를 비공개로 선언하고 다른 애플리케이션으로부터의 액세스를 차단할 수도 있습니다. 이 내용은 매니페스트에서 서비스 선언에 관한 섹션에서 더 자세히 설명합니다.

  

## 서비스와 스레드 간의 선택

서비스는 그저 백그라운드에서 실행될 수 있는 component일 뿐입니다. 이는 사용자가 애플리케이션과 상호작용하지 않아도 관계없이 해당됩니다. 그러므로 필요한 경우에만 서비스를 사용해야 합니다.

사용자가 애플리케이션과 상호작용하는 동안 메인 스레드 밖에서 작업을 수행해야 하는 경우, 다른 application component의 context안에서 새 스레드를 생성해야 합니다. 예를 들어 액티비티가 실행되는 중에만 음악을 재생하고자 하는 경우, onCreate() 안에 스레드를 생성하고 이를 onStart()에서 실행하기 시작한 다음, onStop()에서 중단하면 됩니다. 또한 전통적인 Thread 클래스 보다 `java.util.concurrent` 패키지 또는 Kotlin coroutines의 thread pool과 executors 사용을 고려해보세요

서비스를 사용하는 경우 기본적으로 애플리케이션의 기본 스레드에서 계속 실행되므로 서비스가 집약적이거나 차단 작업을 수행하는 경우 여전히 서비스 내에 새 스레드를 생성해야 한다는 점을 명심하세요.

  

## 기본 사항

서비스를 생성하려면 Service의 하위 클래스(또는 이것의 기존 하위 클래스 중 하나)를 생성해야 합니다. 구현에서는 서비스 수명 주기의 주요 측면을 처리하는 콜백 메서드를 몇 가지 override해야 하며 서비스에 바인딩할 component에 대한 메커니즘을 제공해야 합니다(해당되는 경우). 다음은 override가 필요한 가장 중요한 콜백 메서드입니다.

  

### `onStartCommand()`

시스템이 이 메서드를 호출하는 것은 또 다른 component(예: 액티비티)가 서비스를 시작하도록 요청하는 경우입니다. 이때 startService()를 호출하는 방법을 씁니다. 이 메서드가 실행되면 서비스가 시작되고 백그라운드에서 무한히 실행될 수 있습니다. 이것을 구현하면 서비스의 작업이 완료되었을 때 해당 서비스를 중단하는 것은 개발자 본인의 책임이며, 이때 stopSelf() 또는 stopService()를 호출하면 됩니다. 바인딩만 제공하고자 하는 경우, 이 메서드를 구현하지 않아도 됩니다.

  

### `onBind()`

시스템은 다른 component가 해당 서비스에 바인딩되고자 하는 경우(예를 들어 RPC를 수행하기 위해)에도 이 메서드를 호출합니다. 이때 bindService()를 호출하는 방법을 사용합니다. 이 메서드를 구현할 때에는 클라이언트가 서비스와 통신을 주고받기 위해 사용할 인터페이스를 제공해야 합니다. 이때 IBinder를 반환하면 됩니다. 이 메서드는 항상 구현해야 하지만, 바인딩을 허용하지 않으려면 null을 반환해야 합니다.

  

### `onCreate()`

시스템은 서비스가 처음 생성되었을 때(즉 서비스가 onStartCommand() 또는 onBind()를 호출하기 전에) 이 메서드를 호출하여 일회성 설정 절차를 수행합니다. 서비스가 이미 실행 중인 경우, 이 메서드는 호출되지 않습니다.

  

### `onDestroy()`

시스템이 이 메서드를 호출하는 것은 서비스를 더 이상 사용하지 않고 소멸시킬 때입니다. 서비스는 스레드, 등록된 리스너 또는 수신기 등의 각종 리소스를 정리하기 위해 이것을 구현해야 합니다. 이는 서비스가 수신하는 마지막 호출입니다.

한 component가 startService()를 호출하여 서비스를 시작하면(onStartCommand()에 대한 호출 발생), 해당 서비스는 알아서 stopSelf()로 스스로 중단할 때까지 또는 다른 component가 stopService()를 호출하여 서비스를 중단시킬 때까지 실행 중인 상태로 유지됩니다.

한 component가 bindService()를 호출하여 서비스를 생성하는 경우(그리고 onStartCommand()를 호출하지 않은 경우), 해당 서비스는 해당 component가 바인딩된 경우에만 실행됩니다. 서비스가 모든 클라이언트로부터 바인딩이 해제되면 시스템이 이를 소멸시킵니다.

Android 시스템이 서비스를 강제 중단하는 것은 메모리가 부족하여 사용자 포커스를 가진 액티비티를 위해 시스템 리소스를 회복해야만 하는 경우로만 국한됩니다. 서비스가 사용자 포커스를 가진 액티비티에 바인딩된 경우 종료될 가능성이 적고, 서비스가 포그라운드에서 실행되도록 선언된 경우에는 종료될 가능성이 희박합니다. 서비스가 시작되어 장시간 실행 중이라면 시스템은 시간이 지나면서 백그라운드 작업 목록에서 이 서비스가 차지하는 위치를 낮추고, 따라서 서비스가 종료될 가능성이 높아집니다. 서비스가 시작되었다면 시스템에 의한 재시작을 정상적으로 처리하도록 설계해야 합니다. 시스템이 서비스를 중단하는 경우, 리소스를 다시 사용할 수 있게 되면 가능한 한 빨리 서비스가 다시 시작됩니다. 다만 개발자가 onStartCommand()에서 반환하는 값에 따라 달라집니다. 시스템이 서비스를 소멸시킬 수 있는 경우에 대한 자세한 내용은 [프로세스와 스레드](https://developer.android.com/guide/components/processes-and-threads?hl=ko) 문서를 참조하세요.

다음 섹션에서는 startService()와 bindService() 서비스 메서드를 생성하는 방법과 다른 애플리케이션 component에서 이를 사용하는 방법에 대해 설명합니다.

  

## 매니페스트에서 서비스 선언

액티비티 및 다른 component와 마찬가지로, 서비스는 모두 애플리케이션의 매니페스트 파일에서 선언해야 합니다.

서비스를 선언하려면 `<service>` 요소를 `<application>` 요소의 하위로 추가하면 됩니다. 다음은 이에 대한 예입니다.

```kotlin
<manifest ... >
  ...
  <application ... >
      <service android:name=".ExampleService" />
      ...
  </application>
</manifest>
```

매니페스트에서 서비스를 선언하는 방법에 대한 자세한 내용은 `<service>` 요소 참조 문서를 확인하세요.

`<service>` 요소에 포함할 수 있는 다른 특성도 있습니다. 이 요소를 포함하여 서비스를 시작하는 데 필요한 권한과 서비스를 실행해야 하는 프로세스 등의 특성을 정의할 수 있습니다. `android:name` 특성이 유일한 필수 특성인데, 이는 서비스의 클래스 이름을 나타냅니다. 애플리케이션을 게시한 후에는 이 이름을 그대로 두어야 서비스를 시작 또는 바인딩할 명시적 인텐트에 대한 종속성 때문에 코드가 깨질 위험을 막을 수 있습니다(블로그 게시물 [변경할 수 없는 것](https://android-developers.googleblog.com/2011/06/things-that-cannot-change.html) 참조).

> 주의 : 앱의 보안을 지키기 위해서는 Service를 시작할 때에는 항상 명시적 인텐트만 사용하고 서비스에 대한 인텐트 필터는 선언하지 마세요. 암시적 인텐트를 사용하여 서비스를 시작하면 보안 위험을 초래합니다. 인텐트에 어느 서비스가 응답할 것인지 확신할 수 없고, 사용자는 어느 서비스가 시작되는지 볼 수 없기 때문입니다. Android 5.0(API 레벨 21)부터 시스템은 개발자가 암시적 인텐트로 bindService()를 호출하면 예외를 발생시킵니다.

`android:exported` 특성을 포함하고 이를 false로 설정하면 서비스를 본인의 앱에만 사용 가능하게 할 수 있습니다. 이렇게 하면 다른 앱이 여러분의 서비스를 시작하지 못하도록 효과적으로 방지하며, 이는 명시적 인텐트를 사용하는 경우에도 문제없이 적용됩니다.

> 노트 : 사용자는 기기에서 어떤 서비스가 실행되는지 볼 수 있습니다. 정체를 모르거나 신뢰할 수 없는 서비스를 보면 사용자가 서비스를 중단할 수 있습니다. 사용자에 의해 우발적으로 서비스가 중단되는 불상사를 막으려면 앱 매니페스트의 `<service>` 요소에 android:description 특성을 추가해야 합니다. Description에 서비스가 하는 일과 서비스의 장점을 설명하는 간략한 문장을 기재하세요.

  

## 시작된 서비스 생성

시작된 서비스란 다른 component가 startService()를 호출하여 시작하고, 그 결과로 서비스의 onStartCommand() 메서드가 호출되는 경우를 말합니다.

서비스가 시작되면 이를 시작한 component와 독립적인 수명 주기를 가지게 됩니다. 서비스는 백그라운드에서 무한히 실행될 수 있으며, 이는 해당 서비스를 시작한 component가 소멸되었더라도 무관합니다. 따라서 서비스는 작업이 완료되면 stopSelf()를 호출하여 스스로 중단하거나 다른 component가 stopService()를 호출하여 중단시킬 수도 있습니다.

애플리케이션 component(예: 액티비티)가 서비스를 시작하려면 startService()를 호출하고, Intent를 전달하면 됩니다. 인텐트에서 서비스를 지정하고 이 서비스가 사용할 모든 데이터를 포함합니다. 서비스는 onStartCommand() 메서드에서 이 Intent를 받습니다.

예를 들어 어느 액티비티가 온라인 데이터베이스에 어떤 데이터를 저장해야 한다고 가정해보겠습니다. 액티비티가 동반 서비스를 시작하고, 인텐트를 startService()에 전달하여 서비스에 저장할 데이터를 전달할 수 있습니다. 서비스는 이 인텐트를 onStartCommand()에서 수신하고, 인터넷에 연결한 다음, 데이터베이스 트랜잭션을 수행합니다. 트랜잭션이 완료되면 서비스가 스스로 중단되어 소멸됩니다.

> 주의 : 서비스는 기본적으로 자신이 선언된 애플리케이션의 같은 프로세스에서 실행되기도 하고, 해당 애플리케이션의 메인 스레드에서 실행되기도 합니다. 따라서 사용자가 같은 애플리케이션의 액티비티와 상호작용하는 동안 서비스가 집약적이거나 차단 작업을 수행하는 경우, 해당 서비스 때문에 액티비티 성능이 느려지게 됩니다. 애플리케이션 성능에 영향을 미치는 것을 방지하려면, 서비스 내에서 새 스레드를 시작해야 합니다.

Service 클래스는 모든 서비스들의 base class 입니다. 이 클래스를 상속받을 때 서비스가 모든 작업을 완료할 수 있는 새 스레드를 생성하는 것이 중요합니다. 서비스는 application의 메인 스레드를 기본으로 사용하기 때문에 애플리케이션이 실행 중인 액티비티의 성능을 저하시킬 수 있습니다.

안드로이드 프레임워크는 또한 Service의 하위 클래스인 IntentService를 제공합니다. worker 스레드를 사용하여 모든 시작 요청을 한 번에 하나씩 처리합니다. 안드로이드 8 오레오(백그라운드 제한 도입)에서 제대로 동작하지 않기 때문에 이 클래스를 사용하는 건 **추천하지 않습니다.** 게다가 안드로이드 11부터 deprecated되었습니다. JobIntentService를 IntentService대신 사용할 수 있습니다. 이는 안드로이드의 새로운 버전과 호환됩니다.

다음 섹션은 어떻게 custom 서비스를 만드는지 설명합니다. 그러나 대부분의 사용 사례에서 WorkManager를 대신 사용하는 것을 강력하게 고려해야 합니다. [Android 백그라운드 처리에 대한 가이드](https://developer.android.com/guide/background)를 참조하십시오. 사용자의 요구에 맞는 솔루션이 있는지 확인합니다.

  

## Service class 상속

다가올 intent를 처리하기 위해 Service 클래스를 상속받을 수 있습니다. 기본 구현은 다음과 같습니다

```Kotlin
class HelloService : Service() {

    private var serviceLooper: Looper? = null
    private var serviceHandler: ServiceHandler? = null

    // Handler that receives messages from the thread
    private inner class ServiceHandler(looper: Looper) : Handler(looper) {

        override fun handleMessage(msg: Message) {
            // Normally we would do some work here, like download a file.
            // For our sample, we just sleep for 5 seconds.
            try {
                Thread.sleep(5000)
            } catch (e: InterruptedException) {
                // Restore interrupt status.
                Thread.currentThread().interrupt()
            }

            // Stop the service using the startId, so that we don't stop
            // the service in the middle of handling another job
            stopSelf(msg.arg1)
        }
    }

    override fun onCreate() {
        // Start up the thread running the service.  Note that we create a
        // separate thread because the service normally runs in the process's
        // main thread, which we don't want to block.  We also make it
        // background priority so CPU-intensive work will not disrupt our UI.
        HandlerThread("ServiceStartArguments", Process.THREAD_PRIORITY_BACKGROUND).apply {
            start()

            // Get the HandlerThread's Looper and use it for our Handler
            serviceLooper = looper
            serviceHandler = ServiceHandler(looper)
        }
    }

    override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
        Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show()

        // For each start request, send a message to start a job and deliver the
        // start ID so we know which request we're stopping when we finish the job
        serviceHandler?.obtainMessage()?.also { msg ->
            msg.arg1 = startId
            serviceHandler?.sendMessage(msg)
        }

        // If we get killed, after returning from here, restart
        return START_STICKY
    }

    override fun onBind(intent: Intent): IBinder? {
        // We don't provide binding, so return null
        return null
    }

    override fun onDestroy() {
        Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show()
    }
}
```

예시 코드는 각 호출을 onStartCommand()로 처리하고 백그라운드 스레드에서 동작 중인 Handler에게 작업을 게시합니다. IntentService처럼 작동하며 모든 요구 사항을 순서대로 연달아 처리합니다. 작업을 thread pool에서 동작하도록 코드를 수정할 수 있습니다. 예를 들어 여러 요구 사항을 동시에 작동시킬 수 있습니다.

onStartCommand()메서드가 반드시 정수를 반환함을 주의하세요. 이 정수는 이 서비스를 종료할 경우 서비스를 유지하는 방법을 설명하는 값입니다. onStartCommand() 로부터의 반환 값은 다음 상수 중 하나여야 합니다.

  

### [START\_NOT\_STICKY](https://developer.android.com/reference/android/app/Service?hl=ko#START_NOT_STICKY)

시스템이 서비스를 onStartCommand() 반환 후에 중단시키면 서비스를 재생성하면 안 됩니다. 다만 전달할 보류 인텐트가 있는 경우는 예외입니다. 이는 서비스가 불필요하게 실행되는 일을 피할 수 있는 가장 안전한 옵션이며, 애플리케이션이 완료되지 않은 모든 작업을 단순히 다시 시작할 수 있을 때 유용합니다.

  

### [START\_STICKY](https://developer.android.com/reference/android/app/Service#START_STICKY)

시스템이 onStartCommand() 반환 후에 서비스를 중단하면 서비스를 다시 생성하고 onStartCommand()를 호출하되, 마지막 인텐트는 전달하지 않습니다. 그 대신 시스템이 null 인텐트로 onStartCommand()를 호출합니다. 단, 서비스를 시작하기 위한 보류 인텐트가 있는 경우는 예외입니다. 이 경우에는 그러한 인텐트가 전달됩니다. 이것은 명령을 실행하지는 않지만 무한히 실행 중이며 작업을 기다리고 있는 미디어 플레이어(또는 그와 비슷한 서비스)에 적합합니다.

  

### [START\_REDELIVER\_INTENT](https://developer.android.com/reference/android/app/Service#START_REDELIVER_INTENT)

시스템이 onStartCommand() 반환 후에 서비스를 중단하는 경우, 서비스를 다시 생성하고 이 서비스에 전달된 마지막 인텐트로 onStartCommand()를 호출하세요. 모든 보류 인텐트가 차례로 전달됩니다. 이것은 즉시 재개되어야 하는 작업을 능동적으로 수행 중인 서비스(예: 파일 다운로드 등)에 적합합니다.

이 반환 값에 대한 자세한 내용은 각 상수에 대해 링크로 연결된 참조 문서를 확인하세요.

  

## 서비스 시작

액티비티 또는 다른 component에서 서비스를 시작하려면 Intent(시작할 서비스를 지정)를 startService() 또는 startForegroundService()에 전달하면 됩니다. Android 시스템이 서비스의 onStartCommand() 메서드를 호출하고 여기에 시작할 서비스를 지정하는 Intent를 전달합니다.

> 노트 : 앱이 API 레벨 26 이상을 대상으로 한다면 앱이 포그라운드에 있지 않을 때 시스템에서 백그라운드 서비스 사용하거나 생성하는 것에 제한을 적용합니다. 앱이 포그라운드 서비스를 생성해야 하는 경우, 해당 앱은 startForegroundService()를 호출해야 합니다. 이 메서드는 백그라운드 서비스를 생성하지만, 메서드가 시스템에 신호를 보내 서비스가 자체적으로 포그라운드로 승격될 것이라고 알립니다. 서비스가 생성되면 5초 이내에 startForeground() 메서드를 호출해야 합니다.

예를 들어 아래에서 볼 수 있듯이, 이전 섹션의 예시 서비스(HelloService)를 액티비티가 시작하려면 startService()로 명시적 인텐트를 사용하면 됩니다.

```Kotlin
Intent(this, HelloService::class.java).also { intent ->
    startService(intent)
}
```

startService() 메서드가 즉시 반환되며 Android 시스템이 서비스의 onStartCommand() 메서드를 호출합니다. 서비스가 아직 실행되지 않고 있다면 먼저 onCreate()를 호출한 다음, onStartCommand()를 호출해야 합니다.

서비스가 바인딩도 제공하지 않는 경우, startService()와 함께 전달된 인텐트가 애플리케이션 component와 서비스 사이의 유일한 통신 수단입니다. 그러나 서비스가 결과를 돌려보내기를 원하는 경우, 서비스를 시작한 클라이언트가 브로드캐스트를 위해 PendingIntent를 만들고(getBroadcast() 사용) 이를 서비스를 시작한 Intent의 서비스에 전달할 수 있습니다. 그러면 서비스가 이 브로드캐스트를 사용하여 결과를 전달할 수 있게 됩니다.

서비스를 시작하기 위한 요청을 여러 개 보내면 그에 대응하여 서비스의 onStartCommand()에 대해 여러 번 호출이 발생합니다. 하지만 서비스를 중단하려면 한 번만 중단을 요청하면 됩니다(stopSelf() 또는 stopService()).

  

## 서비스 중단

시작된 서비스는 자신의 수명 주기를 직접 관리해야 합니다. 다시 말해, 시스템이 서비스를 중단하거나 소멸시키지 않는다는 뜻입니다. 다만 시스템 메모리를 회복해야 하고 서비스가 onStartCommand() 반환 후에도 계속 실행되는 경우는 예외입니다. 서비스는 stopSelf()를 호출하여 스스로 중지하고 하고, 아니면 다른 component가 stopService()를 호출하여 이를 중지시킬 수 있습니다.

일단 stopSelf() 또는 stopService()로 중단 요청을 보내면 시스템은 가능한 한 빨리 서비스를 소멸시킵니다.

서비스가 onStartCommand()에 대한 여러 요청을 동시에 처리하는 경우에는, 시작 요청의 처리를 끝낸 뒤에도 서비스를 중단하면 안 됩니다. 그 이후 새 시작 요청을 받았을 수 있기 때문입니다(첫 요청이 끝날 때 중단하면 두 번째 요청이 종료될 수 있습니다). 이 문제를 피하려면, stopSelf(int)를 사용하여 서비스 중단 요청이 항상 가장 최근 시작 요청을 기준으로 하도록 해야 합니다. 다시 말해, stopSelf(int)를 호출할 경우 시작 요청의 ID(onStartCommand()에 전달된 startId)를 전달하며, 여기에 중단 요청이 대응됩니다. 그런 다음 stopSelf(int)를 호출할 수 있게 되기 전에 서비스가 새 시작 요청을 수신하면 ID가 일치하지 않으므로 서비스는 중단되지 않습니다.

> 주의 : 시스템 리소스 낭비를 피하고 배터리 전력 소모를 줄이기 위해 서비스의 작업이 완료되면 애플리케이션에서 서비스를 중단해야 합니다. 필요한 경우 stopService()를 호출하여 다른 component가 서비스를 중단할 수 있습니다. 서비스에 대해 바인딩을 활성화하더라도, 서비스가 onStartCommand()에 대한 호출을 한 번이라도 받았으면 항상 직접 서비스를 중단해야 합니다.

서비스의 수명 주기에 대한 자세한 내용은 아래의 서비스 수명 주기 관리 섹션을 참고하세요.

  

## 바인딩된 서비스 생성

바인딩된 서비스는 bindService()를 호출하여 애플리케이션 component를 서비스에 바인딩하고 오래 유지되는 연결을 설정하는 것입니다. 일반적으로는 startService()를 호출하더라도 component가 서비스를 시작하도록 허용하지 않습니다.

액티비티와 애플리케이션의 다른 component에서 서비스와 상호작용하기를 원하는 경우 바인딩된 서비스를 생성해야 합니다. 아니면 애플리케이션의 기능 몇 가지를 프로세스 간 통신(IPC)을 통해 다른 애플리케이션에 노출하고자 하는 경우에도 좋습니다.

바인딩된 서비스를 생성하려면 onBind() 콜백 메서드를 구현하여 서비스와의 통신을 위한 인터페이스를 정의하는 IBinder를 반환하도록 해야 합니다. 그러면 다른 애플리케이션 component가 bindService()를 호출하여 해당 인터페이스를 검색하고, 서비스에 있는 메서드를 호출하기 시작할 수 있습니다. 서비스는 자신에게 바인딩된 애플리케이션 component에 도움이 되기 위해서만 존재하는 것이므로, 서비스에 바인딩된 component가 없으면 시스템이 이를 소멸시킵니다. 바인딩된 서비스는 서비스를 onStartCommand()를 통해 시작했을 때와 같은 방식으로 중단하지 않아도 됩니다.

바인딩된 서비스를 생성하려면 클라이언트가 서비스와 통신할 수 있는 방법을 나타내는 인터페이스를 정의해야 합니다. 서비스와 클라이언트 사이에서 쓰이는 이 인터페이스는 반드시 IBinder의 구현이어야 하며 이를 서비스가 onBind() 콜백 메서드에서 반환해야 합니다. 클라이언트가 IBinder를 수신하면 해당 인터페이스를 통해 서비스와 상호작용을 시작할 수 있습니다.

여러 클라이언트가 서비스에 한꺼번에 바인딩될 수 있습니다. 클라이언트가 서비스와의 상호작용을 완료하면 이는 unbindService()를 호출하여 바인딩을 해제합니다. 서비스에 바인딩된 클라이언트가 하나도 없으면 시스템이 해당 서비스를 소멸시킵니다.

바인딩된 서비스를 구현하는 데에는 여러 가지 방법이 있으며 그러한 구현은 시작된 서비스보다 훨씬 복잡합니다. 따라서 바인딩된 서비스에 대한 설명은 [바인딩된 서비스](https://developer.android.com/guide/components/bound-services)에 관한 별도의 문서에서 다룹니다.

  

## 사용자에게 알림 전송

서비스가 실행되고 있을 때 사용자에게 스낵바 알림 또는 상태 표시줄 알림을 이용해 이벤트를 알릴 수 있습니다.

스낵바 알림은 현재 창의 표면에 잠간 나타났다가 사라지는 메세지입니다. 상태 표시줄 알림은 상태 표시줄에 메세지가 담긴 아이콘을 제공하여 사용자가 이를 선택하여 조치를 취할 수 있게 하는 것입니다. (예 : 액티비티 시작)

일반적으로 일종의 백그라운드 작업이 완료되었고(예: 파일 다운로드 완료) 이제 사용자가 그에 대해 조치를 취할 수 있는 경우에는 상태 표시줄 알림을 사용하는 것이 가장 좋습니다. 사용자가 확장된 뷰의 알림을 선택하면, 해당 알림이 액티비티를 시작할 수 있습니다(예: 다운로드한 파일 보기).

  

## 서비스 수명 주기 관리

서비스의 수명 주기는 액티비티의 수명 주기보다 훨씬 간단합니다. 하지만 서비스를 생성하고 소멸하는 방법에 특히 주의를 기울여야 한다는 면에서 중요도는 이쪽이 더 높습니다. 서비스는 사용자가 모르는 채로 백그라운드에서 실행될 수 있기 때문입니다.

서비스 수명 주기(생성된 시점부터 소멸되는 시점까지)는 두 가지 서로 다른 경로를 따를 수 있습니다.

시작된 서비스 다른 component가 startService()를 호출하면 서비스가 생성됩니다. 그러면 서비스가 무기한으로 실행될 수 있으며, stopSelf()를 호출하여 자체적으로 중단해야 합니다. 다른 component도 stopService()를 호출하면 서비스를 중단시킬 수 있습니다. 서비스가 중단되면 시스템이 이를 소멸시킵니다.

바인딩된 서비스 다른 component(클라이언트)가 bindService()를 호출하면 서비스가 생성됩니다. 그러면 클라이언트가 IBinder 인터페이스를 통해 서비스와 통신을 주고받을 수 있습니다. 클라이언트가 연결을 종료하려면 unbindService()를 호출하면 됩니다. 여러 클라이언트가 같은 서비스에 바인딩될 수 있으며, 모든 클라이언트가 바인딩을 해제하면 시스템이 해당 서비스를 소멸시킵니다 서비스가 스스로 중단하지 않아도 됩니다.

이 두 가지 경로는 완전히 별개는 아닙니다. 이미 startService()로 시작된 서비스에 바인딩할 수도 있습니다. 예를 들어 재생할 음악을 식별하는 Intent를 포함해 startService()를 호출하면 백그라운드 음악 서비스를 시작할 수 있습니다. 나중에 아마도 사용자가 플레이어에 좀 더 많은 통제력을 발휘하려고 하거나 현재 노래에 대한 정보를 얻고자 할 때, 액티비티가 bindService()를 호출하여 서비스에 바인딩할 수 있습니다. 이와 같은 경우에는 모든 클라이언트가 바인딩을 해제할 때까지 stopService() 또는 stopSelf()가 서비스를 중단하지 않습니다.

  

### 수명 주기 콜백 구현

액티비티와 마찬가지로 서비스에도 수명 주기 콜백 메서드가 있어, 이를 구현하면 서비스의 상태 변경 내용을 모니터할 수 있고 적절한 시기에 작업을 수행할 수 있습니다. 다음의 기본 서비스는 각 수명 주기 메서드를 보여줍니다.

```kotlin
class ExampleService : Service() {
    private var startMode: Int = 0             // indicates how to behave if the service is killed
    private var binder: IBinder? = null        // interface for clients that bind
    private var allowRebind: Boolean = false   // indicates whether onRebind should be used

    override fun onCreate() {
        // The service is being created
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // The service is starting, due to a call to startService()
        return startMode
    }

    override fun onBind(intent: Intent): IBinder? {
        // A client is binding to the service with bindService()
        return binder
    }

    override fun onUnbind(intent: Intent): Boolean {
        // All clients have unbound with unbindService()
        return allowRebind
    }

    override fun onRebind(intent: Intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }

    override fun onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
```

> 노트 : 액티비티 수명 주기 콜백 메서드와는 달리 이와 같은 콜백 메서드를 구현하는 데 슈퍼클래스 구현을 호출하지 않아도 됩니다.

![https://developer.android.com/static/images/service_lifecycle.png](https://developer.android.com/static/images/service_lifecycle.png align="left")

그림 2. 서비스 수명 주기. 왼쪽의 다이어그램은 서비스가 startService()로 생성된 경우의 수명 주기를 나타내며 오른쪽의 다이어그램은 서비스가 bindService()로 생성된 경우의 수명 주기를 나타냅니다.

그림 2는 서비스에 대한 일반적인 콜백 메서드를 나타낸 것입니다. 이 그림에서는 startService()로 생성된 서비스와 bindService()로 생성된 서비스를 구분하고 있지만, 어떤 식으로 시작되었든 모든 서비스는 클라이언트와 바인딩되도록 허용할 수 있다는 점을 명심하세요. onStartCommand()로 처음 시작된 서비스(클라이언트가 startService()호출하는 경우)라고 해도 여전히 onBind()에 대한 호출을 받을 수 있습니다(클라이언트가 bindService()를 호출하는 경우).

이와 같은 메서드를 구현함으로써, 서비스 수명 주기의 두 가지 중첩된 루프를 모니터링할 수 있습니다.

* 서비스의 전체 수명은 onCreate()가 호출된 시점부터 onDestroy() 반환 시점까지입니다. 액티비티와 마찬가지로 서비스는 자신의 초기 설정을 onCreate()에서 수행하며, 남은 리소스를 모두 onDestroy()에서 릴리스합니다. 예를 들어 음악 재생 서비스는 스레드를 생성하고, 이 스레드의 onCreate()에서 음악이 재생됩니다. 그런 다음, onDestroy()에서 스레드를 중단할 수 있습니다.
    

> 노트 : onCreate() 및 onDestroy() 메서드는 모든 서비스에 대해 호출됩니다. 이는 서비스가 startService()로 생성되었든 bindService()로 생성되었든 상관없이 적용됩니다.

* 서비스의 활성 수명은 onStartCommand() 또는 onBind()에 대한 호출에서부터 시작됩니다. 각 메서드는 Intent를 받아서 startService() 또는 bindService()에 전달합니다. 서비스가 시작되면 수명 주기 전체가 종료되는 것과 동시에 활성 수명 주기도 종료됩니다(서비스는 onStartCommand()가 반환된 뒤에도 여전히 활성 상태입니다). 서비스가 바인딩된 경우, onUnbind()가 반환되면 활성 수명 주기가 종료됩니다.
    

> 노트 : 시작된 서비스를 중단하려면 stopSelf() 또는 stopService()를 호출하면 되지만, 서비스에 대한 개별 콜백은 없습니다(즉 onStop() 콜백이 없습니다). 그러므로 서비스가 클라이언트에 바인딩되어 있지 않은 한, 시스템은 서비스가 중단되면 이를 소멸시킵니다. 수신되는 콜백은 onDestroy()가 유일합니다.

바인딩을 제공하는 서비스 생성에 대한 자세한 내용은 [바인딩된 서비스](https://developer.android.com/guide/components/bound-services) 문서를 참조하세요. [바인딩된 서비스의 수명 주기 관리](https://developer.android.com/guide/components/bound-services#Lifecycle)에 대한 섹션에 onRebind()에 관한 자세한 정보가 있습니다.