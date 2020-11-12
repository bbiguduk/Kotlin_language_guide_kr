# 클래스와 상속 \(Classes and Inheritance\)

## 클래스 \(Classes\)

Kotlin에서 클래스는 _class_ 키워드를 사용하여 선언합니다:

```kotlin
class Invoice { /*...*/ }
```

클래스는 클래스 이름, 클래스 헤더 \(타입 파라미터, 생성자 등\) 그리고 괄호\(```{``}```\)로 묶인 클래스 바디로 이루어져 있습니다. 헤더와 바디는 선택사항입니다; 만약에 클래스가 바디가 없다면 괄호를 생략하면 됩니다.

```kotlin
class Empty
```

### 생성자 \(Constructors\)

Kotlin에서 class는 **기본 생성자 \(primary constructor\)** 와 하나 또는 그 이상의 **보조 생성자 \(secondary constructors\)**를 가지고 있습니다. 기본 생성자는 클래스 헤더의 부분입니다: 클래스 이름 다음에 위치합니다 \(타입 파라미터를 가질 수도 있습니다\).

```kotlin
class Person constructor(firstName: String) { /*...*/ }
```

기본 생성자는 어떠한 annotation 또는 접근 제어자를 가지고 있지 않다면, _constructor_ 키워드는 생략 가능합니다:

```kotlin
class Person(firstName: String) { /*...*/ }
```

기본 생성자는 어떠한 코드도 포함되지 않습니다. 초기화 코드는 _init_ 키워드로 시작하는 **초기화 블럭 \(initializer blocks\)**에 위치 할 수 있습니다.

인스턴스 초기화 중에 초기화 블럭은 클래스 바디에 나타나는 순서대로 프로퍼티 초기화와 인터리브됩니다.

```kotlin
//sampleStart
class InitOrderDemo(name: String) {
    val firstProperty = "First property: $name".also(::println)

    init {
        println("First initializer block that prints ${name}")
    }

    val secondProperty = "Second property: ${name.length}".also(::println)

    init {
        println("Second initializer block that prints ${name.length}")
    }
}
//sampleEnd

fun main() {
    InitOrderDemo("hello")
}
```

기본 생성자의 파라미터는 초기화 블럭에서 사용 가능합니다. 그리고 클래스 바디에서 선언 된 프로퍼티 초기화 시에도 사용 가능합니다.

```kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```

Kotlin에서는 기본 생성자에서 초기화하거나 프로퍼티에서 선언을 간단하게 할 수 있습니다:

```kotlin
class Person(val firstName: String, val lastName: String, var age: Int) { /*...*/ }
```

