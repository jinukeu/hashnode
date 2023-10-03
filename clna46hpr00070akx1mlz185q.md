---
title: "[Kotlin in Action] 9장. 제네릭스"
datePublished: Tue Oct 03 2023 09:25:22 GMT+0000 (Coordinated Universal Time)
cuid: clna46hpr00070akx1mlz185q
slug: kotlin-in-action-9
tags: kotlin

---

## 제네릭 타입 파라미터

## 제네릭 함수와 프로퍼티

리스트를 다루는 함수를 작성한다면 어떤 특정 타입을 저장하는 리스트뿐 아니라 모든 리스트를 다룰 수 있는 함수를 원할 것이다. 이때 제네릭 함수를 작성해야 한다.

```kotlin
fun <T> List<T>.slice(indices: IntRange) : List<T>
```

### 제네릭 클래스 선언

자바와 마찬가지로 코틀린에서도 타입 파라미터를 넣은 꺽쇠 기호(`<>`)를 클래스 또는 인터페이스 뒤에 붙이면 제네릭하게 만들 수 있다.

심지어 클래스가 자기 자신을 타입 인자로 참조할 수도 있다.

```kotlin
interface Comparable<T> {
    fun compareTo(other: T): Int
}

class String : Comparable<String> {
    override fun compareTo(other: String): Int = /*...*/
}
```

String 클래스는 제네릭 Comparable 인터페이스를 구현하면서 그 인터페이스의 타입 파라미터 T로 String 자신을 지정한다.

### 타입 파라미터 제약

타입 파라미터 제약은 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능이다. 예를 들어 sum이라는 함수를 List&lt;Int&gt; 또는 List&lt;Double&gt;에만 적용가능하게 만들 수 있다.

어떤 타입을 제네릭 타입의 타입 파라미터에 대한 상한으로 지정하면 된다.

```kotlin
fun <T : Number> List<T>.sum() : T
```

타입 파라미터에 대해 둘 이상의 제약을 가하는 방법은 다음과 같다.

```kotlin
fun <T> ensureTrailingPeriod(seq: T) 
    where T : CharSequence, T : Appendable {}
```

### 타입 파라미터를 널이 될 수 없는 타입으로 한정

아무런 상한을 정하지 않은 타입 파라미터는 결과적으로 Any?를 상한으로 정한 파라미터와 같다.

널 가능성을 제외한 아무런 제약도 필요 없다면 Any를 상한으로 하면 된다.

```kotlin
class Processor<T> {
    fun process(value: T) {
        value?.hashCode() // value는 null이 될 수 있다.
    }
}

//
class Processor<T : Any> {
    fun process(value: T) {
        value?.hashCode() // value는 null이 될 수 없다.
    }
}
```

## 실행 시 제네릭스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터

### 실행 시점의 제네릭: 타입 검사와 캐스트

```kotlin
val list1: List<String> = listOf("a", "b")
val list2: List<Int> = listOf(1, 2, 3)
```

컴파일러는 두 리스트를 서로 다른 타입으로 인식하지만 **런타임**에는 같은 타입의 객체(List)로 인식한다. 즉, List의 타입 인자 정보인 String, Int가 사라진다. 이를 `타입 소거` 라고 한다.

List 뿐만 아니라 모든 제네릭에도 마찬가지의 현상이 일어난다.

타입 소거로 인해 실행 시점에 타입 인자를 검사할 수 없다는 단점이있다.

```kotlin
if (value is List<String>) { ... }
ERROR: Cannot check for instance of erased type
```

대신 타입 소거를 하게되면 저장해야 하는 타입 정보의 크기가 줄어들어서 메모리 사용량이 줄어든다는 장점이 있다.

그러면 위 예시에서 단순히 value가 List타입인지 확인하려면 어떻게 해야할까? 스타 프로젝션을 사용하면 된다.

```kotlin
if (value is List<*>) { ... }
```

### 실체화한 타입 파라미터를 사용한 함수 선언

제네릭 함수의 타입 인자는 런타임때 타입 소거 되지만, 인라인 함수의 경우는 다르다. 인라인 함수의 타입 파라미터는 실체화되므로 실행 시점에 인라인 함수의 타입 인자를 알 수 있다. 단, `reified` 키워드를 같이 사용해야 한다.

```kotlin
inline fun <reified T> isA(value: Any) = value is T
// T는 이제 런타임에 접근 가능해진다.

println(isA<String>("abc"))
```

### 실체화한 타입 파라미터로 클래스 참조 대신

#### ::class.java 에서 :: 는 뭘 의미할까?

::는 리플렉션을 의미한다.

