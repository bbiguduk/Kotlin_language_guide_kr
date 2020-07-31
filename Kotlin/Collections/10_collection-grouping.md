# 그룹 (Grouping)

Kotlin 표준 라이브러리는 콜렉션 요소를 그룹핑 하기 위한 확장 함수를 제공합니다.
기본 함수 [`groupBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/group-by.html)는 람다 함수이며 하나의 `Map`을 반환합니다.
맵에서 각 키는 람다의 결과이며 일치하는 값은 그룹 된 요소의 `List` 입니다.
예를 들어 `String`의 첫 글자를 그룹으로 할 때 이 함수를 사용할 수 있습니다.

`groupBy()`의 두번째 람다 인자를 이용하여 값 변환 함수를 호출할 수 있습니다.
두개의 람다로 `groupBy()`를 통해 나온 결과 맵은 `keySelector` 함수로 생성된 키와 값 변환 함수로 나온 값 결과로 매핑 됩니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five")

    println(numbers.groupBy { it.first().toUpperCase() })
    println(numbers.groupBy(keySelector = { it.first() }, valueTransform = { it.toUpperCase() }))
//sampleEnd
}
```

요소를 그룹화 후 한번에 모든 그룹에 동작을 적용하려면 [`groupingBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/grouping-by.html) 함수를 사용하면 됩니다.
이 함수는 [`Grouping`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-grouping/index.html) 타입의 인스턴스를 반환합니다.
`Grouping` 인스턴스는 늦은 방법으로 작업을 수행할 수 있습니다. 그룹은 실제로 작업이 실행되기 직전에 생성됩니다.

`Grouping`는 다음의 동작을 지원합니다:

* [`eachCount()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/each-count.html)는 각 그룹의 요소의 갯수를 반환합니다.
* [`fold()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/fold.html) 와 [`reduce()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reduce.html)은 분리된 콜렉션으로 각 그룹에서 [fold and reduce](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/collection-aggregate-operations#fold-and-reduce) 작업을 수행하고 결과를 반환합니다.
* [`aggregate()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/aggregate.html)는 각 그룹의 모든 요소에 주어진 작업을 적용하고 결과를 반환합니다.
   이것은 가장 일반적인 `Grouping` 동작입니다. fold와 reduce 외에 다른 작업을 구현할 때 사용하면 됩니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.groupingBy { it.first() }.eachCount())
//sampleEnd
}
```