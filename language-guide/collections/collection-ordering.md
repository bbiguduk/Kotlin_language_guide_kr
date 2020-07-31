# 콜렉션 정렬 \(Collection Ordering\)

요소의 순서는 특정 콜렉션 타입에서 중요합니다. 예를 들어 요소가 다르게 정렬 된 두 리스트의 경우 두 리스트는 동일하지 않습니다.

Kotlin에서 객체의 순서는 여러가지 방법으로 정의되어 있습니다.

먼저 _natural_ 정렬이 있습니다. [`Comparable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/index.html) 인터페이스 상속자를 위해 정의되었습니다. 자연 정렬을 다른 특별한 정렬을 사용하지 않을 때 사용됩니다.

대부분의 타입은 다음과 같습니다:

* 숫자 타입은 기본적인 숫자 정렬로 사용합니다: `1`이 `0`보다 크고; `-3.4f`이 `-5f`보다 큽니다.
* `Char` 와 `String` [lexicographical order](https://en.wikipedia.org/wiki/Lexicographical_order) 사용합니다: `b`가 `a`보다 크고 `world`가 `hello`보다 큽니다.

사용자 정의 타입을 자연 정렬로 정의하려면 타입이 `Comparable` 상속을 받아야 합니다. `compareTo()` 함수를 구현해야 합니다. `compareTo()`는 같은 타입의 다른 객체를 인자로 받고 더 큰 객체에 대해 정수형 값으로 반환합니다:

* 양수값은 수신자 객체가 더 크다는 의미입니다.
* 음수값은 인자보다 더 작다는 의미입니다.
* 0은 두 객체가 같다는 의미입니다.

아래는 메이저와 마이너로 구성 된 버전에 대한 정렬을 사용할 수 있습니다.

```kotlin
class Version(val major: Int, val minor: Int): Comparable<Version> {
    override fun compareTo(other: Version): Int {
        if (this.major != other.major) {
            return this.major - other.major
        } else if (this.minor != other.minor) {
            return this.minor - other.minor
        } else return 0
    }
}

fun main() {    
    println(Version(1, 2) > Version(1, 3))
    println(Version(2, 0) > Version(1, 5))
}
```

_Custom_ 정렬을 원하는 방식으로 어떠한 타입의 객체든 정렬 할 수 있습니다. 특히 비교할 수 없는 객체의 순서를 정의하거나 비교가능한 타입의 정렬을 정의할 수 있습니다. 커스텀 정렬을 정의하려면 [`Comparator`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparator/index.html)을 생성해야 합니다. `Comparator`는 `compare()` 함수를 포함합니다: class의 두 인스턴스를 받아 비교하고 정수형 값으로 반환합니다. 결과는 위에서 설명한 `compareTo()`의 결과와 같습니다.

```kotlin
fun main() {
//sampleStart
    val lengthComparator = Comparator { str1: String, str2: String -> str1.length - str2.length }
    println(listOf("aaa", "bb", "c").sortedWith(lengthComparator))
//sampleEnd
}
```

`lengthComparator`를 사용하면 기본 사전적 순서 대신에 길이로 정렬이 가능합니다.

`Comparator`를 정의하는 짧은 방법은 표준 라이브러리에서 [`compareBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.comparisons/compare-by.html) 함수 입니다. `compareBy()`는 인스턴스에서 `Comparable` 값을 생성하는 람다 함수를 가지고 커스텀 정렬을 정의합니다. `compareBy()`을 사용하면 위의 예제를 아래와 같이 표현 가능합니다:

```kotlin
fun main() {
//sampleStart    
println(listOf("aaa", "bb", "c").sortedWith(compareBy { it.length }))
//sampleEnd
}
```

