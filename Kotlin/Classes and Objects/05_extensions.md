# 확장 (Extensions)

Kotlin은 class를 상속하거나 Decorator와 같은 디자인 패턴을 사용하지 않고 새로운 기능을 확장할 수 있도록 제공해 줍니다.
이것은 _extensions_ 을 통해 선언 됩니다.
예를 들어 수정할 수 없는 서드파티 라이브러리에 있는 class에 대해 새로운 함수를 작성할 수 있습니다.
이러한 함수는 기존 class에 있는 메서드처럼 호출하여 사용이 가능합니다.
이 메카니즘을 _(함수 확장) extension functions_ 이라 부릅니다. 기존 class에 새로운 프로퍼티를 정의할 수 있으며, 이것을 _(프로퍼티 확장) extension properties_ 이라 합니다.

## 함수 확장 (Extension functions)

확장 함수를 선언하려면 이름 앞에 _receiver type_ 즉, 확장되는 타입을 접두어에 붙여야 합니다.
다음은 `MutableList<Int>`에 `swap` 함수를 추가합니다:

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

확장 함수내에 *this* 키워드는 리시버 객체에 해당합니다.
이제, `MutableList<Int>`에서 다음과 같은 함수를 호출 할 수 있습니다:

```kotlin
val list = mutableListOf(1, 2, 3)
list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'
```

이 함수는 `MutableList<T>`에서 잘 동작할 수 있으며, 제너릭으로 선언 가능합니다:

```kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' corresponds to the list
    this[index1] = this[index2]
    this[index2] = tmp
}
```

리시버 타입 표현식에서 사용 가능하도록 함수 이름 앞에 제너릭 타입 파라미터를 선언합니다.
자세한 내용은 [Generic functions](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/generics)를 참고 바랍니다.

## 확장은 **정적**으로 처리된다 (Extensions are resolved **statically**)

확장은 실제로 class를 확장하여 수정하는 것이 아닙니다. 확장에 대해 정의해보면 class에 새로운 멤버를 추가하지 않지만 확장하면 그 타입의 변수에 .로 확장해서 만든 새로운 함수를 호출 할 수 있다는 의미입니다.

확장 함수는 정적으로 전달 된다는 것을 명심해야 합니다. 즉, 리시버 타입에 따라 동작하지 않습니다.
이 뜻은 런타임 시 요청에 따라 다른 타입으로 동작하는 것이 아니라 확장 함수에 선언 된 타입으로 동작 한다는 의미입니다. 예를 들어:

```kotlin
fun main() {
//sampleStart
    open class Shape
    
    class Rectangle: Shape()
    
    fun Shape.getName() = "Shape"
    
    fun Rectangle.getName() = "Rectangle"
    
    fun printClassName(s: Shape) {
        println(s.getName())
    }    
    
    printClassName(Rectangle())
//sampleEnd
}
```

이 예제의 결과는 "_Shape_" 입니다. 이유는 확장 함수에 정의 된 파라미터 `s`는 `Shape` class 타입이므로 선언 된 타입에 의존하기 때문입니다.

class에 같은 리시버 타입과 같은 이름 같은 인자가 주어진 멤버 함수와 확장 함수가 존재 할 때 항상 **멤버가 더 우선순위가 높습니다 (member always wins)**.
예:

```kotlin
fun main() {
//sampleStart
    class Example {
        fun printFunctionType() { println("Class method") }
    }
    
    fun Example.printFunctionType() { println("Extension function") }
    
    Example().printFunctionType()
//sampleEnd
}
```

위 예제는 "_Class method_"가 출력됩니다.

그러나 멤버 함수를 오버로드하는 확장 함수는 언제나 완벽하게 동작합니다:

```kotlin
fun main() {
//sampleStart
    class Example {
        fun printFunctionType() { println("Class method") }
    }
    
    fun Example.printFunctionType(i: Int) { println("Extension function") }
    
    Example().printFunctionType(1)
//sampleEnd
}
```

## null이 가능한 리시버 (Nullable receiver)

