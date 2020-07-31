# 인라인 class (Inline classes)

> 오직 Kotlin 1.3 이후 버전에서만 가능한 인라인 class는 **실험적**인 기능입니다. 자세한 내용은 아래의 Experimental status of inline classes을 참고 바랍니다.

때로 비지니스 로직이 특정 타입의 랩퍼를 생성해야 할 수도 있습니다. 그러나 추가 힙 할당으로 인해 런타임 오버헤드가 발생합니다. 또한 랩핑 된 타입이 기본 타입일 경우 기본 타입은 런타임에 의해 대게 최적화 되는 반면에 랩퍼는 특별한 처리를 하지 않으므로 성능 저하가 심각합니다.

이러한 이슈를 해결하기 위해 Kotlin은 class 이름 앞에 `inline`을 붙여 선언하고 `inline class`라고 부르는 class가 있습니다:

```kotlin
inline class Password(val value: String)
```

인라인 class는 주 생성자에 초기화 된 단일 프로퍼티가 존재해야 합니다. 런타임 시 인라인 class의 인스턴스는 단일 프로퍼티를 사용하여 표시됩니다 (런타임 표현은 아래 Representation을 참고 바랍니다):

```kotlin
// No actual instantiation of class 'Password' happens
// At runtime 'securePassword' contains just 'String'
val securePassword = Password("Don't try this in production") 
```

이것이 인라인 class의 주요 기능으로 "inline" 이름에서 영감을 얻었습니다: class의 데이터는 "inlined" 입니다 ([inline functions](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/inline-functions) 내용이 사이트를 호출하도록 인라인 되는 것과 비슷).

## 멤버 (Members)

인라인 class는 일반적인 class의 기능을 제공합니다. 특히, 프로퍼티와 함수 선언이 가능합니다:

```kotlin
inline class Name(val s: String) {
    val length: Int
        get() = s.length

    fun greet() {
        println("Hello, $s")
    }
}    

fun main() {
    val name = Name("Kotlin")
    name.greet() // method `greet` is called as a static method
    println(name.length) // property getter is called as a static method
}
```