Kotlin 콜렉션 패키지는 콜렉션의 자연적, 커스텀 그리고 랜덤 정렬을 위한 함수를 제공합니다. 이 페이지에서 [read-only](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/kotlin-kotlin-collections-overview#collection-types) 콜렉션에 적용하는 정렬 함수를 설명합니다. 이 함수는 정렬이 요청된 기존 콜렉션의 요소를 포함한 새로운 콜렉션을 반환합니다. [mutable](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/kotlin-kotlin-collections-overview#collection-types) 콜렉션 정렬에 대한 자세한 설명은 [List Specific Operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/list-specific-operations#sorting)을 참고 바랍니다.

## 자연 정렬 \(Natural order\)

기본 함수 [`sorted()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sorted.html) 와 [`sortedDescending()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sorted-descending.html)은 자연적인 정렬에 따라 오름차순이나 내림차순으로 콜렉션 요소를 정렬하고 반환합니다. 이 함수는 `Comparable` 요소의 콜렉션에 적용합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")

    println("Sorted ascending: ${numbers.sorted()}")
    println("Sorted descending: ${numbers.sortedDescending()}")
//sampleEnd
}
```

## 커스텀 정렬 \(Custom orders\)

커스텀 정렬이나 비교 불가한 객체의 정렬은 [`sortedBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sorted-by.html) 와 [`sortedByDescending()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sorted-by-descending.html) 함수를 사용합니다. 콜렉션 요소를 `Comparable` 값에 매핑하고 콜렉션 값을 자연 정렬을 하는 selector 함수를 사용합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")

    val sortedNumbers = numbers.sortedBy { it.length }
    println("Sorted by length ascending: $sortedNumbers")
    val sortedByLast = numbers.sortedByDescending { it.last() }
    println("Sorted by the last letter descending: $sortedByLast")
//sampleEnd
}
```

콜렉션에 커스텀 정렬을 정의하기 위해 고유한 `Comparator`를 제공할 수 있습니다. 이렇게 하려면 `Comparator`에 전달된 [`sortedWith()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sorted-with.html) 함수를 호출해야 합니다. 이 함수를 사용하여 문자열 길이별로 정렬하려면 아래와 같습니다:

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    println("Sorted by length ascending: ${numbers.sortedWith(compareBy { it.length })}")
//sampleEnd
}
```

## 역정렬 \(Reverse order\)

반대로 정렬 된 콜렉션을 사용하려면 [`reversed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reversed.html) 함수를 사용하면 됩니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.reversed())
//sampleEnd
}
```

`reversed()`는 새로운 콜렉션에 복사한 요소를 반환합니다. 그러므로 기존에 콜렉션을 변경하기 전에 `reversed()`의 결과는 기존 콜렉션이 변경되더라도 반영되지 않습니다.

다른 역정렬 함수인 [`asReversed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/as-reversed.html)은 같은 콜렉션 인스턴스를 역정렬로 반환합니다. 그래서 기존 리스트가 변경되지 않는다면 `reversed()` 보다 더 가볍고 더 유용합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    val reversedNumbers = numbers.asReversed()
    println(reversedNumbers)
//sampleEnd
}
```

기존 리스트가 변경가능하면 모든 역정렬에 대한 변환 또는 그 반대는 리스트에 영향을 줍니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four")
    val reversedNumbers = numbers.asReversed()
    println(reversedNumbers)
    numbers.add("five")
    println(reversedNumbers)
//sampleEnd
}
```

그러나 리스트가 변경가능성을 모르거나 리스트가 아닌경우 `reversed()`의 결과가 사본이므로 더 유용합니다.

## 랜덤정렬 \(Random order\)

마지막으로 [`shuffled()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/shuffled.html) 함수는 콜렉션의 요소를 랜덤 정렬 한 새로운 `List`를 반환합니다. 인자값 없이 호출 하거나 [`Random`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.random/-random/index.html) 객체를 이용하여 호출할 수 있습니다.

```kotlin
fun main() {
//sampleStart
     val numbers = listOf("one", "two", "three", "four")
     println(numbers.shuffled())
//sampleEnd
}
```

