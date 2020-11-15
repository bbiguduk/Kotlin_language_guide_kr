# 요소 추출 \(Retrieving Single Elements\)

Kotlin 콜렉션은 콜렉션에서 하나의 요소를 뽑을 수 있는 함수 셋을 제공합니다. 함수는 이 페이지에서 설명되어 있으면 리스트와 셋에 모두 적용 됩니다.

[리스트의 정의 \(definition of list\) ](kotlin-kotlin-collections-overview.md)에서 보았듯이 리스트는 순서가 있는 콜렉션입니다. 그러므로 리스트의 모든 요소는 참조에 필요한 위치를 가지고 있습니다. 이 페이지에 설명 된 함수 외에도 리스트는 인덱스로 요소를 검색하고 추출하는 방법을 제공합니다. 더 자세한 내용은 [리스트 동작 \(List Specific Operations\)](list-specific-operations.md) 참고 바랍니다.

집합은 [정의 \(definition\) ](kotlin-kotlin-collections-overview.md)에 의해 순서가 있는 콜렉션이 아닙니다. 그러나 Kotlin `Set`은 요소의 순서를 저장합니다. 입력 순서 \(`LinkedHashSet`\), 자연 정렬 \(`SortedSet`\) 또는 다른 순서일 수 있습니다. 요소의 set의 순서는 모를 수도 있습니다. 이러한 경우 요소는 여전히 순서대로 정렬되므로 위치에 대한 함수는 순서 결과를 반환합니다. 그러나 사용된 `Set`에 특정 구현을 모르면 호출자는 이러한 결과를 예측할 수 없습니다.

## 위치로 추출 \(Retrieving by position\)

[`elementAt()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/element-at.html) 함수는 특정 위치의 요소를 추출할 때 사용합니다. 인자로 정수형 숫자와 함께 호출하면 주어진 위치에 콜렉션 요소를 알 수 있습니다. 첫번째 요소는 위치 `0`이며 마지막은 `(size - 1)`입니다.

`elementAt()`은 인덱스 접근은 지원하지 않거나 정적으로 알려지지 않은 콜렉션에 유용합니다. `List`의 경우 [인덱스된 접근 연산자 \(indexed access operator\)](list-specific-operations.md#retrieving-elements-by-index) \(`get()` or `[]`\)을 자주 사용합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = linkedSetOf("one", "two", "three", "four", "five")
    println(numbers.elementAt(3))    

    val numbersSortedSet = sortedSetOf("one", "two", "three", "four")
    println(numbersSortedSet.elementAt(0)) // elements are stored in the ascending order
//sampleEnd
}
```

콜렉션의 첫번째와 마지막 요소를 추출하는 유용한 방법도 제공합니다: [`first()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/first.html) 와 [`last()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/last.html).

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five")
    println(numbers.first())    
    println(numbers.last())    
//sampleEnd
}
```

존재하지 않는 위치의 요소를 검색할 때 예외를 피하려면 `elementAt()`의 변형 함수를 사용하면 됩니다:

* [`elementAtOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/element-at-or-null.html) 콜렉션 범위를 넘어가는 위치를 요청할 경우 null을 반환합니다.
* [`elementAtOrElse()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/element-at-or-else.html)은 추가적으로 `Int` 인자를 매핑하는 람다 함수를 가집니다.

   콜렉션 범위를 넘어가는 위치를 요청할 경우 `elementAtOrElse()`은 람다에 주어진 값을 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five")
    println(numbers.elementAtOrNull(5))
    println(numbers.elementAtOrElse(5) { index -> "The value for index $index is undefined"})
//sampleEnd
}
```

## 조건 추출 \(Retrieving by condition\)

[`first()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/first.html) 와 [`last()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/last.html) 함수는 주어진 조건에 맞는 콜렉션 요소를 검색할 수도 있습니다. 조건과 함께 `first()`를 호출하면 조건이 `true`인 콜렉션의 첫번째 요소를 반환합니다. 반대로 `last()`는 조건이 맞는 마지막 요소를 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.first { it.length > 3 })
    println(numbers.last { it.startsWith("f") })
//sampleEnd
}
```

조건에 맞지 않는 요소가 없다면 두 함수는 예외를 발생합니다. 예외를 피하려면 조건에 맞지 않을 경우 `null`을 반환하는 [`firstOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/first-or-null.html) 와 [`lastOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/last-or-null.html) 함수를 이용해야 합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.firstOrNull { it.length > 6 })
//sampleEnd
}
```

또는 해당 상황이 더 맞는 경우 아래와 같은 함수도 가능합니다:

* `firstOrNull()` 대신에 [`find()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/find.html)
* `lastOrNull()` 대신에 [`findLast()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/find-last.html)

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.find { it % 2 == 0 })
    println(numbers.findLast { it % 2 == 0 })
//sampleEnd
}
```

## 랜덤 요소 \(Random element\)

[`random()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/random.html) 함수를 사용하면 콜렉션의 임의의 요소를 추출할 수 있습니다. 인자 없이 호출하거나 [`Random`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.random/-random/index.html) 객체를 사용하여 호출할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.random())
//sampleEnd
}
```

빈 콜렉션에서 `random()` 은 예외를 발생합니다. 예외 대신에 null 을 받으려면 [`randomOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/random-or-null.html) 을 사용합니다.

## 존재 여부 판단 \(Checking existence\)

콜렉션에서 요소가 존재하는지 체크하려면 [`contains()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/contains.html) 함수를 사용해야 합니다. 함수 인자와 콜렉션 요소가 `equals()` 이라면 `true`를 반환합니다. `contains()` 함수는 `in` 키워드로 호출이 가능합니다.

여러 인스턴스를 한번에 체크하려면 [`containsAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/contains-all.html) 함수를 사용해야 합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.contains("four"))
    println("zero" in numbers)

    println(numbers.containsAll(listOf("four", "two")))
    println(numbers.containsAll(listOf("one", "zero")))
//sampleEnd
}
```

추가적으로 콜렉션에 요소 존재 여부를 판단하려면 [`isEmpty()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/is-empty.html) 또는 [`isNotEmpty()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/is-not-empty.html) 함수를 사용해야 합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.isEmpty())
    println(numbers.isNotEmpty())

    val empty = emptyList<String>()
    println(empty.isEmpty())
    println(empty.isNotEmpty())
//sampleEnd
}
```