그러나 인라인 class 멤버는 몇가지 제한 사항이 있습니다:  
* 인라인 class는 *init* 블럭을 가질 수 없습니다.  
* 인라인 class 프로퍼티는 [backing fields](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/untitled#backing-fields)를 가질 수 없습니다.  
	* 인라인 class는 간단한 계산 가능한 프로퍼티만 가질 수 있습니다 (lateinit/위임된 프로퍼티가 아닌).


## 상속 (Inheritance)

인라인 class는 인터페이스 상속이 가능합니다:

```kotlin
interface Printable {
    fun prettyPrint(): String
}

inline class Name(val s: String) : Printable {
    override fun prettyPrint(): String = "Let's $s!"
}    

fun main() {
    val name = Name("Kotlin")
    println(name.prettyPrint()) // Still called as a static method
}
```

인라인 class가 class 계층에 참여되는 것은 금지됩니다. 이것은 인라인 class는 다른 class를 확장할 수 없으며 반드시 *final*이어야 합니다.

## 표현 (Representation)

생성된 코드에서 Kotlin 컴파일러는 각 인라인 class에 대해 **랩퍼**를 유지합니다. 인라인 class 인스턴스는 런타임에 랩퍼 또는 일반적 타입으로 표현 됩니다. 이것은 `Int`가 원시적 `int`나 `Integer` 랩퍼로 [표현 (represented)](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/untitled#representation) 되는 것과 비슷합니다.

Kotlin 컴파일러는 랩퍼 대신 최적화 되고 성능이 뛰어난 코드를 생성하기 위해 기본 타입을 선호합니다. 그러나 때때로 랩퍼를 유지해야 합니다. 일반적으로 인라인 class는 다른 타입으로 사용될 때마다 박스 됩니다.

```kotlin
interface I

inline class Foo(val i: Int) : I

fun asInline(f: Foo) {}
fun <T> asGeneric(x: T) {}
fun asInterface(i: I) {}
fun asNullable(i: Foo?) {}

fun <T> id(x: T): T = x

fun main() {
    val f = Foo(42) 
    
    asInline(f)    // unboxed: used as Foo itself
    asGeneric(f)   // boxed: used as generic type T
    asInterface(f) // boxed: used as type I
    asNullable(f)  // boxed: used as Foo?, which is different from Foo
    
    // below, 'f' first is boxed (while being passed to 'id') and then unboxed (when returned from 'id') 
    // In the end, 'c' contains unboxed representation (just '42'), as 'f' 
    val c = id(f)  
}
```  
인라인 class는 기본 값과 랩퍼로 표시될 수 있으므로 [referential equality](https://kotlinlang.org/docs/reference/equality.html#referential-equality)는 의미가 없으므로 금지합니다.

### 맹글링 (Mangling) - 컴파일러가 자신만 알아보는 타입으로 변경하는 것을 뜻함

인라인 class는 기본 타입으로 컴파일 되므로 플랫폼 시그니쳐 충돌과 같은 예기치 않은 에러가 발생할 수 있습니다:

```kotlin
inline class UInt(val x: Int)

// Represented as 'public final void compute(int x)' on the JVM
fun compute(x: Int) { }

// Also represented as 'public final void compute(int x)' on the JVM!
fun compute(x: UInt) { }
```

이러한 이슈를 완화하기 위해 인라인 class에서 사용하는 함수 이름에 안정적인 해시코드를 추가합니다. 따라서 `fun compute(x: UInt)`는 `public final void compute-<hashcode>(int x)`와 같이 표시 됩니다.

> Java에서 인라인 class에 접근하는 함수를 호출하는 것은 불가능 하므로 `-`은 *유효하지 않은 (invalid)* 심볼입니다.

## 인라인 class vs 타입 별칭 (Inline classes vs type aliases)

인라인 class는 [타입 별칭 (type aliases)](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/type-aliases)와 비슷해 보일 수 있습니다. 실제로 둘다 새로운 타입을 도입하고 런타임 시 기본 타입으로 표시 합니다.

그러나 결정적 차이는 타입 별칭은 기본 타입 (그리고 같은 기본 타입에 다른 타입 별칭)과 *할당-호환 (assignment-compatible)*되지만 인라인 class는 그렇지 않습니다.

즉, 인라인 class는 기존 타입의 대체 이름만 도입하는 타입 별칭과 다르게 실제로 _new_ 타입을 도입합니다:

```kotlin
typealias NameTypeAlias = String
inline class NameInlineClass(val s: String)

fun acceptString(s: String) {}
fun acceptNameTypeAlias(n: NameTypeAlias) {}
fun acceptNameInlineClass(p: NameInlineClass) {}

fun main() {
    val nameAlias: NameTypeAlias = ""
    val nameInlineClass: NameInlineClass = NameInlineClass("")
    val string: String = ""

    acceptString(nameAlias) // OK: pass alias instead of underlying type
    acceptString(nameInlineClass) // Not OK: can't pass inline class instead of underlying type

    // And vice versa:
    acceptNameTypeAlias(string) // OK: pass underlying type instead of alias
    acceptNameInlineClass(string) // Not OK: can't pass underlying type instead of inline class
}
```


## 인라인 class의 실험 상태 (Experimental status of inline classes)

인라인 class의 디자인은 실험적이며 빠르게 변하며 호환성이 보장되지 않습니다. Kotlin 1.3+에서 인라인 class를 사용하면 이 기능은 실험적이라는 경고가 나타납니다.

경고를 삭제하려면 컴파일러 인자에 `-Xinline-classes`을 전달하여 실험적 기능 사용이 가능하도록 해야 합니다.

### Gradle에서 인라인 class 사용 (Enabling inline classes in Gradle)

```groovy
compileKotlin {
    kotlinOptions.freeCompilerArgs += ["-Xinline-classes"]
}
```

```kotlin
tasks.withType<KotlinCompile> {
    kotlinOptions.freeCompilerArgs += "-Xinline-classes"
}
```


See [Compiler options in Gradle](https://kotlinlang.org/docs/reference/using-gradle.html#compiler-options) for details. For [Multiplatform Projects](https://kotlinlang.org/docs/reference/whatsnew13.html#multiplatform-projects) settings, see [building Multiplatform Projects with Gradle](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html#language-settings) section.

### Maven에서 인라인 class 사용 (Enabling inline classes in Maven)

```xml
<configuration>
    <args>
        <arg>-Xinline-classes</arg> 
    </args>
</configuration>
```

See [Compiler options in Maven](https://kotlinlang.org/docs/reference/using-maven.html#specifying-compiler-options) for details.

## 추가 의견 (Further discussion)

다른 기술적 상세 내용과 의견은 [language proposal for inline classes](https://github.com/Kotlin/KEEP/blob/master/proposals/inline-classes.md)을 참고 바랍니다.