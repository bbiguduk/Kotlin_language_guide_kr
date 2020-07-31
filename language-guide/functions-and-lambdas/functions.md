# 함수 \(Functions\)

## 함수 선언 \(Function declarations\)

Kotlin에서 함수는 _fun_ 키워드를 이용하여 선언합니다:

```kotlin
fun double(x: Int): Int {
    return 2 * x
}
```

## 함수 사용 \(Function usage\)

함수 호출은 일반적인 호출방식을 사용합니다:

```kotlin
val result = double(2)
```

멤버 함수 호출은 점 표기를 사용합니다:

```kotlin
Stream().read() // create instance of class Stream and call read()
```

### 파라미터 \(Parameters\)

함수 파라미터는 Pascal 표기법으로 정의합니다: _name_: _type_. 파라미터는 콤마로 구분됩니다. 각 파라미터는 반드시 타입을 명시해야 합니다:

```kotlin
fun powerOf(number: Int, exponent: Int) { /*...*/ }
```

### 기본 인자 \(Default arguments\)

함수 파라미터는 기본값을 가질 수 있으며 기본값이 있는 인자는 생략 가능합니다. 다른 언어 대비 오버로드에 이점이 있습니다:

```kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) { /*...*/ }
```

기본값은 타입 선언 뒤에 **=** 와 값을 넣어 정의할 수 있습니다.

오버라이드 메서드는 항상 기존 메서드의 파라미터 기본값을 사용합니다. 기본 파라미터 값이 있는 메서드를 오버라이드 할 때 기본 파라미터 값은 생략되어 표기됩니다:

```kotlin
open class A {
    open fun foo(i: Int = 10) { /*...*/ }
}

class B : A() {
    override fun foo(i: Int) { /*...*/ }  // no default value allowed
}
```

