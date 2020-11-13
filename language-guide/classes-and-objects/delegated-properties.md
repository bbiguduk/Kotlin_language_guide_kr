# 위임된 프로퍼티 \(Delegated Properties\)

매번 필요할 때마다 구현하지만 한 번 구현해서 라이브러리에 넣으면 좋을만한 공통 프로퍼티가 있습니다. 예를 들어:

* 지연 \(lazy\) 프로퍼티: 처음 접근할 때만 계산되는 값;
* 관찰자 \(observable\) 프로퍼티: 리스너는 프로퍼티가 변하면 알림을 받음
* 분리 된 프로퍼티 대신 맵에 프로퍼티 저장

이러한 역할을 위해 Kotlin은 _위임된 프로퍼티 \(delegated properties\)_를 지원합니다:

```kotlin
class Example {
    var p: String by Delegate()
}
```

문법: `val/var <property name>: <Type> by <expression>`. _by_ 뒤에 표현은 프로퍼티의 `get()` 과 `set()`이 호출되면 `getValue()` 와 `setValue()` 함수로 위임하는 _위임자 \(delegate\)_ 입니다. 프로퍼티 위임자는 인터페이스를 구현할 필요는 없지만 `getValue()` 함수 \(_var_를 위한 `setValue()` 함수\)를 제공해야 합니다. 예를 들어:

```kotlin
import kotlin.reflect.KProperty

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

`Delegate`의 인스턴스에 위임하는 `p`를 읽을 때 첫 번째 파라미터가 `p`를 읽은 객체이고 두 번째 파라미터는 `p`자체가 됩니다. 예를 들어:

```kotlin
val e = Example()
println(e.p)
```

출력 값은:

```text
Example@33a17727, thank you for delegating ‘p’ to me!
```

`p`에 값을 할당할 때는 `setValue()` 함수가 호출됩니다. 첫 번째 두 번째 파라미터는 `getValue()`와 같고 세 번째 파라미터가 할당할 값에 해당합니다:

```kotlin
e.p = "NEW"
```

이 출력 값은

```text
NEW has been assigned to ‘p’ in Example@33a17727.
```

위임된 객체에 대한 요구사항은 [아래](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/delegated-properties#property-delegate-requirements) 를 참고 바랍니다.

Kotlin 1.1 부터 위임된 프로퍼티는 클래스의 멤버일 필요가 없으며 함수나 코드 블럭 안에 선언할 수 있습니다. 아래의 [예제](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/delegated-properties#local-delegated-properties-since-1-1) 를 참고 바랍니다.

## 표준 위임자 \(Standard Delegates\)

Kotlin 표준 라이브러리는 위임자를 위한 몇개의 유용한 팩토리 메서드를 제공합니다.

### 지연 \(Lazy\)

[`lazy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html)는 람다를 받아 지연 프로퍼티를 구현한 위임자인 `Lazy<T>`의 인스턴스를 반환하는 함수입니다: `get()`을 처음 호출하면 `lazy()` 전달한 람다가 실행되며 결과가 저장됩니다. 이후에 `get()`을 호출하면 저장 된 값을 반환합니다.

```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}
```

기본적으로 지연 프로퍼티는 **동기적**입니다: 값은 오직 하나의 스레드에서 계산되고 모든 스레드는 같은 값을 주시합니다. 위임자 초기화를 동기화가 필요하지 않아 여러 스레드가 동시에 실행할 수 있도록 하려면 `lazy()` 함수 파라미터에 `LazyThreadSafetyMode.PUBLICATION`을 전달하면 됩니다. 항상 하나의 스레드에서만 초기화가 일어난다면 다중 스레드 안전성을 보장하지 않는 대신 관련된 부하를 줄일 수 있는 `LazyThreadSafetyMode.NONE`을 사용할 수 있습니다.

### 관찰자 \(Observable\)

[`Delegates.observable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/observable.html)은 초기값과 수정을 위한 핸들러 인자를 가지고 있습니다. 핸들러는 프로퍼티에 값이 할당될 때마다 호출 \(할당이 일어난 _후 \(after\)_\)됩니다. 핸들러는 3개의 파라미터를 가지고 있으며 할당된 프로퍼티, 기존 값, 새로운 값 입니다:

```kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main() {
    val user = User()
    user.name = "first"
    user.name = "second"
}
```

할당이 일어나기 전에 가로채고 싶다면 `observable()` 대신에 [`vetoable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html)을 사용하면 됩니다. 프로퍼티에 값이 할당되기 _전 \(before\)_에 핸들러는 `vetoable`에 전달합니다.

