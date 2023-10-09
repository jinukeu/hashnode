---
title: "[Kotlin in Action] 8장. 고차 함수: 파라미터와 반환 값으로 람다 사용"
datePublished: Tue Sep 19 2023 07:35:05 GMT+0000 (Coordinated Universal Time)
cuid: clmq02qp2000309le4kmr1c8w
slug: kotlin-in-action-8
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1696821352743/8dc113ee-9889-4817-80b0-064edf95f1b3.jpeg
tags: kotlin

---

## 고차 함수 정의

고차 함수는 다른 함수를 인자로 받거나 함수를 반환하는 함수다.

### 함수 타입

함수 타입을 정의하려면 함수 파라미터의 타입을 괄호 안에 넣고, 그 뒤에 화살표(-&gt;)를 추가한 다음, 함수의 반환 타입을 지정하면 된다.

```kotlin
val sum = (Int, Int) -> Int = { x, y -> x + y }
```

함수의 반환 타입이 아니라 함수 타입 전체가 널이 될 수 있는 타입임을 선언할 때는 함수 타입을 괄호로 감싸고 그 뒤에 물음표를 붙인다.

```kotlin
var funOrNull: ((Int, Int) -> Int)? = null
```

### 인자로 받은 함수 호출

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("The result is $result")
}

twoAndThree { a, b -> a + b }
The result is 5
```

### 함수를 함수에서 반환

```kotlin
enum class Delivery { STANDARD, EXPEDITED }
class Order(val itemCount: Int)

fun getShippingCostCaluculator(delivery: Delivery): (Order) -> Double {
    if(delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 1.2 * order.itemCount }
}

val calculator = getShippingCostCaluculator(Delivery.EXPEDITED)
println("${calculator(Order(3))}")
```

## 인라인 함수: 람다의 부가 비용 없애기

inline 변경자를 어떤 함수에 붙이면 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해준다.

### 인라이닝이 작동하는 방식

```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()
    try {
        return action()
    }
    finally {
        lock.unlock()
    }
}

fun foo(l: Lock) {
    println("Before sync")
    synchronized(l) {
        println("Action")
    }
    println("After sync")
}
```

foo는 아래와 같은 바이트코드를 만들어낸다.

```kotlin
fun __foo__(l: Lock) {
    println("Before sync")
    l.lock() //인라이닝
    try { //인라이닝
        println("Action") //인라이닝
    } //인라이닝
    finally { //인라이닝
        l.unlock() //인라이닝
    } //인라이닝
    println("After sync")
}
```

인라인 함수를 호출하면서 람다를 넘기는 대신에 함수 타입의 변수를 넘길 수도 있다. 이런 경우 함수 타입의 변수는 인라이닝되지 않는다.

```kotlin
class LockOwner(val lock: Lock) {
    fun runUnderLock(body: () -> Unit) {
        synchronized(lock, body)
    }
}

//컴파일 시
class LockOwner(val lock: Lock) {
    fun __runUnderLock__(body: () -> Unit) {
        lock.lock()
        try {
            body() // body내용은 인라이닝되지않는다.
        }
        finally {
            lock.unlock()
        }
    }
}
```

### 인라인 함수의 한계

함수 본문에서 파라미터로 받은 람다를 호출한다면 그 호출을 쉽게 람다 본문으로 바꿀 수 있다. 하지만 파라미터로 받은 람다를 다른 변수에 저장하고 나중에 그 변수를 사용한다면 람다를 표현하는 객체가 어딘가에는 존재해야 하기 때문에 람다를 인라이닝할 수 없다.

둘 이상의 람다를 인자로 받는 함수에서 일부 람다만 인라이닝하고 싶을 때는 noinline 변경자를 파라미터 이름 앞에 붙이면 된다.

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit {
    // ..
}
```

### 함수를 인라인으로 선언해야 하는 경우

일반 함수 호출의 경우 JVM은 이미 인라이닝을 알아서 해준다. (가장 이익이 되는 방향으로) 즉, 굳이 할 필요 없다.

그러면 언제 해야할까?

**람다를 인자로 받는 함수**는 인라이닝하면 이익이 더 많다.

람다를 표현하는 클래스와 람다 인스턴스에 해당하는 객체를 만들 필요가 없어지기 때문.  
또한 JVM은 함수 호출과 람다를 인라이닝해 줄 정도로 똑똑하진 않기 때문.  
인라이닝을 사용하면 넌로컬 반환을 사용할 수 있기 때문. (인라이닝에서만 사용할 수 있는 기능이 몇 가지 있다.)

## 고차 함수 안에서 흐름 제어

### 람다 안의 return문: 람다를 둘러싼 함수로부터 반환

람다를 인자로 받는 함수가 인라인 함수인 경우 return은 바깥쪽 함수를 반환시킨다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return //forEach가 인라인 함수이기 때문에 return은 lookForAlice 함수를 리턴한다.
        }
    }
    println("Alice is not found")
}
```

자산을 둘러싸고 있는 블록보다 더 바깥에 있는 블록을 반환하게 만드는 return문을 넌로컬 return이라 부른다.

### 람다로부터 반환: 레이블을 사용한 return

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach @label {
        if (it.name == "Alice") {
            println("Found!")
            return@label
        }
    }
    println("Alice is not found")
}
```

람다에 레이블을 붙여서 사용하는 대신 람다를 인자로 받는 인라인 함수의 이름을 return 뒤에 레이블로 사용해도 된다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach {
        if (it.name == "Alice") {
            println("Found!")
            return@forEach
        }
    }
    println("Alice is not found")
}
```

### 무명 함수: 기본적으로 로컬 return

무명 함수를 사용해서 코드 블록을 함수에 넘길 수 있다.

```kotlin
fun lookForAlice(people: List<Person>) {
    people.forEach(fun (person) {
        if (it.name == "Alice") return
        println("${person.name} is not Alice")
    })
}
```

무명 함수 안에서 레이블이 붙지 않은 return 식은 무명 함수 자체를 반환시킬 뿐 무명 함수를 둘러싼 다른 함수를 반환시키지 않는다.

return은 fun 키워드를 사용해 정의된 가장 안쪽 함수를 반환시킨다.