---
title: "[Kotlin in Action] 2장. 코틀린 기초"
datePublished: Wed Jul 05 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmesa750000t09l1ass46jn9
slug: kotlin-in-action-2
tags: kotlin

---

## 기본 요소 : 함수와 변수

### Hello, World!

* 함수를 선언할 때 `fun` 키워드 사용
    
* 파라미터 이름 뒤에 그 파라미터의 타입을 쓴다.
    
* 함수 최상위 수준 정의 가능
    
* 배열도 일반적인 클래스로 취급 (자바의 `int[]` 같은 함수 없음)
    
* 여러가지 표준 자바 라이브러리 함수를 간결하게 사용 가능
    

### 함수

```kotlin
fun max(a: Int, b: Int): Int {
	return if(a>b) a else b
}
```

코틀린 if는 결과를 만드는 식이다.

* 문과 식 식 : 값을 만들어 냄 문 : 아무런 값을 만들어내지 못함
    

#### 식이 본문인 함수

```kotlin
fun max(a: Int, b: Int) = if(a>b) a else b // 반환 타입 생략 가능
```

### 변수

```kotlin
val kotlin = "kotlin"
val answer: Int = 42
```

#### 변경 가능한 변수와 변경 불가능한 변수

* val - 변경 불가능, 자바로 말하자면 final
    
* var - 변경 가능
    

val의 경우 블록을 실행할 때 정확히 한 번만 초기화돼야 한다. 하지만 어떤 블록이 실행될 때 초기화 문장이 하나만 실행된다면 조건에 따라 val 값을 다른 여러 값으로 초기화 가능하다.

```kotlin
val message: String
if (canPerformOperation()) { message = "Success" }
else { message = "Fail" }
```

***val 참조 자체는 불변일지라도 그 참조가 가리키는 객체의 내부 값은 변경될 수 있다.***

## 문자열 템플릿

```kotlin
printf("Hello, $name")
```

## 클래스와 프로퍼티

```kotlin
class Person(val name: String)
```

### 프로퍼티

* 필드(field) : 자바에서 데이터를 저장하는 곳
    
* 접근자 메소드(accessor method) 데이터에 접근하는 통로로 사용 게터(getter), 세터(setter)가 있음.
    
* **프로퍼티(property)** : 필드와 접근자를 묶은 것
    

val : 읽기 전용 프로퍼티 var : 변경 가능한 프로퍼티

**뒷받침하는 필드(**`backing field`) : 프로퍼티의 값을 저장하기 위한 필드

---

코틀린 클래스에서는 필드를 직접적으로 선언할 수 없습니다. 따라서 값을 저장하는 동시에 로직을 실행할 수 있게 하기 위해서는 접근자 안에서 프로퍼티를 뒷받침하는 필드(backing field)가 있어야 합니다. 접근자의 본문에서는 field 식별자를 이용하여 backing field에 접근할 수 있습니다. getter에서는 field값을 읽을수만 있고 setter에서는 field 값을 읽거나 쓸 수 있습니다.

```kotlin
var counter = 0 // 이 initializer는 backing field를 직접 할당함.
    set(value) {
        if(value >= 0) field = value
    }
```

여기서 사용된 field 식별자는 이처럼 프로퍼티의 접근자에서만 사용될 수 있습니다. backing field는 최소한 하나 이상의 접근자로 기본 구현을 사용하거나, 커스텀 접근자가 field 식별자를 이용해 backing field를 참조할 때 생성됩니다. 예를 들어 아래와 같은 예시에서는 backing field가 없습니다.

```kotlin
// 예시 1.
val isEmpty: Boolean
    get() = this.size == 0

// 예시 2.
var name: String // get, set
    get() {
        return "User"
    }
```