## 맵에서 프로퍼티 저장 \(Storing Properties in a Map\)

맵에 프로퍼티 값을 저장하는 것도 위임된 프로퍼티의 케이스 중 하나입니다. JSON을 파싱하거나 다른 "동적"인 일을 하는 애플리케이션에서 주로 사용합니다. 이 경우에 위임된 프로퍼티에 위임자로 맵 인스턴스 자체를 사용할 수 있습니다.

```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

이 예제에선 생성자는 맵을 가집니다:

```kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```

위임된 프로퍼티는 맵으로 부터 값을 가져옵니다:

```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}

fun main() {
    val user = User(mapOf(
        "name" to "John Doe",
        "age"  to 25
    ))
//sampleStart
    println(user.name) // Prints "John Doe"
    println(user.age)  // Prints 25
//sampleEnd
}
```

읽기 전용인 `Map` 대신에 `MutableMap`을 이용해 _var_ 프로퍼티를 사용할 수 있습니다.

```kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```

## 지역 위임된 프로퍼티 \(Local Delegated Properties\) \(since 1.1\)

위임된 프로퍼티를 지역 변수로 선언할 수 있습니다. 예를 들어 지연 지역 변수를 만들 수 있습니다:

```kotlin
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)

    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
```

`memoizedFoo` 변수는 처음 접근할 때 계산될 것입니다. `someCondition`이 실패라면 변수는 계산되지 않을 것입니다.

## 프로퍼티 위임자 요구사항 \(Property Delegate Requirements\)

위임자 객체 요구사항을 정리하였습니다.

**읽기전용 \(read-only\)** 프로퍼티 \(_val_\)는 다음 파라미터를 가지는 `getValue()` 함수를 제공해야 합니다:

* `thisRef` --- _프로퍼티 소유_와 같은 타입 또는 슈퍼타입이어야 합니다. \(확장 프로퍼티의 경우 --- 확장된 타입\).
* `property` --- `KProperty<*>` 타입 또는 슈퍼타입이어야 합니다.

`getValue()` 프로퍼티와 같은 타입이거나 서브타입을 반환해야 합니다.

```kotlin
class Resource

class Owner {
    val valResource: Resource by ResourceDelegate()
}

class ResourceDelegate {
    operator fun getValue(thisRef: Owner, property: KProperty<*>): Resource {
        return Resource()
    }
}
```

**가변 \(mutable\)** 프로퍼티 \(_var_\)는 아래의 파라미터를 가지는 `setValue()` 함수도 제공해야 합니다:

* `thisRef` --- _property owner_ 와 같은 타입 또는 슈퍼타입이어야 합니다. \(확장 프로퍼티의 경우 --- 확장된 타입\).
* `property` --- `KProperty<*>` 타입 또는 슈퍼타입이어야 합니다.
* `value` --- 프로퍼티와 같은 타입이거나 서브타입이어야 합니다.

```kotlin
class Resource

class Owner {
    var varResource: Resource by ResourceDelegate()
}

class ResourceDelegate(private var resource: Resource = Resource()) {
    operator fun getValue(thisRef: Owner, property: KProperty<*>): Resource {
        return resource
    }
    operator fun setValue(thisRef: Owner, property: KProperty<*>, value: Any?) {
        if (value is Resource) {
            resource = value
        }
    }
}
```

`getValue()` 와 `setValue()` 함수는 위임자 class 또는 확장 함수의 멤버 함수로 제공되어야 합니다. 후자의 경우 해당 함수를 제공하지 않는 객체에 위임된 프로퍼티를 추가할 때 편리합니다. 두 함수 모두 `operator` 키워드를 붙여야 합니다.

위임 class는 `operator` 메서드를 가진 `ReadOnlyProperty` 와 `ReadWriteProperty` 인터페이스를 구현하기도 합니다. 이 인터페이스는 Kotlin 표준 라이브러리에 선언되어 있습니다:

```kotlin
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```

### Translation Rules

위임된 프로퍼티를 위해 Kotlin 컴파일러는 보조 프로퍼티를 만들어 위임합니다. 예를 들어 `prop` 프로퍼티를 위해 숨겨진 프로퍼티 `prop$delegate`를 생성되고 위임합니다:

```kotlin
class C {
    var prop: Type by MyDelegate()
}

