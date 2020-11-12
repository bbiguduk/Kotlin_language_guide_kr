# 프로퍼티와 필드 \(Properties and Fields\)

## 프로퍼티 선언 \(Declaring Properties\)

Kotlin 클래 안에서 프로퍼티는 _var_로 수정가능하게 또는 _val_로 읽기전용으로 선언이 가능합니다.

```kotlin
class Address {
    var name: String = "Holmes, Sherlock"
    var street: String = "Baker"
    var city: String = "London"
    var state: String? = null
    var zip: String = "123456"
}
```

간단하게 프로퍼티를 사용하는 방법은 선언한 이름을 이용합니다:

```kotlin
fun copyAddress(address: Address): Address {
    val result = Address() // there's no 'new' keyword in Kotlin
    result.name = address.name // accessors are called
    result.street = address.street
    // ...
    return result
}
```

## Getter와 Setter \(Getters and Setters\)

프로퍼티를 선언하는 전체 구문은 아래와 같습니다

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

Getter와 setter는 선택사항입니다. 프로퍼티 타입이 초기화 값이나 아래와 같이 getter 반환 타입으로 유추가 가능할 경우 생략해도 됩니다.

예:

```kotlin
var allByDefault: Int? // error: explicit initializer required, default getter and setter implied
var initialized = 1 // has type Int, default getter and setter
```

읽기전용 프로퍼티를 선언하는 방식은 수정가능 한 프로퍼티 선언과 2가지가 다릅니다. 첫번째로 `var` 대신 `val`로 선언해야 하며, setter를 사용할 수 없습니다:

```kotlin
val simple: Int? // has type Int, default getter, must be initialized in constructor
val inferredType = 1 // has type Int and a default getter
```

프로퍼티는 커스텀 접근자를 지정할 수 있습니다. 커스텀 getter를 지정하면 프로퍼티에 접근할 때마다 호출됩니다 \(이를 통해 계산 된 프로퍼티 제공 가능\). 예제를 참고 바랍니다:

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

커스텀 setter를 지정할 수 있습니다. 프로퍼티에 값을 할당 할 때마다 호출됩니다. 커스텀 setter는 아래를 참고 바랍니다:

```kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
```

편의를 위해 setter의 파라미터는 `value`라는 이름을 가집니다. 그러나 원하면 다른 이름도 사용이 가능합니다.

Kotlin 1.1 이후부터, getter에서 프로퍼티 타입이 유추가 가능하면 생략해도 됩니다:

```kotlin
val isEmpty get() = this.size == 0  // has type Boolean
```

접근자의 가시성은 변경하지만 기본 구형을 변경할 필요가 없는 경우 바디 정의 없이 접근자를 정의할 수 있습니다:

```kotlin
var setterVisibility: String = "abc"
    private set // the setter is private and has the default implementation

var setterWithAnnotation: Any? = null
    @Inject set // annotate the setter with Inject
```

### 뒷받침 필드 \(Backing Fields\)

Kotlin 클래스에서 필드는 직접적으로 선언할 수 없습니다. 그러나 프로퍼티가 뒷받침 필드 \(backing field\)가 필요할 때 Kotlin은 자동으로 제공해줍니다. `field` 식별자를 통해 뒷받침 필드에 접근할 수 있습니다:

```kotlin
var counter = 0 // Note: the initializer assigns the backing field directly
    set(value) {
        if (value >= 0) field = value
    }
```

`field` 식별자는 프로퍼티 접근을 위해서만 사용 가능합니다.

하나 이상의 접근자로 기본 구현을 하거나 `field` 식별자를 통해 커스텀 접근자를 참조하는 경우 프로퍼티에 뒷받침 필드는 생성됩니다.

다음의 예에는 뒷받침 필드가 없는 케이스 입니다:

