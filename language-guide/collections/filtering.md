# 필터링 \(Filtering\)

필터링은 콜렉션 작업에서 가장 중요한 작업 중 하나입니다. Kotlin에서 필터링 조건은 콜렉션 요소를 가져와 조건과 일치하면 `true` 그 반대면 `false`를 반환하는 람다 함수인 _속성 \(predicates\)_ 으로 정의 됩니다

표준 라이브러리는 단일 호출로 콜렉션을 필터링 할 수 있는 확장 함수의 그룹이 포함되어 있습니다. 이러한 함수는 기존 콜렉션을 변경하지 않기 때문에 [변경 가능과 읽기-전용 \(mutable and read-only\)](kotlin-kotlin-collections-overview.md#collection-types) 콜렉션 사용이 가능합니다. 필터링 결과를 변경하려면 변수에 저장하거나 필터링 후 함수를 연결해야 합니다.

## 조건 필터링 \(Filtering by predicate\)

필터링 함수의 기본은 [`filter()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter.html) 입니다. 조건과 함께 호출할 때 `filter()`는 조건과 일치하는 콜렉션의 요소를 반환합니다. `List` 와 `Set`의 결과 콜렉션은 `List`로 `Map`은 `Map`으로 결과 콜렉션을 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")  
    val longerThan3 = numbers.filter { it.length > 3 }
    println(longerThan3)

    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
    println(filteredMap)
//sampleEnd
}
```

`filter()`의 조건은 요소의 값만 체크할 수 있습니다. 필터에서 요소의 위치를 사용하고 싶다면 [`filterIndexed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-indexed.html)을 사용해야 합니다. 이 함수는 요소의 인덱스와 값에 대한 두개의 인자를 조건으로 받습니다.

반대 조건으로 콜렉션을 필터하고 싶다면 [`filterNot()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-not.html)을 사용해야 합니다. `false`인 조건들의 요소의 리스트를 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")

    val filteredIdx = numbers.filterIndexed { index, s -> (index != 0) && (s.length < 5)  }
    val filteredNot = numbers.filterNot { it.length <= 3 }

    println(filteredIdx)
    println(filteredNot)
//sampleEnd
}
```

주어진 타입의 요소로 필터링 하는 함수도 있습니다:

* [`filterIsInstance()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-is-instance.html)는 주어진 타입의 콜렉션 요소를 반환합니다. `List<Any>`에서 호출 된 `filterIsInstance<T>()`은 `List<T>`을 반환하므로 `T` 타입의 함수를 호출 할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(null, 1, "two", 3.0, "four")
    println("All String elements in upper case:")
    numbers.filterIsInstance<String>().forEach {
        println(it.toUpperCase())
    }
//sampleEnd
}
```

* [`filterNotNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-not-null.html)은 null이 아닌 요소를 반환합니다. `List<T?>`에서 호출 된 `filterNotNull()`은 `List<T: Any>`을 반환하므로 요소를 null이 아닌 객체로 취급할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(null, "one", "two", null)
    numbers.filterNotNull().forEach {
        println(it.length)   // length is unavailable for nullable Strings
    }
//sampleEnd
}
```

## 분배 \(Partitioning\)

다른 필터링 함수 [`partition()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/partition.html)은 조건에 따라 콜렉션을 필터하고 조건에 맞지 않은 리스트도 따로 가지고 있습니다. 그래서 첫번째 리스트는 조건에 맞는 요소들이 포함되어 있고 두번째 리스트는 조건에 맞지 않은 요소들이 포함 된 `List`의 `Pair`로 반환된 값을 가지고 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    val (match, rest) = numbers.partition { it.length > 3 }

    println(match)
    println(rest)
//sampleEnd
}
```

## 테스트 조건 \(Testing predicates\)

마지막으로 콜렉션 요소에 대해 간단히 테스트하는 함수가 있습니다:

* [`any()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/any.html)는 주어진 조건에 하나라도 요소가 일치하면 `true`를 반환합니다.
* [`none()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/none.html)은 주어진 조건에 요소가 일치하지 않으면 `true`를 반환합니다.
* [`all()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/all.html)은 주어진 조건에 모든 요소가 일치하면 `true`를 반환합니다. `all()`은 빈 콜렉션에서 호출되면 항상 `true`를 반환합니다. 이러한 결과는 로직에서 [_공허한 진실 \(vacuous truth\)_ ](https://en.wikipedia.org/wiki/Vacuous_truth)이라고 알고 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")

    println(numbers.any { it.endsWith("e") })
    println(numbers.none { it.endsWith("a") })
    println(numbers.all { it.endsWith("e") })

    println(emptyList<Int>().all { it > 5 })   // vacuous truth
//sampleEnd
}
```

`any()` 와 `none()`은 콜렉션의 비어있는 여부를 판단하기 위해 조건 없이 사용할 수 있습니다. `any()`는 요소가 있으면 `true`를 반환하고 요소가 없으면 `false`를 반환합니다: `none()`은 반대로 동작합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    val empty = emptyList<String>()

    println(numbers.any())
    println(empty.any())

    println(numbers.none())
    println(empty.none())
//sampleEnd
}
```

