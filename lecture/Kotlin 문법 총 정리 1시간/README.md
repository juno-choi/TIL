# 🔴 코틀린 1시간 총정리

## 🟠 기본

### 🟢 변수
코틀린은 ;을 붙이지 않는게 좋다

변수는 var

코틀린은 자동 타입 추정을 진행해서 궅이 타입을 명시하지 않아도 된다.


### 🟢 상수

val

java의 final과 동일하다.

```kotlin
fun main() {
    var name = "준호"
    name = "고은"
    val age : Int = 3

    println(name)
    println(age)
}
```

### 🟢 const

compile type 상수로 정의한다면 const 예약어를 사용해주면 main이 실행되기 전 컴파일때 상수를 등록하여 성능 향상을 기대해볼 수 있다.

```kotlin

const val age : Int = 3

fun main() {

    var name = "준호"
    name = "고은"

    println(name)
    println(age)

}
```

### 🟢 형변환

기존 java에서 `(Long) 3` 과 같이 형변환을 하거나 `Integer.parseInt()` 와 같은 작업이 필요하지 않다

```kotlin
fun main() {
    
    var i = 10
    var l = 20L
    var s = ""
    
    l = i.toLong()
    s = i.toString()
}
```

### 🟢 String

String 기본 사용법

```kotlin
fun main() {
    var name = "준호"
    println(name.uppercase())
    println(name.lowercase())
    println(name[0])
    println("제 이름은 ${name} 입니다.")
}
```

### 🟢 Max, Min

```kotlin
import kotlin.math.*

fun main() {

    var i = 10
    var j = 20
    
    println(max(i, j))
    println(min(i, j))
}
```

### 🟢 random

```kotlin
import kotlin.random.Random

fun main() {
    val randomNumber = Random.nextInt(0, 100)   // 0~99
    println(randomNumber)

    val randomDouble = Random.nextDouble(0.0, 1.0)   // 0~99
    println(randomDouble)
}
```

### 🟢 키보드 입력

```kotlin
import java.util.Scanner

fun main() {
    val reader = Scanner(System.`in`)
    val next = reader.next()
    println(next)
}
```

### 🟢 조건문

기존 if else 사용 가능
```kotlin
fun main() {
    var i = 5
    if (i > 10) {
        println("10보다 크다")
    } else if (i > 5) {
        println("5보다 크다")
    } else {
        println("5보다 작다")
    }
}
```

if문은 when문으로 치환이 가능하다
```kotlin
fun main() {
    var i = 5

    when {
        i > 10 -> {
            println("10보다 크다")
        }
        i > 5 -> {
            println("5보다 크다")
        }
        else -> {
            println("5보다 작다")
        }
    }
}
```

if와 when은 모두 식으로서 결과값은 전달받을 수 있다.
```kotlin
fun main() {
    var i = 5

    val result = if (i > 10) {
        "10보다 크다"
    } else if (i > 5) {
        "5보다 크다"
    } else {
        "5보다 작다"
    }

    println(result)
}
```

```kotlin
fun main() {
    var i = 5

    val result = when {
        i > 10 -> {
            "10보다 크다"
        }
        i > 5 -> {
            "5보다 크다"
        }
        else -> {
            "5보다 작다"
        }
    }

    println(result)
}
```

java의 삼항 연산자에 대해서도 아래와 같이 사용할 수 있다
```kotlin
fun main() {
    // result = i > 10 ? true : false;
    var i = 5
    var result = if (i > 10) true else false
}
```

### 🟢 반복문

```kotlin
fun main() {

    val items = listOf<Int>(1,2,3,4,5)

    for (item in items) {
        print(item)
    }

    println()

    items.forEach { item ->
        print(item)
    }
    println()

    for(i in 0..4) {
        print(i)
    }
    println()

    for (i in 0..(items.size - 1)) {
        print(items[i])
    }
    println()
    
    var j = 0
    while(j < 5) {
        print(items[j])
        j++
    }
}
```

### 🟢 List

