---
title: "[Kotlin in Action] 6장. 코틀린 타입 시스템"
datePublished: Wed Aug 09 2023 15:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clmesg4ej000009l79bzrbk17
slug: kotlin-in-action-6
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696821317997/ac3895c0-6128-407b-83ad-8a0ac78105c2.jpeg
tags: kotlin

---

# 널 가능성

널이 될 수 있는지 여부를 타입 시스템에 추가함으로써 컴파일러가 여러 가지 오류를 컴파일 시 미리 감지해서 실행 시점에 발생할 수 있는 예외의 가능성을 줄일 수 있다.

## 널이 될 수 있는 타입

널이 될 수 있는 변수에 대한 메소드를 호출하면 NullPointerException이 발생할 수 있다. 코틀린은 이런 메소드 호출을 금지할 수 있다.

## 타입의 의미

널이 될 수 있는 타입과 될 수 없는 타입을 구분하면 각 타입의 값에 대해 어떤 연산이 가능할지 명확히 이해할 수 있고, 실행 시점에 예외를 발생시킬 수 있는 연산을 판단할 수 있다.

## 안전한 호출 연산자: ?.

?. 은 null 검사와 메소드 호출을 한 번의 연산으로 수행한다. 예를 들어 s?.toUppderCase()는 if(s!=null) s.toUpperCase() else null과 같다.

## 엘비스 연산자: ?:

```kotlin
fun foo(s: String?) {
	val t: String = s ?: ""
}
```

"" 대신 return이나 throw등의 연산을 넣을 수 있다.

## 안전한 캐스트: as?

as? 연산자는 어떤 값을 지정한 타입으로 캐스트한다. as?는 값을 대상 타입으로 변환할 수 없으면 null을 반환한다.

## 널 아님 단언: !!

!!를 사용하면 어떤 값이든 널이 될 수 없는 타입으로 바꿀 수 있다.

## let 함수

let함수는 자신의 수신 객체를 인자로 전달받은 람다에게 넘긴다. email?.let { sendEmailTo(it) }

## 나중에 초기화할 프로퍼티

lateinit 변경자를 붙이면 프로퍼티를 나중에 초기화할 수 있다.

## 널이 될 수 있는 타입 확장

널이 될 수 있는 타입에 대한 확장 함수를 정의하면 널이 될 수 있는 값에 대해 그 확장 함수를 호출할 수 있다.

s.toUpperCase()의 경우 s가 null이면 NPE를 발생시키지만 s.isNullOrEmpty()는 NPE를 발생시키지 않는다.

## 타입 파라미터의 널 가능성

코틀린에서는 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다. 따라서 타입 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 T가 널이 될 수 있는 타입이다.

```kotlin
fun <T> printHashCode(t: T) {
	println(t?.hashCode()) //t는 null이 될 수 있다.
}
```

타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상한를 지정해야 한다.

```kotlin
fun <T: Any> printHashCode(t: T) {
	println(t.hashCode()) //t는 null이 될 수 없다.
}
```

## 널 가능성과 자바

자바에서 @Nullable과 같은 코틀린이 이해할 수 있는 어노테이션이 있다면 코틀린에서 이 정보를 활용하여 널 가능성을 판단한다.

### 플랫폼 타입

플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다. 이 타입을 널이 될 수 있는 타입으로 처리해도 되고 널이 될 수 없는 타입으로 처리해도 된다. 단, NPE 발생 가능성이 있다.

### 상속

코틀린에서 자바 메소드를 오버라이드할 때 그 메소드의 파라미터와 반환 타입을 널이 될 수 있는 타입으로 선언할지 널이 될 수 없는 타입으로 선언할지 결정해야 한다.

```java
/* 자바 */
interface StringProcessor {
	void process(String value);
}
```

코틀린 컴파일러는 다음과 같은 두 구현을 다 받아들인다.

```kotlin
class StringPrinter : StringProcessor{
	override fun process(value: String) {
    	println(value)
    }
}

class StringPrinter : StringProcessor{
	override fun process(value: String?) {
    	println(value)
    }
}
```

# 코틀린의 원시 타입

## 원시 타입: Int, Boolean 등

코틀린은 자바와 달리 원시 타입과 참조 타입을 구분하지 않는다. 상황에 따라 자바의 원시 타입 또는 참조 타입으로 컴파일된다.

## 널이 될 수 있는 원시 타입: Int?, Boolean? 등

코틀린에서 널이 될 수 있는 원시 타입을 사용하면 그 타입은 자바의 래퍼 타입으로 컴파일된다.

제네릭 클래스의 경우 래퍼 타입을 사용한다. 예를 들어 다음 문장에서는 null 값이나 널이 될 수 있는 타입을 전혀 사용하지 않았지만 만들어지는 리스트는 래퍼인 Integer 타입으로 이뤄진 리스트다.

```kotlin
val listOfInts = listOf(1, 2, 3)
```

### Any, Any?: 최상위 타입

자바에서 Object가 클래스 계층의 최상위 타입이듯 코틀린에서는 Any 타입이 모든 널이 될 수 없는 타입의 조상 타입이다. 자바에서 원시 타입은 Object가 최상위 타입이 아닌 반면, 코틀린에서는 Any가 Int 등의 원시 타입을 포함한 모든 타입의 조상 타입이다.

코틀린에서 널을 포함하는 모든 값을 대입할 변수를 선언하려면 Any? 타입을 사용해야 한다.

