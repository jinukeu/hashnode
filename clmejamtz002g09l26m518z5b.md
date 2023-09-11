---
title: "[Android] MVVM 패턴 박살내기"
datePublished: Sat Aug 27 2022 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmejamtz002g09l26m518z5b
slug: android-mvvm
tags: android

---

## MVVM 패턴이란?

코드를 ***M***odel, ***V***iew, ***V***iew***M***odel의 역할에 맞게 분리하서 작성하는 디자인 패턴

  

## 왜 MVVM을 사용할까?

### 1\. 사용안하면 뭐 어쩔건데?

`MVVM 패턴`을 사용하지 않고 `액티비티/프래그먼트`에 모든 코드를 넣게 되면 어떻게 될까요?

1. `액티비티/프래그먼트` 코드가 매우 복잡해짐
    
2. 스파게티 코드가 될 확률이 높아짐
    
3. 유지보수 & 테스트가 어려워짐
    
4. 다른 사람이 코드를 이해하기 어려워짐 (본인 포함)
    

이런 단점이 있기 때문에 코드를 분리해서 작성하는 `MVVM 패턴`을 사용한다.

  

### 2\. MVVM말고 다른 패턴(MVC, MVP)를 사용하면 안되나요?

`MVC`의 경우 Model, View, Controller로 코드를 관리  
하지만 View와 Model 사이의 **의존성**이 높다는 단점이 있다.

`MVP`의 경우 Model, View, Presenter로 코드를 관리  
하지만 View와 Presenter 사이의 **의존성**이 높다는 단점이 있다.

  

***아니 ... 의존성은 또 뭐고 그게 높으면 뭐 어떻게 되는데요???*** -&gt; [이거 ... 보고 오시죠!](https://kotlinworld.com/64)

\-&gt; MVVM &gt; (MVC, MVP)

  

### 3\. 협업이 쉬워짐

(다른 디자인 패턴도 마찬가지이지만) 팀원들이 전부 `MVVM 패턴`을 적용하고 이해하고 있으면 개발 커뮤니케이션이 수월해진다!

  

### 4\. 그리고 ...

* 각각의 부분이 독립적 -&gt; 유닛 테스트 용이
    
* 모듈화가 잘 되어 있다면 내장 DB를 통채로 변경하는게 비교적 쉬움 ex) realm -&gt; room
    
* `ViewModel`과 `View`가 1:n 연결이 가능 -&gt; 뷰모델 재활용 가능
    
* `View`가 데이터를 실시간으로 관찰하고 자동으로 UI 갱신
    

  

## 자세히 알아보자!

### 구조

> **Model**
> 
> * 어플리케이션에서 사용하는 데이터와 그 데이터를 처리하는 부분
>     

> **View**
> 
> * 액티비티/프래그먼트, 사용자에게 보여지는 UI
>     
> * 사용자와 상호작용
>     
> * `ViewModel`의 데이터를 관찰하여 UI 갱신
>     

> **ViewModel**
> 
> * `Model`과 `View`를 이어주는 역할. `Model`의 데이터를 가공하여 `View`에게 `data stream`을 제공
>     

  

### 어떻게 구현?

AAC(Android Architecture Components)를 사용하면 비교적 쉽게 구현 가능

> **AAC**
> 
> * 강력하고 테스트와 유지관리가 쉬운 앱을 디자인하도록 돕는 라이브러리 모음
>     

  

주로 안드로이드 `MVVM 패턴`에는 `AAC`의 `LiveData`와 `ViewModel`을 사용한다.

  

## 주의사항

### 1\. AAC의 ViewModel != MVVM의 ViewModel

어 뭐야? 둘이 같은거 아닌가요??? -&gt; 아뇨 ... 이름만 같아요 ... ㅋㅋㅋ

