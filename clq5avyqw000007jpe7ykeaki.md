---
title: "[Android] rememberUpdatedState 완벽 이해"
datePublished: Thu Dec 14 2023 14:33:25 GMT+0000 (Coordinated Universal Time)
cuid: clq5avyqw000007jpe7ykeaki
slug: android-rememberupdatedstate
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/ktpymCAIvGc/upload/18f6ae11634822646df7c0a9b1e62e0f.jpeg
tags: compose, android

---

# rememberUpdatedState

## 정의

[공식 문서](https://developer.android.com/jetpack/compose/side-effects?hl=ko#rememberupdatedstate)에는 다음과 같이 적혀있다.

> 값이 변경되는 경우 다시 시작되지 않아야 하는 효과(Effect)에서 값 참조

포스팅을 정리하면서 정의한 `rememberUpdateState`는 아래와 같다.

> `remember`는 [초기 컴포지션에서만 값을 저장](https://developer.android.com/jetpack/compose/state?hl=ko#state-in-composables)하고 리컴포지션 때 들어온 값은 저장하지 않는다. 리컴포지션 때 들어온 값도 저장하고 싶을 때 `rememberUpdateState`를 사용한다.

이게 도대체 무슨 말일까? 아래 퀴즈를 풀면서 이해해보자.

## 퀴즈

아래 코드를 보고 2초 후에 보여질 Screen을 맞춰보세요! (Splash Screen? CraneHome?)

```kotlin
/*
* 2초 후에 onTimeout 실행
**/
@Composable
fun SplashScreen(onTimeout: () -> Unit) {
    Box(modifier = modifier.fillMaxSize()) {
        LaunchedEffect(Unit) {
            delay(2000)
            onTimeout()
        }
        Image(
            painterResource(id = R.drawable.ic_crane_drawer), 
            contentDescription = null
        )
    }
}

/*
* 빈 람다 {}인 onTimeout을 SpalshScreen에 넘겨준다.
* 0.5초 뒤 onTimeout을 showLandingScreen을 false로 만드는 람다로 변경한다.
**/
@Composable
private fun MainScreen(onExploreItemClicked: OnExploreItemClicked) {
    Surface(color = MaterialTheme.colors.primary) {
        var showLandingScreen by remember {
            mutableStateOf(true)
        }

        var onTimeout: () -> Unit by remember {
            mutableStateOf({ /* empty */ })
        }

        if (showLandingScreen) {
            SplashScreen(onTimeout = onTimeout) // onTimeout = {}

            LaunchedEffect(key1 = true) {
                delay(500L)
                onTimeout = { showLandingScreen = false } // SplashScreen 리컴포지션 발생
            }
        } else {
            CraneHome(onExploreItemClicked = onExploreItemClicked)
        }
    }
}
```

.

.

.

.

.

정답은 **SplashScreen**

왜 이런 결과가 나왔을까? 또 CreanHome이 보이게 하려면 어떻게 해야할까? 차근 차근 알아보자.

위 코드의 동작 순서는 다음과 같다.

1. MainScreen - SplashScreen(onTimeOut) 실행 - `onTimeout = { /* empty */ }`
    
2. SplashScreen 2초간 대기 중 ...
    
3. 0.5초 이후 MainScreen - `onTimeout = { showLandingScreen = false }`
    
4. MainScreen - SplashScreen(onTimeOut) 리컴포지션 - `onTimeout = { showLandingScreen = false }`
    
5. 2초가 지나 SpalshScreen의 onTimeout이 실행되고 showLandingScreen이 false가 되어 SpalshScreen이 보이면 안되지만 여전히 보이는 문제 발생
    

왜 showLandingScreen의 값이 false가 되지 않았을까? 그 이유는 LaunchedEffect는 리컴포지션이 일어나도 재실행되지 않으며 오직 key가 바뀐 경우에만 재실행되기 때문이다.  
따라서 onTimeout이 0.5초 이후 값이 바뀌더라도 LaunchedEffect안의 onTimeout은 바뀌기 전의 값( `{ /* empty */ }` )이 있는 것이다.

### 해결방법

```kotlin
data class Refer(
    var onTimeout: () -> Unit,
)

val refer = Refer {}

@Composable
fun SplashScreen(onTimeout: () -> Unit, modifier: Modifier = Modifier) {
    Box(modifier = modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        // Adds composition consistency. Use the value when LaunchedEffect is first called
        refer.onTimeout = onTimeout

        LaunchedEffect(Unit) {
            delay(SplashWaitTime)
            refer.onTimeout()
        }

        Image(painterResource(id = R.drawable.ic_crane_drawer), contentDescription = null)
    }
}
```

onTimeout을 참조로 만들면 된다.

위와 같이 onTimeout을 가지고 있는 객체를 SplashScreen 밖에 선언해두고 LaunchedEffect 내부에서는 해당 객체의 onTimeout을 실행하면 된다.

그러면 `remember`를 활용하면 어떨까?

```kotlin
@Composable
inline fun <T> remember(crossinline calculation: @DisallowComposableCalls () -> T): T =
    currentComposer.cache(false, calculation)
```

remember 구현은 위와 같이 되어있고 ...

```kotlin
@ComposeCompilerApi
inline fun <T> Composer.cache(invalid: Boolean, block: @DisallowComposableCalls () -> T): T {
    @Suppress("UNCHECKED_CAST")
    return rememberedValue().let {
        if (invalid || it === Composer.Empty) {
            val value = block()
            updateRememberedValue(value)
            value
        } else it
    } as T
}
```

Composer.cache 구현은 위와 같다.

그러니까 remember { } 블록의 값을 cache 해놓는다는 것이다.

onTimeout을 캐시해서 사용하는 것과 참조해서 사용하는 것 둘다 동일한 동작을 할 것으로 예상된다.

```kotlin
@Composable
fun SplashScreen(onTimeout: () -> Unit, modifier: Modifier = Modifier) {
    Box(modifier = modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        // Adds composition consistency. Use the value when LaunchedEffect is first called
        // 정상 동작
        val currentOnTimeout by remember { mutableStateOf(onTimeout) }

        LaunchedEffect(Unit) {
            delay(SplashWaitTime)
            currentOnTimeout()
        }

        Image(painterResource(id = R.drawable.ic_crane_drawer), contentDescription = null)
    }
}
```

하지만 ... 정상 작동하지 않는다.

왜냐하면 remember는 **초기** 컴포지션이 발생할 때에만 계산된 값을 저장하기 때문이다.

> Composable functions can use the remember API to store an object in memory. A value computed by remember is stored in the Composition during initial composition, and the stored value is returned during recomposition. remember can be used to store both mutable and immutable objects. - [공식 문서](https://developer.android.com/jetpack/compose/state?hl=ko#state-in-composables)

그러니까 초기 컴포지션에서 { /\* empty \*/ } 값은 계산되지만 recomposition때 들어온 { showLandingScreen = false } 값은 remember에 저장되지 않기 때문이다.

이를 해결하려면 `rememberUpdatedState`를 사용해야한다. 코드는 아래와 같다.

```kotlin
@Composable
fun <T> rememberUpdatedState(newValue: T): State<T> = remember {
    mutableStateOf(newValue)
}.apply { value = newValue }
```

리컴포지션때 새로운 값이 들어와도 remember 내부의 State에 이를 반영하도록 .apply { value = newValue } 를 사용한 것이다.

최종 해결 코드는 아래와 같다.

```kotlin
@Composable
fun SplashScreen(onTimeout: () -> Unit, modifier: Modifier = Modifier) {
    Box(modifier = modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        // Adds composition consistency. Use the value when LaunchedEffect is first called
        val currentOnTimeout by rememberUpdatedState(onTimeout)

        LaunchedEffect(Unit) {
            delay(SplashWaitTime)
            currentOnTimeout()
        }

        Image(painterResource(id = R.drawable.ic_crane_drawer), contentDescription = null)
    }
}
```

`rememberUpdatedState`는 그래서 이렇게 말할 수 있다.

> `remember`는 [초기 컴포지션에서만 값을 저장](https://developer.android.com/jetpack/compose/state?hl=ko#state-in-composables)하고 리컴포지션 때 들어온 값은 저장하지 않는다. 리컴포지션 때 들어온 값도 저장하고 싶을 때 `rememberUpdateState`를 사용한다.

## 언제 사용?

언제 사용하는게 좋을까?

1. [`rememberUpdatedState` 는 오랫동안 실행되는 `side-effect`에서 변수에 대한 업데이트된 참조를 유지하고 싶을 때 사용 (`side-effect`는 리컴포지션이 일어나도 재시작되지 않는 LaunchedEffect 등을 의미)](https://proandroiddev.com/jetpack-compose-side-effects-iii-rememberupdatedstate-c8df7b90a01d)
    
2. [상위 컴포저블에서 변경된 최신 State를 하위 컴포저블에서 `rememberUpdatedState`로 추적할 때 사용](https://stackoverflow.com/questions/69085027/difference-between-remember-and-rememberupdatedstate-in-jetpack-compose)
    

2번 예시 코드는 아래와 같다.

```kotlin
@Composable
private fun Calculation(input: Int) {
    val rememberUpdatedStateInput by rememberUpdatedState(input)
    val rememberedInput = remember { input }

    Text("updatedInput: $rememberUpdatedStateInput, rememberedInput: $rememberedInput")
}

var myInput by remember {
    mutableStateOf(0)
}

OutlinedButton(
    onClick = {
        myInput++
    }
) {
    Text("Increase $myInput")
}
Calculation(input = myInput)
```

버튼을 클릭해보면 rememberedInput 값은 업데이트 되지 않고 rememberUpdatedStateInput값만 업데이트 되는 걸 확인할 수 있다.