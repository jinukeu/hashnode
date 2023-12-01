---
title: "[Android] 이벤트 처리 Channel vs SharedFlow"
datePublished: Fri Dec 01 2023 06:19:45 GMT+0000 (Coordinated Universal Time)
cuid: clpm8j1pw00020alf1qc1ds16
slug: android-channel-vs-sharedflow
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/_QRor5GK3po/upload/25c4edb7162de97eb1b7716e2e1f6046.jpeg
tags: android, kotlin, channel, sharedflow

---

> 안드로이드에서 이벤트(사이드 이펙트)는 주로 channel 또는 sharedFlow를 사용해 처리한다.

#### channel을 이용한 이벤트 처리 예시 코드

```kotlin
private val _effect: Channel<A> = Channel()
val effect = _effect.receiveAsFlow()
```

#### sharedFlow를 이용한 이벤트 처리 예시 코드

```kotlin
private val _sideEffectFlow: MutableSharedFlow<SE>
val sideEffectFlow: SharedFlow<SE> = _sideEffectFlow.asSharedFlow()
```

그렇다면 `SharedFlow`와 `Channel`을 사용한 이벤트 처리의 차이점은 뭘까?

각각의 장단점에 대해 알아보자.

# Channel

## 장단점

장점 : 백그라운드에서 발생한 이벤트도 수집 가능

단점 : 여러 개의 구독자를 가지기에는 적합하지 않음

## 테스트 1 - 백그라운드

### MainViewModel

```kotlin
class MainViewModel : ViewModel() {
    private val _channel = Channel<Int>()
    val channel = _channel.receiveAsFlow()

    init {
        viewModelScope.launch {
            repeat(100) {
                Log.d("Channel", "MainViewModel - Send $it")
                _channel.send(it)
                delay(1000)
            }
        }
    }
}
```

### MainActivity

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.channel.collect { number ->
                    Log.d("Channel","MainActivity - Collected $number from channel")
                }
            }
        }
        // ...
}
```

### 결과

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701356938622/3562a630-b753-4bce-b8e1-e03d4205339d.png align="center")

1. MainActivity - onStop (백그라운드 진입)
    
2. MainViewModel - channel send 7 (백그라운드에서 send)
    
3. MainActivity - onStart (포그라운드 진입)
    
4. MainActivity - collect 7 (포그라운드에서 collect)
    

백그라운드에서 send한 7을 올바르게 collect한 모습을 볼 수 있다.

이게 가능한 이유는 `channel`의 `send()`는 [channel의 버퍼가 꽉 차있거나 존재하지 않으면 suspend되기 때문이다.](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/send.html) (suspending the caller while the buffer of this channel is full or if it does not exist, or throws an exception if the channel [is closed for `send`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/is-closed-for-send.html) (see [close](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/close.html) for details).)

## 테스트 2 - 여러 개의 구독자를 가지는 경우

### MainActivity

```kotlin
lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.channel.collect { number ->
                    Log.d("Subscriber","Subscriber[1] - Collected $number from channel")
                }
            }
        }

lifecycleScope.launch {
        repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.channel.collect { number ->
                Log.d("Subscriber","Subscriber[2] - Collected $number from channel")
            }
        }
    }
```

### 결과

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701315939435/30568252-eef4-4fd1-8698-74801b5558c9.png align="center")

Channel에서 여러 개의 구독자가 있다면, 각각의 구독자가 번갈아가면서 collect하게 된다.

1. channel - send 1
    
2. subscriber\[1\] - collect 1
    
3. channel - send 2
    
4. subscriber\[2\] - collect 2
    

[공식 문서](https://kotlinlang.org/docs/channels.html#channels-are-fair)에 따르면 `Channel`은 평등하기 때문이다.

따라서 여러 개의 구독자를 가지는 경우, 각각의 구독자가 같은 이벤트를 수신하지 않으므로 `Channel`보다는 `SharedFlow`가 더 적합하다.

예를 들어, 앱 전체에 tick을 보내서 정기적으로 앱의 데이터를 refresh 해야한다면 `Channel`을 사용하는 것은 부적합하다.

# sharedFlow

장점 : 여러 개의 구독자를 가질 수 있음

단점 : 백그라운드에서 발생한 이벤트 수집 불가능

## 테스트 1 - 백그라운드

### MainActivity

```kotlin
lifecycleScope.launch {
    repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.sharedFlow.collect { number ->
            Log.d("SharedFlow","MainActivity - Collected $number from sharedFlow")
        }
    }
}
```

### 결과

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701357173634/28420e06-e990-4d0f-90b3-2e4e5935f96a.png align="center")

1. MainActivity - onStop (백그라운드 진입)
    
2. MainViewModel - sharedFlow emit 8, 9, 10, 11 (백그라운드에서 emit)
    
3. MainActivity - onStart (포그라운드 진입)
    
4. MainActivity - collect 12 (포그라운드에서 collect) - 8, 9, 10, 11은 유실됨
    

백그라운드에서 emit한 8, 9, 10, 11은 유실된 것을 확인할 수 있다.

## 테스트 2 - 여러 개의 구독자를 가지는 경우

### MainActivity

```kotlin
lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.sharedFlow.collect { number ->
                    Log.d("Subscriber","Subscriber[1] - Collected $number from sharedFlow")
                }
            }
        }

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.sharedFlow.collect { number ->
                    Log.d("Subscriber","Subscriber[2] - Collected $number from sharedFlow")
                }
            }
        }
