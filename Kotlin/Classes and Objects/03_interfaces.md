# 인터페이스 (Interfaces)

Kotlin에서 인터페이스는 메서드 구현 뿐만 아니라 추상 메서드 선언도 포함될 수 있습니다. 추상 class와의 차이점은 상태 저장을 할 수 없다는 것입니다. 인터페이스는 프로퍼티를 가질 수 있으나 추상 프로퍼티 이거나 접근자 구현은 제공해야 합니다.

*interface* 키워드를 사용해 인터페이스를 정의합니다.

```kotlin
interface MyInterface {
    fun bar()
    fun foo() {
      // optional body
    }
}
```

## 인터페이스 구현 (Implementing Interfaces)

class나 object는 하나 이상의 인터페이스를 구현할 수 있습니다

```kotlin
class Child : MyInterface {
    override fun bar() {
        // body
    }
}
```

## 인터페이스에서의 프로퍼티 (Properties in Interfaces)

인터페이스에서 프로퍼티를 선언할 수 있습니다. 인터페이스에 선언 된 프로퍼티는 추상 프로퍼티 이거나 접근자를 위한 구현부를 제공 할 수 있습니다. 인터페이스에 선언 된 프로퍼티는 backing field를 가질 수 없기 때문에 인터페이스 선언 된 접근자는 참조가 불가능합니다.

```kotlin
interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```

## 인터페이스 상속 (Interfaces Inheritance)

인터페이스는 다른 인터페이스에서 파생 될 수 있으므로 멤버에 대한 구현을 제공하고 새로운 함수와 프로퍼티를 선언을 제공합니다. 이러한 인터페이스를 구현하는 class는 누락 없이 모두 구현 되어야 합니다:

```kotlin
interface Named {
    val name: String
}

interface Person : Named {
    val firstName: String
    val lastName: String
    
    override val name: String get() = "$firstName $lastName"
}

data class Employee(
    // implementing 'name' is not required
    override val firstName: String,
    override val lastName: String,
    val position: Position
) : Person
```

## 오버라이딩 충돌 해결 (Resolving overriding conflicts)
슈퍼타입에 많은 타입을 선언할 때 같은 메서드를 구현해야 할 경우가 발생합니다. 예를 들어

```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
    }
}
```

*foo()* 와 *bar()* 함수가 선언 된 2개의 인터페이스가 존재한다고 생각해 봅시다. *foo()* 함수는 각각의 인터페이스에 구현이 되어 있으며, *bar()* 함수는 *B* 인터페이스에서만 구현이 되어 있습니다 (*A* 인터페이스에서 *bar()* 함수는 바디를 가지고 있지 않기 때문에 추정 메서드로 표시하지 않습니다). *A*로 부터 파생 된 *C* class가 존재 한다면 *bar()* 함수를 오버라이드하고 구현하기 위해 제공해 주어야 합니다.

그러나 *A* 와 *B*에서 *D*를 파생하는 경우 모든 메서드를 구현해야 합니다. 그리고 *D*가 구현하는 방법을 정확히 지정해야 합니다. 이것은 한 곳에만 구현 된 (*bar()*)와 여러 군데 구현 된 (*foo()*) 모두 적용 됩니다.