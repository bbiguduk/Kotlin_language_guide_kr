# 타입 별칭 \(Type aliases\)

타입 별칭은 타입에 대해 다른 이름을 제공합니다. 타입 이름이 너무 길다면 짧은 이름이나 새로운 이름으로 제공 가능합니다.

긴 제너릭 타입을 짧게 하는데 유용합니다. 예를 들어 콜렉션 타입을 간결하게 표현할 수 있습니다:

```kotlin
typealias NodeSet = Set<Network.Node>

typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

함수 타입을 위한 별칭도 제공 가능합니다:

```kotlin
typealias MyHandler = (Int, String, Any) -> Unit

typealias Predicate<T> = (T) -> Boolean
```

내부 클래스와 중첩 클래스의 별칭도 제공 가능합니다:

```kotlin
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```

타입 별칭은 새로운 타입을 나타낼 수는 없습니다. 해당하는 기본 유형과 같습니다. 코드에서 `typealias Predicate<T>`를 추가하고 `Predicate<Int>`를 사용할 때 Kotlin 컴파일러는 항상 `(Int) -> Boolean`으로 확장합니다. 따라서 일반 함수 타입이 필요할 때마다 타입의 변수를 전달 할 수 있고 그 반대도 가능합니다:

```kotlin
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)

fun main() {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // prints "true"

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // prints "[1]"
}
```

