# 객체 표현과 선언 \(Object Expressions and Declarations\)

때로는 새로운 서브 class를 명시적으로 선언하지 않고 일부 class를 약간 수정한 객체를 만들어야 할 때가 있습니다. Kotlin은 이 경우에 _객체 표현 \(object expressions\)_ 와 _객체 선언 \(object declarations\)_로 처리합니다.

## 객체 표현 \(Object expressions\)

일부 타입 \(또는 타입들\)에서 상속되는 익명 class의 객체를 생성하려면 다음과 같이 작성해야 합니다:

```kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { /*...*/ }

    override fun mouseEntered(e: MouseEvent) { /*...*/ }
})
```

슈퍼타입에 생성자가 있으면 적절한 생성자 파라미터를 전달해야 합니다. 콜론 뒤에 많은 슈퍼타입이 쉼표로 구분되어 지정할 수 있습니다:

```kotlin
open class A(x: Int) {
    public open val y: Int = x
}

interface B { /*...*/ }

val ab: A = object : A(1), B {
    override val y = 15
}
```

슈퍼타입 없이 단지 객체만 필요로 한다면 쉽게 표현할 수 있습니다:

```kotlin
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
}
```

익명 객체는 로컬과 private에 선언 해야 타입으로 사용할 수 있습니다. public 함수 또는 public 프로퍼티 타입을 반환하는 익명 객체를 사용한다면 실제 타입은 익명 객체에 선언 된 슈퍼타입이거나 `Any`입니다. 익명 객체에 추가한 멤버는 접근할 수 없습니다.

```kotlin
class C {
    // Private function, so the return type is the anonymous object type
    private fun foo() = object {
        val x: String = "x"
    }

    // Public function, so the return type is Any
    fun publicFoo() = object {
        val x: String = "x"
    }

    fun bar() {
        val x1 = foo().x        // Works
        val x2 = publicFoo().x  // ERROR: Unresolved reference 'x'
    }
}
```

객체 표현식의 코드는 괄호안에서는 변수에 접근할 수 있습니다.

```kotlin
fun countClicks(window: JComponent) {
    var clickCount = 0
    var enterCount = 0

    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }

        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
    // ...
}
```

## 객체 선언 \(Object declarations\)

[Singleton](http://en.wikipedia.org/wiki/Singleton_pattern)은 유용한 객체 선언 방법 중 하나이며, Kotlin은 이것을 쉽게 선언할 수 있습니다:

```kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```

_객체 선언 \(object declaration\)_이라 부르며 항상 이름 앞에 _object_ 키워드를 넣어야 합니다. 변수 선언과 마찬가지로 객체 선언은 표현식이 아니며 대입 문의 오른쪽에서 사용할 수 없습니다.

객체 선언의 초기화는 스레드로부터 안전하며 첫번째 접근 시 수행됩니다.

객체를 참조하려면 이름을 직접적으로 사용하면 됩니다:

```kotlin
DataProviderManager.registerDataProvider(...)
```

이러한 객체들은 슈퍼타입을 가질 수 있습니다:

```kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { ... }

    override fun mouseEntered(e: MouseEvent) { ... }
}
```

**참고**: 객체 선언은 함수내에 직접 중첩될 수 있지만 로컬일 수 없습니다. 다른 객체 선언이나 내부 class가 아닌 곳에는 중첩이 가능합니다.

### 동반 객체 \(Companion Objects\)

class 안에 객체 선언은 _companion_ 키워드와 함께 사용할 수 있습니다:

```kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

동반 객체의 멤버는 간단하게 class 이름으로 호출 할 수 있습니다:

```kotlin
val instance = MyClass.create()
```

동반 객체 이름은 생략할 수 있으며 이런 경우 `Companion`을 사용합니다:

```kotlin
class MyClass {
    companion object { }
}

val x = MyClass.Companion
```

다른 이름이 아닌 해당 class 자체의 이름으로 동반 객체에 참조가 가능합니다:

```kotlin
class MyClass1 {
    companion object Named { }
}

val x = MyClass1

class MyClass2 {
    companion object { }
}

val y = MyClass2
```

동반 객체의 멤버는 다른 언어에서 static 처럼 보이지만 런타임 시 객체의 인스턴스 멤버이며 인터페이스를 구현할 수 있습니다:

```kotlin
interface Factory<T> {
    fun create(): T
}

class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}

val f: Factory<MyClass> = MyClass
```

`@JvmStatic` annotation을 사용하면 동반 객체의 멤버를 static 메서드와 static 필드로 생성할 수 있습니다. 자세한 내용은 [Java interoperability](https://kotlinlang.org/docs/reference/java-to-kotlin-interop.html#static-fields)을 참고 바랍니다.

### 객체 표현과 선언의 의미상 차이 \(Semantic difference between object expressions and declarations\)

객체 표현과 객체 선언에는 아주 중요한 하나의 차이가 있습니다:

* 객체 표현은 사용할 때 **즉시** 실행과 초기화 됩니다.
* 객체 선언은 처음 접근 할 때 **늦게** 초기화 됩니다.
* 동반 객체는 class가 로드 될 때 초기화 되며, Java의 static 초기화와 의미가 일치합니다.