자바 메소드에서 Object를 인자로 받거나 반환하면 코틀린에서는 Any!로 그 타입을 취급한다. 코틀린 함수가 Any를 사용하면 자바 바이트코드의 Object로 컴파일된다.

java.lang.Object에 있는 메소드(wait, notify)는 Any에서 사용할 수 없다. 사용하려면 java.lang.Object로 값을 캐스트해야 한다.

## Unit 타입: 코틀린의 void

코틀린 Unit 타입은 자바 void와 같은 기능을 한다. 코틀린 함수의 반환 타입이 Unit이고 그 함수가 제네릭 함수를 오버라이드하지 않는다면 그 함수는 내부에서 자바 void 함수로 컴파일된다. 그런 코틀린 함수를 자바에서 오버라이드하는 경우 void를 반환 타입으로 해야 한다.

코틀린의 Unit과 자바 void의 차이점은 Unit은 모든 기능을 갖는 일반적인 타입이며, void와 달리 Unit를 타입 인자로 쓸 수 있다.

## Nothing 타입: 이 함수는 결코 정상적으로 끝나지 않는다.

Nothing 타입은 아무 값도 포함하지 않는다. 따라서 Nothing은 함수의 반환 타입이나 반환 타입으로 쓰일 타입 파라미터로만 쓸 수 있다.

```kotlin
fun fail(message: String) : Nothing {
	throw IllegalStateException(message) // 반환 X
}
```

컴파일러는 Nothing이 반환 타입인 함수가 결코 정상 종료되지 않음을 알고 그 함수를 호출하는 코드를 분석할 때 사용한다.

# 컬렉션과 배열

## 읽기 전용과 변경 가능한 컬렉션

코틀린 컬렉션과 자바 컬렉션을 나누는 가장 중요한 특성 하나는 코틀린에서는 컬렉션 안의 데이터에 접근하는 인터페이스와 컬렉션 안의 데이터를 변경하는 인터페이스를 분리했다는 점이다. 컬렉션의 읽기 전용 인터페이스와 변경 가능 인터페이스를 구별한 이유는 프로그램에서 데이터에 어떤 일이 벌어지는지를 더 쉽게 이해하기 위함이다.

kotlin.collections.Collection -&gt; 원소 추가, 제거 불가 kotlin.collections.MutableCollection -&gt; 원소 추가, 제거 가능

동일한 컬렉션 객체를 읽기 전용 컬렉션 타입의 참조와 변경 가능한 컬렉션 타입이 참조할 수 있다. 이 컬렉션을 참조하는 다른 코드를 호출하거나 병렬 실행한다면 컬렉션을 사용하는 도중에 다른 컬렉션이 그 컬렉션의 내용을 변경하는 상황이 생길 수 있고 이런 상황에서는ConcurrentModificationException이나 다른 오류가 발생할 수 있다.

## 코틀린 컬렉션과 자바

자바는 읽기 전용 컬렉션과 변경 가능 컬렉션을 구분하지 않으므로, 코틀린에서 읽기 전용 Collection으로 선언된 객체라도 자바 코드에서는 그 컬렉션 객체의 내용을 변경할 수 있다.

또한 널이 아닌 원소로 이뤄진 컬렉션을 자바 메소드로 넘겼는데 자바 메소드가 널을 컬렉션에 넣을 수도 있다.

따라서 컬렉션을 자바 코드에 넘길 때는 특별히 주의를 기울여야 한다.

## 컬렉션을 플랫폼 타입으로 다루기

```java
/* 자바 */
interface FileContentProcessor {
	void processContents(
    	File path, 
        byte[] binaryContents, 
        List<String> textContents);
}
```

이 인터페이스를 코틀린으로 구현하려면 다음을 선택해야 한다. (읽기 전용으로 구현할지, null 가능성을 열어둘지 코틀린에서 선택해야한다.)

* 일부 파일은 이진 파일이며 이진 파일 안의 내용은 텍스트로 표현할 수 없는 경우가 있으므로 리스트는 널이 될 수 있다.
    
* 파일의 각 줄은 널일 수 없으므로 이 리스트의 원소는 널이 될 수 없다.
    
* 이 리스트는 파일의 내용을 표현하며 그 내용을 바꿀 필요가 없으므로 읽기 전용이다.
    

```kotlin
class FileIndexer : FileContentProcessor {
	override fun processContents(path: File,
    	binaryContents: ByteArray?,
        textContents: List<String>?) {
     // ...
    }
```

## 객체의 배열과 원시 타입의 배열

코틀린 배열은 타입 파라미터를 받는 클래스다. 배열의 원소 타입은 바로 그 타입 파라미터에 의해 정해진다. 보통 arrayOf 함수를 이용해 배열을 만든다.

코틀린에서는 배열을 인자로 받는 자바 함수를 호출하거나 vararg 파라미터를 받는 코틀린 함수를 호출하기 위해 자주 배열을 만든다. 데이터가 이미 컬렉션에 들어가 있다면 toTypedArray 메서드를 사용해 배열로 변환한다.

`Array<Int>` 같은 타입을 선언하면 그 배열은 박싱된 정수의 배열이다. 원시 타입을 사용하고 싶다면 IntArray, ByteArray 등의 배열을 사용해야 한다.

이 밖에 박싱된 값이 들어있는 컬렉션이나 배열이 있다면 toIntArray 등의 변환 함수를 사용할 수 있다.