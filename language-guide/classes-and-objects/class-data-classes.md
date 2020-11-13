# 데이터 클래스 \(Data Classes\)

데이터를 가지는 것이 목적인 클래스를 자주 만듭니다. 이러한 클래스에서 일부 표준 기능과 유틸리티 기능은 종종 데이터로 부터 기계적으로 파생될 수 있습니다. Kotlin에서 이것을 _데이터 클래스   
\(data class\)_ 라 부르며 `data`로 표기합니다:

```kotlin
data class User(val name: String, val age: Int)
```

컴파일러는 자동적으로 주 생성자에 선언 된 모든 프로퍼티로 부터 다음 멤버들을 파생합니다:

* `equals()`/`hashCode()` 쌍
* `"User(name=John, age=42)"` 의 `toString()`;
* 프로퍼티 선언된 순서대로의 [`componentN()` 함수 \(`componentN()` functions\)](https://kotlinlang.org/docs/reference/multi-declarations.html);
* `copy()` 함수 \(아래 내용 참고\).

생성 된 코드의 일관성과 동작을 보장하기 위해 데이터 클래스는 아래의 요구사항을 충족해야 합니다:

* 주 생성사는 적어도 하나 이상의 파라미터가 있어야 합니다;
* 모든 주 생성자 파라미터는 `val` 또는 `var` 로 표기해야 합니다;
* 데이터 클래스는 abstract, open, sealed 또는 inner 로 사용할 수 없습니다.
* \(Kotlin 1.1 이전만 해당\) 데이터 클래스 인터페이스로만 구현이 가능합니다.

추가적으로 멤버 생성은 멤버 상속과 관련하여 아래의 규칙을 따라야 합니다:

* 데이터 클래스의 본문에 `equals()`, `hashCode()` 또는 `toString()`을 명시적으로 구현하거나 슈퍼 클래스를 _final_로 선언하면 이 함수들은 생성되지 않고 구현된 함수를 사용합니다;
* 슈퍼타입이 _open_이고 반환 타입이 호환이 되는 `componentN()` 함수를 가지고 있다면 데이터 클래스에 해당 함수가 생성되며 슈퍼타입을 재정의 합니다. 슈퍼타입의 함수가 호환되지 않은 유형이거나 final로 재정의 할 수 없다면 에러가 발생합니다;
* `copy(...)` 함수를 가지고 있는 상속하는 것은 Kotlin 1.2에서 deprecated 되었고 1.3에선 금지 되었습니다.
* `componentN()` 과 `copy()` 함수를 직접적으로 구현하는 것은 불가합니다.

Kotlin 1.1부터 데이터 클래스는 다른 클래스를 확장 할 수 있습니다 \(자세한 내용은 [한정 클래스 \(Sealed classes\)](class-sealed-classes.md) 를 참고 바랍니다\).

JVM에서 파리미터가 없는 생성자가 필요하다면 모든 프로퍼티는 기본값을 가지고 있어야 합니다 \(자세한 내용은 [생성자 \(Constructors\)](class-classes-and-inheritance.md#constructors) 를 참고 바랍니다\).

```kotlin
data class User(val name: String = "", val age: Int = 0)
```

## 클래스에 선언 된 프로퍼티 \(Properties Declared in the Class Body\)

컴파일러는 자동으로 생성 된 함수를 위한 주 생성자에 선언 된 프로퍼티만 사용합니다. 특정 프로퍼티를 제외하려면 클래스 바디 안에 선언 하면 됩니다:

```kotlin
data class Person(val name: String) {
    var age: Int = 0
}
```

`name` 프로퍼티만 `toString()`, `equals()`, `hashCode()`, `copy()`에서 사용되며 하나의 컴퍼넌트 함수 `component1()`를 가지고 있습니다. 2개의 `Person` 객체가 다른 age 값을 가지고 있더라도 그 객체는 같은 것으로 간주합니다.

```kotlin
data class Person(val name: String) {
    var age: Int = 0
}
fun main() {
//sampleStart
    val person1 = Person("John")
    val person2 = Person("John")
    person1.age = 10
    person2.age = 20
//sampleEnd
    println("person1 == person2: ${person1 == person2}")
    println("person1 with age ${person1.age}: ${person1}")
    println("person2 with age ${person2.age}: ${person2}")
}
```

## 복사 \(Copying\)

프로퍼티의 일부를 변경하지만 나머지는 변경하지 않기 위해 가끔 복사를 사용해야 합니다. 이러한 기능을 위해 `copy()` 함수가 생성됩니다. 위에서 `User` 클래스는 아래와 같이 구현 됩니다:

```kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)
```

아래와 같이 쓸 수 있습니다:

```kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

## 데이터 클래스와 구조해체 선언 \(Data Classes and Destructuring Declarations\)

데이터 클래스의 _컴포넌트 함수 \(Component functions\)_ 은 [구조해체 선언 \(destructuring declarations\)](https://kotlinlang.org/docs/reference/multi-declarations.html) 에 사용할 수 있습니다:

```kotlin
val jane = User("Jane", 35) 
val (name, age) = jane
println("$name, $age years of age") // prints "Jane, 35 years of age"
```

## 표준 데이터 클래스 \(Standard Data Classes\)

기본 라이브러리는 `Pair` 와 `Triple`을 제공합니다. 대부분의 경우 지정된 데이터 클래스는 의미있는 프로퍼티 이름을 제공하여 코드를 더 읽기 쉽게 하기 때문에 더 나은 디자인입니다.

