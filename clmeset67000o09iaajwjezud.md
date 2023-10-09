---
title: "[Kotlin in Action] 5장. 람다로 프로그래밍"
datePublished: Wed Aug 02 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmeset67000o09iaajwjezud
slug: kotlin-in-action-5
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696821298094/e59bb760-bc6d-4aac-88e2-60cb3008868c.jpeg
tags: kotlin

---

람다 식 또는 람다는 기본적으로 다른 함수에 넘길 수 있는 작은 코드 조각을 뜻한다.

## 람다 식과 멤버 참조

### 람다와 컬렉션

#### 람다를 사용해 컬렉션 검사하기

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.maxBy { it.age })

>>> Person(name=Bob, age=31)
```

maxBy는 가장 큰 원소를 찾기 위해 비교에 사용할 값을 돌려주는 함수를 인자로 받는다. 중괄호로 둘러싸인 코드 { it.age }는 바로 비교에 사용할 값을 돌려주는 함수이다.

이런 식으로 단지 함수나 프로퍼티를 반환하는 역할을 수행하는 람다는 멤버 참조로 대치할 수 있다.

```kotlin
people.maxBy(Person::age)
```

### 현재 영역에 있는 변수에 접근

자바와 달리 코틀린 람다 안에서는 파이널 변수가 아닌 변수에도 접근할 수 있다.

```kotlin
var clientErrors = 0

responses.forEach {
	if(it.startsWith("4")) clientErrors++
}
```

### 멤버 참조

멤버 참조 문법 `클래스::멤버`

멤버 참조는 그 멤버를 호출하는 람다와 같은 타입이다.

```kotlin
people.maxBy(Person::age)
people.maxBy { p -> p.age }
people.maxBy { it.age }
```

최상위에 선언된 함수나 프로퍼티를 참조할 수도 있다.

```kotlin
fun salute() = println("Salute!")

::salute
```

람다가 인자가 여럿인 다른 함수한테 작업을 위임하는 경우 람다를 정의하지 않고 직접 위임 함수에 대한 참조를 제공하면 편리하다.

```kotlin
val action = { person: Person, message: String -> 
	sendEmail(person, message)
    }

val nextAction = ::sendEmail
```

생성자 역시 참조를 쓸 수 있다.

```kotlin
data class Person(val name: String, val age: Int)

val createPerson = ::Person
val p = createPerson("Alice", 29)
```

확장 함수도 멤버 함수와 똑같은 방식으로 참조할 수 있다.

```kotlin
fun Person.isAdult() = age >= 21
val predicate = Person::isAdult
```

## 컬렉션 함수형 API

map과 멤버 참조를 유용하게 조합할 수 있다.

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.map { it.name })

// 더 유용하게 사용
people.map(Person::name)
```

### flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리

flatMap 함수는 먼저 인자로 주어진 람다를 컬렉션의 모든 객체에 적용하고 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 한데 모은다.

```kotlin
>>> val strings = listOf("abc", "def")
>>> println(strings.flatMap { it.toList() }
[a, b, c, d, e, f]
```

1. map { it.toList() } 적용 : "abc" -&gt; \[a, b, c\] / "def" -&gt; \[d, e, f\]
    
2. flatten -&gt; \[a, b, c, d, e, f\]
    

1번의 과정을 수행하고 싶지 않다면 `flatten()`을 사용하면 된다.

## 지연 계산(lazy) 컬렉션 연산

map이나 filter 같은 컬렉션 함수는 결과 컬렉션을 **즉시** 생성한다. 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담기 때문에 성능이 안좋다. **시퀀스**를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.

```kotlin
people.asSequence()
	.map(Person::name)
    .filter { it.startsWith("A") }
    .toList()
```

### 시퀀스 연산 실행: 중간 연산과 최종 연산

시퀀스의 경우 모든 연산은 각 원소에 대해 순차적으로 적용된다. (컬렉션은 한번에 적용)

### 시퀀스 만들기

generateSequence 함수를 사용하여 시퀀스를 만들 수 있다. 이 함수는 이전의 함수를 원소로 받아 다음 원소를 계산한다.

```kotlin
val naturalNumbers = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
println(numbersTo100.sum())
```

어떤 객체의 조상이 자신과 같은 타입이고 모든 조상의 시퀀스에서 어떤 특성을 알고 싶을 때 시퀀스를 사용한다.

```kotlin
fun File.isInsideHiddenDirectory() = 
	generateSequence(this) { it.parentFile }.any { it.isHidden }
```

## 자바 함수형 인터페이스 활용

추상 메소드가 단 하나만 있는 인터페이스를 함수형 인터페이스 또는 SAM 인터페이스라고 한다. SAM은 단일 추상 메소드(Single Abstract Method)라는 뜻이다. 코틀린은 SAM 인터페이스를 람다로 넘길 수 있게 해준다.

예시)

```java
button.setOnClickListener(new OnClickListener() {
	@Override
    public void onClick(View v) { }
```

```kotlin
button.setOnClickListener { view -> }
```

### 자바 메소드에 람다를 인자로 전달

함수형 인터페이스를 인자로 원하는 자바 메소드에 코틀린 람다를 전달할 수 있다.

```java
void postponeComputation(int delay, Runnable computation);
```

```kotlin
postponeComputation(1000) { println(42) }
```

무명 객체를 만들어서 사용할 수도 있다.

```kotlin
postponeComputation(1000, object : Runnable {
	override fun run() {
    	println(42)
    }
}
```

무명 객체를 명시적으로 선언하는 경우 메소드를 호출할 때마다 새로운 객체가 생성되는 반면 람다는 대응하는 무명 객체 메소드를 호출할 때마다 반복 사용한다.

단, 람다가 주변 영역의 변수를 포획한다면 매 호출마다 같은 인스턴스를 사용할 수 없다.

```kotlin
fun handleComputation(id: String) {
	postponeComputation(1000) { println(id) }
}
```

위 함수에서는 id를 필드로 저장하는 새로운 Runnable 인스턴스를 매번 새로 만들어 사용한다.

### SAM 생성자: 람다를 함수형 인터페이스로 명시적으로 변경

컴파일러가 자동으로 람다를 함수형 인터페이스 무명 클래스로 바꾸지 못하는 경우 SAM 생성자를 사용한다. 예를 들어 함수형 인터페이스의 인스턴스를 반환하는 메소드가 있다면 람다를 직접 반환할 수 없고, 반환하고픈 람다를 SAM 생성자로 감싸야 한다.

```kotlin
fun createAllDoneRunnable() : Runnable {
	return Runnable { println("All done!") } // { println("All done!") } 이런 식으로 람다만 쓰는거 불가능
}
```

SAM 생성자의 이름은 함수형 인터페이스의 이름과 같다.

## 수신 객체 지정 람다: with와 apply

### with 함수

with 함수는 첫 번째 인자로 바든 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다. 인자로 만든 람다 본문에서는 this를 사용해 그 수신 객체에 접근할 수 있다.

```kotlin
fun alphabet() : String {
	val stringBuilder = StringBuilder()
    return with(stringBuilder) {
    	for (letter in 'A' .. 'Z') {
        	this.append(letter)
        }
        append("\nNow I know the alphbet!")
        this.toString()
    }
}
```

with가 반환하는 값은 람다 코드를 실행한 결과며, 그 결과는 람다 식의 본문에 있는 마지막 식의 값이다.

때로는 람다의 결과 대신 수신 객체가 필요한 경우도 있다. 그럴 때는 apply를 사용한다.

### apply 함수

apply와 with의 차이는 apply는 항상 자신에게 전달된 객체를 반환한다.

apply 함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다.