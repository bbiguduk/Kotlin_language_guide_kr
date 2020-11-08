# 기본 구문 \(Basic Syntax\)

## Package 정의와 import

Package는 소스파일의 최상단에 위치:

```kotlin
package my.demo

import kotlin.text.*

// ...
```

디렉토리와 packages는 서로 동일할 필요는 없습니다: 소스 파일들은 임의의 위치에 둘 수 있습니다.

See [Packages](../basics/import-packages-and-imports.md).

## 프로그램 시작점

Kotlin application의 시작점은 `main` 함수입니다.

```kotlin
fun main() {
    println("Hello world!")
}
```

## 함수

2개의 `Int` 파라미터와 `Int` 리턴타입인 함수는 아래와 같이 표현합니다:

```kotlin
//sampleStart
fun sum(a: Int, b: Int): Int {
    return a + b
}
//sampleEnd

fun main() {
    print("sum of 3 and 5 is ")
    println(sum(3, 5))
}
```

간단한 표현식 body로 부터 리턴타입 추론이 가능한 함수는 아래와 같이 표현합니다:

```kotlin
//sampleStart
fun sum(a: Int, b: Int) = a + b
//sampleEnd

fun main() {
    println("sum of 19 and 23 is ${sum(19, 23)}")
}
```

무의미한 값을 리턴하는 함수는 아래와 같이 표현합니다:

```kotlin
//sampleStart
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
//sampleEnd

fun main() {
    printSum(-1, 8)
}
```

`Unit` 리턴타입은 생략이 가능합니다:

```kotlin
//sampleStart
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
//sampleEnd

fun main() {
    printSum(-1, 8)
}
```

See [Functions](../functions-and-lambdas/functions.md).

## 변수

읽기만 가능한 변수는 키워드 `val`로 정의합니다. 해당 변수는 최초 한번만 정의 가능합니다.

```kotlin
fun main() {
//sampleStart
    val a: Int = 1  // immediate assignment
    val b = 2   // `Int` type is inferred
    val c: Int  // Type required when no initializer is provided
    c = 3       // deferred assignment
//sampleEnd
    println("a = $a, b = $b, c = $c")
}
```

쓰기가 가능한 변수는 `var`로 정의합니다:

```kotlin
fun main() {
//sampleStart
    var x = 5 // `Int` type is inferred
    x += 1
//sampleEnd
    println("x = $x")
}
```

최상위 변수\(전역변수\):

```kotlin
//sampleStart
val PI = 3.14
var x = 0

fun incrementX() { 
    x += 1 
}
//sampleEnd

fun main() {
    println("x = $x; PI = $PI")
    incrementX()
    println("incrementX()")
    println("x = $x; PI = $PI")
}
```

See also [Properties And Fields](../classes-and-objects/untitled.md).

## 주석

다른 언어와 같이 Kotlin도 single-line \(or _end-of-line_\) 과 multi-line \(_block_\) 주석을 지원합니다.

```kotlin
// This is an end-of-line comment

/* This is a block comment
   on multiple lines. */
```

Kotlin에서 주석 블락은 중첩하요 사용할 수 있습니다.

```kotlin
/* The comment starts here
/* contains a nested comment */     
and ends here. */
```