클래스 프로퍼티를 선언할 때 [후행 콤마 \(trailing comma\)](../getting-started/coding-conventions.md#trailing-commas) 를 사용할 수 있습니다:

```kotlin
class Person(
    val firstName: String,
    val lastName: String,
    var age: Int, // trailing comma
) { /*...*/ }
```

기본적인 프로퍼티 선언 처럼 기본 생성자에 선언 된 프로퍼티는 변경가능 \(_var_\) 또는 읽기전용 \(_val_\)으로 선언할 수 있습니다.

생성자가 annotation 또는 접근 제어자를 가지고 있으면, _constructor_ 키워드를 반드시 표기해야 하며, _constructor_ 키워드 전에 annotation 또는 접근 제어자를 표기해야 합니다:

```kotlin
class Customer public @Inject constructor(name: String) { /*...*/ }
```

자세한 내용은 [접근 제한자 \(Visibility Modifiers\)](visibility-modifiers.md) 참고 바랍니다.

#### 보조 생성자 \(Secondary constructors\)

클래스는 _constructor_ 키워드를 통해 **보조 생성자 \(secondary constructors\)**를 선언할 수 있습니다.

```kotlin
class Person {
    var children: MutableList<Person> = mutableListOf<>()
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

클래스가 기본 생성자를 가지고 있다면, 각각의 보조 생성자는 직접 또는 다른 보조 생성자를 통해 간접적으로 기본 생성자로 위임해야 합니다. 같은 클래스에 다른 생성자로 위임을 하기 위해선 _this_ 키워드를 사용합니다:

```kotlin
class Person(val name: String) {
    var children: MutableList<Person> = mutableListOf<>()
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

초기화 블럭 안에 코드는 효과적으로 기본 생성자의 일부가 됩니다. 기본 생성자로의 위임은 보조 생성자의 첫 번째 문에서 발생합니다. 보조 생성자의 바디가 실행되기 전에 모든 초기화 블럭과 프로퍼티 초기화가 실행됩니다. 클래스가 기본 생성자를 가지고 있지 않더라도 위임은 암묵적으로 이루어 지며, 초기화 블럭도 실행됩니다:

```kotlin
//sampleStart
class Constructors {
    init {
        println("Init block")
    }

    constructor(i: Int) {
        println("Constructor")
    }
}
//sampleEnd

fun main() {
    Constructors(1)
}
```

비추상 클래스에 어떠한 생성자 \(기본 생성자 또는 보조 생성자\)도 선언하지 않으면, 자동으로 인자 없는 기본 생성자를 생성합니다. 생성자의 가시성은 public 으로 선언 됩니다. 클래스에 public 한 생성자를 원하지 않으면 접근 제어자와 함께 인자가 없는 기본 생성자를 선언하면 됩니다:

```kotlin
class DontCreateMe private constructor () { /*...*/ }
```

> **NOTE**: JVM에서 기본 생성자 파라미터에 모두 기본값이 있을경우 컴파일러는 기본값을 사용하는 파라미터가 없는 생성자를 생성합니다. 이것은 파라미터가 없는 생성자로 클래스 인스턴스를 만드는 Jackson 또는 JPA와 같은 라이브러리에서 Kotlin을 쉽게 사용할 수 있게 합니다.
>
> ```kotlin
> class Customer(val customerName: String = "")
> ```

### 클래스 인스턴스 생성 \(Creating instances of classes\)

클래스의 인스턴스를 생성하려면 생성자를 일반 함수처럼 호출 하면 됩니다:

```kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

참고: Kotlin은 _new_ 키워드를 가지고 있지 않습니다.

중첩, 내부와 익명 내부 클래스의 인스턴스 생성은 [중첩된 클래스 \(Nested classes\)](class-nested-and-inner-classes.md) 을 참고 바랍니다.

### 클래스 멤버 \(Class members\)

클래스는 아래를 포함할 수 있습니다:

* [생성자와 초기화 구문 블럭 \(Constructors and initializer blocks\)](class-classes-and-inheritance.md#constructors)
* [함수 \(Functions\)](../functions-and-lambdas/functions.md)
* [프로퍼티 \(Properties\)](untitled.md)
* [중첩된 클래스와 내부 클래스 \(Nested and Inner Classes\)](class-nested-and-inner-classes.md)
* [객체 선언 \(Object Declarations\)](object-expressions-and-declarations.md)

## 상속 \(Inheritance\)

Kotlin은 임의로 지정한 슈퍼타입이 없는 class에 대해 `Any` 슈퍼 class를 기본적으로 가지고 있습니다:

```kotlin
class Example // Implicitly inherits from Any
```

`Any`는 3개의 메서드를 가지고 있습니다: `equals()`, `hashCode()`, `toString()`. 따라서, Kotlin의 모든 class에는 3개의 메서드가 정의되어 있습니다.

기본적으로 Kotlin class는 상속이 불가능 한 final 입니다. 상속 가능한 class를 만들려면 `open` 키워드를 붙여야 합니다.

```kotlin
open class Base //Class is open for inheritance
```

슈퍼타입을 선언하려면 class 헤더 다음에 콜론을 넣고 콜론 다음에 타입을 입력합니다:

```kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

파생 class가 기본 생성자를 가지고 있다면 베이스 class는 기본 생성자의 파라미터를 이용하여 초기화 되거나 꼭 해줘야 합니다.

파생 class가 기본 생성자가 없으면 각 보조 생성자는 _super_ 키워드를 이용하여 기본타입 초기화를 하거나 다른 생성자에 위임 해줘야 합니다. 참고: 이 경우 다른 보조 생성자는 기본타입의 다른 생성자를 호출 할 수 있습니다:

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

### 오버라이딩 메서드 \(Overriding methods\)

앞에서 언급했듯이, Kotlin은 명시적으로 표현하는 것을 추구합니다. 그래서, Kotlin은 오버라이드 가능한 멤버 \(_open_이라 부름\)와 오버라이드에 대해 명시적 나타내야 합니다.

```kotlin
open class Shape {
    open fun draw() { /*...*/ }
    fun fill() { /*...*/ }
}

class Circle() : Shape() {
    override fun draw() { /*...*/ }
}
```

`Circle.draw()`는 _override_ 키워드가 필요합니다. 만약에 _override_ 키워드가 없으면 컴파일 시 문제가 생길 수 있습니다. final class \(_open_ 수식어 없는 class\) 멤버에 _open_ 수식어를 사용하여도 아무런 변화가 없습니다. `Shape.fill()`와 같이 _open_ 수식어가 없는 함수를 서브 class에서 _override_ 수식어를 사용하든 안하든 같은 이름으로 선언되지 않습니다.

_override_로 표시 된 멤버는 자체적으로 open 되어 있습니다. 즉, 서브 class에서 오버라이드 가능합니다. 만약 재 오버라이딩을 방지하려면 _final_ 키워드를 사용하면 됩니다.

```kotlin
open class Rectangle() : Shape() {
    final override fun draw() { /*...*/ }
}
```

### 프로퍼티 오버라이드 \(Overriding properties\)

프로퍼티 오버라이드는 메서드 오버라이드와 유사하게 동작합니다; 슈퍼 class에 선언 된 프로퍼티는 자식 class에 유효한 타입으로 _override_ 키워드와 함께 재 선언 해야 합니다. 각 선언 된 프로퍼티는 프로퍼티 초기화나 `get` 메서드로 재정의 할 수 있습니다.

```kotlin
open class Shape {
    open val vertexCount: Int = 0
}

class Rectangle : Shape() {
    override val vertexCount = 4
}
```

`val`를 `var`로 오버라이드 가능하지만 그 반대는 불가합니다. 이것은 기본적으로 `val`는 `get` 메서드를 선언하고 `var`로 오버라이드 시 `set` 메서드를 추가로 선언하기 때문에 가능합니다.

참고: _override_ 키워드는 기본 생성자 프로퍼티 선언 부분에도 사용할 수 있습니다.

```kotlin
interface Shape {
    val vertexCount: Int
}

class Rectangle(override val vertexCount: Int = 4) : Shape // Always has 4 vertices

class Polygon : Shape {
    override var vertexCount: Int = 0  // Can be set to any number later
}
```

### 파생 class 초기화 순서 \(Derived class initialization order\)

파생 class의 새로운 인스턴스를 생성하는 동안 기본 class 초기화는 첫 번째 단계 \(기본 class 생성자 인수로만 실행\)로 진행 되므로 파생 class의 초기화가 실행 되기 전에 발생합니다.

```kotlin
//sampleStart
open class Base(val name: String) {

    init { println("Initializing Base") }

    open val size: Int = 
        name.length.also { println("Initializing size in Base: $it") }
}

class Derived(
    name: String,
    val lastName: String
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {

    init { println("Initializing Derived") }

    override val size: Int =
        (super.size + lastName.length).also { println("Initializing size in Derived: $it") }
}
//sampleEnd

fun main() {
    println("Constructing Derived(\"hello\", \"world\")")
    val d = Derived("hello", "world")
}
```

이 말은 기본 class 생성자가 실행 될 때, 파생 class에 선언 된 프로퍼티나 오버라이드는 아직 초기화 되지 않았다는 의미입니다. 기본 class 초기화 로직 \(다른 오버라이드 된 _open_ 멤버를 통해 직접 또는 간접적\)에 이러한 프로퍼티를 사용할 경우 런타임 에러가 발생 할 수 있습니다.

### 슈퍼 class 호출 구현 \(Calling the superclass implementation\)

파생 class는 슈퍼 class 함수와 프로퍼티를 _super_ 키워드를 통해 호출 할 수 있습니다:

```kotlin
open class Rectangle {
    open fun draw() { println("Drawing a rectangle") }
    val borderColor: String get() = "black"
}

class FilledRectangle : Rectangle() {
    override fun draw() {
        super.draw()
        println("Filling the rectangle")
    }

    val fillColor: String get() = super.borderColor
}
```

내부 class 안에서 outer class의 슈퍼 class를 접근하기 위해선 _super_ 키워드와 outer class 이름을 명시해 줘야 합니다: `super@Outer`:

```kotlin
class FilledRectangle: Rectangle() {
    fun draw() { /* ... */ }
    val borderColor: String get() = "black"

    inner class Filler {
        fun fill() { /* ... */ }
        fun drawAndFill() {
            super@FilledRectangle.draw() // Calls Rectangle's implementation of draw()
            fill()
            println("Drawn a filled rectangle with color ${super@FilledRectangle.borderColor}") // Uses Rectangle's implementation of borderColor's get()
        }
    }
}
```

### 오버라이드 규칙 \(Overriding rules\)

Kotlin에서 상속 구현은 아래의 규칙을 따라야 합니다: 만약에 상속 된 여러개의 슈퍼 class에 같은 이름의 멤버가 있는 경우, 멤버를 오버라이드 하거나 재 구현할 수 있도록 제공해야 합니다 \(상속 된 class 중 하나\). 상속 된 구현 부분에서 슈퍼 타입을 표현하려면 _super_ 키워드에 꺾쇠 괄호에 슈퍼 타입 이름을 표기해야 합니다. `super<Base>`:

```kotlin
open class Rectangle {
    open fun draw() { /* ... */ }
}

interface Polygon {
    fun draw() { /* ... */ } // interface members are 'open' by default
}

class Square() : Rectangle(), Polygon {
    // The compiler requires draw() to be overridden:
    override fun draw() {
        super<Rectangle>.draw() // call to Rectangle.draw()
        super<Polygon>.draw() // call to Polygon.draw()
    }
}
```

`Rectangle` 와 `Polygon` 둘 다 상속하는 것이 좋지만, 둘 다 `draw()`를 가지고 있으므로, `Square`에서 `draw()`를 오버라이드 하고 구현을 할 수 있게 제공해줘야 합니다.

## 추상 class \(Abstract classes\)

class와 멤버 중 일부는 _추상 \(abstract\)_으로 선언 될 수 있습니다. 추상 멤버는 해당 class 안에 구현 부분을 가지고 있지 않습니다. 참고: 추상 class나 함수는 open 으로 선언 할 필요가 없습니다.

추상 멤버로 비추상 멤버를 오버로이드 할 수 있습니다.

```kotlin
open class Polygon {
    open fun draw() {}
}

abstract class Rectangle : Polygon() {
    abstract override fun draw()
}
```

## Companion objects

class 인스턴스 없이 class 내부 함수 \(예를 들어, 팩토리 메서드\)에 접근하려면 class 안에 [object declaration](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/object-expressions-and-declarations)으로 작성하면 가능합니다.

구체적으로, class 안에 [companion object](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/object-expressions-and-declarations#companion-objects)를 선언하면 class 이름만 가지고 해당 멤버에 접근할 수 있습니다.