```

### 결과

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410032713/c24cc5b9-00ba-4e78-9bca-b6b5b9d85df7.png align="center")

여러 개의 구독자를 가져도 모든 구독자가 같은 이벤트를 collect하는 것을 확인할 수 있다.

## 단점 극복 방법

`sharedFlow`의 단점은 [MVVM의 ViewModel에서 이벤트를 처리하는 방법 6가지](https://medium.com/prnd/mvvm%EC%9D%98-viewmodel%EC%97%90%EC%84%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8%EB%A5%BC-%EC%B2%98%EB%A6%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-6%EA%B0%80%EC%A7%80-31bb183a88ce)를 통해 해결할 수 있지만 ... 이러면 여러 개의 구독자를 가질 수 없게 된다. 이게 무슨 말일까? 자세히 알아보자. (그 전에 [MVVM의 ViewModel에서 이벤트를 처리하는 방법 6가지](https://medium.com/prnd/mvvm%EC%9D%98-viewmodel%EC%97%90%EC%84%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8%EB%A5%BC-%EC%B2%98%EB%A6%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-6%EA%B0%80%EC%A7%80-31bb183a88ce) 포스팅을 읽고 오자.)

### 문제 발생 시나리오

`event` 객체가 있고 이를 `AFragment`, `BFragment`에서 `collect`하고 있다고 가정하자.

1.`event`가 `emit`되면 `AFragment`, `BFragment`에서 `collect` 된다. (근소한 차이로 `AFragment`에서 먼저 `collect` 되었다고 가정)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410405436/eb1f7456-9d94-4f1d-b848-c931c6856f06.png align="center")

2.이때 `AFragment`에서 `event`의 `comsumed`는 `true`가 된다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410458010/03f11d34-b3ec-4370-a0f1-6d3db0571808.png align="center")

3.그 이후 `BFragment`에서 `event`가 `collect`되어야하지만, `event`는 이미 `comsumed`되었기 때문에 `collect`되지 않는다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410489299/a6103a2d-7364-4210-9d61-acedf7a25daf.png align="center")

### 해결 방안

`slotStore`라는 `HashMap`을 만들었다.

`slotStore`의 `key`에는 현재 `collect`하고 있는 `collector`의 이름과 `slot`의 `toString()`값이 들어간다.  
`value`에는 `event`와 동일한 값을 가지고 있는 새로운 `event`가 들어간다.

아까와 마찬가지로 `event` 객체가 있고 이를 `AFragment`, `BFragment`에서 `collect`하고 있다고 가정한다.

1\. `event`가 `emit`되면 `AFragment`, `BFragment`에서 `collect` 된다. (근소한 차이로 `AFragment`에서 먼저 `collect` 되었다고 가정)

2.이때 `slotStore`에 `{ AFragment + event.toString() : Event(event.value) }`와 같은 값이 저장된다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410632726/57ab9c44-5f57-40c9-a944-db1e56b3a855.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410648176/cf29dc4d-3812-4e45-a3ee-21f56a494f75.png align="center")

3.그 이후 `slotStore`의 `key` 값이 `AFragment + event.toString()`인 `value`의 `consumed`는 `true`가 된다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410675138/144be55b-b4b6-47d3-a266-3a0af1554ed8.png align="center")

4.`BFragment`에서도 동일한 동작이 수행된다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410701358/84185d24-bcce-4c8d-b088-eb7571d7f4da.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410715738/5fa72e7e-2f67-4b91-851a-918ff78574e0.png align="center")

5.`collect` 이후 `BFragment`의 `event`는 `comsumed`처리 된다.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1701410735751/9ff719bc-0162-45a2-9865-795b005136a9.png align="center")

위 방법을 사용하여 2개 이상의 구독자를 가진 `EventFlow`가 `emit`되면 단 하나의 구독자만 `collect`하는 문제를 해결했다.  
`slotStore`를 사용한 코드는 [여기](https://github.com/jinukeu/jinukeu/issues/2#issue-1843226523)서 볼 수 있다.

### 해결방안 - 개선

`Event` 객체가 더 이상 사용되지 않아도 `slotStore`에서 계속 참조를 하고 있기 때문에 GC되지 않는 현상이 있었다. `ArrayDeque`를 사용하여 최대 20개까지의 `EventFlowSlot`을 저장하게끔 구현했다. 최종 코드는 다음과 같다.

```kotlin
private class EventFlowImpl<T>(
    replay: Int
) : MutableEventFlow<T> {

    private val flow: MutableSharedFlow<EventFlowSlot<T>> = MutableSharedFlow(replay = replay)

    private val slotStore: ArrayDeque<Slot<EventFlowSlot<T>>> = ArrayDeque()

    @InternalCoroutinesApi
    override suspend fun collect(collector: FlowCollector<T>) = flow
        .collect { slot ->

            val slotKey = collector.javaClass.name + slot

            if(isContainKey(slotKey)) {
                if(slotStore.size > MAX_CACHE_EVENT_SIZE) slotStore.removeFirst()
                slotStore.addLast(Slot(slotKey, EventFlowSlot(slot.value)))
            }

            val slotValue = slotStore.find { it.key == slotKey }?.value ?: slot

            if (slotValue.markConsumed().not()) {
                collector.emit(slotValue.value)
            }
        }

    override suspend fun emit(value: T) {
        flow.emit(EventFlowSlot(value))
    }

    fun isContainKey(findKey: String): Boolean {
        return slotStore.find { it.key == findKey } == null
    }
}