> 리플렉션은 힙 영역에 로드된 Class 타입의 객체를 통해, 원하는 클래스의 인스턴스를 생성할 수 있도록 지원하고, 인스턴스의 필드와 메소드를 접근 제어자와 상관 없이 사용할 수 있도록 지원하는 API이다.
> 
> 여기서 로드된 클래스라고 함은, JVM의 클래스 로더에서 클래스 파일에 대한 로딩을 완료한 후, 해당 클래스의 정보를 담은 **Class 타입의 객체**를 생성하여 메모리의 힙 영역에 저장해 둔 것을 의미한다. new 키워드를 통해 만드는 객체와는 다른 것임을 유의하자. [https://steady-coding.tistory.com/609](https://steady-coding.tistory.com/609)

* **클래스 리플렉션** `className::class`
    
* **인스턴스를 통한 클래스 리플렉션** `instanceName::class`
    

클래스 이름을 통해서 바로 리플렉션을 불러오거나 인스턴스를 사용해서 리플렉션을 불러올 수 있다.

이렇게 불러온 리플렉션은 `KClass` 라는 레퍼런스 객체이다. `KClass`를 통해서 클래스가 추상클래스인지, 데이터 클래스인지에 대한 정보를 불린 변수로 얻거나, 생성자, 멤버변수들에 대한 정보를 얻을 수 있다.

`KClass` 가 아닌 자바의 `Class` 를 가져오고 싶다면 `.java` 를 붙이면 된다.

ex) `className::class.java`

