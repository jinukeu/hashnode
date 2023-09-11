---
title: "[Kotlin in Action] 3장. 함수 정의와 호출"
datePublished: Tue Jul 11 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmesbbiw000v09l1480y9e1f
slug: kotlin-in-action-3
tags: kotlin

---

## 함수를 호출하기 쉽게 만들기

### 이름 붙인 인자 (Named arguments)

```kotlin
joinToString(collection, separator = " ", prefix = " ", postfix = ".")
```

인자로 전달한 각 문자열이 어떤 역할을 하는지 명시적으로 적을 수 있다.

### 디폴트 파라미터 값

코틀린에서는 함수 선언에서 파라미터의 디폴트 값을 지정할 수 있다.

```kotlin
fun <T> joinToString(
	collection: Collectioin<T>,
    separator: String = "",
    prefix: String = "",
    postfix: String = ""
): String
```

#### 디폴트 값과 자바

자바에는 디폴트 파라미터라는 개념이 없어서 코틀린 함수를 자바에서 호출하는 경우에는 그 코틀린 함수가 디폴트 파라미터 값을 제공하더라도 모든 인자를 명시해야 한다. 자바에서 코틀린 함수를 호출할 때 @JvmOverloads 애노테이션을 함수에 추가하면 코틀린 컴파일러가 자동으로 맨 마지막 파라미터로부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 추가해준다.

---