기본값이 있는 파라미터가 없는 파라미터보다 앞에 위치한다면, 기본값은 [named arguments](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/functions#named-arguments)로 호출해야 사용합니다:

```kotlin
fun foo(bar: Int = 0, baz: Int) { /*...*/ }

foo(baz = 1) // The default value bar = 0 is used
```

기본값이 있는 파라미터 뒤에 마지막 인자가 [lambda](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#lambda-expression-syntax)인 경우 named argument나 [outside the parentheses](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#passing-trailing-lambdas)로 전달할 수 있습니다:

```kotlin
fun foo(bar: Int = 0, baz: Int = 1, qux: () -> Unit) { /*...*/ }

foo(1) { println("hello") }     // Uses the default value baz = 1
foo(qux = { println("hello") }) // Uses both default values bar = 0 and baz = 1 
foo { println("hello") }        // Uses both default values bar = 0 and baz = 1
```

### 인자 명칭 \(Named arguments\)

함수 호출 시 함수 파라미터에 이름을 붙일 수 있습니다. 이것은 함수의 파라미터가 많거나 기본값이 있는경우 편리하게 이용할 수 있습니다.

아래 함수를 살펴봅시다:

```kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
/*...*/
}
```

기본 인자를 이용하여 함수 호출을 할 수 있습니다:

```kotlin
reformat(str)
```

그러나 기본값을 이용하지 않고 호출하면 아래와 같은 형태가 됩니다:

```kotlin
reformat(str, true, true, false, '_')
```

인자에 명칭과 함께 함수를 호출하면 보다 보기 쉬운 코드가 됩니다:

```kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
)
```

그리고 모든 인자에 명칭을 붙일 필요는 없습니다:

```kotlin
reformat(str, wordSeparator = '_')
```

순차적 인자 \(positional arguments\)와 명칭 인자 \(named arguments\)를 통해 함수를 호출 할 때 모든 순차적 인자는 명칭 인자 앞에 와야 합니다. 예를 들어 `f(1, y = 2)`은 가능하지만 `f(x = 1, 2)`은 불가능 합니다.

[길이가 가변인 인자 \(_vararg_\)](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/functions#variable-number-of-arguments-varargs)는 아래와 같이 **spread \(`*`\)** 연산자를 사용하여 전달합니다:

```kotlin
fun foo(vararg strings: String) { /*...*/ }

foo(strings = *arrayOf("a", "b", "c"))
```

> **JVM**: 인자 명칭은 Java 함수를 호출 할 때는 사용할 수 없습니다. Java 바이트코드가 항상 함수 파라미터 명칭을 가지고 있지 않기 때문입니다.

### Unit 반환 함수 \(Unit-returning functions\)

어떠한 값도 반환하지 않는 함수의 반환 타입은 `Unit`입니다. `Unit`은 오직 하나의 값을 가지는 타입입니다. 이 값을 명시적으로 반환할 필요는 없습니다:

```kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` or `return` is optional
}
```

`Unit` 반환 타입 선언은 항상 선택사항입니다. 위에 코드는 아래와 같습니다:

```kotlin
fun printHello(name: String?) { ... }
```

### 단일 표현 함수 \(Single-expression functions\)

함수의 반환하는 식이 한 줄이라면 괄호를 생략하고 **=** 뒤에 표현할 수 있습니다:

```kotlin
fun double(x: Int): Int = x * 2
```

컴파일러가 반환 타입 추론이 가능하다면 반환 타입도 [optional](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/functions#explicit-return-types) 입니다:

```kotlin
fun double(x: Int) = x * 2
```

### 명시적 반환 타입 \(Explicit return types\)

함수의 바디 블럭은 `Unit`을 반환하는 형태가 아니라면 항상 명시적으로 반환 타입을 지정해야 합니다. Kotlin은 바디 블럭을 가진 함수의 반환 타입은 유추하지 않습니다. 이 경우 함수는 복잡할 수 있고 컴파일러도 함수의 리턴 타입을 명확히 알 수 없기 때문입니다.

### 가변 길이 인자 \(Variable number of arguments\) \(Varargs\)

함수의 파라미터 \(일반적으로 마지막\)에 `vararg` 수식어를 표기할 수 있습니다:

```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```

길이가 가변인 인자를 함수에 전달할 수 있습니다:

```kotlin
val list = asList(1, 2, 3)
```

함수 안에 `T` 타입의 `vararg` 파라미터는 `T`의 배열로 나타납니다. 즉, 위 예제의 변수 `ts`는 `Array<out T>`라는 타입을 가집니다.

오직 하나의 파라미터만 `vararg`로 지정할 수 있습니다. `vararg` 파라미터는 함수에서 마지막 파라미터에 위치 하지 않으면 인자 명칭을 이용하여 전달하거나 파라미터에 함수 타입이 있는 경우에는 람다를 이용해 전달합니다.

`vararg` 함수를 호출할 때 인자를 하나씩 전달 할 수 있습니다. 예를 들어 `asList(1, 2, 3)`, 또는 배열이 이미 있고 그 내용을 함수에 전달하려면 **spread** 연산자 \(`*`를 배열 앞에 붙임\)를 이용하여 전달합니다:

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

### 인픽스 표기 \(Infix notation\)

_infix_ 키워드로 표기된 함수는 인픽스 표기를 이용하여 호출할 수 있습니다 \(호출할 때 점과 괄호를 생략할 수 있습니다\). 인픽스 함수는 아래의 요구사항을 만족해야 합니다:

* 멤버 함수이거나 [확장 함수 \(extension functions\)](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/extensions) 이어야 합니다.
* 하나의 파라미터를 가지고 있어야 합니다.
* [가변 길이 인자 \(accept variable number of arguments\)](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/functions#variable-number-of-arguments-varargs)나 [기본값 \(default value\)](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/functions#default-arguments)은 사용할 수 없습니다.

```kotlin
infix fun Int.shl(x: Int): Int { ... }

// calling the function using the infix notation
1 shl 2

// is the same as
1.shl(2)
```

> 인픽스 함수 호출은 산술적 연산, 타입 캐스트, `rangeTo` 연산자보다 낮은 우선순위를 갖습니다. 다음의 표현은 같은 표현입니다:
>
> * `1 shl 2 + 3` 와 `1 shl (2 + 3)`  
> * `0 until n * 2` 와 `0 until (n * 2)`  
> * `xs union ys as Set<*>` 와 `xs union (ys as Set<*>)`
>
> 반면에 인픽스 함수 호출은 boolean 연산자 `&&` 와 `||`, `is`- 와 `in`-checks 그리고 몇개 다른 연산자보다 높은 우선순위를 갖습니다. 다음 표현은 같은 표현입니다:
>
> * `a && b xor c` is equivalent to `a && (b xor c)`  
> * `a xor b in c` is equivalent to `(a xor b) in c`
>
> 연산자 우선순위는 [Grammar reference](https://kotlinlang.org/docs/reference/grammar.html#expressions)을 참고 바랍니다.

인픽스 함수는 항상 리시버와 파라미터를 지정해야 합니다. 인픽스 표기를 사용하여 현재 리시버에 메서드를 호출 할 때 명시적으로 `this`를 표기해야 합니다. 일반 메서드 호출처럼 생략이 불가합니다. 구문 분석을 위해 반드시 필요합니다.

```kotlin
class MyStringCollection {
    infix fun add(s: String) { /*...*/ }

    fun build() {
        this add "abc"   // Correct
        add("abc")       // Correct
        //add "abc"        // Incorrect: the receiver must be specified
    }
}
```

## 함수 범위 \(Function scope\)

Kotlin에서 함수는 파일에 가장 최상위에 선언할 수 있습니다. 이 말은 Java, C\#, Scala와 같은 언어처럼 함수를 생성하기 위해 class를 만들 필요가 없다는 말입니다. Kotlin 함수는 최상위 함수 외에도 멤버 함수와 확장 함수로 로컬에 선언할 수 있습니다.

### 지역 함수 \(Local functions\)

Kotlin은 함수 안에 다른 함수를 선언하는 로컬 함수를 지원합니다:

```kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: MutableSet<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

로컬 함수는 바깥 함수의 로컬 변수에 접근할 수 있으므로 위에 예제에서 _visited_은 로컬 변수입니다:

```kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

### 멤버 함수 \(Member functions\)

멤버 함수는 class나 객체 안에 정의 된 함수입니다:

```kotlin
class Sample() {
    fun foo() { print("Foo") }
}
```

멤버 함수는 점 표기를 이용하여 호출됩니다:

```kotlin
Sample().foo() // creates instance of class Sample and calls foo
```

class와 오버라이드 멤버에 대한 자세한 내용은 [Classes](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-classes-and-inheritance) 와 [Inheritance](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-classes-and-inheritance#inheritance)을 참고 바랍니다.

## 제너릭 함수 \(Generic functions\)

함수는 함수 이름 전에 괄호 \(```<``>```\)를 이용하여 제너릭 파라미터를 가질 수 있습니다:

```kotlin
fun <T> singletonList(item: T): List<T> { /*...*/ }
```

자세한 내용은 [Generics](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/generics)을 참고 바랍니다.

## 인라인 함수 \(Inline functions\)

인라인 함수는 [인라인 함수 section](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/inline-functions)를 참고 바랍니다.

## 확장 함수 \(Extension functions\)

확장 함수는 [their own section](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/extensions)를 참고 바랍니다.

## 고차 함수와 람다 \(Higher-order functions and lambdas\)

고차 함수와 람다는 [their own section](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas)를 참고 바랍니다.

## 꼬리 재귀 함수 \(Tail recursive functions\)

Kotlin은 [tail recursion](https://en.wikipedia.org/wiki/Tail_call)라고 알려진 함수형 프로그래밍 스타일을 제공합니다. 이 알고리즘을 통해 스택 오버플로우의 리스크 없이 재귀 함수를 사용할 수 있습니다. 함수에 `tailrec` 수식어를 붙이고 필요한 형식에 맞는다면 컴파일러는 재귀를 최적화 하고 빠르고 효율적인 루프 기반 버전으로 남겨줍니다:

```kotlin
val eps = 1E-10 // "good enough", could be 10^-15

tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (Math.abs(x - Math.cos(x)) < eps) x else findFixPoint(Math.cos(x))
```

이 코드는 수학적인 상수인 코사인의 고정점을 계산합니다. 이것은 1.0 부터 결과값이 바뀌지 않을 때까지 반복적으로 Math.cos을 호출하여 지정된 `eps`와 비교하여 0.7390851332151611라는 결과를 도출합니다. 위 코드는 아래의 코드와 같은 내용입니다:

```kotlin
val eps = 1E-10 // "good enough", could be 10^-15

private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (Math.abs(x - y) < eps) return x
        x = Math.cos(x)
    }
}
```

`tailrec` 수식어를 사용하려면 함수는 마지막으로 수행할 때 자기자신을 호출해야 합니다. 재귀 호출 후 더많은 코드가 있으면 사용할 수 없습니다. try/catch/finally 블럭 안에서도 사용이 불가합니다. 현재는 JVM 기반의 Kotlin과 Kotlin/Native에서 지원합니다.