확장은 null이 가능한 리시버 타입으로 정의 될 수 있습니다. null인 객체가 확장 함수를 호출 할 수도 있고 `this == null`와 같은 구문으로 체크도 가능합니다.
Kotlin에서 toString()은 null 체크 없이 호출이 가능합니다: null 체크는 확장 함수에서 체크합니다.

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // after the null check, 'this' is autocast to a non-null type, so the toString() below
    // resolves to the member function of the Any class
    return toString()
}
```

## 프로퍼티 확장 (Extension properties)

함수와 비슷하게 Kotlin은 프로퍼티 확장을 지원합니다:

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

class에 멤버를 추가하는 것이 아니기 때문에 확장 프로퍼티는 [backing field](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/untitled#backing-fields)를 가질 수 없습니다. 그래서 **프로퍼티 확장은 초기화를 사용할 수 없습니다**. getter/setter를 통해서만 정의할 수 있습니다.

예:

```kotlin
val House.number = 1 // error: initializers are not allowed for extension properties
```


## 컴패니언 객체 확장 (Companion object extensions)

class에 [companion object](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/object-expressions-and-declarations#companion-objects)가 정의되어 있으면 companion object도 함수와 프로퍼티를 확장할 수 있습니다. companion object의 일반 멤버를 호출하듯이 class 이름을 사용하여 호출 할 수 있습니다:

```kotlin
class MyClass {
    companion object { }  // will be called "Companion"
}

fun MyClass.Companion.printCompanion() { println("companion") }

fun main() {
    MyClass.printCompanion()
}
```


## 확장의 범위 (Scope of extensions)

대부분 확장은 패키지 바로 아래인 최상위에 정의 합니다:

```kotlin
package org.example.declarations
 
fun List<String>.getLongestString() { /*...*/}
```

선언한 패키지의 외부에서 확장을 사용하려면 import를 해야 합니다:

```kotlin
package org.example.usage

import org.example.declarations.getLongestString

fun main() {
    val list = listOf("red", "green", "blue")
    list.getLongestString()
}
```

자세한 사항은 [Imports](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/import-packages-and-imports#imports) 참고 바랍니다.

## 확장을 멤버로 선언 (Declaring extensions as members)

class 안에서 다른 class를 위한 확장을 선언할 수 있습니다. 확장에는 여러개의 _암시적 리시버_ 가 있습니다 - 한정자 없이 접근 가능한 객체. 확장이 선언 된 class의 인스턴스를 _dispatch receiver_ 라고 부르며 확장 메서드의 리시버 타입 인스턴스를 _extension receiver_ 라고 부릅니다.

```kotlin
class Host(val hostname: String) {
    fun printHostname() { print(hostname) }
}

class Connection(val host: Host, val port: Int) {
     fun printPort() { print(port) }

     fun Host.printConnectionString() {
         printHostname()   // calls Host.printHostname()
         print(":")
         printPort()   // calls Connection.printPort()
     }

     fun connect() {
         /*...*/
         host.printConnectionString()   // calls the extension function
     }
}

fun main() {
    Connection(Host("kotl.in"), 443).connect()
    //Host("kotl.in").printConnectionString(443)  // error, the extension function is unavailable outside Connection
}
```

dispatch 수신자와 확장 수신자의 멤버 이름이 같은 경우 확장 수신자가 더 우선입니다. dispatch 수신자의 멤버를 참조하기 위해선 [qualified `this` syntax](https://kotlinlang.org/docs/reference/this-expressions.html#qualified)을 사용하면 됩니다.

```kotlin
class Connection {
    fun Host.getConnectionString() {
        toString()         // calls Host.toString()
        this@Connection.toString()  // calls Connection.toString()
    }
}
```

멤버로 선언 된 확장은 `open`로 선언할 수 있고 subclass에서 오버라이드 할 수 있습니다. 이것은 dispatch 수신자 타입에 따라 가변적이기도 하지만 확장 수신자 타입에 따라 정적이라는 의미입니다.

```kotlin
open class Base { }

class Derived : Base() { }

open class BaseCaller {
    open fun Base.printFunctionInfo() {
        println("Base extension function in BaseCaller")
    }

    open fun Derived.printFunctionInfo() {
        println("Derived extension function in BaseCaller")
    }

    fun call(b: Base) {
        b.printFunctionInfo()   // call the extension function
    }
}

class DerivedCaller: BaseCaller() {
    override fun Base.printFunctionInfo() {
        println("Base extension function in DerivedCaller")
    }

    override fun Derived.printFunctionInfo() {
        println("Derived extension function in DerivedCaller")
    }
}

fun main() {
    BaseCaller().call(Base())   // "Base extension function in BaseCaller"
    DerivedCaller().call(Base())  // "Base extension function in DerivedCaller" - dispatch receiver is resolved virtually
    DerivedCaller().call(Derived())  // "Base extension function in DerivedCaller" - extension receiver is resolved statically
}
```

## Note on visibility

확장은 일반 함수에 선언 된 범위와 같은 [visibility of other entities](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/visibility-modifiers)을 이용합니다. 예:

* 파일에 가장 최상위에 선언 된 확장은 같은 파일에 있는 다른 `private` 최상위 선언에 접근할 수 있습니다;
* 확장이 리시버 타입 외부에 선언 된 경우 이러한 확장은 리시버의 `private` 멤버에 접근할 수 없습니다.
