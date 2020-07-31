# 중첩과 내부 class (Nested and Inner Classes)

class는 다른 class 안에 중첩 될 수 있습니다:

```kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

## 내부 class (Inner classes)

*inner*로 명시 된 중첩 class는 외부 class의 멤버에 접근할 수 있습니다. 내부 class는 외부 class의 객체에 대한 참조를 가지고 있습니다:

```kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

내부 class의 *this*의 차이점에 대해서는 [Qualified *this*{: .keyword } expressions](https://kotlinlang.org/docs/reference/this-expressions.html)를 참고 바랍니다.

## 익명 내부 class (Anonymous inner classes)

익명 내부 class 인스턴스는 [object expression](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/object-expressions-and-declarations#object-expressions)을 사용하여 생성됩니다:

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