[https://velog.io/@evergreen\_tree/Android-%EC%BD%94%ED%8B%80%EB%A6%B0-class.java%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C](https://velog.io/@evergreen_tree/Android-%EC%BD%94%ED%8B%80%EB%A6%B0-class.java%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)

---

자바의 `Class` 타입 인자를 파라미터로 받는 경우에도 inline과 `reified` 키워드는 유용하게 사용된다.

```kotlin
inline fun <reified T> loadService() {
    return ServiceLoader.load(T::class.java)
}
```

### 실체화한 타입 파라미터의 제약

다음과 같은 경우 실체화한 타입 파라미터를 사용할 수 있다.

* 타입 검사와 캐스팅 (is, !is, as, as?)
    
* 코틀린 리플렉션 API (::class)
    
* 코틀린 타입에 대응하는 java.lang.Class 얻기(::class.java)
    
* 다른 함수를 호출할 때 타입 인자로 사용
    

하지만 다음과 같은 일은 할 수 없다.

* 타입 파라미터 클래스의 인스턴스 생성하기
    
* 타입 파라미터 클래스의 동반 객체 메소드 호출하기
    
* 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입 인자로 넘기기
    
* 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 파라미터를 reified로 지정하기
    

실체화한 타입 파라미터 (inline + reified)는 인라인 함수에만 사용할 수 있으므로 실체화한 타입 파라미터를 사용하는 함수는 자신에게 전달되는 모든 람다와 함께 인라이닝된다.

람다를 인라이닝할 수 없는 경우가 생길 수도 있고 성능 문제로 람다를 인라이닝 하고 싶지 않을 때도 있다. 이때는 noinline을 함수 타입 파라미터에 붙여서 인라이닝을 금지할 수 있다.

## 변성: 제네릭과 하위 타입

변성 개념은 List&lt;String&gt;, List&lt;Any&gt;와 같이 기저 타입(List)이 같고 타입 인자(String, Any)가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념이다.

### 변성이 있는 이유: 인자를 함수에 넘기기

### 클래스, 타입, 하위 타입

1. 제네릭이 아닌 클래스의 경우  
    타입이 될 수 있다. String 클래스는 String 타입, String? 타입이 될 수 있다.
    
2. 제네릭 클래스인 경우  
    타입 파라미터를 구체적인 타입 인자로 바꿔줘야 한다.  
    List는 타입이 될 수 없다. 단, List&lt;String&gt;은 타입이다.
    

하위 타입 : 어떤 타입 A의 값이 필요한 모든 장소에 어떤 타입 B의 값을 넣어도 아무 문제 없다면 타입 B는 타입 A의 하위 타입이다.

예를 들어 Int는 Number의 하위 타입이지만 String의 하위 타입은 아니다.

상위 타입은 하위 타입의 반대다.

---

#### 무공변

**타입 S가 T의 하위 타입일 때, Box\[S\]와 Box\[T\] 사이에 상속 관계가 없는 것**을 나타냅니다.

예를 들어 String은 Any의 하위 타입이지만, MutableList&lt;String&gt;은 MutableList&lt;Any&gt;의 하위 타입이 아니다.

[https://sungjk.github.io/2021/02/20/variance.html](https://sungjk.github.io/2021/02/20/variance.html)

---

### 공변성: 하위 타입 관계를 유지

`Producer<T>`를 예로 공변성 클래스를 설명하자. A가 B의 하위 타입일 때 `Producer<A>`가 `Producer<B>`의 하위 타입이면 `Producer`는 공변적이다. 즉, 타입 인자(`A, B`)가 하위 타입이면 기저 타입(`Producer`)도 하위 타입이다.

코틀린 제네릭 클래스가타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 이름 앞에 out을 넣어야 한다.

```kotlin
interface Producer<out T> {
    fun produce(): T
}
```

공변적으로 만들면 불필요한 타입 캐스팅을 하지 않아도 된다.

```kotlin
open class Animal {
    fun feed() {}
}

class Cat : Animal() {
    fun cleanLitter() { ... }
}

class Herd<out T: Animal> {
    ...
}

fun takeCareOfCats(cats: Herd<Cat>) {
    for (i in 0 until cats.size) {
        cats[i].cleanLitter()
    }
    feedAll(cats) // feedAll은 Herd<Animal> 타입을 받고 있다.
    // 하지만 cats는 Herd<Cat> 타입이다.
    // 공변성을 설정해주지 않았다면 타입 캐스팅을 해줘야하는 번거로움이 있다.
}

fun feedAll(animals: Herd<Animal>) {
    animals.feed()
}
```

---

단, 공변성을 가지게 되면 값에 대한 read만 가능하고, write은 불가능해진다. 이유는 다음과 같다. in 위치에서만 사용가능하고 out 위치에서는 사용할 수 없다.

```kotlin
operator fun get(index: Int): T // 여기서의 T는 out 위치이다.
override fun add(element: T): Boolean // 여기서의 T는 in 위치이다.
```

**\[READ\]**

`feedAll` 메서드에서 `Herd<out Animal>` 타입인 변수(`animals`)은 부모가 `Animal`인 것을 컴파일러가 인지하고 있다.

그렇다면 `animals`의 값을 read 할 때에는 Animal, Cat 중 하나일 것도 알 것이며,

이는 모두를 포함하는 Animal로 할당 해 줄 수 있으므로 read를 할 때에는 문제가 발생하지 않는다.

**\[WRITE\]**

그러나 write의 경우에는 좀 다르다.

공변성을 사용한 `animals`에게 실제로는 `Herd<Cat>`을 넘겨줬다.

그러나 메소드에서 `animals`은 `Herd<out Animal>`로 선언되어 있으므로,

실제 `animals`이 `Herd<Animal>`인지, `Herd<Cat>`인지 모른다.

`animals`의 실제 Type을 모르는 메소드가 함부로 값을 write 할 수 없으므로 문제가 발생한다.

`List`도 공변적이기 때문에 List에 T 타입의 값을 추가하거나 리스트에 있는 기존 값을 변경하는 메소드는 없다.

[https://hungseong.tistory.com/30](https://hungseong.tistory.com/30)

---

### 반공변성: 뒤집힌 하위 타입 관계

반공변 클래스의 하위 타입 관계는 공변 클래스의 경우와 반대다. 예를 들어 Comparator 인터페이스를 살펴 보자.

```kotlin
interface Comparator<in T> {
    fun compare(e1: T, e2: T): Int { ... }
}
```

이 인터페이스의 메소드는 T 타입의 값을 소비하기만 한다. 이는 T가 in 위치에서만 쓰인다는 뜻이다.

```kotlin
val anyComparator = Comparator<Any> {
    e1, e2 -> e1.hashCode() - e2.hashCode()
}
val strings: List<String> = ...
strings.sortedWith(anyComparator)
```

sortedWith은 `Comparator<String>`을 요구한다. 하지만 위 코드를 보면 String의 상위 타입인 Any가 들어간 `Comparator<Any>`가 들어간 것을 확인할 수 있다. 즉 `Any`는 `String`의 상위 타입 이지만 `Comparator<Any>`는 `Comparator<String>`의 하위 타입이다. 상위 타입 -&gt; 하위 타입으로 관계가 뒤집혔다.

| 공변성 | 반공변성 | 무공변성 |
| --- | --- | --- |
| Producer&lt;out T&gt; | Consumer&lt;in T&gt; | MutableList&lt;T&gt; |
| 타입 인자의 하위 타입 관계가 제네릭 타입에서도 유지된다. | 타입 인자의 하위 타입 관계가 제네릭 타입에서 뒤집힌다. | 하위 타입 관계가 성립하지 않는다. |
| Producer&lt;Cat&gt;은 Producer&lt;Animal&gt;의 하위 타입이다. | Consumer&lt;Animal&gt;은 Consumer&lt;Cat&gt;의 하위 타입이다. |  |
| T를 아웃 위치에서만 사용할 수 있다. | T를 인 위치에서만 사용할 수 있다. | T를 아무 위치에서 사용할 수 있다. |

### 사용 지점 변성: 타입이 언급되는 지점에서 변성 지정

클래스 안에서 어떤 타입 파라미터가 공변적이거나 반공변적인지 선언할 수 없는 경우에도 특정 타입 파라미터가 나타나는 지점에서 변성을 정할 수 있다. 즉, 함수에서도 변성 지정이 가능하다.

### 스타 프로젝션: 타입 인자 대신 \* 사용

제네릭 타입 인자 정보가 없음을 표현하기 위해 스타 프로젝션을 사용한다. 예를 들어 원소 타입이 알려지지 않은 리스트는 `List<*>` 라는 구문으로 표현할 수 있다.

`MutableList<*>`은 `MutableList<Any?>`와 같지 않다. Any?는 모든 타입의 원소를 담을 수 있다는 리스트이고 \*은 어떤 정해진 구체적인 타입의 원소만들 담는 리스트이다.

`MutableList<*>`은 `MutableList<out Any?>` 처럼 동작한다. 어떤 리스트의 타입을 모르더라도 그 리스트에서 안전하게 Any? 타입의 원소를 꺼내올 수는 있지만, 타입을 모르는 리스트에 원소를 마음대로 넣을 수는 없다.