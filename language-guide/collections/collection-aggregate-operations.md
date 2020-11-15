# 콜렉션 집합 작업 \(Collection Aggregate Operations\)

Kotlin 콜렉션은 콜렉션 내용을 기반으로 하나의 값을 반환하는 작업에 대한 일반적으로 사용되는 _집합 연산 \(aggregate operations\)_ 함수를 포함하고 있습니다. 대부분 알고 있고 다른 언어와 같은 동작을 합니다:

* [`min()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/min.html) 과 [`max()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/max.html)은 요소의 가장 작은 값과 가장 큰 값을 반환합니다;
* [`average()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/average.html)은 콜렉션 요소의 평균을 반환합니다;
* [`sum()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sum.html)은 콜렉션 요소의 합을 반환합니다;
* [`count()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/count.html)은 콜렉션의 요소의 수를 반환합니다;

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(6, 42, 10, 4)

    println("Count: ${numbers.count()}")
    println("Max: ${numbers.max()}")
    println("Min: ${numbers.min()}")
    println("Average: ${numbers.average()}")
    println("Sum: ${numbers.sum()}")
//sampleEnd
}
```

특정 선택 함수 또는 커스텀 [`Comparator`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparator/index.html)에 의해 가장 작은 요소와 가장 큰 요소를 검색하는 함수도 있습니다:

* [`maxBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/max-by.html)/[`minBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/min-by.html)은 선택 함수를 사용하여 요소의 가장 크거나 작은 값을 반환합니다.
* [`maxWith()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/max-with.html)/[`minWith()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/min-with.html)은 `Comparator` 객체를 사용하여 `Comparator`에 따라 가장 크거나 작은 요소를 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(5, 42, 10, 4)
    val min3Remainder = numbers.minBy { it % 3 }
    println(min3Remainder)

    val strings = listOf("one", "two", "three", "four")
    val longestString = strings.maxWith(compareBy { it.length })
    println(longestString)
//sampleEnd
}
```

추가로 모든 요소에 일정 동작 후 합을 반환하는 함수도 있습니다:

* [`sumBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sum-by.html)는 콜렉션 요소에 `Int` 값을 반환하는 함수를 적용합니다.
* [`sumByDouble()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sum-by-double.html)은 `Double`을 반환하는 함수를 적용합니다.

```kotlin
fun main() {
//sampleStart    
    val numbers = listOf(5, 42, 10, 4)
    println(numbers.sumBy { it * 2 })
    println(numbers.sumByDouble { it.toDouble() / 2 })
//sampleEnd
}
```

## 접기와 줄이기 \(Fold and reduce\)

보다 특별한 경우로는 제공된 동작을 콜렉션 요소에 순차적으로 적용하고 누적된 결과를 반환하는 [`reduce()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce.html) 와 [`fold()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/fold.html) 함수가 있습니다. 이 동작은 두개의 인자를 받습니다: 이전에 누적된 값과 콜렉션 요소입니다.

`fold()`는 첫번째 단계에서 초기값을 받아 누적된 값을 사용하지만 `reduce()` 첫번째 단계에서 첫번째와 두번째 요소를 사용한다는 차이점이 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(5, 2, 10, 4)

    val sum = numbers.reduce { sum, element -> sum + element }
    println(sum)
    val sumDoubled = numbers.fold(0) { sum, element -> sum + element * 2 }
    println(sumDoubled)

    //val sumDoubledReduce = numbers.reduce { sum, element -> sum + element * 2 } //incorrect: the first element isn't doubled in the result
    //println(sumDoubledReduce)
//sampleEnd
}
```

위 예제에서 차이를 나타냅니다: `fold()`는 배가 되는 요소의 합을 나타냅니다. 만약 위에서 `reduce()`를 사용하면 다른 결과를 반환합니다. `reduce()`는 첫 단계에서 첫번째와 두번째 요소를 인자로 사용하기 때문에 첫번째 요소는 2배가 되지 않습니다.

역순으로 적용하려면 [`reduceRight()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce-right.html) 와 [`foldRight()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/fold-right.html) 함수를 사용합니다. `fold()` 와 `reduce()` 동작과 똑같지만 시작이 마지막 요소에서 시작하고 그 다음 요소는 이전 요소가 됩니다. 접기 또는 줄이기 작업을 하면 연산 인자의 순서를 변경합니다: 먼저 요소로 이동하고 누적값으로 이동합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(5, 2, 10, 4)
    val sumDoubledRight = numbers.foldRight(0) { element, sum -> sum + element * 2 }
    println(sumDoubledRight)
//sampleEnd
}
```

요소 인덱스를 파라미터로 함수를 적용할 수 있습니다. 이를 위해 첫번째 인자로 요소 인덱스를 전달하는 [`reduceIndexed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce-indexed.html) 와 [`foldIndexed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/fold-indexed.html) 함수를 사용합니다.

[`reduceRightIndexed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce-right-indexed.html) 와 [`foldRightIndexed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/fold-right-indexed.html)은 콜렉션 요소의 오른쪽에서 왼쪽으로 동작을 적용할 때 사용합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(5, 2, 10, 4)
    val sumEven = numbers.foldIndexed(0) { idx, sum, element -> if (idx % 2 == 0) sum + element else sum }
    println(sumEven)

    val sumEvenRight = numbers.foldRightIndexed(0) { idx, element, sum -> if (idx % 2 == 0) sum + element else sum }
    println(sumEvenRight)
//sampleEnd
}
```

모든 줄이기 연산은 빈 콜렉션에서 예외를 발생합니다. 예외 대신에 `null` 을 받으려면 `*OrNull()` 을 사용하면 됩니다:

* [`reduceOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce-or-null.html)
* [`reduceRightOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce-right-or-null.html)
* [`reduceIndexedOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce-indexed-or-null.html)
* [`reduceRightIndexedOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce-right-indexed-or-null.html)