See [Documenting Kotlin Code](https://kotlinlang.org/docs/reference/kotlin-doc.html) for information on the documentation comment syntax.

## 문자열\(String\) 템플릿

```kotlin
fun main() {
//sampleStart
    var a = 1
    // simple name in template:
    val s1 = "a is $a" 

    a = 2
    // arbitrary expression in template:
    val s2 = "${s1.replace("is", "was")}, but now is $a"
//sampleEnd
    println(s2)
}
```

See [String templates](../basics/untitled.md#string-templates) for details.

## 조건부 표현

```kotlin
//sampleStart
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
//sampleEnd

fun main() {
    println("max of 0 and 42 is ${maxOf(0, 42)}")
}
```

Kotlin에선 `if`문을 추론 리턴타입으로 표기 가능합니다:

```kotlin
//sampleStart
fun maxOf(a: Int, b: Int) = if (a > b) a else b
//sampleEnd

fun main() {
    println("max of 0 and 42 is ${maxOf(0, 42)}")
}
```

See [_if_-expressions](../basics/control-flow-if-when-for-while.md#if-expression).

## Null이 가능한 변수와 _null_ 체크

_null_이 가능한 변수일 경우 반드시 표시\(?\)가 필요합니다.

`str`이 정수로 변환이 불가능할 경우 _null_을 리턴합니다:

```kotlin
fun parseInt(str: String): Int? {
    // ...
}
```

null로 리턴이 가능한 함수는 아래와 같이 표현합니다:

```kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

//sampleStart
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    // Using `x * y` yields error because they may hold nulls.
    if (x != null && y != null) {
        // x and y are automatically cast to non-nullable after null check
        println(x * y)
    }
    else {
        println("'$arg1' or '$arg2' is not a number")
    }    
}
//sampleEnd


fun main() {
    printProduct("6", "7")
    printProduct("a", "7")
    printProduct("a", "b")
}
```

또는

```kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

//sampleStart
    // ...
    if (x == null) {
        println("Wrong number format in arg1: '$arg1'")
        return
    }
    if (y == null) {
        println("Wrong number format in arg2: '$arg2'")
        return
    }

    // x and y are automatically cast to non-nullable after null check
    println(x * y)
//sampleEnd
}

fun main() {
    printProduct("6", "7")
    printProduct("a", "7")
    printProduct("99", "b")
}
```

See [Null-safety](https://kotlinlang.org/docs/reference/null-safety.html).

## 타입체크와 자동변환

어떠한 타입으로 표기가 가능한지 체크하기 위해선 _is_를 사용합니다. 불변하는 변수나 프로퍼티를 특정 타입에 대한 체크를 진행했다면, 추가로 타입변환\(캐스팅\)을 표현하지 않아도 됩니다:

```kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // `obj` is automatically cast to `String` in this branch
        return obj.length
    }

    // `obj` is still of type `Any` outside of the type-checked branch
    return null
}
//sampleEnd


fun main() {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, not a string"} ")
    }
    printLength("Incomprehensibilities")
    printLength(1000)
    printLength(listOf(Any()))
}
```

또는

```kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` is automatically cast to `String` in this branch
    return obj.length
}
//sampleEnd


fun main() {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, not a string"} ")
    }
    printLength("Incomprehensibilities")
    printLength(1000)
    printLength(listOf(Any()))
}
```

또는

```kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    // `obj` is automatically cast to `String` on the right-hand side of `&&`
    if (obj is String && obj.length > 0) {
        return obj.length
    }

    return null
}
//sampleEnd


fun main() {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, is empty or not a string at all"} ")
    }
    printLength("Incomprehensibilities")
    printLength("")
    printLength(1000)
}
```

See [Classes](../classes-and-objects/class-classes-and-inheritance.md#classes) and [Type casts](https://kotlinlang.org/docs/reference/typecasts.html).

## `for` 문

```kotlin
fun main() {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    for (item in items) {
        println(item)
    }
//sampleEnd
}
```

또는

```kotlin
fun main() {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    for (index in items.indices) {
        println("item at $index is ${items[index]}")
    }
//sampleEnd
}
```

See [for loop](../basics/control-flow-if-when-for-while.md#for-loops).

## `while` 문

```kotlin
fun main() {
//sampleStart
    val items = listOf("apple", "banana", "kiwifruit")
    var index = 0
    while (index < items.size) {
        println("item at $index is ${items[index]}")
        index++
    }
//sampleEnd
}
```

See [while loop](../basics/control-flow-if-when-for-while.md#while-loops).

## `when` 표현

```kotlin
//sampleStart
fun describe(obj: Any): String =
    when (obj) {
        1          -> "One"
        "Hello"    -> "Greeting"
        is Long    -> "Long"
        !is String -> "Not a string"
        else       -> "Unknown"
    }
