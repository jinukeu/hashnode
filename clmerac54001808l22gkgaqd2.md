---
title: "[Android] Bundle, 직렬화, 역직렬화, ViewModel"
datePublished: Thu Feb 02 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmerac54001808l22gkgaqd2
slug: android-bundle-viewmodel
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/6GMq7AGxNbE/upload/5f7babe2d718d5be5b5f9664db15d166.jpeg
tags: android

---

# Bundle

## 정의

> 여러가지 타입의 값을 저장하는 Map 클래스  
> 기본 타입인 int, double, long, String 부터 FloatArray, StringArrayList, Serializable, Parcelable까지 구현함.

## 사용처

1. Activity간에 데이터를 주고 받을 때 Bundle 클래스를 사용하여 데이터를 전송함. (Intent의 PutExtra메서드에서 사용하는 것이 Bundle 객체.)
    
2. Activity를 생성할 때 Bundle savedInstanceState 객체를 가지고 와서, 액티비티를 중단할 때 savedInstanceState 메서드를 호출하여 임시적으로 데이터를 저장한다. 즉 전에 저장된 데이터가 있으면, 그 데이터를 가지고 Activity를 다시 생성한다.
    

# 직렬화와 역직렬화

> * 데이터 직렬화
>     
>     * 메모리를 디스크에 저장하거나, 네트워크 통신에서 사용하기 위한 형식으로 변환하는 것
>         
> * 데이터 역직렬화
>     
>     * 디스크에 저장한 데이터를 읽거나, 네트워크 통신으로 받은 데이터를 메모리에 쓸 수 있도록 변환하는 것
>         

## 객체와 같은 참조 형식 데이터를 디스크에 저장하거나, 네트워크 통신으로 보낼 수 없는 이유

참조 형식 데이터의 경우 힙에 할당되어 있는 메모리 번지 주소를 가지고 있기 때문 ... !

따라서 메모리 번지 주소는 프로그램이 종료되면 기존에 할당되었던 메모리는 해제되기 때문에 의미가 없고, 각 PC 마다 사용하고 있는 메모리 공간 주소는 서로 다르기 때문에 네트워크 통신으로 주소를 보내도 의미가 없다.

***하지만 직렬화를 사용하게 되면 각 주소 값이 가지는 데이터를 전부 끌어 모아서 값 형식 데이터로 변환해 준다.*** 즉, 데이터를 파일 저장, 데이터 통신에서 사용할 수 있는 형식으로 바꿔준다.

## 자바에서 직렬화하는 방법

