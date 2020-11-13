# 인라인 함수 \(Inline Functions\)

[higher-order functions](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas)를 사용하는 것은 런타임 패널티를 가지게 됩니다: 각 함수는 객체가 되며 클로저를 가지고 있게 되는데 이는 함수의 본문내에서 접근되는 변수도 가지고 있다는 말입니다. 함수 객체와 class를 메모리에 할당하고 가상 호출은 런타임 오버헤드를 가집니다.

하지만 이런한 오버헤드 케이스들은 람다 표현식을 인라인 방식으로 적용함으로써 제거할 수 있습니다. 아래 예에서의 함수는 이러한 상황의 좋은 예입니다. `lock()` 함수는 호출 위치에서 쉽게 인라인 될 수 있습니다:

```kotlin
lock(l) { foo() }
```

함수 객체의 파라미터와 호출하는 동작을 반드는 대신에 컴파일러가 아래와 같은 코드로 변경하면 동일한 동작에 대해 오버헤드가 줄어드는 결과를 얻을 수 있습니다:

```kotlin
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}
```

`lock()` 함수에 `inline` 수식어를 사용하여 컴파일러가 위와 같이 동작하도록 할 수 있습니다:

```kotlin
inline fun <T> lock(lock: Lock, body: () -> T): T { ... }
```

`inline` 수식어는 함수 자체와 함수에 전달 된 람다 모두 적용 됩니다: 모두 호출되는 위치에 인라인 됩니다.

인라인은 생성된 코드가 커질 수 있습니다. 그러나 합리적으로 사용하면 \(큰 함수를 인라인 하지 않음\) 루프 내부의 호출 위치에서 성능이 향상 됩니다.

## noinline

특정 람다만 인라인 함수로 전달할 때 `noinline` 수식어를 이용하면 가능합니다:

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { ... }
```

인라인 가능한 람다는 인라인 함수 내에서 호출되거나 인라인 가능한 인자만 전달 할 수 있습니다. 그러나 `noinline` 수식어가 붙으면 다양하게 사용할 수 있습니다: 필드에 저장, 주변에 전달 등.

인라인 함수가 인라인 가능한 함수 파라미터가 없거나 [reified type parameters](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/inline-functions#reified-type-parameters) 아니면 컴파일러는 경고를 나타냅니다. 이는 그런 함수는 이득이 적기 때문에 발생합니다. 그럼에도 사용하려면 `@Suppress("NOTHING_TO_INLINE")` annotation을 사용하면 가능합니다.

## Non-local returns

Kotlin에서 일반적으로 `return`은 함수나 익명 함수를 빠져나오게 됩니다. 람다를 종료하기 위해선 [label](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/returns-and-jumps#return-at-labels)을 써야 하며 람다는 함수 안에서 리턴할 수 없으므로 `return` 만 사용하는 것을 금지합니다:

```kotlin
fun ordinaryFunction(block: () -> Unit) {
    println("hi!")
}
//sampleStart
fun foo() {
    ordinaryFunction {
        return // ERROR: cannot make `foo` return here
    }
}
//sampleEnd
fun main() {
    foo()
}
```

그러나 람다가 전달 된 함수가 인라인이면 반환도 인라인이므로 이러한 경우 허용됩니다:

```kotlin
inline fun inlined(block: () -> Unit) {
    println("hi!")
}
//sampleStart
fun foo() {
    inlined {
        return // OK: the lambda is inlined
    }
}
//sampleEnd
fun main() {
    foo()
}
```

람다안에 있지만 함수를 종료하기 위한 반환은 _non-local_ 반환이라 부릅니다. 인라인 함수가 종종 애워싸는 이런 종류의 루프 구조에 익숙합니다:

```kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // returns from hasZeros
    }
    return false
}
```

어떤 인라인 함수는 함수 본문에서 바로 파라미터로 전달된 것이 아닌 로컬 객체 또는 중첩 함수와 같은 또다른 실행 문맥으로부터 파라미터로 전달된 람다를 호출할 수도 있습니다. 이러한 경우 non-local control flow은 허용되지 않습니다. 이것을 명시하기 위해 람다 파라미터는 `crossinline` 수식어가 필요합니다:

```kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ...
}
```

> `break` 와 `continue`는 아직 인라인 람다에서 사용이 불가하지만 지원되도록 계획 중입니다.

## 구체화된 타입 파라미터 \(Reified type parameters\)

파라미터로 전달된 타입을 접근 해야 할 경우가 있습니다.

```kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```

트리를 걸어 리플렉션을 사용하여 해당 노드에 특정 타입이 있는지 확인합니다. 모든 것이 좋지만 호출하는 부분이 매끄럽지 않습니다:

```kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```

실제로 아래처럼 타입을 함수로 전달하는 방법이 간단합니다:

```kotlin
treeNode.findParentOfType<MyTreeNode>()
```

_reified type parameters_를 지원하는 인라인 함수를 통해 아래와 같이 사용할 수 있습니다:

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

`reified` 수식어가 사용되어 타입 파라미터 `T`는 일단 class 처럼 함수내에서 접근이 가능합니다. 인라인 된 함수는 이제 리플렉션이 필요치 않으며 `!is` 와 `as` 같은 일반 연산자 사용이 가능해집니다. 또한 `myTree.findParentOfType<MyTreeNodeType>()` 호출도 가능합니다.

대부분의 경우 리플렉션은 필요치 않으나 reified 타입 파라미터와 함께 사용할 수 있습니다:

```kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}
```

인라인이 표기 안된 일반 함수는 reified 파라미터를 가질 수 없습니다. run-time representation을 가지고 있는 타입 \(non-reified 타입 파라미터 또는 `Nothing`\)은 reified 타입 파라미터 인자로 사용할 수 없습니다.

자세한 내용은 [spec document](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md)를 참고 바랍니다.

## 인라인 프로퍼티 \(Inline properties\) \(since 1.1\)

`inline` 수식어는 백킹 필드가 없는 프로퍼티에 사용할 수 있습니다. 각각의 프로퍼티 접근자에 사용할 수 있습니다:

```kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

2개의 접근자를 다 가진 프로퍼티에도 사용할 수 있습니다:

```kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

호출되는 위치에서 인라인 접근자는 일반 인라인 함수처럼 처리됩니다.

## Restrictions for public API inline functions

인라인 함수가 `public` 또는 `protected`이고 `private` 또는 `internal` 선언의 일부가 아니면 [module](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/visibility-modifiers#modules)의 public API로 간주합니다. 다른 모듈에서 호출될 수 있고 호출되는 위치에 인라인 될 수 있습니다.

이는 변경 후 모듈이 다시 컴파일되지 않을 경우 인라인 기능을 선언하는 모듈의 변경으로 인해 발생하는 바이너리 비호환성의 위험을 가지고 있습니다.

모듈의 **non**-public API 변경으로 인해 이러한 비호환성 위험을 제거하기 위해 public API 인라인 함수는 non-public-API 선언을 허락하지 않습니다.

`internal` 선언은 `@PublishedApi` annotation을 선언하면 public API 인라인 함수 안에서 사용가능 하도록 해줍니다. `internal` 인라인 함수가 `@PublishedApi`로 표시되면 본문도 public 인 것처럼 체크 됩니다.