```kotlin
fun main() {
    
    // 불변 리스트
    val items = listOf<Int>(1,2,3,4,5)
    
    // 변경 가능 리스트
    val list = mutableListOf<Int>(1,2,3,4,5)
    
    list.add(6)
    list.add(7)
    
}
```

### 🟢 Array

```kotlin
fun main() {
    val items = arrayOf(1,2,3)
    // java는 length인데 kotlin은 size다
    items.size
}
```

### 🟢 try-catch-finally

```kotlin
fun main() {
    val items = arrayOf(1,2,3)

    try {
        items[4]
    } catch (e: Exception) {
        println(e.message)
    } finally {
        println("finally")
    }
}
```

### 🟢 null safety

```kotlin
fun main() {
    var name:String? = null
    name = null
    name = "junho"
    
    var name2:String = ""
    // name2 = name , String과 String?은 타입이 다르기 때문에
    if (name != null) {
        name2 = name
    }
    
    // 개발자가 직접 null이 아닐거라고 명시해주는 경우인데 추천하진 않는다.
    name2 = name!!
    
    // ?.let{}은 name값이 null이 아니라면 과 같다.
    name?.let { it -> println(it) }
    
}
```

### 🟢 function (method)

```kotlin
fun main() {
    println(sum(1, 2))
    // 입력 parameter를 명시적으로 지정할 수 있다
    println(sum(a = 1, b = 2, c = 3))
}

fun sum(a: Int, b: Int) : Int {
    return a + b
}

// 간단한 경우 아래와 같이 변경 가능
fun sum2(a: Int, b: Int) : Int = a + b
```

```kotlin
fun main() {
    println(sum(1, 2))
    // 입력 parameter를 명시적으로 지정할 수 있다
    println(sum(a = 1, b = 2, c = 3))
}

// 자바에서는 아래와 같이 c가 추가된다면 오버로딩을 해야하지만 kotlin은 기존 함수에 c값에 default 값을 넣어주면 그대로 사용이 가능하다
fun sum(a: Int, b: Int, c: Int = 0) = a + b + c
```

### 🟢 class

```kotlin
fun main() {
    val juno = Person("Juno", 30)
    // getter setter가 모두 사용 가능
    println(juno.name)
    
    juno.age = 31
    println(juno.age)
}


class Person (
    val name: String,
    var age: Int,
) {}
```

class 생성시 parameter에 `private`으로 막아주면 getter, setter가 노출되지 않는다.

### 🟢 data class

```kotlin
fun main() {
    val juno1 = Person("Juno", 30)
    val juno2 = Person("Juno", 30)

    println(juno1 == juno2)
}


class Person (
    val name: String,
    var age: Int,
) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false

        other as Person

        if (age != other.age) return false
        if (name != other.name) return false

        return true
    }

    override fun hashCode(): Int {
        var result = age
        result = 31 * result + name.hashCode()
        return result
    }

    override fun toString(): String {
        return "Person(name='$name', age=$age)"
    }
}
```

equals와 hashCode, toString을 구현하려면 java에서는 다음과 같이 진행해야 한다. 혹은 롬복을 쓰거나

```kotlin
fun main() {
    val juno1 = Person("Juno", 30)
    val juno2 = Person("Juno", 30)

    println(juno1 == juno2)
}


data class Person (
    val name: String,
    var age: Int,
)
```

kotlin에서는 `data class`를 선언해주면 된다

```kotlin
fun main() {
    val juno1 = Person("Juno", 30)
    val juno2 = Person("Juno", 30)

    println(juno1 == juno2)
    println(juno1)
    println(juno2)
}


data class Person (
    val name: String,
    var age: Int,
    var hobby: String = ""
) {
    // 생성자 만든 후 초기값 세팅
    init {
        hobby = "축구"
    }
}
```

`init {}`을 통해 생성자를 만든 후 초기값을 세팅할 수 있다

### 🟢 getter, setter 제어