직렬화를 하고 싶은 객체에 Serializable 인터페이스를 implement한 후, [java.io](http://java.io).ObjectOutputStream을 사용하여 직렬화를 수행한다.

## 자바 역직렬화 방법

[java.io](http://java.io).ObjectInputStream 을 사용하여 역직렬화를 진행한다.

## 자바 직렬화/역직렬화 단점

역직렬화를 할 때 클래스 구조가 변경되면 [java.io](http://java.io).InvalidClassException 이 발생한다. Java 직렬화 대상 객체는 동일한 serialVersionUID를 가지고 있어야 하는데, 클래스의 멤버 변수가 추가되거나 삭제되면 serialVersionUID가 달라지기 때문이다.

## 안드로이드 스튜디오에서의 직렬화/역직렬화

[참고 자료](https://youngest-programming.tistory.com/108)

### 1\. Serializable 사용

> 데이터 클래스(POJO)에 Serializable인터페이스만 implements하면 된다.

단, Serializable은 내부에서 Reflection을 사용하여 직렬화를 처리한다. Reflection 은 프로세스 동작 중에 사용되며 처리 과정 중에 많은 추가 객체를 생성 한다. 이 많은 쓰레기들은 가비지 컬렉터의 타겟이 되고 가비지 컬렉터의 과도한 동작으로 인하여 성능 저하 및 배터리 소모가 발생한다.

(Reflection이란 리플렉션이란 객체를 통해 클래스의 정보를 분석해 내는 프로그램 기법을 말한다. 메소드를 호출 할 때, 타겟이 실제로 메소드 선언 자의 인스턴스인지, 올바른 인수 번호를 가지고 있는지 여부, 각 인수의 유형이 올바른지 여부 등을 확인해야한다. 즉 런타임에 코드의 동작을 유지하고 수정하는 유연하지만 느린 방법이다.)

### 2\. Parcelable 사용

Reflection을 사용하지 않도록 설계되어 Serializable보다 속도가 빠르다. Serializable 과는 달리 **직렬화 처리 방법을 사용자가 명시적으로 작성**해야 한다. Parcelable은 안드로이드 전용 인터페이스이고 Serializable은 표준 자바 인터페이스이다.

**@Parcelize**을 사용하면 **직렬화 처리 방법을 자동으로 작성**해준다. 컴파일타임에 바이트 코드 변조를 하기 때문에 추가되는 메서드 및 런타임시 오버헤드 비용도 발생하지 않는다. 그리고 무엇보다 이 플러그인은 구글과 JetBrains가 협업하여 만든 플러그인이기 때문에 다른 3rd-party 라이브러리와는 다르게 추후 계속 유지보수 될 것이라 기대한다.

[Parcelize 사용 방법](https://www.charlezz.com/?p=44613)

# ViewModel

> Ui 관련 데이터를 저장하고 관리해주는 역할

## 왜 쓰는거야?

Configuration 변경이 (예:화면 회전) 발생하면 액티비티가 다시 시작 되며 기존의 데이터가 날라간다. 기존에는 이러한 문제를 saveInstanceState를 통해 해결했지만 ... 문제가 많다.

1\. 담을 수 있는 데이터가 적다. 공식 문서에서는 50k 미만의 데이터를 권장하고 있다. 2. 담을 수 있는 데이터의 형태가 제한된다. 3. onCreate에서 작업을 처리하므로 UI 컨트롤러가 해야할 일이 늘어난다.

하지만 ViewModel을 사용하면 이러한 문제를 해결할 수 있다.

![](https://developer.android.com/static/images/topic/libraries/architecture/viewmodel-lifecycle.png?hl=ko align="left")

ViewModel은 범위로 지정된 ViewModelStoreOwner가 사라질 때까지 메모리에 남아있다.

1. 액티비티의 경우 finish될 때
    
2. 프래그먼트의 경우 detach될 때
    
3. jetpack navigation의 경우 백 스택에서 삭제될 때
    

제거 된다. 즉 Configuration 변경에 영향을 받지 않는다!!!

[AAC ViewModel은 어떻게 onDestroy에서 살아남을 수 있었을까](https://seokzoo.tistory.com/9)

## ViewModel은 어떻게 생성될까

ViewModel은 ViewModelStore라는 객체에서 관리를 한다. ViewModelStore 클래스는 내부적으로 `HashMap<String, ViewModel>` 를 두어 ViewModel을 관리한다.

그러면 이 ViewModelStore 객체는 누가 어떻게 만들고 관리할까? 그건 바로 ViewModelStoreOwner 라는 녀석이 한다.

액티비티나 프래그먼트가 ViewModelStoreOwner를 관리하고 있기 때문에, ViewModel 객체를 생성할 때 액티비티나 프래그먼트가 필요하고 어떤 Owner를 통해 생성하냐에 따라 ViewModel의 Scope이 결정된다.

액티비티나 프래그먼트를 ViewModelProvider에 전달하여 ViewModel 인스턴스를 생성할 수 있다.

```kotlin
ViewModelProvider(this).get(UserViewModel::class.java)
```

**생성 과정**

1. ViewModelProvider를 통해 ViewModel 인스턴스를 요청한다. 이때 owner를 넘겨준다. (액티비티 또는 프래그먼트)
    
2. ViewModelProvider 내부에서는 ViewModelStoreOwner를 참조하여 ViewModelStore를 가져온다.
    
3. ViewModelStore에게 이미 생성된(저장된) ViewModel 인스턴스를 요청한다.
    
4. 만약 ViewModelStore가 적합한 ViewModel 인스턴스를 가지고 있지 않다면, Factory를 통해 ViewModel인스턴스를 생성한다.
    
5. 생성한 ViewModel 인스턴스를 ViewModeStore에 저장하고 만들어진 ViewModel 인스턴스를 클라이언트에게 반환한다.
    
6. 똑같은 ViewModel 인스턴스 요청이 들어온다면, 1~3번의 과정을 반복하게 된다.
    

android-ktx / fragment-ktx 모듈을 사용하면 보다 다음과 같이 편리하게 뷰모델 인스턴스를 생성할 수도 있다.

```kotlin
private val model: UserViewModel by viewModels()
```

## 주의 사항

> ViewModel은 절대로 Activity나 Fragmemt 또는 View의 Context를 참조해서는 안된다.

언제든 생성 및 파괴될수있는 객체의 참조를 ViewModel에게 유지시키는 일은 파괴된 Activity를 ViewModel에서 유지 시킴으로서 메모리 누수를 발생시킨다. (파괴된 Activity Context를 ViewModel이 계속 참조하고 있으면 가비지 컬렉터가 작동하지 않는다.) 시스템의 Context가 필요한경우 AndroidViewModel을 사용해라. (AndroidViewModel을 사용하면 applicationContext를 사용할 수 있다.)

## ViewModelScope

> Coroutine Scope, viewModelScope는 ViewModel이 onCleared()를 호출 할때 자동으로 coroutine 작업을 취소한다.

[(액티비티, 프래그먼트에서도 생명주기에 맞춰 코루틴을 사용해야 한다.)](https://www.charlezz.com/?p=46044)

ViewModelScope는 withContext를 통해 Dispatchers 전환이 없다면 기본적으로 Main Thread 로 작업을 한다.. 그런데 신기한 점은 ViewModelScope 에서 Retrofit2를 사용할 때 IO Thread가 아니더라도 문제없이 Retrofit2를 사용할 수 있다.

Retrofit2에서 Coroutine을 사용하기 위해서는 interface에 suspend 함수를 적어줘야 한다.

```kotlin
interface ApiService {

    // 회원가입 요청 API
    @POST(SIGN_UP)
    suspend fun signUp(@Body info: SignUpFormat): SignUpResponse
```

interface로 작성한 suspend fun signUp()은 Retrofit2의 내부 코드에서 IO를 알아서 처리하고, 이를 리턴해주는 구현체를 가지고 있다. 결국 Retrofit2를 사용해 coroutines 의로 값을 불러온다면 알아서 새로운 쓰레드를 생성해 내부에서 데이터를 불러오는 작업을 하고, 이를 UI로 바꿔준다.

# 참고

[자료 1](https://todaycode.tistory.com/33) [자료 2](https://charlezz.medium.com/viewmodel%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-viewmodel-%EC%B4%88%EB%B3%B4%EB%A5%BC-%EC%9C%84%ED%95%9C-%EA%B0%80%EC%9D%B4%EB%93%9C-e1be5dc1ac18) [자료 3](https://developer.android.com/topic/libraries/architecture/coroutines?hl=ko) [자료 4](https://kotlinworld.com/87?category=971011) [자료 5](https://kotlinworld.com/198) [자료 6](https://kotlinworld.com/230) [자료 7](https://gift123.tistory.com/60) [자료 8](https://thdev.tech/kotlin/2021/01/12/Retrofit-Coroutines/)