> **AAC ViewModel**
> 
> * `ViewModel` 클래스는 수명 주기를 고려하여 UI 관련 데이터를 저장하고 관리하도록 설계되었습니다. `ViewModel` 클래스를 사용하면 화면 회전과 같이 구성을 변경할 때도 데이터를 유지할 수 있습니다. - [공식 문서](https://developer.android.com/topic/libraries/architecture/viewmodel)
>     

> **MVVM ViewModel**
> 
> * `Model`과 `View`를 이어주는 역할. `Model`의 데이터를 가공하여 `View`에게 `data stream`을 제공
>     

  

그러면 어떻게 `AAC ViewModel`을 `MVVM ViewModel`처럼 사용하느냐...! 단순히 ***AAC ViewModel이 가지고 있는 데이터를 관찰가능하게 해주고 View는 데이터바인딩으로 그것을 구독하고 있으면 된다!*** 이렇게 하면 데이터를 수명 주기를 고려하여 관리할 수 있으므로 더 좋다!

  

하지만 단점도 있다.  
`AAC ViewModel`은 `액티비티`에 단 하나만 존재할 수 있다. 예를 들어 `MainViewModel`을 한번 생성하면 그 `액티비티`에서 `MainViewModel`을 여러번 생성해도 하나의 객체만 사용된다. `MVVM 패턴`은 `View`와 `ViewModel`이 1:n 관계인데, `AAC ViewModel`은 이를 지킬 수 없다. 또한 구글은 `View`와 `AAC ViewModel`을 1:1 관계로 사용하고 `ViewModel`안에 여러 `Model`과 `LiveData`를 사용하도록 권장한다. 이 역시 `MVVM` 원칙을 위반한다.

  

### 2\. LiveData의 한계

우선 `LiveData`가 뭔지 알아보자.  
`LiveData`란 `LiveCycle`을 알고 있는 `DataType`이라 생각하면 된다. ***LiveData는 Observer 객체를 이용하여 데이터의 변화를 관찰하며 UI를 업데이트 합니다.*** 어라? `MVVM`에서 `View`의 역할은 ***ViewModel의 데이터를 관찰하여 UI를 갱신하는건데*** ... 아! `ViewModel`의 데이터를 `LiveData`로 만들면 되겠네!

  

#### LiveData vs MutableLiveData

`LiveData`는 `read-only`이고 `MutableLiveData`는 `LiveData`를 수정할 수 있다.  
`ViewModel`를 캡슐화 하기위해 외부로 노출되는 값인 `read-only` `LiveData`를 `public`으로 내부에서는 데이터를 편집할 수 있는 `MutableLiveData`를 `private`로 만들자!

> **캡슐화를 하는 이유**  
> 데이터 내용을 변경할 수 있는 기능을 외부로 노출하면 디버그하기 어렵고 데이터의 내용이 어디서 왔는지 추론하기 어려워지기 때문 [다른 이유는 링크 참고](https://velog.io/@chawani47/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5%EC%97%90%EC%84%9C-%EC%BA%A1%EC%8A%90%ED%99%94%EA%B0%80-%EC%A4%91%EC%9A%94%ED%95%9C-%EC%9D%B4%EC%9C%A0)

  

그래서 `LiveData`의 한계가 뭔데?  
바로 비동기 데이터 스트림을 지원하지 않는다는 것! 이유는 `LiveData`는 UI와 밀접하게 연관되어 있기 때문에 오직 `Main Thread`에서만 관찰되기 때문이다. 따라서 데이터베이스(Local)과 서버(Remote)와의 통신이 이루어지는 `Repository`에서 `LiveData`를 사용할 수 없다.

클린 아키텍처의 `Domain Layer`에서도 `LiveData`를 사용할 수 없다. `Domain Layer`는 순수 Java 및 Kotlin 코드로만 구성하는데 `LiveData`는 안드로이드 플랫폼에 종속적이기 때문

이러한 이유로 `LiveData`는 사용하지 않는게 좋다! (`Repositroy`는 해당 포스트에서 클린 아키텍처는 나중 포스트에서 다룰 예정)

  

#### Flow

하지만~!! 비동기 데이터 스트림을 지원하는 Kotlin 클래스 `Flow`가 존재한다. 이를 통해 `ViewModel`에서는 `LiveData`를 사용하고 DB, 서버와 통신을 하는 `Repository`같은 곳에서는 `Flow`를 사용하면 된다. 하지만 ... 이런 방법을 사용해도 `Flow`는 `LiveData`를 완전히 대체할 수 없다.

* `Flow`는 스스로 안드로이드 생명주기에 대해 알 수 없다.
    
* `Flow`는 상태가 없어 값이 할당된 것인지, 현재 값은 무엇인지 알기가 어렵다.
    
* `Flow`는 `Cold Stream` 방식으로, 연속해서 계속 들어오는 데이터를 처리할 수 없으며 `collect` 되었을 때만 생성되고 값을 반환한다. 만약, 하나의 `flow builder` 에 대해 다수의 `collector`가 있다면 `collector` 하나마다 하나씩 데이터를 호출하기 때문에 업스트림 로직이 비싼 비용을 요구하는 DB 접근이나 서버 통신 등이라면 여러 번 리소스 요청을 하게 될 수 있다.
    

  

### StateFlow, SharedFlow

위의 단점을 보완하기 위해 `StateFlow`, `SharedFlow`가 나왔다. (`SharedFlow`는 `StateFlow`의 일반적인 버전이다. 자세한 설명은 다른 포스트에서 하겠다.)

  

#### StateFlow 특징

* `StateFlow`는 항상 값을 가지고 있고, 오직 한 가지 값을 가진다.
    
* `StateFlow`는 여러 개의 `collector`를 지원한다. 이는 `flow`가 공유된다는 의미이며 앞서 설명했던 `flow`의 단점과는 다르게 업스트림이 `collector` 마다 중복으로 처리되지 않는다.
    
* `StateFlow` 는 `collector` 수에 관계없이 항상 구독하고 있는 것의 최신 값을 받는다.
    
* `StateFlow` 는 `flow builder` 를 사용하여 빌드된 `flow` 가 `cold stream` 이었던 것과 달리, `hot stream`이다. 따라서 `collector` 에서 수집하더라도 생산자 코드가 트리거 되지 않고, 일반 `flow`는 마지막 값의 개념이 없었던 것과 달리 `StateFlow` 는 마지막 값의 개념이 있으며 생성하자마자 활성화된다.
    

  

`LiveData`는 `StateFlow`로 완전히 대체 가능하다.

  

## Repository 패턴

아! 이제 MVVM을 사용하는 이유, 기본 개념, AAC ViewModel과 MVVM ViewModel의 차이점, LiveData대신 StateFlow를 사용해라! 까지 알았으니 이제 실습만 하면 되겠군요? ㅋㅋ

  

아뇨 Repositroy 패턴까지만 ... 보고 가시죠!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694415533198/be79a1f3-6f9d-4575-8f82-675fb12ae711.png align="center")

  

> **Repositroy**  
> 데이터 출처(로컬 DB인지 API응답인지 등)와 관계 없이 동일 인터페이스로 데이터에 접속할 수 있도록 만드는 것을 Repository 패턴이라고 한다. 레포지토리는 데이터 소스에 액세스하는 데 필요한 논리를 캡슐화하는 클래스 또는 구성 요소이다. Repository 의 존재 덕분에 ViewModel 이 데이터를 관리할 필요가 없게 된다.

  

## 마무리

이렇게 Repository에 대한 간단한 설명을 마지막으로 포스팅을 끝내려한다. 이제 남은 일은 Github에서 MVVM 예제 소스를 찾아서 공부하는 것 뿐 ... (클린 아키텍처도 공부 ...)

  

## 레퍼런스

[https://velog.io/@haero\_kim/Android-%EA%B9%94%EC%8C%88%ED%95%98%EA%B2%8C-MVVM-%ED%8C%A8%ED%84%B4%EA%B3%BC-AAC-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0  
https://devvkkid.tistory.com/196  
https://readystory.tistory.com/207  
https://yjyoon-dev.github.io/android/2022/02/12/android-02/  
https://velog.io/@k7120792/Model-View-ViewModel-Pattern  
https://leveloper.tistory.com/216](https://velog.io/@haero_kim/Android-%EA%B9%94%EC%8C%88%ED%95%98%EA%B2%8C-MVVM-%ED%8C%A8%ED%84%B4%EA%B3%BC-AAC-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0￼https://devvkkid.tistory.com/196￼https://readystory.tistory.com/207￼https://yjyoon-dev.github.io/android/2022/02/12/android-02/￼https://velog.io/@k7120792/Model-View-ViewModel-Pattern￼https://leveloper.tistory.com/216) [https://blog.yena.io/studynote/2019/03/16/Android-MVVM-AAC-1.html](https://blog.yena.io/studynote/2019/03/16/Android-MVVM-AAC-1.html)