//sampleEnd

fun main() {
    println(describe(1))
    println(describe("Hello"))
    println(describe(1000L))
    println(describe(2))
    println(describe("other"))
}
```

See [when expression](../basics/control-flow-if-when-for-while.md#when-expression).

## 범위

어떠한 숫자가 범위안에 포함되는지 체크하기 위해선 _in_을 사용한다:

```kotlin
fun main() {
//sampleStart
    val x = 10
    val y = 9
    if (x in 1..y+1) {
        println("fits in range")
    }
//sampleEnd
}
```

범위에 포함하지 않는경우를 체크하기 위해선 아래와 같이 표현한다:

```kotlin
fun main() {
//sampleStart
    val list = listOf("a", "b", "c")

    if (-1 !in 0..list.lastIndex) {
        println("-1 is out of range")
    }
    if (list.size !in list.indices) {
        println("list size is out of valid list indices range, too")
    }
//sampleEnd
}
```

범위내 반복 표현은 아래와 같이 합니다:

```kotlin
fun main() {
//sampleStart
    for (x in 1..5) {
        print(x)
    }
//sampleEnd
}
```

또는 아래와 같은 반복 표현도 가능합니다:

```kotlin
fun main() {
//sampleStart
    for (x in 1..10 step 2) {
        print(x)
    }
    println()
    for (x in 9 downTo 0 step 3) {
        print(x)
    }
//sampleEnd
}
```

See [Ranges](../collections/ranges-and-progressions-1.md#range).

## 콜렉션

콜렉션의 반복 표현은 아래와 같습니다:

```kotlin
fun main() {
    val items = listOf("apple", "banana", "kiwifruit")
//sampleStart
    for (item in items) {
        println(item)
    }
//sampleEnd
}
```

콜렉션 안에 어떠한 object가 포함되어 있는지 확인하기 위해선 _in_을 사용합니다:

```kotlin
fun main() {
    val items = setOf("apple", "banana", "kiwifruit")
//sampleStart
    when {
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }
//sampleEnd
}
```

콜렉션 filter와 map은 람다 표현을 이용하여 표기해줍니다:

```kotlin
fun main() {
//sampleStart
    val fruits = listOf("banana", "avocado", "apple", "kiwifruit")
    fruits
      .filter { it.startsWith("a") }
      .sortedBy { it }
      .map { it.toUpperCase() }
      .forEach { println(it) }
//sampleEnd
}
```

See [Collections overview](../collections/kotlin-kotlin-collections-overview.md).

## 기본 class 생성과 인스턴스 생성

```kotlin
fun main() {
//sampleStart
    val rectangle = Rectangle(5.0, 2.0)
    val triangle = Triangle(3.0, 4.0, 5.0)
//sampleEnd
    println("Area of rectangle is ${rectangle.calculateArea()}, its perimeter is ${rectangle.perimeter}")
    println("Area of triangle is ${triangle.calculateArea()}, its perimeter is ${triangle.perimeter}")
}

abstract class Shape(val sides: List<Double>) {
    val perimeter: Double get() = sides.sum()
    abstract fun calculateArea(): Double
}

interface RectangleProperties {
    val isSquare: Boolean
}

class Rectangle(
    var height: Double,
    var length: Double
) : Shape(listOf(height, length, height, length)), RectangleProperties {
    override val isSquare: Boolean get() = length == height
    override fun calculateArea(): Double = height * length
}

class Triangle(
    var sideA: Double,
    var sideB: Double,
    var sideC: Double
) : Shape(listOf(sideA, sideB, sideC)) {
    override fun calculateArea(): Double {
        val s = perimeter / 2
        return Math.sqrt(s * (s - sideA) * (s - sideB) * (s - sideC))
    }
}
```

See [classes](../classes-and-objects/class-classes-and-inheritance.md#classes) and [objects and instances](../classes-and-objects/object-expressions-and-declarations.md).

