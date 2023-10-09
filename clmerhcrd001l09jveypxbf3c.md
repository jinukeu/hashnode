---
title: "[Android] Multi Module"
datePublished: Tue Jun 13 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmerhcrd001l09jveypxbf3c
slug: android-multi-module
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/rEM3cK8F1pk/upload/4fec87203fcfdcbdb61e4e0ed619100d.jpeg
tags: android

---

# 모듈

## 모듈이란?

* 모듈은 소스 파일 및 빌드 설정으로 구성된 모음이며, 이를 통해 프로젝트를 별개의 기능 단위로 분할할 수 있다.
    
* 프로젝트에서 하나 이상의 모듈이 포함될 수 있으며, 하나의 모듈이 다른 모듈을 종속성으로 사용할 수 있습니다. 각 모듈은 개별적으로 빌드, 테스트 및 디버그 할 수 있다.
    
* 안드로이드 프로젝트를 만들면 자동으로 생성되는 app도 모듈의 한 종류.
    

## 장점

* 의존성을 줄일 수 있다. (관심사 분리)  
    기존의 단일 모듈 방식에서는 실수로 의존성 규칙을 위반할 수 있지만 멀티 모듈 방식을 사용하면 build.gradle 파일에서 의존성을 추가하지 않으면 다른 모듈의 코드를 사용할 수 없기 때문에 의존성 규칙을 쉽게 관리 가능.
    
* 빌드 시간 감소를 기대할 수 있다.  
    기본적으로 빌드를 할때 변경된 모듈만 빌드하므로 빌드 시간 감소를 기대할 수 있다. 하지만 모듈간 종속성이 복잡해지고 모듈의 수정이 많다면 빌드 시간이 증가될 수 있다.
    
* 코드 재사용성이 높아진다.  
    레이어별, 기능별로 모듈을 나눠서 코드를 작성하게되면 해당 기능이 필요할때 해당 모듈에 대한 의존성을 추가해서 사용하면 되기 때문에 재사용성이 높아진다.
    
* 모듈 단위 테스트를 할 수 있다.
    

## 단점

* 하나의 앱만 있는데, 모듈을 여러개로 나눠놓으면 코드량이 더 많게 느껴진다.
    
* 해당 프로젝트를 처음보면 코드 전체를 보고 이해하기 어렵다.
    
* 위의 이유로 오히려 빌드 시간이 늘어날 수 있다.
    

## 버전 관리

멀티 모듈을 사용하게되면 build.gradle 파일이 여러개 생긴다.

> * **gradle == 빌드 도구(build tool)**  
>     빌드도구는 빌드의 모든 과정(컴파일, 테스트, 배포, 문서화 등의 작업을 포함하는 절차)을 자동으로 처리할 수 있도록 도와주는 것이다. 즉 개발을 좀 더 쉽게 할 수 있도록 도움을 준다! 빌드 도구 중 하나가 Gradle이다. Gradle에서 사용되는 언어로는 그루비 기반의 gradle dsl(Domain-Specific Languages, 도메인 특화 언어)를 사용한다.
>     
> * **settings.gradle 파일과 build.gradle 파일의 차이**  
>     `settings.gradle` : 프로젝트의 구성 정보를 기록하는 파일이다. 어떤 하위 프로젝트들이 어떤 관계로 구성되어 있는지를 기술하면 된다.  
>     `build.gradle` : 의존성이나 플러그인 설정 등을 하는 파일이다. 빌드 작업에 필요한 기본 설정, 동작 등을 기술하면 된다.
>     

여러개의 build.gradle에서 공통적으로 사용할 변수를 쓴다거나, dependency들의 버전을 통일하고자 할 때 사용할 수 있는 방법들은 여러가지가 있다.

### 1\. ext(extra) 사용

* 장점
    
    * 간단하다.
        
* 단점
    
    * ide 자동 완성이 안되기 때문에 오타를 낼 확률이 있다.
        

### 2\. Kotlin DSL + buildSrc 사용

* 장점
    
    * 컴파일 타임에 에러 확인
        
    
    * 코드 탐색
        
    * 자동 완성
        
    * 구문 강조
        
    * IDE의 지원으로 향상된 편집환경
        
    * 소스코드와 동일한 언어의 사용
        
* 단점
    
    * 빌드 캐시가 Invalidation 되거나 클린 빌드시에 Groovy DSL보다 느리다.
        
    * Java8이상에서 동작
        
    * 새로운 라이브러리 버전 Inspection 기능 미지원
        
    * buildSrc 구성 파일에서 버전을 변경하는 경우 Gradle 이 정의한 플러그인을 포함하여 전체 buildSrc 를 재구성한다. (빌드 캐시 무효화)  
        \-&gt; 아래에서 설명하는 version catalog 의 경우 함수 호출의 형태라 상수 값의 변경 여부는 무관하기 때문에, 단순 버전 변경이 일어난다고 해도 리빌드를 하지 않으니 그만큼 시간 절약에 도움이 될 수 있다.
        

### 3\. Version catalog

* 장점
    
    * IDE 에서 자동 완성을 통해 종속성을 쉽게 추가할 수 있다.
        
    * 하나의 파일로 모든 모듈의 종속성 버전을 관리할 수 있다.
        
    * bundle 을 이용할 수 있다.
        

# 참고

[https://brunch.co.kr/@purpledev/43  
https://velog.io/@dabin/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EB%A9%80%ED%8B%B0%EB%AA%A8%EB%93%88  
https://hhyeok1026.tistory.com/38  
https://velog.io/@7lo9ve3/gradle  
https://brunch.co.kr/@oemilk/221](https://brunch.co.kr/@purpledev/43￼https://velog.io/@dabin/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EB%A9%80%ED%8B%B0%EB%AA%A8%EB%93%88￼https://hhyeok1026.tistory.com/38￼https://velog.io/@7lo9ve3/gradle￼https://brunch.co.kr/@oemilk/221)