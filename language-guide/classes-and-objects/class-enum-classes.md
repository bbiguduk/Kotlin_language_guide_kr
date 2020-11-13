# 열거형 클래스 \(Enum Classes\)

열거형 클래스의 가장 기본적인 사용은 타입 안전 열거형을 구현하는 것입니다:

```kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```

각 열거형 상수는 객체입니다. 열거형 상수는 콤마로 구분합니다.

## 초기화 \(Initialization\)

각 열거형은 열거형 클래스의 인스턴스 이기 때문에 초기화가 가능합니다:

```kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

## 익명 클래스 \(Anonymous Classes\)

열거형 상수는 기본 메서드를 재정의 할 수 있을 뿐 아니라 메서드를 사용하여 익명 클래스를 선언 할 수도 있습니다.

```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```

열거형 클래스가 멤버를 정의하는 경우 열거형 상수 정의를 멤버 정의와 세미콜론으로 구분합니다.

열거형 항목은 내부 클래스 외에 다른 중첩 타입을 포함할 수 없습니다 \(Kotlin 1.2에서 deprecated 되었음\).

## 열거형 클래스 안에 인터페이스 구현 \(Implementing Interfaces in Enum Classes\)

열거형 클래스는 인터페이스 \(클래스에서 파생되지 않음\)를 구현할 수 있으며, 모든 항목에 단일 인터페이스 멤버 구현을 제공하거나 익명 클래스 내에 각 항목에 대해 별도의 인터페이스 멤버를 제공할 수 있습니다. 이것은 다음과 같이 열거형 클래스 선언에 인터페이스를 추가하여 수행합니다:

```kotlin
import java.util.function.BinaryOperator
import java.util.function.IntBinaryOperator

//sampleStart
enum class IntArithmetics : BinaryOperator<Int>, IntBinaryOperator {
    PLUS {
        override fun apply(t: Int, u: Int): Int = t + u
    },
    TIMES {
        override fun apply(t: Int, u: Int): Int = t * u
    };

    override fun applyAsInt(t: Int, u: Int) = apply(t, u)
}
//sampleEnd

fun main() {
    val a = 13
    val b = 31
    for (f in IntArithmetics.values()) {
        println("$f($a, $b) = ${f.apply(a, b)}")
    }
}
```

## 열거형 상수 동작 \(Working with Enum Constants\)

Kotlin에서 열거형 클래스는 정의 된 열거형 상수를 나열하고 이름별로 열거형 상수를 얻을 수 있는 메서드가 있습니다. 아래를 참고하시기 바랍니다 \(열거형 클래스를 `EnumClass`로 가정\):

```kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

클래스에 정의 된 열거형 상수와 일치 하지 않으면 `valueOf()` 메서드는 `IllegalArgumentException`을 발생시킵니다.

Kotlin 1.1 부터 `enumValues<T>()` 와 `enumValueOf<T>()` 함수를 이용하여 제너릭으로 열거형 상수에 접근할 수 있습니다:

```kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>() // prints RED, GREEN, BLUE
```

모든 열거형 상수는 선언 된 열거형 클래스에서 이름과 포지션을 얻을 수 있는 프로퍼티를 가지고 있습니다:

```kotlin
val name: String
val ordinal: Int
```

열거형 상수는 [비교 \(Comparable\)](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/) 인터페이스를 구현하며 자연적으로 열거형 클래스 안에 정의 된 순서대로 포지션을 갖습니다.

