# 기본 유형 (Basic Types)

Kotlin에서 모든 변수에서 멤버 함수와 프로퍼티를 호출 할 수 있다는 의미에서 모두다 객체입니다.
일부 타입들은 특별한 내부 표현을 가지고 있습니다 - 예를 들어, number, character, boolean은 런타임 시 기본값으로 표시 될 수 있습니다 - 그러나 일반 class 처럼 보입니다.
이번 섹션에서는 Kotlin 에서 사용하는 기본 유형에 대해 설명하겠습니다: number, character, boolean, array, string.

## 숫자 (Numbers)

Kotlin은 숫자를 나타내는 자체 내장 된 타입을 제공합니다.
정수 숫자의 경우 크기와 값 범위가 다른 4가지 타입이 있습니다.

| 유형	 |크기 (bits)| 최소값| 최대값|
|--------|-----------|----------|--------- |
| Byte	 | 8         |-128      |127       |
| Short	 | 16        |-32768    |32767     |
| Int	 | 32        |-2,147,483,648 (-2<sup>31</sup>)| 2,147,483,647 (2<sup>31</sup> - 1)|
| Long	 | 64        |-9,223,372,036,854,775,808 (-2<sup>63</sup>)|9,223,372,036,854,775,807 (2<sup>63</sup> - 1)|

모든 변수는 `Int`의 최대값을 넘지 않는 값으로 초기화 시 `Int`타입으로 추론 합니다. 만약에 초기화 값이 `Int`의 최대값을 넘어갈 경우 `Long`으로 추론하게 됩니다.
명시적으로 `Long`으로 지정하고 싶다면 `L`접미사를 추가해 주면 됩니다.

```kotlin
val one = 1 // Int
val threeBillion = 3000000000 // Long
val oneLong = 1L // Long
val oneByte: Byte = 1
```

부동 소수점 숫자의 경우 Kotlin에서는 `Float` 와 `Double`타입을 제공합니다.
[IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754) 따르면, 부동 소수점 타입은 _decimal place_, 즉 저장할 수 있는 소수 자릿수에 따라 다릅니다.
`Float`는 IEEE 754 _single precision_ 을 반영하는 반면에 `Double`은 _double precision_ 을 제공합니다. 

| 유형	 |크기 (bits)|최상위 비트|지수 비트|가수|
|--------|-----------|--------------- |-------------|--------------|
| Float	 | 32        |24              |8            |6-7            |
| Double | 64        |53              |11           |15-16          |    

소수로 초기화 된 변수는 컴파일러가 `Double` 타입으로 추론 합니다.
명시적으로 `Float` 으로 지정하고 싶다면 `f` 또는 `F`접미사를 추가해 주면 됩니다.
만약 값의 가수가 6-7 보다 많을 경우 반올림 됩니다.

```kotlin
val pi = 3.14 // Double
val e = 2.7182818284 // Double
val eFloat = 2.7182818284f // Float, actual value is 2.7182817
```
Kotlin은 다른언어와 다르게 숫자에 대해선 자동 형변환이 되지 않습니다.
예를 들어, `Double` 타입의 파라미터를 가지고 있는 함수는 오직 `Double` 타입 파라미터를 적용해야 정상 호출이 가능합니다. `Float`, `Int` 또는 다른 숫자 타입에 대해선 동작하지 않습니다.

```kotlin
fun main() {
    fun printDouble(d: Double) { print(d) }

    val i = 1    
    val d = 1.1
    val f = 1.1f 

    printDouble(d)
//    printDouble(i) // Error: Type mismatch
//    printDouble(f) // Error: Type mismatch
}
```

