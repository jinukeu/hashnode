---
title: "[Kotlin in Action] 부록 E. 코루틴과 Async/Await"
datePublished: Thu Oct 12 2023 02:24:40 GMT+0000 (Coordinated Universal Time)
cuid: clnmk44gx000009jv8oie737d
slug: kotlin-in-action-e-asyncawait
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696900307647/e0b3ec8d-a2d3-46d5-af18-b6d9f25938d6.jpeg
tags: kotlin

---

## 코루틴이란?

> 코루틴은 컴퓨터 프로그램 구성 요소 중 하나로 비선점형 멀티태스킹을 수행하는 일반화한 서브루틴이다. 코루틴은 실행을 일시 중단하고 재개할 수 있는 여러 진입 지점을 허용한다.

서브루틴 == 함수

비선점형 멀티태스킹 : 멀티태스킹의 각 작업을 수행하는 참여자들의 실행을 운영체제가 강제로 일시 중단시키고 다른 참여자를 실행하게 만들 수 없다는 뜻

어떤 함수 A가 실행되다가 제네레이터인 코루틴 B를 호출하면 A가 실행되던 스레드 안에서 코루틴 B의 실행이 시작된다. 코루틴 B는 실행을 진행하다가 실행을 A에 양보한다. A는 다시 코루틴을 호출했던 **바로 다음 부분부터** 실행을 계속 진행하다가 또 코루틴 B를 호출한다.

B가 일반적인 함수라면 로컬 변수를 초기화하면서 처음부터 실행을 다시 시작하겠지만, 코루틴이면 이전에 **실행을 양보했던 지점부터** 실행을 계속하게 된다.

## 코틀린의 코루틴 지원: 일반적인 코루틴

### 여러 가지 코루틴

#### kotlinx.coroutines.CoroutineScope.launch

launch는 코루틴을 Job으로 반환하며, 만들어진 코루틴은 기본적으로 즉시 실행된다.

#### kotlinx.coroutines.CoroutineScope.async

async는 사실상 launch와 같은 일을 한다. 유일한 차이는 launch가 Job을 반환하는 반면 async는 Deffered를 반환한다는 점뿐이다.

Deffered와 Job의 차이는, Job은 아무 타입 파라미터가 없는데 Deffered는 타입 파라미터가 있는 제네릭 타입이라는 점과 Deffered 안에는 await() 함수가 정의돼 있다는 점이다.

Deffered의 타입 파라미터는 바로 Deffered 코루틴이 계산을 하고 돌려주는 값의 타입이다.

```kotlin
fun sumAll() {
    runBlocking {
        val d1 = async { delay(1000L); 1 }
        log("after async(d1)")
        val d2 = async { delay(2000L); 2 }
        log("after async(d2)")
        val d3 = async { delay(3000L); 3 }
        log("after async(d3)")

        log("1+2+3 = ${d1.await() + d2.await() + d3.aait()}")
        log("after await all & add")
    }
}
```

실행 결과

```kotlin
00:46:45.405:Thread[main, 5, main]: after async(d1)
00:46:45.409:Thread[main, 5, main]: after async(d2)
00:46:45.409:Thread[main, 5, main]: after async(d3)
00:46:48.417:Thread[main, 5, main]: 1+2+3 = 6
00:46:48.418:Thread[main, 5, main]: after await all & add
```

d1, d2, d3를 하나하나 순서래도 실행하면 6초 이상이 걸리지만, 여기서는 3초가 걸린걸 볼 수 있다.

그럼에도 불구하고 스레드를 여럿 사용하는 병렬 처리와 달리 **모든 async 함수들이 메인 스레드 안에서 실행됨**을 볼 수 있다.

실행하려는 작업이 시간이 얼마 걸리지 않거나 I/O에 의한 대기 시간이 크고, CPU 코어 수가 작아 동시에 실행할 수 있는 스레드 개수가 한정된 경우에는 특히 코루틴과 일반 스레드를 사용한 비동기 처리 사이에 차이가 커진다.

### 코루틴 컨텍스트와 디스패처

CoroutineContext는 실제로 코루틴이 실행 중인 여러 작업(Job 타입)과 디스패처를 저장하는 일종의 맵이라 할 수 있다. 코틀린 런타임은 이 CoroutineContext를 사용해서 다음에 실행할 작업을 선정하고, 어떻게 스레드를 배정할지 대한 방법을 결정한다.

```kotlin
launch {}
launch(Dispatchers.Default) {}
```

### 코루틴 빌더와 일시 중단 함수

launch, async, runBlocking 이외에도 코루틴 빌더는 2가지가 더 있다.

> produce: 정해진 채널로 데이터를 스트림으로 보내는 코루틴을 만든다. 이 함수는 ReceiveChannel&lt;&gt;을 반환한다. 그 채널로부터 메시지를 전달받아 사용할 수 있다.
> 
> actor: 정해진 채널로 메세지를 받아 처리하는 액터를 코루틴으로 만든다. 이 함수가 반환하는 SendChannel&lt;&gt; 채널의 send() 메서드를 통해 액터에게 메시지를 보낼 수 있다.

## suspend 키워드와 코틀린의 일시 중단 함수 컴파일 방법

suspend 함수는 어떻게 동작할까? 예를 들어 일시 중단 함수 안에서 yield()를 해야 하는 경우 어떤 동작이 필요할지 생각해보자.

* 코루틴에 진입할 때와 코루틴에서 나갈 때 코루틴이 실행 중이던 상태를 저장하고 복구하는 등의 작업을 할 수 있어야 한다.
    
* 현재 실행 중이던 위치를 저장하고 다시 코루틴이 재개될 때 해당 위치부터 실행을 재개할 수 있어야 한다.
    
* 다음에 어떤 코루틴을 실행할지 결정한다.
    

이 세 가지 동작 중 마지막 동작은 코루틴 컨텍스트에 있는 디스패처에 의해 수행된다. 일시 중단 함수를 컴파일하는 컴파일러는 앞의 두 가지 작업을 할 수 있는 코드를 생성해 내야 한다.

이때 코틀린은 Continuation Passing Style(CPS)과 State Machine를 활용해 코드를 생성해낸다.

CPS 변환은 프로그램의 실행 중 특정 시점 이후에 진행해야 하는 내용을 별도의 함수(Continuation 이라 부른다.)로 뽑고 그 함수에게 현재 시점 까지 실행한 결과를 넘겨서 처리하게 만드는 소스코드 변환 기술이다.

## 코루틴 빌더 만들기

skip ...