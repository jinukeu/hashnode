---
title: "[Kotlin in Action] 7장. 연산자 오버로딩과 기타 관례"
datePublished: Wed Sep 06 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmesha2d000408jpdmf047ao
slug: kotlin-in-action-7
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696821337231/137fc3cb-2a48-48b8-b391-129834fb2215.jpeg
tags: kotlin

---

## 산술 연산자 오버로딩

### 이항 산술 연산 오버로딩

```kotlin
data class Point(val x: Int, val y: Int) {
	operator fun plus(other: Point): Point {
    	return Point(x + other.x, y + other.y)
        }
    }


val p1 = Point(10, 20)
val p2 = Point(30, 40)
println(p1 + p2)
```

연산자를 오버로딩하는 함수 앞에는 꼭 operator가 있어야한다. operator를 plus 앞에 붙임으로써 + 기호를 오버라이딩했다.

연산자를 확장 함수로도 정의할 수 있다.

```kotlin
operator fun Point.plus(other: Point)
```

| 식 | 함수 이름 |
| --- | --- |
| a \* b | times |
| a / b | div |
| a % b | rem |
| a + b | plus |
| a - b | minus |

두 피연산자의 타입이 다른 연산자도 정의할 수 있다.

```kotlin
operator fun Point.times(scale: Double): Point {
	return Point((x * scale).toInt(), (y * scale).toInt())
    }
```

위 예제는 p *1.5로만 사용할 수 있으며 1.5* p와 같은 형식으로 사용하려면 대응하는 연산자 함수를 또 만들어줘야 한다.

### 복합 대입 연산자 오버로딩

+=, -= 등의 연산자를 복합 대입 연산자라 한다.

반환 타입이 Unit인 plusAssign 함수를 정의하면 코틀린은 += 연산자에 그 함수를 사용한다.

```kotlin
operator fun <T> MutableCollection<T>.plusAssign(element: T) {
	this.add(element)
}
```

a += b는 a = [a.plus](http://a.plus)(b) 또는 a.plusAssign(b) 두 가지로 컴파일될 수도 있다.

+, -는 항상 새로운 컬렉션을 반환하며 +=과 -= 연산자는 변경 가능한 컬렉션에 작용해 메모리에 있는 객체 상태를 변화시킨다.

### 단항 연산자 오버로딩

단항 연산자 예시 : -a

```kotlin
operator fun Point.unaryMinus(): Point {
	return Point(-x, -y)
}

val p = Point(10, 20)
println(-p)
```

오버로딩할 수 있는 단항 산술 연산자

| 식 | 함수 이름 |
| --- | --- |
| +a | unaryPlus |
| \-a | unaryMinus |
| !a | not |
| ++a, a++ | inc |
| \--a, a-- | dec |

## 비교 연산자 오버로딩

### 동등성 연산자: equals

코틀린은 == 연산자 호출을 equals 메소드 호출로 컴파일한다. ===는 자신의 두 피연산자가 서로 같은 객체를 가리키는지 비교한다.

### 순서 연산자: compareTo

```kotlin
class Person(
	val firstName: String, val lastName: String
) : Comparable<Person> {
	override fun compareTo(other: Person): Int {
    	return compareValuesBy(this, other, Person::lastName, Person::firstName)
    }
}
```

## 컬렉션의 범위에 대해 쓸 수 있는 관례

### 인덱스로 원소에 접근: get과 set

```kotlin
operator fun Point.get(index: Int): Int {
	return when(index) {
    	0 -> x
        1 -> y
        else -> throw ...
    }
}

val p = Point(10, 20)
println(p[1])
```

2차원 행렬이나 배열을 표현하는 클래스에 operator fun get(rowIndex: Int, colIndex: Int)를 정의하면 matrix\[row, col\] 로 호출할 수 있다.

### in 관례

in은 객체가 컬렉션에 들어있는지 검사한다. 그런 경우 in 연산자와 대응하는 함수는 contains다.

```kotlin
data class Rectangle(val upperLeft: Point, val lowerRight: Point)

operator fun Rectangle.contains(p: Point): Boolean {
	return p.x in upperLeft.x until lowerRight.x && p.y in upperLeft.y until lowerRight.y
}

val rect = Rectangle(Point(10, 20), Point(50, 50))
println(Point(20, 20) in rect)
```

### rangeTo 관례

.. 연산자는 rangeTo 함수를 간략하게 표현하는 방법이다. 어떤 클래스가 Comparable 인터페이스를 구현하면 rangeTo를 정의할 필요 없다.

코틀린 표준 라이브러리에는 모든 Comparable 객체에 대해 적용 가능한 rangeTo 함수가 들어있다.

```kotlin
val now = LocalDate.now()
val vacation = now..now.plusDays(10)
println(now.plusWeeks(1) in vacation)
```

### for 루프를 위한 iterator 관례

```kotlin
operator fun ClosedRange<LocalDate>.iterator() : Iterator<LocalDate> = object : Iterator<LocalDate> {
	var current = start
    
    override fun hasNext() = current <= endInclusive
    override fun next() = current.apply {
    	current = plusDays(1)
    }
}

val newYear = LocalDate.ofYearDay(2017, 1)
val daysOff = newYear.minusDays(1)..newYear
for (dayOff in daysOff) { println(dayOff) }

2016-12-31
2017-01-01
```

## 구조 분해 선언과 component 함수

구조 분해를 사용하면 복합적인 값을 분해해서 여러 다른 변수를 한꺼번에 초기화할 수 있다.

data 클래스의 주 생성자에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 componentN 함수를 만들어준다.

```kotlin
class Point(val x: Int, val y: Int) {
	operator fun component1() = x
    operator fun component2() = y
}

val (a, b) = p // val a = p.component1(), val b = p.component2()
```

## 프로퍼티 접근자 로직 재활용: 위임 프로퍼티

### 위임 프로퍼티 소개

간단 예시

```kotlin
class Delegate {
	operator fun getValue(...) { ... }
    operator fun setValue(..., value: Type) { ... }
}

class Foo {
	var p: Type by Delegate()
}
```

p의 set, get은 Delegate 클래스의 getValue, setValue로 대체된다.

```kotlin
val ov = foo.p // foo.p는 delegate의 getValue를 호출
foo.p = nv // delegate의 setValue를 호출
```

### 위임 프로퍼티 사용: by lazy()를 사용한 프로퍼티 초기화 지연

지연 초기화는 객체의 일부분을 초기화하지 않고 남겨뒀다가 실제로 그 부분의 값이 필요한 경우 초기화할때 흔히 사용한다.

### 프로퍼티 값을 맵에 저장

```kotlin
class Person {
	private val _attributes = hashMapOf<String, String>()
    
    fun setAttribute(attrName: String, value: String) {
     _attributes[attrName] = value
 }
 
 	val name: String by _attributes
}
```

표준 라이브러리가 Map과 MutableMap 인터페이스에 대해 getValue와 setValue 확장 함수를 제공하기 때문에 가능하다.

[p.name](http://p.name)\["name"\] 호출은 \_attributes\["name"\]과 같다.