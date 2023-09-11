---
title: "[Kotlin in Action] 4장. 클래스, 객체, 인터페이스"
datePublished: Wed Jul 19 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmescb91000108jpfx5heab9
slug: kotlin-in-action-4
tags: kotlin

---

## 클래스 계층 정의

### 코틀린 인터페이스

코틀린 인터페이스 안에는 추상 메소드뿐 아니라 구현이 있는 메소드도 정의할 수 있다. 단, 인터페이스에는 아무런 상태(필드)도 들어갈 수 없다.

```kotlin
interface Clickable {
	fun click() // 일반 메소드 선언
    fun showOff() = println("I'm clickable!") // 디폴트 구현이 있는 메소드
```

showOff 메소드를 정의하는 다른 인터페이스가 있으면 어떻게 될까?

```kotlin
interface Focusable {
	fun setFocus(b: Boolean) =  
    	println("I ${if (b) "got" else "lost"} focus.")
        
    fun showOff() = println("I'm focusable!")
```

이름과 시그니처가 같은 멤버 메서도에 대해 둘 이상의 디폴트 구현이 있는 경우 인터페이스를 구현하는 하위 클래스에서 명시적으로 새로운 구현을 제공해야 한다.

```kotlin
class Button: Clickable, Focusable {
	override fun click() = println("I was clicked")
    override fun showOff() {
    	super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

### open, final, abstract 변경자: 기본적으로 final

어떤 클래스의 상속을 허용하려면 클래스 앞에 open 변경자를 붙여야 한다. 그와 더불어 오버라이드를 허용하고 싶은 메소드나 프로퍼티의 앞에도 open 변경자를 붙여야 한다.

> 왜 open을 붙여야할까? 기반 클래스를 작성한 사람의 의도와 다른 방식으로 메소드를 오버라이드할 위험이 있기 때문. 기반 클래스를 변경하는 경우 하위 클래스의 동작이 예기치 않게 바뀔 수도 있다.

기반 클래스나 인터페이스의 멤버를 오버라이드하는 경우 그 메소드는 기본적으로 열려있다.

```kotlin
open class RichButton : Clickable {
	fun disable() {} // 파이널, 하위 클래스에서 오버라이드 불가
    
    open fun animate() // 하위 클래스에서 오버라이드 가능
    
