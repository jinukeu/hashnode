---
title: "동기와 비동기 vs 블락킹과 논블락킹"
datePublished: Sat Jul 08 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cltcf00br000008l9fa4n7skf
slug: vs
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/1j9Yrl0nW10/upload/008374c9d3bb24284dd57fc19cff17f5.jpeg

---

## 동기와 비동기 vs 블락킹과 논블락킹

[참고 자료 1](https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-%EB%8F%99%EA%B8%B0%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC) [참고 자료 2](https://evan-moon.github.io/2019/09/19/sync-async-blocking-non-blocking/#%EB%8F%99%EA%B8%B0-%EB%B0%A9%EC%8B%9D--%EB%85%BC%EB%B8%94%EB%A1%9D%ED%82%B9-%EB%B0%A9%EC%8B%9D) [참고 자료 3](http://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/) [참고 자료 4](https://velog.io/@codemcd/Sync-VS-Async-Blocking-VS-Non-Blocking-sak6d01fhx)

### 동기 & 비동기

여러 개의 함수들이 ***시간을 맞춰서 실행되느냐***에 따라 구분

> * **동기**  
>     함수가 두 개 이상 존재할 때, 이 함수들이 작업을 동시에 시작하거나, 끝나는 타이밍을 맞추거나, 하나가 끝나고 다른 하나를 차례로 실행하는 것
>     
> * **비동기**  
>     두 함수는 서로가 언제 시작하고, 언제 일을 마치는지 전혀 신경쓰지 않는 것
>     

### 블로킹 & 논블로킹

> * **제어권**  
>     자신(함수)의 코드를 실행할 권리 같은 것. 제어권을 가진 함수는 자신의 코드를 끝까지 실행한 후, 자신을 호출한 함수에게 돌려준다.
>     
> * **블로킹**  
>     다른 함수를 호출할 때, 제어권도 아예 함께 넘겨주고 작업이 끝난 후에 돌려받는 방식
>     
> * **논블로킹**  
>     호출할 때 제어권을 넘겨주기는 하지만, 바로 돌려받는다.
>     

### 요약

> 동기/비동기 : 시간을 맞추냐, 안맞추냐  
> 블로킹/논블로킹 : 제어권을 넘겨주냐, 안넘겨주냐

백문이 불여일견! 코드로 직접 한번 봐보자!

```javascript
function employee() {
  for (let i = 1; i < 101; i++) {
    console.log(`직원: 인형 눈알 붙히기 ${i}번 수행`);
  }
}

function boss() {
  console.log('사장: 출근');
  employee();
  console.log('사장: 퇴근');
}

boss();
```

사장: 출근 직원: 인형 눈알 붙히기 1번 수행 직원: 인형 눈알 붙히기 2번 수행 ... 직원: 인형 눈알 붙히기 100번 수행 사장: 퇴근

* 블락킹일까? 논블락킹일까?
    

1. `boss()`함수 내에서 `employee()`함수가 실행되면 `boss()`함수의 *제어권*이 `employee()`로 넘어간다.
    
2. `employee()`함수는 종료된 이후에 제어권을 다시 `boss()`함수로 넘겨준다.  
    \-&gt; 음! 제어권을 넘겨주고 다시 받았으니 *블락킹* 이구나!
    

* 동기일까? 비동기일까?
    

1. `boss()`함수가 실행되다가 `employee()`함수가 실행된다.
    
2. `employee()`함수가 종료된 이후에 `boss()`함수가 마저 실행되고 종료된다. -&gt; 음! 두 개의 함수가 순차적으로 실행되니 *동기* 이구나!
    

즉 위 코드는 **동기-블락킹**이다.

다음 문제~!

```c
int main() {
    Future ft = asyncFileChannel.read(~~~); //논블락킹 함수 - 대략 60분 동안 실행된다고 가정

    while(!ft.isDone()) {
        // isDone()은 asyncChannle.read() 작업이 완료되지 않았다면 false를 바로 리턴해준다.
        // isDone()은 물어보면 대답을 해줄 뿐 작업 완료를 스스로 신경쓰지 않고,
        // isDone()을 호출하는 쪽에서 계속 isDone()을 호출하면서 작업 완료를 신경쓴다.
        // asyncChannle.read()이 완료되지 않아도 여기에서 다른 작업 수행 가능 
    }
}
// 작업이 완료되면 작업 결과에 따른 다른 작업 처리
```

그냥 블로그에서 퍼온 코드이다...! 주석을 자세히 읽으면 동기/비동기, 블락킹/논블락킹을 판단할 수 있다.

[`asyncFileChannel.read`](http://asyncFileChannel.read)`()`함수는 *논블락킹* 함수이다. 즉 `main()` 함수는 제어권을 [`asyncFileChannel.read`](http://asyncFileChannel.read)`()`함수로 넘겨준 뒤 바로 돌려받는다.

`isDone()`은 단순히 [`asyncFileChannel.read`](http://asyncFileChannel.read)`()`함수의 작업이 완료되었는지 파악하는 함수이다. `isDone()`이 `true`일 때, 즉 [`asyncFileChannel.read`](http://asyncFileChannel.read)`()`가 종료된 이후에 `main()`함수의 나머지 부분이 실행되므로 *동기*이다.

\-&gt; 정답! **논블락킹-동기**

선생님, 다음 문제는 **블락킹-비동기** 예제 인가요?? ㅋㅋ  
아쉽게도 **블락킹-비동기**에 대한 예제 코드는 ... 없다 ... ! 대신 언제 블락킹-비동기가 발생하는지 서술하겠다.

> Asynchronous Non-Blocking 모델 중에서 Blocking 으로 동작하는 작업이 있는 경우 의도와 다르게 Asynchronous Blocking으로 동작할 때가 있다고 한다.
> 
> 대표적인 예로는 Node.js와 MySQL을 함께 사용하는 경우이다. Node.js는 비동기로 작업하려 하지만 MySQL 드라이버가 Blocking 방식으로 동작하므로 어쩔 수 없이 Asynchronous Blocking 방식으로 동작하게 된다.

  

후후 ... 분명 블락킹/논블락킹, 동기/비동기에 대한 설명을 들으면서 블락킹 == 동기, 비동기 == 논블락킹 아냐? 라는 생각을 했을 것이다 ... ! 하지만 위 예시를 보면 전혀 아니라는 것을 알 수 있다!

```c
int main() {
    Future ft1 = asyncFileChannel.read(A); //논블락킹 함수 - 5분 동안 실행
    Future ft2 = asyncFileChannel.read(B); //논블락킹 함수 - 10분 동안 실행
}
```

위 코드는 ... 어디보자 [`asyncFileChannel.read`](http://asyncFileChannel.read)`(A)`, [`asyncFileChannel.read`](http://asyncFileChannel.read)`(B)` 모두 논블락킹 함수이고 2개의 함수는 시간에 맞춰 실행되지 않으므로 비동기 ... !  
\-&gt; 정답! **논블락킹-비동기**!

아래는 실제 프로젝트에서 동기 처리를 해주지 않아 생긴 문제이다.

### Access Token 갱신 요청이 동시에 2번 발생하면서 생긴 문제

### 🚨 ***Issue***

`okhttp3.Authenticator`를 사용하여 서버에서 내려온 `HTTP Status Code`가 `401`인 경우 자동으로 `Access Token` 갱신 요청을 하도록 만들어놨습니다.

1. 유저의 `Access Token`이 만료된 상태에서 `Access Token`이 필요한 api 2개를 비동기 호출
    
2. `okhttp3.Authenticator`에 의해 `Access Token Refresh Api`가 2번 비동기 호출
    
3. 2개의 `Access Token Refresh Api(ㄱ, ㄴ)`의 `Body`에는 `A Refresh Token` 값이 들어감
    
4. `ㄱ Api` 호출에서 `A Refresh Token`이 갱신되어 `B Refresh Token`으로 변경됨
    

|  | ㄱ Api Http Body | 서버 DB에 저장된 유저의 Refresh Token |
| --- | --- | --- |
| 요청 | refresh : A Refresh Token | A Refresh Token |
| 처리 완료 |  | B Refresh Token |

5. `ㄴ Api`가 호출되지만 이미 `ㄱ Api` 호출에서 서버 DB에 저장된 유저의 `Refresh Token` 값이 바뀌었기 때문에 토큰 갱신 실패
    

|  | ㄴ Api Http Body | 서버 DB에 저장된 유저의 Refresh Token |
| --- | --- | --- |
| 요청 | refresh : A Refresh Token | B Refresh Token |
| 처리 실패 |  | A Refresh Token invaild |

### ✅ ***Solution***

Api 호출을 동기 처리하여 문제를 해결했습니다.

```kotlin
//ViewModel.kt

// 서버에서 강의 평가의 통합 정보를 불러온다. 요청이 성공했을 경우 onSuccess()를 실행한다.
suspend fun fetchLectureIntegratedInfo(onSuccess: () -> Unit) {
        val response = lectureInfoRepository.getLectureDetailInfo(pageViewModel.lectureId)
        when {         
        	response.isSuccessful -> {
                response.body()?.data?.let { data ->
                    _lectureDetailInfoData.value = data
                    onSuccess()
                }
            }
            else -> {
                handleError(response.code())
            }
        }
    }

// 서버에서 강의평가 또는 시험정보 리스트를 불러온다.
fun fetchLectureList() {
        if(pageViewModel.page.value!! == LAST_PAGE)
            return
        when (_writeBtnText.value) {
            R.string.write_evaluation -> getEvaluationList()
            else -> getExamList()
        }
    }
```

```kotlin
//Fragment.kt
lifecycleScope.launch {
	lectureInfoViewModel.fetchLectureIntegratedInfo {
        lectureInfoViewModel.fetchLectureList()
   }
}
```