```kotlin
fun main() {
    val juno1 = Person("Juno", 30)
    val juno2 = Person("Juno", 30)

    println(juno1 == juno2)
    println(juno1)
    println(juno2)
    println(juno1.married)
}


data class Person (
    val name: String,
    var age: Int,
    var hobby: String = ""
) {
    var married: String = "미혼"
    private set // set 접근하지 못하도록 막는다
        get() = "결혼 여부 : $field"

    // 생성자 만든 후 초기값 세팅
    init {
        hobby = "축구"
    }
}
```

getter에서는 `get()`을 오버라이드 하여 사용할 수 있고

setter는 private set을 주어 내부에서 사용되는 변수에 대한 접근을 막을 수 있다.

### 🟢 Extends

```kotlin
fun main() {

    val dog = Dog()
    dog.move()
}

abstract class Animal {
    fun move() {
        println("move")
    }
}

class Dog : Animal() {

}

class Cat : Animal() {

}
```

추상 class를 정의하여 내부적으로 funtion을 정의해두면 동일하게 가져다가 사용할 수 있다

하지만 여기서 move() function은 각 class에서 override 해서 사용할 수 없도록 막혀있다

```kotlin
fun main() {

    val dog = Dog()
    dog.move()
}

abstract class Animal {
    open fun move() {
        println("move")
    }
}

class Dog : Animal() {
    override fun move() {
        println("이동")
    }
}

class Cat : Animal() {

}
```

`open` 예약어를 통해 overrid 가동하도록 처리할 수 있다.

```kotlin
fun main() {

    val dog = Dog()
    dog.move()
}

open class Animal {
    open fun move() {
        println("move")
    }
}

class Dog : Animal() {
    override fun move() {
        println("이동")
    }
}

class Cat : Animal() {

}
```

추상 class가 아닌 일반 class의 경우 상속이 가능하도록 처리하려면 class 앞에 open 예약어를 처리해주면 가능해진다

### 🟢 interface


```kotlin
fun main() {

    val dog = Dog()
    dog.move()
}

interface Drawable {
    fun draw()
}

abstract class Animal {
    open fun move() {
        println("move")
    }
}

class Dog : Animal(), Drawable {
    override fun move() {
        println("이동")
    }

    override fun draw() {
        TODO("Not yet implemented")
    }
}

class Cat : Animal(), Drawable {
    override fun draw() {
        TODO("Not yet implemented")
    }
}
```

interface의 경우는 class와 달리 class 명을 나열해주면 된다.

### 🟢 타입 체크

```kotlin
fun main() {

    val animal : Animal = Dog()
    
    if (animal is Dog) {
        println("멍멍")
    }
}

...
```

타입 체크의 경우 `is`를 통해서 타입 체크가 된다.

### 🟢 강제 타입 캐스팅

```kotlin
fun main() {

    val animal : Animal = Dog()
    animal as Dog
    animal.draw()
}

...
```

`as`를 통해 강제로 타입을 캐스팅할 수 있다.

Dog만 사용할 수 있는 draw()를 as를 통해 먼저 선언해주면 사용이 가능해진다.

### 🟢 제네릭

```kotlin
fun main() {
    val box = Box(10)
    val box2 = Box("hello")

    println(box.value)
    println(box2.value)
}

class Box<T>(val value: T) {

}
```

제네릭은 java와 비슷하게 사용할 수 있다.

### 🟢 콜백 함수(고차 함수)

```kotlin
fun main() {
    myFunc({
        println("함수 호출 $it")
    })
}

fun myFunc(callback : (a: Int) -> Unit) {
    println("함수 시작")
    callback(2)
    println("함수 종료")
}
```

kotlin은 다음과 같이 콜백 함수를 정의해서 사용할 수 있다


```kotlin
fun main() {
    myFunc {
        println("함수 호출 $it")
    }
}

// Unit = Void
fun myFunc(callback : (a: Int) -> Unit) {
    println("함수 시작")
    callback(2)
    println("함수 종료")
}
```

또한 위의 코드와 아래 코드는 동일한 코드이다

### 🟢 코루틴

코루틴은 나중에 자세히 공부해보자.