	override fun click() {} //상위 클래스에서 메소드 오버라이드함. 오버라이드한 메소드는 기본적으로 열려있다.
}
```

오버라이드하는 메소드의 구현을 하위 클래스에서 오버라이드하지 못하게 금지하려면 오버라이드하는 메소드 앞에 final을 명시해야 한다.

```kotlin
open class RichButton : Clickable {
	final override fun click() {} //final이 없는 override 메소드나 프로퍼티는 기본적으로 열려있다.
}
```

abstract로 선언한 추상 클래스는 구현이 없는 추상 멤버가 있기 때문에 인스턴스화할 수 없다.

```kotlin
abstract class Animated {
	abstract fun animate()  // 하위 클래스에서 반드시 오버라이드
    open fun stopAnimation() {} // 오버라이드 허용
    fun animateTwice() {} // final 오버라이드 불가
```

| 변경자 | 이 변경자가 붙은 멤버는 ... | 설명 |
| --- | --- | --- |
| final | 오버라이드 불가 | 클래스 멤버의 기본 변경자 |
| open | 오버라이드 가능 | 반드시 open을 명시해야 오버라이드할 수 있다. |
| abstract | 반드시 오버라이드 | 추상 클래스의 멤버에만 이 변경자를 붙일 수 있다. 추상 멤버에는 구현이 있으면 안된다. |
| override | 상위 클래스나 상위 인스턴스의 멤버를 오버라이드 하는 중 | 오버라이드하는 멤버는 기본적으로 열려있다. 하위 클래스의 오버라이드를 금지하려면 final을 명시해야 한다. |

### 가시성 변경자: 기본적으로 공개

| 변경자 | 클래스 멤버 | 최상위 선언 |
| --- | --- | --- |
| public(기본 가시성) | 모든 곳에서 볼 수 있다. | 모든 곳에서 볼 수 있다. |
| internal | 같은 모듈 안에서만 볼 수 있다. | 같은 모듈 안에서만 볼 수 있다. |
| protected | 하위 클래스 안에서만 볼 수 있다. | (최상위 선언에 적용 불가) |
| private | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 안에서만 볼 수 있다. |

```kotlin
internal open class TalkactiveButton : Focusable {
	private fun yell() = println("Hey!")
    protected fun whisper() = println("Let's talk!")
}

fun TalkactiveButton.giveSpeech() { //public fun에서 internal class에 접근 불가
	yell() // 확장 함수는 private에 접근 불가
    whisper() // 확장 함수는 protected에 접근 불가
}
```

> protected 자바와 코틀린 차이점 자바 : 같은 패키지안에서 protected 접근 가능 코틀린 : 오직 어떤 클래스나 그 클래스를 상속한 클래스 안에서만 보임

> 코틀린에서는 자바와 달리 외부 클래스가 내부 클래스나 중첩된 클래스의 private 멤버에 접근할 수 없다.

### 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

코틀린의 중첩 클래스는 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다. 내부 클래스로 변경해서 바깥쪽 클래스에 대한 참조를 포함하게 만들고 싶다면 inner 변경자를 붙여야 한다.

### 봉인된 (sealed) 클래스: 클래스 계층 정의 시 계층 확장 제한

상위 클래스에 sealed 변경자를 붙이면 그 상위 클래스를 상속한 하위 클래스 정의를 제한할 수 있다.

```kotlin
sealed class Expr {
	class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}

// "when" 식이 모든 하위 클래스를 검사하므로 별도의 "else" 분기가 없어도 된다.
fun eval(e: Expr): Int = 
	when(e) { 
    	is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```

## 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

### 클래스 초기화: 주 생성자와 초기화 블록

```kotlin
class User(val nickname: String)
```

클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 주 생성자라고 부른다.

```kotlin
class User(_nickname: String) {
	val nickname: String
    
    init {
    	nickname = _nickname
    }
}
```

constructor 키워드는 주 생성자나 부 생성자 정의를 시작할 때 사용한다. init 키워드는 초기화 블록을 시작한다.

```kotlin
class User(_nickname: String) {
	val nickname = _nickname
}
```

프로퍼티를 초기화하는 식이나 초기화 블록 안에서만 주 생성자의 파라미터를 참조할 수 있다.

```kotlin
class User(val nickname: String) // val은 이 파라미터에 상응하는 프로퍼티가 생성된다는 뜻이다.
```

### 상위 클래스를 다른 방식으로 초기화

부 생성자는 constructor 키워드로 시작한다.

```kotlin
class MyButton : View {
	constructor(ctx: Context) : super(ctx) {
    	// ...
    }
    
    constructor(ctx: Context, attr: AttributeSet) : super(ctx, attr) {
    	// ...
    }
    
    constructor(ctx: Context): this(ctx, MY_STYLE)
```

this()를 통해 클래스 자신의 다른 생성자를 호출할 수 있다.

### 인터페이스에 선언된 프로퍼티 구현

```kotlin
interface User {
	val nickname: String
}
```

### 게터와 세터에서 뒷받침하는 필드에 접근

프로퍼티에 저장된 값의 변경 이력을 로그에 남기려는 경우

```kotlin
class User(val name: String) {
	var address: String = "unspecified"
    	set(value: String) {
        	println("""
            	Address was changed for $name:
                "$field" -> "$value".""".trimIndent())
            field = value
        }
}
```

### 접근자의 가시성 변경

```kotlin
class LengthCounter {
	var counter: Int = 0
    	private set // 클래스 안에서만 값을 바꿀 수 있음
    
    fun addWord(word: String) {
    	counter += word.length
    }
}
```

## 컴파일러가 생성한 메소드: 데이터 클래스와 클래스 위임

### 모든 클래스가 정의해야 하는 메소드

코틀린에서 == 연산자는 참조 동일성을 검사하지 않고 객체의 동등성을 검사한다. 따라서 == 연산은 equals를 호출하는 식으로 컴파일된다. 참조 비교를 위해서는 === 연산자를 사용할 수 있다.

#### 해시 컨테이너: hashCode()

HashSet은 원소를 비교할 때 비용을 줄이기 위해 먼저 객체의 해시 코드를 비교하고 해시 코드가 같은 경우에만 실제 값을 비교한다.

```kotlin
val processed = hashSetOf(Client("오현석", 4122))
println(processed.contains(Client("오현석", 4122)))
```

false

Client에서 equals를 올바르게 구현했다 해도 해시 코드가 다르기 때문에 false가 나온다.

따라서 이 문제를 고치려면 hashCode를 직접 구현해야 한다.

```kotlin
class Client(val name: String, val postalCode: Int) {

 ...
 override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

### 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

data라는 변경자를 클래스 앞에 붙이면 toString, equals, hashCode를 컴파일러가 자동으로 만들어준다.

#### 데이터 클래스와 불변성: copy() 메소드

데이터 클래스의 모든 프로퍼티를 읽기 전용으로 만들어서 데이터 클래스를 불변 클래스로 만들라고 권장한다.

why?

1. 데이터 클래스 객체를 키로 하는 값을 컨테이너에 담은 다음에 키로 쓰인 데이터 객체의 프로퍼티를 변경하면 컨테이너 상태가 잘못될 수 있다.
    

```kotlin
data class Data(val id: Int, var name: String)

fun main() {
    val container = hashMapOf<Data, String>()

    val data1 = Data(1, "Alice")
    val data2 = Data(2, "Bob")

    container[data1] = "Value 1"
    container[data2] = "Value 2"

    println(container)
    println(container[data1])

    data1.name = "Charlie"

    println(container)
    println(container[data1])
}
```

출력 결과

```kotlin
{Data(id=2, name=Bob)=Value 2, Data(id=1, name=Alice)=Value 1}
Value 1
{Data(id=2, name=Bob)=Value 2, Data(id=1, name=Charlie)=Value 1}
null
```

container에 값을 추가한 후, data1의 name을 변경하면 container의 상태가 바뀌게 됩니다. 이는 HashMap이 내부적으로 키 객체의 해시값을 사용하여 데이터를 저장하고 검색하기 때문입니다. data1의 name을 변경하면 해시값이 변경되어 컨테이너 내에서 올바른 위치를 찾지 못하게 됩니다.

1. 다른 코드에서 객체를 사용하더라도 객체의 상태가 변하지 않기 때문에 쉽게 추론할 수 있다.
    
2. 스레드가 사용 중인 데이터를 다른 스레드가 변경할 수 없으므로 스레드를 동기화해야 할 필요가 줄어든다.
    

copy 메소드를 사용하면 일부 프로퍼티를 변경하면서 객체의 복사본을 만들 수 있다. 이렇게하면 원본을 참조하는 다른 부분에 전혀 영향을 미치지 않는다.

### 클래스 위임: by 키워드 사용

상속을 허용하지 않는 클래스에 새로운 동작을 추가할 때 주로 데코레이터 패턴을 사용한다. 상속을 허용하지 않는 클래스(A) 대신 새로운 클래스(B)를 만들되 기존 클래스(A)와 같은 인터페이스를 데코레이터(B)가 제공하게 만들고 기존 클래스(A)를 데코레이터 내부에 필드로 유지하는 것이다.

새로 정의해야 하는 기능은 데코레이터의 메소드에 새로 정의하고 기존 기능이 그대로 필요한 부분은 데코레이터의 메소드가 기존 클래스의 메소드에게 요청을 전달한다.

```kotlin
class DelegatingCollection<T> : Collection<T> {
	private val innerList = arrayListOf<T>()
    
    override val size: Int get() = innerList.size
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun ...
    override fun ...
    override fun ...
    override fun ...
}
```

인터페이스를 구현할 때 by 키워드를 통해 그 인터페이스에 대한 구현을 다른 객체에 위임 중이라는 사실을 명시할 수 있다.

```kotlin
class DelegatingCollection<T>(
	innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList {}
```

최종 예시

```kotlin
class CountingSet<T>(
	val innerSet: MutableCollection<T> = HashSet<T>()
) : MutableCollection<T> by innerSet {
    
    var objectsAdded = 0
    
    override fun add(element: T): Boolean {
    	objectsAdded++
        return innerSet.add(element)
    }
    
    override fun addAll(c: Collection<T>): Boolean {
    	objectsAdded += c.size
        return innserSet.addAll(c)
    }
```

## object 키워드: 클래스 선언과 인스턴스 생성

### 객체(object) 선언: 싱글턴을 쉽게 만들기

코틀린은 객체 선언 기능을 통해 싱글턴을 언어에서 기본 지원한다.

```kotlin
object Payroll {
	val allEmployees = arrayListOf<Person>()
    
    fun calculateSalary() {
    	for (person in allEmployees) {
        	...
        }
    }
}
```

일반 클래스 인스턴스와 달리 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출 없이 즉시 만들어진다. 따라서 생성자를 만들 수 없다.

object도 클래스나 인스턴스를 상속할 수 있다.

### 동반 객체: 팩토리 메소드와 정적 멤버가 들어갈 장소

최상위 함수는 private로 표시된 클래스 비공개 맴버에 접근할 수 없다. 클래스의 인스턴스와 관계없이 호출해야 하지만, 클래스 내부 정보에 접근해야 하는 함수가 필요할 때 클래스에 중첩된 객체 선언(object)의 멤버 함수로 정의해야 한다.

클래스 안에 정의된 object 중 하나에 companion이라는 걸 붙이면 동반 객체로 만들 수 있다.

동반 객체는 자신을 둘러싼 클래스의 모든 private 멤버에 접근할 수 있다. (일반 object도 가능) 동반 객체는 바깥쪽 클래스의 private 생성자를 호출할 수 있기 때문에 팩토리 패턴을 구현하기 가장 적합하다.

```kotlin
class User private construtor(val nickname: String) {
	companion object {
    	fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
        
        fun newFacebookUser(accountId: Int) = User(getFacebookName(accountId))
    }
}
```

#### 팩토리 메소드 장점

1. 목적에 따라 팩토리 메소드 이름을 정할 수 있다.
    
2. 팩토리 메소드가 선언된 클래스의 하위 클래스 객체를 반환할 수도 있다. (SubscribingUser와 FacebookUser가 따로 존재한다면 필요에 따라 적당한 클래스의 객체 반환 가능)
    
3. 생성할 필요가 없는 객체를 생성하지 않을 수도 있다. (이메일 주소별로 유일한 User 인스턴스를 만드는 경우 팩토리 메소드가 이미 존재하는 인스턴스에 해당하는 이메일 주소를 전달받으면 새 인스턴스를 만들지 않고 캐시에 있는 기존 인스턴스를 반환할 수 있다.)
    

### 동반 객체를 일반 객체처럼 사용

동반 객체에 이름을 붙이거나, 인터페이스를 상속하거나, 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.

### 객체 식: 무명 내부 클래스를 다른 방식으로 작성

무명 객체를 정의할 때도 object 키워드를 쓴다.

```kotlin
val listener = object : MouseAdapter() {
	override fun ...
    override fun ...
}
```

객체 선언과 달리 무명 객체는 싱글턴이 아니다.

자바 무명 내부 클래스와 달리 코틀린 무명 클래스는 여러 인터페이스를 구현하거나 클래스를 확장하면서 인터페이스를 구현할 수 있다.