private data class Slot<T>(
    val key: String,
    val value: T
)
```

---

## 결론

> 이벤트(side-effect)는 보통 한 곳에서만 처리를 한다.

`sharedFlow`를 사용하여 백그라운드에서 발생한 이벤트를 처리하면 `eventFlow`를 만들어줘야한다. `eventFlow`를 만드는 것 자체만으로 별도의 코드를 추가해야한다는 단점이 있다. 또 `eventFlow`를 만드는 순간 `sharedFlow`의 장점(여러 개의 구독자를 가질 수 있음)이 사라진다. 이것 또한 위에서 설명한 방식으로 해결할 수 있다. 하지만 별도의 코드를 추가해야하며 코드를 쉽게 이해하기 어렵다.

따라서 이벤트는 보통 한 곳에서만 처리를 하므로 `channel`을 사용하여 이벤트를 수신하는게 가장 코드를 덜 작성하고 비교적 쉽게 이해할 수 있다.

필요한 경우에만 `sharedFlow`를 사용하는 것이 좋다 생각한다.

### 참고

[\[Kotlin\] Coroutine의 SharedFlow 와 Channel](https://velog.io/@morning-la/Kotlin-Coroutine%EC%9D%98-SharedFlow-%EC%99%80-Channel)

[StateFlow vs SharedFlow 를 비교해보자 #이벤트 핸들링](https://developer88.tistory.com/entry/StateFlow-vs-SharedFlow-%EB%A5%BC-%EB%B9%84%EA%B5%90%ED%95%B4%EB%B3%B4%EC%9E%90-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%ED%95%B8%EB%93%A4%EB%A7%81)

[MVVM의 ViewModel에서 이벤트를 처리하는 방법 6가지](https://medium.com/prnd/mvvm%EC%9D%98-viewmodel%EC%97%90%EC%84%9C-%EC%9D%B4%EB%B2%A4%ED%8A%B8%EB%A5%BC-%EC%B2%98%EB%A6%AC%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-6%EA%B0%80%EC%A7%80-31bb183a88ce)