[https://holika.tistory.com/entry/%EB%82%B4-%EB%A7%98%EB%8C%80%EB%A1%9C-%EC%A0%95%EB%A6%AC%ED%95%9C-Kotlin-JvmOverloads-constructor%EB%A5%BC-%EC%9D%BC%EC%9D%BC%EC%9D%B4-%EC%83%81%EC%86%8D%EB%B0%9B%EC%95%84-%EB%A7%8C%EB%93%A4%EA%B8%B0-%EA%B7%80%EC%B0%AE%EB%8B%A4%EB%A9%B4](https://holika.tistory.com/entry/%EB%82%B4-%EB%A7%98%EB%8C%80%EB%A1%9C-%EC%A0%95%EB%A6%AC%ED%95%9C-Kotlin-JvmOverloads-constructor%EB%A5%BC-%EC%9D%BC%EC%9D%BC%EC%9D%B4-%EC%83%81%EC%86%8D%EB%B0%9B%EC%95%84-%EB%A7%8C%EB%93%A4%EA%B8%B0-%EA%B7%80%EC%B0%AE%EB%8B%A4%EB%A9%B4)

View를 상속받은 클래스를 생성할 때에는, 아래 3개의 생성자가 필요하다.

```kotlin
View(context: Context)
View(context: Context, attrs: AttributeSet)
View(context: Context, attrs: AttributeSet, defStyleRes: Int)
```

@JvmOverloads는 위와 같은 생성자 오버로딩을 자동으로 생성해 주는 어노테이션이다.

이 어노테이션을 사용하면 위와 같은 보일러플레이트 코드를 최소화할 수 있다. 위의 Student 클래스를 @JvmOverloads를 사용하여 작성하면 이렇다.

```kotlin
class CustomView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    @AttrRes defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr, defStyleRes)
```

이를 디컴파일해 보면 실제로 아래와 같은 생성자들이 모두 생성된다.

```kotlin
public CustomView(@NotNull Context context) 
public CustomView(@NotNull Context context, @Nullable AttributeSet attrs) 
public CustomView(@NotNull Context context, @Nullable AttributeSet attrs, @AttrRes int defStyleAttr) 
public CustomView(Context var1, AttributeSet var2, int var3, int var4, DefaultConstructorMarker var5)
```

---

### 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

```kotlin
package strings

fun joinToString(...): String { ... }
```

위 join.kt를 컴파일한 결과와 같은 클래스를 자바 코드로 보면 다음과 같다.

```java
package strings;

public class JoinKt {
	public static String joinToString(...) { ... }
}
```

코틀린 컴파일러가 생성하는 클래스의 이름은 최상위 함수가 들어있던 코틀린 소스 파일의 이름과 대응한다. 코틀린 파일의 모든 최상위 함수는 이 클래스의 정적인 메소드가 된다.

### 최상위 프로퍼티

함수와 마찬가지로 프로퍼티도 파일의 최상위 수준에 놓을 수 있다. 이런 프로퍼티의 값은 정적 필드에 저장된다. const 변경자를 추가하면 프로퍼티를 public static final 필드로 컴파일하게 만들 수 있다.

```kotlin
const val UNIX_LINE_SEPARATOR = "\n"
```

```java
/* 자바 */
public static final String UNIX_LINE_SEPARATOR = "\n";
```

## 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

확장 함수는 어떤 클래스의 멤버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.

```kotlin
package strings

fun String.lastChar(): Char = this.get(this.length - 1)

println("Kotlin".lastChar())
```

String이 자바나 코틀린 등의 언어 중 어떤 것으로 작성됐는가는 중요하지 않다. 예를 들어 그루비와 같은 다른 JVM 언어로 작성된 클래스도 확장할 수 있다.

확장 함수는 클래스 내부에서만 사용할 수 있는 private, protected 멤버를 사용할 수 없다.

### 임포트와 확장 함수

as 키워드를 사용하면 임포트한 클래스나 함수를 다른 이름으로 부를 수 있다.

```kotlin
import strings.lastChar as last

val c = "Kotlin".last()
```

### 확장 함수는 오버라이드할 수 없다.

```kotlin
fun View.showOff() = println("I'm a view!")
fun Button.showOff() = println("I'm a button!")

val view: View = Button()
view.showOff()
```

I'm a view!

view가 가리키는 객체의 실제 타입이 Button이지만, view의 타입이 View이기 때문에 무조건 View의 확장 함수가 호출된다.

### 확장 프로퍼티

```kotlin
val String.lastChar: Char
	get() = get(length - 1)
```

확장 함수와 마찬가지로 확장 프로퍼티도 일반적인 프로퍼티와 같은데 단지 수신 객체 클래스가 추가되었을 뿐이다.

StringBuilder에 같은 프로퍼티를 정의한다면 StringBuilder의 맨 마지막 문자는 변경 가능하므로 프로퍼티를 var로 만들 수 있다.

```kotlin
var StringBuilder.lastChar: Char
	get() = get(length - 1)
    set(value: Char) {
    	this.setCharAt(length - 1, value)
    }
```

## 컬렉션 처리: 가변 길이 인자, 중위 함수 호출, 라이브러리 지원

### 가변 인자 함수: 인자의 개수가 달라질 수 있는 함수 정의

코틀린의 가변 길이 인자는 파라미터 앞에 vararg 변경자를 붙인다. 코틀린에서는 배열을 명시적으로 풀어서 배열의 각 원소가 인자로 전달되게 해야 한다. 단순히 배열 앞에 `*`를 붙이면 된다.

```kotlin
fun main(args: Array<String>) {
	val list = listOf("args: ", *args)
    println(list)
}
```

### 값의 쌍 다루기: 중위 호출과 구조 분해 선언

중위 호출 시에는 수신 객체와 유일한 메소드 인자 사이에 메소드 이름을 넣는다.

```kotlin
1.to("one")
1 to "one"
```

함수를 중위 호출에 사용하게 허용하고 싶으면 infix 변경자를 함수 선언 앞에 추가하면 된다.

```kotlin
infix un Any.to(other: Any) = Pair(this, other)
```

```kotlin
val (number, nume) = 1 to "one"
```

이런 기능을 구조 분해 선언이라고 부른다.

## 문자열과 정규식 다루기

### 문자열 나누기

코틀린의 split 함수는 정규식이나 일반 텍스트로 문자열을 분리할 수 있다.

### 정규식과 3중 따옴표로 묶은 문자열

3중 따옴표 문자열에서는 역슬래시()를 포함한 어떤 문자도 이스케이프할 필요가 없다.

일반 문자열로 `"C:\\Users\\yole\\kotlin-book"` 이라고 쓴 윈도우 파일 경로를 3중 따옴표 문자열로 쓰면 `"""C:\Users\yole\kotlin-book"""`이다.

3중 따옴표 문자열 안에 $을 넣어야 한다면

```kotlin
"""${'$'}99.9"""
```

처럼 문자열 템플릿 안에 '$' 문자를 넣어야 한다.

## 코드 다듬기: 로컬 함수와 확장

코틀린에서는 함수에서 추출한 함수를 원 함수 내부에 중첩시킬 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
	if(user.name.isEmpty()) {
    	throw IllegalArgumentException(
        	"Can't save user ${user.id}: empty Name")
    }
    
    if(user.address.isEmpty()) {
    	throw IllegalArgumentException(
        	"Can't save user ${user.id}: empty Address")
    }
    
    // user를 데이터베이스에 저장한다.
```

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
	fun validate(user: User,
    			value: String,
                fieldName: String) {
        if(value.isEmpty()) {
        	throw IllegalArgumentException(
        		"Can't save user ${user.id}: empty $fieldName")
        }
	
    validate(user, user.name, "Name:)
    validate(user, user.address, "Address:)
    // user를 데이터베이스에 저장한다.
}
```

로컬 함수는 자신이 속한 바깥 함수의 모든 파라미터와 변수를 사용할 수 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
	//이제 saveUser 함수의 user 파라미터를 중복 사용하지 않는다.
	fun validate(value: String,fieldName: String) {
        if(value.isEmpty()) {
        	throw IllegalArgumentException(
        		"Can't save user ${user.id}: empty $fieldName")
        }
	
    validate(user.name, "Name:)
    validate(user.address, "Address:)
    // user를 데이터베이스에 저장한다.
}
```