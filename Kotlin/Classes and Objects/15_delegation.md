# 위임 (Delegation)

## 프로퍼티 위임 (Property Delegation)

프로퍼티 위임은 다른 페이지에 상세히 설명되어 있습니다: [Delegated Properties](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/delegated-properties).

## 위임으로의 구현 (Implementation by Delegation)

[Delegation pattern](https://en.wikipedia.org/wiki/Delegation_pattern)은 상속 구현에 대한 좋은 대안책으로 Kotlin는 상용구 없는 코드를 요구합니다.
`Derived` class는 모든 public 멤버를 특정 객체에 위임함으로써 `Base` 인터페이스를 구현할 수 있습니다:

```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
    val b = BaseImpl(10)
    Derived(b).print()
}
```

`Derived`의 슈퍼타입에 *by*-clause는 `b`가 `Derived`의 객체에 내부적으로 저장되고 컴파일러는 `b`로 전달되는 모든 `Base`의 메서드를 생성한다는 것을 나타냅니다.

### 위임으로 구현 된 인터페이스의 멤버 오버라이드 (Overriding a member of an interface implemented by delegation)

[Overrides](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-classes-and-inheritance#overriding-methods)는 기존과 동일하게 동작합니다: 컴파일러는 위임 객체 대신 `override` 구현을 사용합니다. `Derived` class에 `override fun printMessage() { print("abc") }`을 추가한다면 `printMessage`을 호출 했을 때 "10"이 아닌 "abc"가 호출 됩니다.

```kotlin
interface Base {
    fun printMessage()
    fun printMessageLine()
}

class BaseImpl(val x: Int) : Base {
    override fun printMessage() { print(x) }
    override fun printMessageLine() { println(x) }
}

class Derived(b: Base) : Base by b {
    override fun printMessage() { print("abc") }
}

fun main() {
    val b = BaseImpl(10)
    Derived(b).printMessage()
    Derived(b).printMessageLine()
}
```
그러나 이렇게 오버라이드 된 멤버는 위임 객체의 멤버에서 호출되지 않으며, 인터페이스 멤버 자체 구현에만 접근 할 수 있습니다:

```kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}

class Derived(b: Base) : Base by b {
    // This property is not accessed from b's implementation of `print`
    override val message = "Message of Derived"
}

fun main() {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}
```

> **JVM**: `default` 메서드가 있는 인터페이스를 위임 (Kotlin `@JvmDefault`가 있는 인터페이스 포함)에 사용하는 경우 실제 위임 타입이 자체 구현을 제공하더라도 기본 구현이 호출됩니다.
>  더 자세한 내용은 [Calling Kotlin from Java](https://kotlinlang.org/docs/reference/java-to-kotlin-interop.html#using-in-delegates)를 참고 바랍니다.