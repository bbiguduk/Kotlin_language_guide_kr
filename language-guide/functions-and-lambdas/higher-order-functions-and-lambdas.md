# 고차 함수와 람다 \(Higher-Order Functions and Lambdas\)

Kotlin 함수는 [_first-class_](https://en.wikipedia.org/wiki/First-class_function) 입니다. 이것은 함수가 변수나 데이터 구조체에 저장되거나 인수로 전달되거나 다른 [higher-order functions](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#higher-order-functions)부터 반환될 수 있다는 의미입니다. 비함수 값을 가능한 함수로 동작하게 할 수 있습니다.

Kotlin은 정적 타입 프로그래밍 언어로써 [function types](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#function-types)을 이용해 함수를 나타내고 [lambda expressions](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#lambda-expressions-and-anonymous-functions)와 같은 특별한 언어 구조를 지원합니다.

## 고차 함수 \(Higher-Order Functions\)

고차 함수는 함수를 파라미터로 받거나 반환할 수 있는 함수를 말합니다.

콜렉션의 \[functional programming idiom `fold`\]\([https://en.wikipedia.org/wiki/Fold\_\(higher-order\_function\)\)이](https://en.wikipedia.org/wiki/Fold_%28higher-order_function%29%29이) 좋은 예입니다. 초기 연산값과 결합 함수를 사용하여 현재 연산값을 각 콜렉션 요소와 연속적으로 결합하여 연산값을 대체한 반환값을 만듭니다:

```kotlin
fun <T, R> Collection<T>.fold(
    initial: R, 
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}
```

위 코드를 보면 2개의 인자인 `R` 과 `T`가 있고 `R`을 반환하는 [function type](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#function-types) `(R, T) -> R`인 파라미터 `combine`이 있습니다. _for_ 루프 안에서 [invoked](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#invoking-a-function-type-instance)되고 `accumulator`에 반환값이 할당 됩니다.

`fold`를 호출하기 위해 [instance of the function type](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#instantiating-a-function-type)을 인자로 전달해야 하고 람다식 \([described in more detail below](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#lambda-expressions-and-anonymous-functions)\)은 고차 함수를 호출하는 곳에서 주로 사용됩니다:

```kotlin
fun main() {
    //sampleStart
    val items = listOf(1, 2, 3, 4, 5)

    // Lambdas are code blocks enclosed in curly braces.
    items.fold(0, { 
        // When a lambda has parameters, they go first, followed by '->'
        acc: Int, i: Int -> 
        print("acc = $acc, i = $i, ") 
        val result = acc + i
        println("result = $result")
        // The last expression in a lambda is considered the return value:
        result
    })

    // Parameter types in a lambda are optional if they can be inferred:
    val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })

    // Function references can also be used for higher-order function calls:
    val product = items.fold(1, Int::times)
    //sampleEnd
    println("joinedToString = $joinedToString")
    println("product = $product")
}
```

다음 섹션에서 더 자세히 다루도록 하겠습니다.

## 함수 타입 \(Function types\)

Kotlin은 함수를 다루는 선언에 `(Int) -> String`와 같은 함수 타입을 사용합니다: `val onClick: () -> Unit = ...`.

이러한 타입은 함수의 서명, 즉 파라미터와 반환값을 표현하는 특별한 표기법이 있습니다:

* 모든 함수 타입은 괄호로 묶인 파라미터 타입과 반환 타입이 있습니다: `(A, B) -> C`은 `A` 와 `B` 2개의 파라미터가 있으며 `C` 타입의 값을 반환하는 형태입니다. 파라미터 타입이 비어 있다면 `() -> A`로 표현 가능합니다. [`Unit` return type](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/functions#unit-unit-returning-functions)은 생략 불가합니다.
* 함수 타입은 선택적으로 _receiver_ 타입을 추가로 가질 수 있습니다. 이것은 점 앞에 표기합니다: `A.(B) -> C` 타입은 파라미터 `B`를 전달 받은 반환값 `C`를 `A` 리시버 객체에서 호출합니다. [Function literals with receiver](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#function-literals-with-receiver)은 주로 이러한 타입에서 사용됩니다.
* [Suspending functions](https://kotlinlang.org/docs/reference/coroutines-overview.html)는 `suspend () -> Unit` 또는 `suspend A.(B) -> C`와 같이 _suspend_ 수식어가 있는 함수 타입입니다.

함수 타입 표기는 선택적으로 파라미터에 이름을 포함할 수 있습니다: `(x: Int, y: Int) -> Point`. 이러한 이름은 파라미터의 의미를 문서화 하는데 사용할 수 있습니다.

> 함수 타입이 [nullable](https://kotlinlang.org/docs/reference/null-safety.html#nullable-types-and-non-null-types)이라면 괄호를 사용하면 됩니다: `((Int, Int) -> Int)?`.
>
> 함수 타입은 괄호를 이용하여 결합이 가능합니다: `(Int) -> ((Int) -> Unit)`
>
> 활살표 표기는 오른편과 연관성을 가지며, `(Int) -> (Int) -> Unit`은 이전 예제와 같은 표기지만 `((Int) -> (Int)) -> Unit`은 다릅니다.

함수 타입에 [a type alias](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/type-aliases)을 사용할 수 있습니다:

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```

### 함수 타입 인스턴스 \(Instantiating a function type\)

함수 타입의 인스턴스를 얻는 방법은 아래와 같습니다:

* 함수 리터럴의 코드 블럭을 사용하여 아래 중 하나의 방식으로 할 수 있습니다:
  * [lambda expression](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#lambda-expressions-and-anonymous-functions): `{ a, b -> a + b }`,
  * [anonymous function](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#anonymous-functions): `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

    [Function literals with receiver](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#function-literals-with-receiver)는 리시버가 있는 함수 타입의 값으로 사용될 수 있습니다.
* 기존 선언에 대한 호출가능한 참조를 사용하는 방법:
  * 최상위, 로컬, 멤버 또는 확장 [function](https://kotlinlang.org/docs/reference/reflection.html#function-references): `::isOdd`, `String::toInt`,
  * 최상위, 멤버 또는 확장 [property](https://kotlinlang.org/docs/reference/reflection.html#property-references): `List<Int>::size`,
  * [constructor](https://kotlinlang.org/docs/reference/reflection.html#constructor-references): `::Regex`

    특정 인스턴스의 멤버를 가리키는 [bound callable references](https://kotlinlang.org/docs/reference/reflection.html#bound-function-and-property-references-since-11)를 포함합니다: `foo::toString`.
* 함수 타입을 인터페이스로 구현한 커스텀 class를 사용하는 방법:

```kotlin
class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
}

val intFunction: (Int) -> Int = IntTransformer()
```

컴파일러는 정보가 충분하다면 함수 타입의 변수를 추론할 수 있습니다:

```kotlin
val a = { i: Int -> i + 1 } // The inferred type is (Int) -> Int
```

리시버가 있거나 없는 함수 타입의 _Non-literal_ 값은 서로 교환할 수 있습니다. 즉 리시버가 첫 번째 파라미터를 대신하거나 그 반대도 가능합니다. 예를 들어 `(A, B) -> C` 타입의 값은 `A.(B) -> C`가 예상되는 곳이나 다른 방법으로 전달 또는 할당할 수 있습니다:

```kotlin
fun main() {
    //sampleStart
    val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
    val twoParameters: (String, Int) -> String = repeatFun // OK

    fun runTransformation(f: (String, Int) -> String): String {
        return f("hello", 3)
    }
    val result = runTransformation(repeatFun) // OK
    //sampleEnd
    println("result = $result")
}
```

> 확장 함수를 참조하여 변수를 초기화 하더라도 리시버가 없는 함수 타입이 기본으로 추론됩니다. 이를 변경하려면 변수 타입을 명시적으로 지정해야 합니다.

### 함수 타입 인스턴스 호출 \(Invoking a function type instance\)

함수 타입의 값은 [`invoke(...)` operator](https://kotlinlang.org/docs/reference/operator-overloading.html#invoke)을 이용하여 호출할 수 있습니다: `f.invoke(x)` 또는 `f(x)`.

리시버 타입을 가지고 있으면 리시버 객체는 첫 번째 인자로 전달되야 합니다. 리시버와 함수 타입을 호출하는 다른 방법은 해당 값이 [extension function](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/extensions) 인 것처럼 수신 객체 앞에 추가하는 방법입니다: `1.foo(2)`.

예:

```kotlin
fun main() {
    //sampleStart
    val stringPlus: (String, String) -> String = String::plus
    val intPlus: Int.(Int) -> Int = Int::plus

    println(stringPlus.invoke("<-", "->"))
    println(stringPlus("Hello, ", "world!")) 

    println(intPlus.invoke(1, 1))
    println(intPlus(1, 2))
    println(2.intPlus(3)) // extension-like call
    //sampleEnd
}
```

### 인라인 함수 \(Inline functions\)

고차 함수의 유연한 제어 흐름을 제공하기 위해 [inline functions](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/inline-functions)를 사용하는 것이 유리합니다.

## 람다 표현과 익명 함수 \(Lambda Expressions and Anonymous Functions\)

람다 표현과 익명 함수는 'function literals' 입니다. 즉, 함수는 선언되지 않고 표현식으로 전달됩니다. 아래 예제 참고 바랍니다:

```kotlin
max(strings, { a, b -> a.length < b.length })
```

`max` 함수는 함수 값을 두 번째 인자로 가지는 고차 함수입니다. 두번 째 인자는 함수 리터럴로 그 자체가 함수 인 표현식입니다. 아래의 함수와 의미가 같다고 할 수 있습니다:

```kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

### 람다 표현식 \(Lambda expression syntax\)

람다식의 전체 구문 형태는 아래와 같습니다:

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
```

람다 표현식은 항상 중괄호로 묶여 있습니다. 파라미터는 중괄호 안에 선언되며 선택적으로 타입을 표기할 수 있습니다. 본문은 `->` 표시 뒤에 옵니다. 람다의 반환 타입이 `Unit`이 아니면 마지막 표현식이 반환 타입으로 취급 됩니다.

모든 선택 사항을 생략하면 아래와 같습니다:

```kotlin
val sum = { x: Int, y: Int -> x + y }
```

### Passing trailing lambdas

Kotlin에서 마지막 파라미터가 함수라면 해당 인자로 전달 된 람다식을 중괄호 밖에 나타낼 수 있습니다:

```kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

이러한 표기를 _trailing lambda_ 라고 합니다.

람다가 유일한 인자라면 소괄호를 생략할 수 있습니다.

```kotlin
run { println("...") }
```

### `it`: 싱글 파라미터의 암묵적 이름 \(implicit name of a single parameter\)

람다 표현식은 파라미터를 하나만 가지고 있는 경우가 많습니다.

컴파일러가 유추 가능한 경우 파라미터를 선언하지 않고 `->`도 생략 가능합니다. 파라미터는 암묵적으로 `it`라는 이름으로 선언됩니다.

```kotlin
ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```

### 람다 표현식의 반환값 \(Returning a value from a lambda expression\)

[qualified return](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/returns-and-jumps#return-at-labels)을 이용하여 람다에서 반환값을 명시적으로 나타낼 수 있습니다. 그렇지 않으면 가장 마지막 표현식이 암묵적 반환값이 됩니다.

따라서 아래 2개의 예는 같은 표기입니다:

```kotlin
ints.filter {
    val shouldFilter = it > 0 
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter
}
```

[passing a lambda expression outside parentheses](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#passing-trailing-lambdas)에 따라 \[LINQ-style\]\([https://docs.microsoft.com/en-us/previous-versions/dotnet/articles/bb308959\(v=msdn.10](https://docs.microsoft.com/en-us/previous-versions/dotnet/articles/bb308959%28v=msdn.10)\)\) 코드가 가능합니다:

```kotlin
strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }
```

### 사용하지 않는 변수에 대한 언더바 표기 \(Underscore for unused variables\) \(since 1.1\)

람다 파라미터를 사용하지 않을경우, 파라미터 이름 대신 언더바로 표현 가능합니다:

```kotlin
map.forEach { _, value -> println("$value!") }
```

### Destructuring in lambdas \(since 1.1\)

자세한 내용은 [destructuring declarations](https://kotlinlang.org/docs/reference/multi-declarations.html#destructuring-in-lambdas-since-11)를 참고 바랍니다.

### 익명 함수 \(Anonymous functions\)

위에 예제에 나온 람다식 구문에서 표현안된 것은 함수의 반환 유형을 지정하는 기능입니다. 대부분 반환 타입을 유추가능하여 필요하지 않습니다. 그러나 명시적으로 반환 타입을 지정해야 할 경우 _익명 함수 \(anonymous function\)_ 를 사용할 수 있습니다.

```kotlin
fun(x: Int, y: Int): Int = x + y
```

익명 함수는 함수의 이름이 생략 된다는 점만 제외하고 일반 함수 선언과 비슷합니다. 본문은 블록 또는 식으로 표현할 수 있습니다:

```kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

파라미터 타입을 유추할 수 있다면 생략할 수 있다는 점을 제외하면 일반 함수 선언과 동일하게 파라미터와 반환 타입을 선언할 수 있습니다:

```kotlin
ints.filter(fun(item) = item > 0)
```

익명 함수에서 반환 타입 유추는 일반 함수와 비슷합니다: 반환 타입은 표현식 바디가 있는 익명 함수에 대해 자동으로 추론되며 블록이 있는 익명 함수에 대해 명시적 \(아니면 `Unit`으로 지정\)으로 지정 해야 합니다.

익명 함수 파라미터는 항상 괄호 안으로 전달됩니다. 함수를 괄호 밖에 위치하게 하는 짧은 문법은 람다 표현식에서만 가능합니다.

익명 함수와 람다 표현식의 차이점은 [non-local returns](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/inline-functions#non-local-returns)의 동작입니다. 라벨 없는 _return_ 구문은 항상 _fun_ 키워드로 선언 된 함수로 부터 반환합니다. 람다 표현식의 _return_은 자신을 둘러싼 함수로부터 반환할 것이라는 뜻입니다. 반면에 익명 함수의 _return_은 익명 함수 자신으로 부터 반환할 것이라는 뜻입니다.

### 클로저 \(Closures\)

람다 표현식 또는 익명 함수 \([local function](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/functions#local-functions) 와 [object expression](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/object-expressions-and-declarations#object-expressions) 마찬가지로\) 자신의 _클로저 \(closure\)_ 에 접근 가능합니다. 예를 들어 외부에 선언 된 변수가 해당 됩니다. 클로저에 있는 변수는 람다에서 수정될 수 있습니다:

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

### Function literals with receiver

`A.(B) -> C`와 같이 리시버가 있는 [Function types](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#function-types)은 function literal의 특별한 형태로 인스턴스화가 가능합니다.

위에서 언급했듯이 Kotlin은 _receiver object_ 를 제공하는 리시버가 있는 함수 타입을 [to call an instance](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#invoking-a-function-type-instance)을 제공합니다.

function literal의 본문 내에서 호출에 전달 된 리시버 객체는 _implicit_ _this_가 되므로 리시버 객체의 멤버에 추가 정의 없이 접근 가능하거나 [`this` expression](https://kotlinlang.org/docs/reference/this-expressions.html)을 사용하여 리시버 객체에 접근 가능합니다.

이런 동작은 함수의 본문안에 리시버 객체의 멤버에 접근 가능한 [extension functions](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/extensions)와 비슷합니다.

다음 예는 타입이 존재하며 리시버가 있는 function literal 입니다. 이곳에서 리시버 객체로 `plus`가 호출됩니다:

```kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) }
```

익명 함수는 function literal의 리시버 타입을 직접 지정할 수 있습니다. 리시버로 함수 타입의 변수를 선언한 뒤 나중에 사용할 때 유용하게 쓸 수 있습니다.

```kotlin
val sum = fun Int.(other: Int): Int = this + other
```

리시버 타입이 유추될 수 있는 람다 표현식은 리시버가 있는 function literal로 사용할 수 있습니다. 이것을 사용하는 가장 중요한 예는 [type-safe builders](https://kotlinlang.org/docs/reference/type-safe-builders.html) 입니다:

```kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // create the receiver object
    html.init()        // pass the receiver object to the lambda
    return html
}

html {       // lambda with receiver begins here
    body()   // calling a method on the receiver object
}
```

