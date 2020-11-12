# 함수형 \(SAM\) 인터페이스 \(Functional \(SAM\) interfaces\)

하나의 추상 메서드만 있는 인터페이스를 _함수형 인터페이스_ 라고 하거나 _단일 추상 메서드 \(Single Abstract Method\) \(SAM\) 인터페이스_ 라고 합니다. 함수형 인터페이스는 비추상 멤버를 여러개 가질 수 있지만 추상 멤버는 오직 하나만 가질 수 있습니다.

Kotlin에서 함수형 인터페이스를 선언하기 위해 `fun` 수식어를 사용합니다.

```kotlin
fun interface KRunnable {
   fun invoke()
}
```

## SAM 변환 \(SAM conversions\)

함수형 인터페이스의 경우 [람다 표현식 \(lambda expressions\)](../functions-and-lambdas/higher-order-functions-and-lambdas.md#lambda-expression-syntax) 을 사용하여 코드를 더 간결하고 읽기 쉽게 만드는데 도움이 되는 SAM 변환을 사용할 수 있습니다.

함수형 인터페이스를 수동으로 구현하는 클래스를 생성하는 대신에 람다 표현식을 사용할 수 있습니다. SAM 변환을 통해 Kotlin은 서명이 인터페이스의 단일 메서드 서명과 일치하는 모든 람다 표현식을 인터페이스를 구현하는 클래스의 인스턴스로 변환할 수 있습니다.

예를 들어 다음의 Kotlin 함수형 인터페이스를 생각해 봅시다:

```kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}
```

SAM 변환을 사용하지 않으면 아래와 같이 코드를 작성해야 합니다:

```kotlin
// Creating an instance of a class
val isEven = object : IntPredicate {
   override fun accept(i: Int): Boolean {
       return i % 2 == 0
   }
}
```

Kotlin의 SAM 변환을 활용하면 대신 다음과 같은 동등한 코드를 작성할 수 있습니다:

```kotlin
// Creating an instance using lambda
val isEven = IntPredicate { it % 2 == 0 }
```

짧은 람다 표현식은 불필요한 코드를 모두 대체합니다.

```kotlin
fun interface IntPredicate {
   fun accept(i: Int): Boolean
}

val isEven = IntPredicate { it % 2 == 0 }

fun main() {
   println("Is 7 even? - ${isEven.accept(7)}")
}
```

[Java 인터페이스에 대한 SAM 변환 \(SAM conversions for Java interfaces\)](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions) 도 사용할 수 있습니다.

## 함수형 인터페이스 vs 타입 별칭 \(Functional interfaces vs. type aliases\)

함수형 인터페이스와 [타입 별칭 \(type aliases\)](type-aliases.md) 는 다른 용도로 사용됩니다. 타입 별칭은 기존 타입의 이름 일 뿐입니다 – 함수형 인터페이스와 다르게 새로운 타입을 생성하지 않습니다.

타입 별칭은 오직 하나의 멤버만 가질 수 있는 반면에 함수형 인터페이스는 여러개의 비추상 멤버와 하나의 추상 멤버를 가질 수 있습니다. 함수형 인터페이스는 다른 인터페이스를 구현하고 확장할 수도 있습니다.

위의 사항을 고려하면 함수형 인터페이스는 더 유연하고 타입 별칭보다 더 많은 기능을 제공합니다.

