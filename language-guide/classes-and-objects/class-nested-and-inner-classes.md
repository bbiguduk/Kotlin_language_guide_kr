# 중첩된 클래스와 내부 클래스 \(Nested and Inner Classes\)

클래스는 다른 클래스 안에 중첩 될 수 있습니다:

```kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

## 내부 클래스 \(Inner classes\)

_inner_로 명시 된 중첩 클래스는 외부 클래스의 멤버에 접근할 수 있습니다. 내부 클래스는 외부 클래스의 객체에 대한 참조를 가지고 있습니다:

```kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

내부 클래스의 _this_의 차이점에 대해서는 [정규화된 _this_ 표현식 \(Qualified _this_ expressions\)](https://kotlinlang.org/docs/reference/this-expressions.html) 을 참고 바랍니다.

## 익명 내부 클래스 \(Anonymous inner classes\)

익명 내부 클래스 인스턴스는 [객체 포현식 \(object expression\) ](object-expressions-and-declarations.md#object-expressions)을 사용하여 생성됩니다:

```kotlin
window.addMouseListener(object : MouseAdapter() {

    override fun mouseClicked(e: MouseEvent) { ... }

    override fun mouseEntered(e: MouseEvent) { ... }
})
```

_참고_: JVM에서 객체가 예를 들어 단일 추상 메서드가 있는 Java 인터페이스 인 경우 람다 표현식으로 나타낼 수 있습니다:

```kotlin
val listener = ActionListener { println("clicked") }
```