By. [https://shinjekim.github.io/kotlin/2019/09/02/Kotlin-%ED%94%84%EB%A1%9C%ED%8D%BC%ED%8B%B0%EC%99%80-%ED%95%84%EB%93%9C(Properties-and-Fields)/](https://shinjekim.github.io/kotlin/2019/09/02/Kotlin-%ED%94%84%EB%A1%9C%ED%8D%BC%ED%8B%B0%EC%99%80-%ED%95%84%EB%93%9C(Properties-and-Fields)/)

---

### 커스텀 접근자

```kotlin
class Rectangle(val height: Int, val width: Int) {
	val isSquare: Boolean
    	get() {
        	return height == width
        }
```

파라미터가 없는 함수를 정의하는 방식과 커스텀 게터를 정의하는 방식에서 구현이나 성능상의 차이는 없다. 일반적으로 클래스의 특성을 정의하고 싶다면 프로퍼티로 그 특성을 정의해야 한다.

### 디텍터리와 패키지

같은 패키지에 속해 있다면 다른 파일에서 정의한 선언일지라도 직접 사용할 수 있다. 반면 다른 패키지에 정의한 선언을 사용하려면 임포트를 통해 불러와야 한다.

## 선택 표현과 처리: enum과 when

```kotlin
enum class Color(
	val r: Int, val g: Int, val b: Int
) {
	RED(255,0,0), BLUE(255,165,0); //세미콜론 주의
    
    fun rgb() = (r * 256 + g) * 256 + b
}
```

emum 클래스 안에 메소드를 정의하는 경우 반드시 enum 상수 목록과 메서드 정의 사이에 세미콜론을 넣어야 한다.

### when으로 enum 다루기

```kotlin
fun getMnemonic(color: Color) = 
	when (color) {
    	Color.RED -> "Richard"
        Color.Blue -> "Of"
        Color.ORANGE, Color.YELLOW -> "warm"
    }
```

자바와 달리 break를 넣지 않아도 된다.

### when과 임의의 객체를 함께 사용

코틀린 when의 분기 조건은 임의의 객체를 허용한다.

```kotlin
fun mix(c1: Color, c2: Color) = 
	when(setOf(c1, c2)) {
    	setOf(RED, YELLOW) -> ORANGE
        else -> throw Exceptioin("Dirty Color")
    }
```

여러 개의 setOf를 생성하기 때문에 비효율적

### 인자 없는 when 사용

```kotlin
fun mix(c1: Color, c2: Color) = 
	when {
    	c1 == RED && c2 == YELLOW -> ORANGE
        else -> throw Exceptioin("Dirty Color")
    }
```

효율적이지만 가독성이 떨어진다.

when에 아무 인자도 없으려면 각 분기의 조건이 불리언 결과를 계산하는 식이어야 한다.

## 스마트 캐스트: 타입 검사와 타입 캐스트를 조합

코틀린에서는 `is`를 사용해 변수 타입을 검사한다. 어떤 변수가 원하는 타입인지 일단 `is`로 검사하고 나면 컴파일러가 자동으로 캐스팅을 수행해준다. 이를 ***스마트 캐스트***라고 부른다.

스마트 캐스트는 반드시 커스텀 접근자를 사용하지 않은 val이어야한다. 그렇지 않으면 항상 같은 Type이 아니기 때문이다.

원하는 타입으로 명시적으로 타입 캐스팅하러면 `as` 키워드를 사용한다.

### if와 when의 분기에서 블록 사용

if나 when 모두 분기에 블록을 사용할 수 있다. 그런 경우 블록의 마지막 문장이 블록 전체의 결과가 된다.

## 대상을 이터레이션: while과 for 루프

코틀린 while 루프는 자바와 동일하다 for는 자바의 for-each 루프에 해당하는 형태만 존재한다.

### 수에 대한 이터레이션: 범위와 수열

초깃값, 증가 값, 최종 값을 루프에서 사용하기 위해 코틀린에서는 범위(range)를 사용한다.

코틀린의 범위는 폐구간(닫힌 구간) 또는 양끝을 포함하는 구간이다.

```kotlin
for(i in 1..100) {
	//100도 포함
}

for(i in 100 downTo 1 step 2) {}

for(i in 1 until 100) {
	//100 미포함
}
```

### 맵에 대한 이터레이션

```kotlin
val binaryReps = TreeMap<Char, String>()

for (c in 'A' .. 'F') {
	val binary = Integar.toBinaryString(c.toInt())
    binaryReps[c] = binary
}

for ((letter, binary) in binaryReps) {
	println("$letter = $binary")
}
```

맵에 사용했던 구조 분해 구문을 맵이 아닌 컬렉션에도 활용할 수 있다.

```kotlin
val list = arrayListOf("10","11","1001")
for ((index, element) in list.withIndex()) {
	println("$index: $element)
}
```

### in으로 컬렉션이나 범위의 원소 검사

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c in '0'..'9'
```

범위는 문자에만 국한되지 않고 비교 가능한 클래스라면 (java.lang.Comparable 인터페이스를 구현한 클래스라면) 그 클래스의 인스턴스 객체를 사용해 범위를 만들 수 있다.

## 코틀린의 예외 처리

자바와 달리 코틀린의 throw는 식이므로 다른 식에 포함될 수 있다.

```kotlin
val percentage = if(number in 0..100) number
				else throw Exception("예외")
```

조건이 거짓인 경우 변수가 초기화되지 않는다. 자세한건 6.2.6절에서 설명한다.

### try를 식으로 사용

```kotlin
fun readNumber(reader: BufferedReader) {
	val number = try {
		Integel.parselnt(reader.readLine())
) catch(e: NumberFormatException){
		null //return도 가능
}
println(number)
```