// this code is generated by the compiler instead:
class C {
    private val prop$delegate = MyDelegate()
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```

Kotlin 컴파일러는 `prop`의 모든 정보를 인자에 제공합니다: 첫 번째 인자 `this`는 `C` class의 인스턴스를 나타내고 `this::prop`는 `prop`을 나타내는 `KProperty`에 반영 된 객체 입니다.

`this::prop` 표현과 같은 참조는 [bound callable reference](https://kotlinlang.org/docs/reference/reflection.html#bound-function-and-property-references-since-11)라고 하며 Kotlin 1.1 부터 지원합니다.

### 위임 제공 \(Providing a delegate\) \(since 1.1\)

`provideDelegate` 연산자를 정의해서 프로퍼티 구현이 위임 된 객체를 생성하는 로직을 확장할 수 있습니다. `by`의 오른편에 `provideDelegate`의 멤버나 확장 함수로 정의 한 객체는 해당 함수는 프로퍼티의 위임자 프로퍼티를 생성하기 위해 호출 됩니다.

`provideDelegate`의 사용 가능 한 케이스 중 하나는 프로퍼티의 정합성 판단입니다. 이것은 getter 또는 setter가 호출 될 때 뿐만 아니라 생성될 때도 판단합니다.

예를 들어 바인딩 전에 프로퍼티 이름을 체크하고 싶으면 아래와 같이 작성하면 됩니다:

```kotlin
class ResourceDelegate<T> : ReadOnlyProperty<MyUI, T> {
    override fun getValue(thisRef: MyUI, property: KProperty<*>): T { ... }
}

class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(
            thisRef: MyUI,
            prop: KProperty<*>
    ): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, prop.name)
        // create delegate
        return ResourceDelegate()
    }

    private fun checkProperty(thisRef: MyUI, name: String) { ... }
}

class MyUI {
    fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { ... }

    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```

`provideDelegate`는 `getValue`의 파라미터와 같습니다:

* `thisRef` --- _property owner_ 와 같은 타입 또는 슈퍼타입이어야 합니다. \(확장 프로퍼티의 경우 --- 확장된 타입\).
* `property` --- `KProperty<*>` 타입 또는 슈퍼타입이어야 합니다.

`provideDelegate` 메서드는 `MyUI` 인스턴스를 생성하는 동안 각 프로퍼티마다 호출되고 바로 검증을 수행합니다.

프로퍼티와 위임자를 연결을 인터셉트 할 수 없다면 같은 기능을 하기위해 불편하지만 프로퍼티 이름을 명시적으로 전달해야 합니다:

```kotlin
// Checking the property name without "provideDelegate" functionality
class MyUI {
    val image by bindResource(ResourceID.image_id, "image")
    val text by bindResource(ResourceID.text_id, "text")
}

fun <T> MyUI.bindResource(
        id: ResourceID<T>,
        propertyName: String
): ReadOnlyProperty<MyUI, T> {
   checkProperty(this, propertyName)
   // create delegate
}
```

생성된 코드에 `provideDelegate` 메서드는 보조 프로퍼티 `prop$delegate`을 초기화 할 때 호출됩니다. 프로퍼티 선언 `val prop: Type by MyDelegate()`에 대해 생성 된 코드와 위의 생성 된 코드 \(`provideDelegate` 메서드가 없는 경우\)를 비교해 보시기 바랍니다:

```kotlin
class C {
    var prop: Type by MyDelegate()
}

// this code is generated by the compiler 
// when the 'provideDelegate' function is available:
class C {
    // calling "provideDelegate" to create the additional "delegate" property
    private val prop$delegate = MyDelegate().provideDelegate(this, this::prop)
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```

`provideDelegate` 메서드가 오직 보조 프로퍼티 생성에 영향을 끼치고 getter 또는 setter를 위한 생성된 코드에는 영향을 미치지 않는 점에 주목하시기 바랍니다.