숫자 타입을 다른 타입으로 변환하려면 [Explicit conversions](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/untitled#explicit-conversions)를 참고하세요.

### 리터럴 상수 (Literal constants)

정수에 대한 리터널 상수는 아래와 같은 종류가 있습니다:

* 10진법: `123`
  * Long은 뒤에 대문자 `L`을 표기해줌: `123L`
* 16진법: `0x0F`
* 2진법: `0b00001011`

NOTE: 8진법은 지원하지 않습니다.

Kotlin은 부동 소수점 숫자를 위한 표기법도 지원합니다:
 
* `Double`이 기본: `123.5`, `123.5e10`
* `Float`는 `f` 또는 `F`로 표기: `123.5f`
 
### 숫자 표기시 언더바 (Kotlin 1.1 이상 지원)
 
숫자를 더 읽기 좋게 언더바를 넣어 표기할 수 있습니다: 

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### 표현 (Representation)

Java 플랫폼에서 숫자는 null이 가능한 숫자 (예. `Int?`) 또는 제너릭을 사용하지 않으면 JVM 원시 타입으로 저장이 됩니다.
null이 가능한 숫자 또는 제너릭을 사용하면 박싱이 되어 객체처럼 사용이 됩니다.

숫자를 박싱하면 identity를 유지하지 않습니다:

```kotlin
fun main() {
//sampleStart
    val a: Int = 10000
    println(a === a) // Prints 'true'
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a
    println(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
//sampleEnd
}
```

반면에, 동등성은 유지됩니다:

```kotlin
fun main() {
//sampleStart
    val a: Int = 10000
    println(a == a) // Prints 'true'
    val boxedA: Int? = a
    val anotherBoxedA: Int? = a
    println(boxedA == anotherBoxedA) // Prints 'true'
//sampleEnd
}
```

### 명시적 변환 (Explicit conversions)

표현이 다르기 때문에 작은 타입은 큰 타입의 속해 있지 않습니다.
만약 그렇게 된다면 아래와 같은 문제가 발생합니다:

```kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(b == a) // Surprise! This prints "false" as Long's equals() checks whether the other is Long as well
```

그래서 동등성이 맞지 않게 됩니다.

결론적으로 작은 타입은 암시적으로 큰 타입으로 변환 되지 않습니다. `Byte`가 `Int`로 명시적으로 언급하지 않으면 변환되지 않습니다.

```kotlin
fun main() {
//sampleStart
    val b: Byte = 1 // OK, literals are checked statically
    val i: Int = b // ERROR
//sampleEnd
}
```
명시적 변환을 통해 넓은 범위의 타입으로 변환이 가능합니다.

```kotlin
fun main() {
    val b: Byte = 1
//sampleStart
    val i: Int = b.toInt() // OK: explicitly widened
    print(i)
//sampleEnd
}
```
모든 숫자 타입은 아래와 같은 변환을 제공합니다:

* `toByte(): Byte`
* `toShort(): Short`
* `toInt(): Int`
* `toLong(): Long`
* `toFloat(): Float`
* `toDouble(): Double`
* `toChar(): Char`

타입이 문맥에서 추론되어지고 산술연산에서는 적절하고 오버로드 되기 때문에 암묵적 형변환 부재에 대한 불편함이 없습니다. 예를 들어

```kotlin
val l = 1L + 3 // Long + Int => Long
```

### 연산 (Operations)

Kotlin은 적절한 class 멤버로 선언 된 숫자에 대해 기본적인 산술연산 (`+` `-` `*` `/` `%`)을 제공합니다 (그러나 컴파일러는 해당 호출에 대해 최적화 합니다).
[Operator overloading](https://kotlinlang.org/docs/reference/operator-overloading.html) 참고 바랍니다.

#### 정수 나누기 (Division of integers)

정수로 나누기를 진행하면 항상 정수를 반환합니다. 정수 외에 소수점은 버림을 진행합니다. 예를 들어:

```kotlin
fun main() {
//sampleStart
    val x = 5 / 2
    //println(x == 2.5) // ERROR: Operator '==' cannot be applied to 'Int' and 'Double'
    println(x == 2)
//sampleEnd
}
```

이것은 어떠한 두 정수일 경우에 해당합니다.

```kotlin
fun main() {
//sampleStart
    val x = 5L / 2
    println(x == 2L)
//sampleEnd
}
```
명시적으로 하나의 인자에 부동 소수 타입으로 지정 시 부동 소수 타입으로 반환합니다.

```kotlin
fun main() {
//sampleStart
    val x = 5 / 2.toDouble()
    println(x == 2.5)
//sampleEnd
}
```

#### 비트 연산자 (Bitwise operations)

비트 연산을 나타내는 특별한 문자는 없습니다. 그러나, 비트 연산자는 infix 형식으로 호출할 수 있습니다. 예를 들어:
* NOTE: infix 형식이란 일반 적으로 비트 연산 함수 호출을 하려면 `1.shl(2)`의 형식으로 호출을 해야 되나 infix 형식이 가능하여 `1 shl 2`의 형식으로 호출 되는 것을 infix 형식이라 합니다.

```kotlin
val x = (1 shl 2) and 0x000FF000
```

비트 연산자는 아래와 같습니다 (`Int` 와 `Long` 타입만 가능):

* `shl(bits)` – signed shift left
* `shr(bits)` – signed shift right
* `ushr(bits)` – unsigned shift right
* `and(bits)` – bitwise __and__
* `or(bits)` – bitwise __or__
* `xor(bits)` – bitwise __xor__
* `inv()` – bitwise inversion

### 부동 소수점 비교 (Floating point numbers comparison)

부동 소수점 비교에 대한 연산자는 아래와 같습니다:

* 동등 여부: `a == b` and `a != b`
* 비교 연산자: `a < b`, `a > b`, `a <= b`, `a >= b`
* 범위 인스턴스와 범위 체크: `a..b`, `x in a..b`, `x !in a..b`

`a`와 `b`가  `Float` 또는 `Double` 타입이거나 null이 가능한 타입 (선언되거나 유추되거나 [smart cast](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts)의 결과)일 경우의 숫자와 범위에 대한 연산은 EEE 754 Standard for Floating-Point Arithmetic 따릅니다.

그러나 부동 소수가 정적 타입으로 되지 않은 경우 (예. `Any`, `Comparable<...>`, 타입 파라미터) 제너릭 사용을 지원하고 total ordering을 지원하기 위해 `Float` 와 `Double` 타입에 대해 `equals` 와 `compareTo`을 사용하는데 아래와 같이 표준과 일치 하지 않습니다:

* `NaN` 은 자신과 동등
* `NaN` 은 `POSITIVE_INFINITY`을 포함하여 어떠한 것보다 큼
* `-0.0` 은 `0.0` 보다 작음

## 문자 (Characters)

문자는 `Char`로 표기합니다. 문자 타입은 숫자를 직접 다룰 수 없습니다.

```kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```

문자 리터럴은 홑따옴표로 나타냅니다: `'1'`.
특수 문자는 백슬래시를 포함하여 표기합니다: `\t`, `\b`, `\n`, `\r`, `\'`, `\"`, `\\`, `\$`.
어떠한 문자를 인코딩 하려면 유니코드 표기를 사용합니다: `'\uFF00'`.

문자는 명시적 변환으로 `Int`로 변환이 가능합니다:

```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```

숫자와 문자는 null이 가능한 참조가 필요한 경우 박싱됩니다.
박싱되면 동일한 객체로 생성되지 않습니다.

## Booleans

`Boolean`로 표기하며 2개의 값을 가지고 있습니다:
*true* 와 *false*.

null이 가능한 참조가 필요한 경우 박싱됩니다.

boolean은 아래와 같은 연산자를 포함합니다.

* `||` – lazy disjunction
* `&&` – lazy conjunction
* `!` - negation

## 배열 (Arrays)

Kotlin에서 배열은 `Array` class로 표기되며, `get` 과 `set` 함수 (오버로드 변환에 따라 `[]`로 전환 됨)를 가지고 있으며, `size` 프로퍼티와 몇가지 유용한 멤버 함수를 가지고 있습니다:

```kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

`arrayOf()` 함수를 통해 배열을 생성할 수 있습니다. `arrayOf(1, 2, 3)`으로 배열 `[1, 2, 3]`을 만들 수 있습니다. 다른 방법으로는 `arrayOfNulls()` 함수를 이용하여 지정된 크기의 null로 채워진 배열을 만들 수 있습니다.

또 다른 방법으론 `Array` 생성자에 배열 크기를 넣고 초기 배열 요소값을 반환 할 수 있는 함수를 사용하는 것 입니다:

```kotlin
fun main() {
//sampleStart
    // Creates an Array<String> with values ["0", "1", "4", "9", "16"]
    val asc = Array(5) { i -> (i * i).toString() }
    asc.forEach { println(it) }
//sampleEnd
}
```

앞에서 말한 것과 같이, `[]` 연산자는 `get()` 과 `set()` 멤버 함수를 호출합니다.

Kotlin에서 배열은 _불변 (invariant)_ 입니다. 이것은 런타임시 발생할 수 있는 오류를 방지하기 위해 `Array<String>`은 `Array<Any>`에 할당할 수 없다는 뜻입니다 (`Array<out Any>`은 사용 가능, [Type Projections](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/generics#type-projections)을 참고). 

### 원시 타입 배열 (Primitive type arrays)

Kotlin은 박싱 오버헤드가 없는 원시 타입의 배열을 나타내는 class를 가지고 있습니다: `ByteArray`, `ShortArray`, `IntArray` 등. 이 class는 `Array` class를 상속 받고 있지 않지만, `Array`와 같은 메서드와 프로퍼티를 가지고 있습니다. 그리고 각각의 대응하는 팩토리 함수를 가지고 있습니다:

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

```kotlin
// Array of int of size 5 with values [0, 0, 0, 0, 0]
val arr = IntArray(5)

// e.g. initialise the values in the array with a constant
// Array of int of size 5 with values [42, 42, 42, 42, 42]
val arr = IntArray(5) { 42 }

// e.g. initialise the values in the array using a lambda
// Array of int of size 5 with values [0, 1, 2, 3, 4] (values initialised to their index value)
var arr = IntArray(5) { it * 1 } 
```


## 부호없는 정수 (Unsigned integers)

> 부호없는 타입은 Kotlin 1.3 이상 버전에서 가능합니다. 자세한 사항은 [below](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/untitled#experimental-status-of-unsigned-integers) 참고바랍니다.

Kotlin은 아래와 같이 부호없는 정수 타입을 따릅니다:

* `kotlin.UByte`: an unsigned 8-bit integer, ranges from 0 to 255
* `kotlin.UShort`: an unsigned 16-bit integer, ranges from 0 to 65535
* `kotlin.UInt`: an unsigned 32-bit integer, ranges from 0 to 2^32 - 1
* `kotlin.ULong`: an unsigned 64-bit integer, ranges from 0 to 2^64 - 1

부호없는 타입은 대부분의 연산을 지원하고 있습니다.

> 부호없는 타입에서 부호있는 정수 타입으로 변경 (그 반대도 포함) 은 호환성이 보장되지 않습니다.

부호없는 타입은 [inline classes](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-inline-classes)를 사용하여 구현되어져 있습니다.

### 특수 class (Specialized classes)

기본 배열 타입과 마찬가지로 각 부호없는 타입에도 배열을 가지고 있으며 아래와 같습니다:

* `kotlin.UByteArray`: an array of unsigned bytes
* `kotlin.UShortArray`: an array of unsigned shorts
* `kotlin.UIntArray`: an array of unsigned ints
* `kotlin.ULongArray`: an array of unsigned longs

부호있는 정수 배열과 마찬가지로 박싱 오버헤드가 없는 `Array`와 유사한 API를 제공합니다.

또한, `kotlin.ranges.UIntRange`, `kotlin.ranges.UIntProgression`, `kotlin.ranges.ULongRange`, `kotlin.ranges.ULongProgression` class를 통해 `UInt` 와 `ULong`의 [ranges and progressions](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/ranges-and-progressions-1)을 지원합니다.

### 리터럴 (Literals)

부호없는 정수를 쉽게 사용하기 위해 Kotlin은 Float/Long 타입처럼 접미사를 제공합니다:
* `u` 와 `U` 태그는 부호없는 정수를 나타냅니다. 타입에 대한 명시를 해주면 그 타입에 맞게 제공되며, 명시하지 않을 경우 `UInt` 또는 `ULong` 중 적절한 타입을 알아서 제공하게 됩니다.

```kotlin
val b: UByte = 1u  // UByte, expected type provided
val s: UShort = 1u // UShort, expected type provided
val l: ULong = 1u  // ULong, expected type provided

val a1 = 42u // UInt: no expected type provided, constant fits in UInt
val a2 = 0xFFFF_FFFF_FFFFu // ULong: no expected type provided, constant doesn't fit in UInt
```

* `uL` 와 `UL`는 부호없는 Long 타입의 태그입니다.

```kotlin
val a = 1UL // ULong, even though no expected type provided and constant fits into UInt
```

### 부호없는 정수의 실험적 상태 (Experimental status of unsigned integers)

부호없는 타입은 실험적인 타입입니다. 이 뜻은 이 타입은 안정성을 보장하지 않으며, 계속해서 특성이 변경됩니다. Kotlin 1.3+ 에서 부호없는 연산을 사용하면 이 기능은 실험적이라는 경고가 나타나게 됩니다. 이 경고를 나타나지 않게 하려면 부호없는 타입에 대한 실험적 기능을 사용하겠다는 설정을 해줘야 합니다.

실험적인 기능에 대한 설정 방법은 2가지가 있습니다: API에 experimental을 표시하거나 표시하지 않는 방법입니다.

- `@ExperimentalUnsignedTypes` annotation을 부호없는 정수에 선언해 줍니다.
- `@OptIn(ExperimentalUnsignedTypes::class)` annotation을 선언하거나 컴파일러에 `-Xopt-in=kotlin.ExperimentalUnsignedTypes` 설정을 해줍니다.

항상 부호없는 타입은 실험적이기 때문에 Kotlin에 변경사항이 생기면 기존에 API를 사용 못할 수도 있다는 것을 명심하시기 바랍니다.

추가적인 정보는 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/experimental.md)를 참고하시기 바랍니다.

### 추가 논의 (Further discussion)

[language proposal for unsigned types](https://github.com/Kotlin/KEEP/blob/master/proposals/unsigned-types.md)를 참고하시기 바랍니다.

## 문자열 (Strings)

문자열은 `String`으로 표기합니다. 문자열은 불변입니다. 문자열의 문자는 인덱싱을 통해 접근할 수 있습니다: `s[i]`.
문자열은 *for*-loop를 통해 문자에 접근할 수도 있습니다:

```kotlin
fun main() {
val str = "abcd"
//sampleStart
for (c in str) {
    println(c)
}
//sampleEnd
}
```

문자열은 `+` 연산을 통해 서로 연결할 수 있습니다. 문자열은 `+` 연산을 통해 다른 타입과도 연결이 가능합니다. 연결 된 타입은 문자열로 나타납니다:

```kotlin
fun main() {
//sampleStart
val s = "abc" + 1
println(s + "def")
//sampleEnd
}
```
대부분의 경우 [string templates](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/untitled#string-templates)나 raw 문자열을 통해 연결하는 것이 더 효율적입니다.

### 문자열 리터럴 (String literals)

Kotlin은 2개의 문자열 리터럴 타입을 가지고 있습니다: 이스케이프 된 문자를 가지고 있을 때는 이스케이프 된 문자열이라고 하며, raw 문자열은  개행이나 임의의 텍스트를 가진 문자열 입니다. 아래는 이스케이프 된 문자열의 예입니다:

```kotlin
val s = "Hello, world!\n"
```

이스케이핑은 백슬래시와 함께 사용됩니다. 이스케이프 문자는 [Characters](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/untitled#characters) 부분을 참고하시기 바랍니다.

raw 문자열은 세개의 큰따옴표로 구분되며 이스케이핑이 포함되지 않고, 개행이나 임의의 어떠한 문자도 포함이 가능합니다:

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

[`trimMargin()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/trim-margin.html) 함수를 이용하여 앞에 공백 제거가 가능합니다:

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

기본적으로 `|`가 마진 접두사로 사용되며, 다른 문자를 사용하고 싶다면 `trimMargin(">")`와 같이 직접 지정해서 사용하면 가능합니다.

### 문자열 템플릿 (String templates)

문자열 리터럴은 템플릿 표현을 포함할 수 있습니다. 예를 들어 계산 코드의 일부분이나 어떠한 결과값을 포함하여 문자열로 나타내 줍니다.
템플릿 표현은 달러 표시($)로 시작합니다:

```kotlin
fun main() {
//sampleStart
    val i = 10
    println("i = $i") // prints "i = 10"
//sampleEnd
}
```

또는 괄호 (`{``}`) 안에 표현할 수 있습니다:

```kotlin
fun main() {
//sampleStart
    val s = "abc"
    println("$s.length is ${s.length}") // prints "abc.length is 3"
//sampleEnd
}
```

템플릿은 raw 문자열과 이스케이프 된 문자열을 모두 지원합니다. 만약에 raw 문자열 (백슬래시 이스케이핑을 지원하지 않음)에서 `$` 문자를 표현하고 싶다면 아래와 같이 표현하면 가능합니다:

```kotlin
val price = """
${'$'}9.99
"""
```