```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

### 뒷받침 프로퍼티 \(Backing Properties\)

"암묵적 뒷받침 필드" 스킴에 맞지 않는 작업을 하려고 한다면, 항상 _뒷받침 프로퍼티 \(backing property\)_를 가질 수 있습니다:

```kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```

> **JVM 경우**: 기본 getter와 setter를 사용하여 private 프로퍼티를 접근하는 것은 최적화 되어있어 함수 호출 오버헤드가 발생하지 않습니다.

## 컴파일-타임 상수 \(Compile-Time Constants\)

컴파일 시 읽기전용 프로퍼티의 값을 알고 있으면 _const_를 사용하여 _컴파일 타입 상수 \(compile time constant\)_ 라고 표시해 줍니다: 이러한 프로퍼티는 다음의 요구사항을 충족해야 합니다:

* [_객체_ 선언 \(_object_ declaration\)](object-expressions-and-declarations.md#object-declarations) 또는 [companion 객체 \(a _companion object_\)](object-expressions-and-declarations.md#companion-objects) __의 최상위 또는 멤버.
* `String` 타입의 값으로 초기화 또는 원시 타입
* 커스텀 getter가 없어야 함

이러한 프로퍼티는 annotation에서도 사용 가능합니다:

```kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```

## 늦은 초기화 프로퍼티와 변수 \(Late-Initialized Properties and Variables\)

일반적으로, 프로퍼티는 생성자에서 null이 아닌 값으로 초기화 되어야 합니다. 이러한 초기화가 꼭 들어맞지 않은 경우가 있습니다. 예를 들어 프로퍼티가 dependency injection이나 유닛 테스트의 setup 메서드에서 초기화 되는 경우 생성자에서 null이 아닌 값으로 초기화가 불가능 합니다. 그러나 클래스의 바디에서 프로퍼티 참조 시 null 체크를 피하고 싶은 경우가 있습니다.

이 경우에, `lateinit`로 프로퍼티를 표기하면 가능합니다:

```kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // dereference directly
    }
}
```

`lateinit`은 클래스의 바디 안에서 \(주 생성자가 아니고 커스텀 getter와 setter가 아닌 경우\) `var` 프로퍼티에 선언 될 수 있고, Kotlin 1.2 이후부턴 최상위 프로퍼티나 지역 변수에도 사용이 가능합니다. `lateinit` 으로 선언 된 프로퍼티나 변수는 반드시 null이 아니어야 하며 원시 타입이 아니어야 합니다.

초기화 되기 전에 `lateinit` 프로퍼티에 접근하면 특정 예외가 발생합니다.

### lateinit var 초기화 여부 확인 \(Checking whether a lateinit var is initialized\) \(since 1.2\)

`lateinit var`이 초기화 되었는지 확인하기 위해선 [프로퍼티 참조 \(reference to that property\)](https://kotlinlang.org/docs/reference/reflection.html#property-references) 의 `.isInitialized`을 사용합니다.

```kotlin
if (foo::bar.isInitialized) {
    println(foo.bar)
}
```

동일한 타입이나 외부 타입 중 하나 또는 동일 파일에서 최상위에 선언 된 접근 가능한 프로퍼티에 대해서만 확인 가능합니다.

## 재정의 프로퍼티 \(Overriding Properties\)

자세한 내용은 [재정의 프로퍼티 \(Overriding Properties\)](class-classes-and-inheritance.md#overriding-properties) 참고 바랍니다.

## 위임 된 프로퍼티 \(Delegated Properties\)

일반적인 프로퍼티는 뒷받침 필드에서 읽거나 쓸 수 있습니다. 반면에 커스텀 getter와 setter를 사용하면 프로퍼티의 모든 동작을 구현할 수 있습니다. 프로퍼티 동작에 대한 일반적인 패턴이 존재합니다. 예를 들어: lazy 값, 주어진 키로 map에서 읽기, database 접근, 접근 알림 등

이러한 동작은 [_위임된 프로퍼티 \(delegated properties\)_](delegated-properties.md) __을 이용하여 라이브러리로 구현 할 수 있습니다.

