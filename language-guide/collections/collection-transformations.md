# 콜렉션 변환 \(Collection Transformations\)

Kotlin 표준 라이브러리는 콜렉션 _변환 \(transformations\)_ 을 위한 확장 함수를 제공합니다. 이 함수는 변환 규칙에 따라 기존 콜렉션을 변환하여 새로운 콜렉션을 만듭니다. 이 페이지에서는 콜렉션 변환 함수에 대해 알아보겠습니다.

## 매핑 \(Mapping\)

_매핑 \(mapping\)_ 변환은 다른 콜렉션 요소의 함수 결과로 부터 콜렉션을 생성합니다. 기본 매핑 함수는 [`map()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map.html) 입니다. 주어진 람다 함수에 각 다음 요소에 적용하고 람다 결과의 리스트를 반환합니다. 결과 순서는 원래 요소의 순서와 동일합니다. 인덱스를 인자로 사용하는 변환을 적용하려면 [`mapIndexed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-indexed.html)을 사용합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = setOf(1, 2, 3)
    println(numbers.map { it * 3 })
    println(numbers.mapIndexed { idx, value -> value * idx })
//sampleEnd
}
```

변환 작업이 특정 요소에 대해 `null`을 생성하는 경우 `map()` 대신 [`mapNotNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-not-null.html) 또는 `mapIndexed()` 대신 [`mapIndexedNotNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-indexed-not-null.html) 함수를 이용해 `null`을 필터링 할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = setOf(1, 2, 3)
    println(numbers.mapNotNull { if ( it == 2) null else it * 3 })
    println(numbers.mapIndexedNotNull { idx, value -> if (idx == 0) null else value * idx })
//sampleEnd
}
```

map을 변환할 때 2가지 옵션이 있습니다: 키를 변환하고 값을 변경하지 않거나 그 반대의 경우가 있습니다. 키를 변환하려면 [`mapKeys()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-keys.html)를 사용하고 값을 변환하려면 [`mapValues()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-values.html) 사용합니다. 두 함수 모두 map 항목을 인자로 사용하므로 키와 값을 모두 조작할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    println(numbersMap.mapKeys { it.key.toUpperCase() })
    println(numbersMap.mapValues { it.value + it.key.length })
//sampleEnd
}
```

## Zipping

_Zipping_ 변환은 두 콜렉션의 같은 위치의 요소를 쌍으로 만들어 줍니다. Kotlin 표준 라이브러리에서는 [`zip()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/zip.html) 확장 함수를 이용하여 수행할 수 있습니다. 콜렉션에서 호출되거나 다른 콜렉션을 인자로 가진 배열에서 호출되면 `zip()`은 `Pair` 객체의 `List`를 반환합니다. 수신자 콜렉션의 요소는 그 쌍의 첫번째 요소입니다. 콜렉션의 사이즈가 다르면 `zip()`의 결과는 더 작은 사이즈를 따릅니다; 더 큰 콜렉션의 요소는 결과에 포함되지 않습니다. `zip()`은 인픽스 형태인 `a zip b`으로 호출 될 수도 있습니다.

```kotlin
fun main() {
//sampleStart
    val colors = listOf("red", "brown", "grey")
    val animals = listOf("fox", "bear", "wolf")
    println(colors zip animals)

    val twoAnimals = listOf("fox", "bear")
    println(colors.zip(twoAnimals))
//sampleEnd
}
```

수신자 요소와 인자 요소의 두개의 매개변수를 가지는 변환함수를 사용하여 `zip()`을 호출 할 수 있습니다. 이 경우 결과 `List`는 같은 위치의 수신자와 인자 요소의 변환 함수의 결과 값을 포함합니다.

```kotlin
fun main() {
//sampleStart
    val colors = listOf("red", "brown", "grey")
    val animals = listOf("fox", "bear", "wolf")

    println(colors.zip(animals) { color, animal -> "The ${animal.capitalize()} is $color"})
//sampleEnd
}
```

`Pair`의 `List`를 가지고 있을 때 역변환도 가능합니다 - _unzipping_ - 이것은 각 쌍으로 부터 2개의 리스트를 만듭니다:

* 첫번째 리스트는 `Pair`의 첫번째 요소를 포함합니다. 
* 두번째 리스트는 `Pair`의 두번째 요소를 포함합니다.

[`unzip()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/unzip.html)을 호출하여 pair의 list를 분리 할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numberPairs = listOf("one" to 1, "two" to 2, "three" to 3, "four" to 4)
    println(numberPairs.unzip())
//sampleEnd
}
```

## 조합 \(Association\)

_Association_ 변환을 통해 콜렉션 요소와 이와 연관된 값에서 맵을 만들 수 있습니다. 다른 조합 타입에서 요소는 조합 맵의 키거나 값일 수 있습니다.

기본 조합 함수 [`associateWith()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/associate-with.html)은 기존 콜렉션이 키인 `Map`을 만들고 주어진 변환 함수에 의해 값을 생성합니다. 만약에 두 요소가 같다면 마지막 요소만 맵에 남아있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.associateWith { it.length })
//sampleEnd
}
```

콜렉션 요소를 값으로 사용하여 맵을 생성하는 경우 [`associateBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/associate-by.html) 함수가 있습니다. 이것은 요소의 값에 따라 키를 반환합니다. 만약에 두 요소가 같다면 마지막 요소만 맵에 남아있습니다. `associateBy()`는 값 변환 함수로 호출 할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")

    println(numbers.associateBy { it.first().toUpperCase() })
    println(numbers.associateBy(keySelector = { it.first().toUpperCase() }, valueTransform = { it.length }))
//sampleEnd
}
```

키와 값이 콜렉션 요소에서 생성되는 맵을 만드는데 또 다른 방법은 [`associate()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/associate.html) 함수 입니다. `Pair`를 반환하는 람다 함수를 사용합니다.

`associate()`는 퍼포먼스에 영향이 있는 수명이 짧은 `Pair` 객체를 생성합니다. 따라서 `associate()`은 퍼포먼스가 중요하지 않거나 다른 옵션보다 더 나을 때 사용합니다.

키와 해당 값이 요소에서 함께 생성되는 경우 `associate()`을 사용합니다.

```kotlin
fun main() {
data class FullName (val firstName: String, val lastName: String)

fun parseFullName(fullName: String): FullName {
    val nameParts = fullName.split(" ")
    if (nameParts.size == 2) {
        return FullName(nameParts[0], nameParts[1])
    } else throw Exception("Wrong name format")
}

//sampleStart
    val names = listOf("Alice Adams", "Brian Brown", "Clara Campbell")
    println(names.associate { name -> parseFullName(name).let { it.lastName to it.firstName } })  
//sampleEnd
}
```

여기서는 먼저 요소에 대해 변환 함수를 호출하고 함수 결과에서 쌍을 만듭니다.

## Flattening

중첩된 콜렉션을 작업할 때 중첩된 콜렉션 요소를 플랫하게 접근할 수 있는 표준 라이브러리 함수를 찾을 것입니다.

첫번째 함수로는 [`flatten()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/flatten.html)이 있습니다. 예를 들어 `Set`으로 구성 된 `List`와 같은 콜렉션의 콜렉션에서 호출 할 수 있습니다. 이 함수는 중첩된 콜렉션의 모든 요소를 하나의 `List`로 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
    println(numberSets.flatten())
//sampleEnd
}
```

또 다른 함수 [`flatMap()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/flat-map.html)은 중첩된 콜렉션을 처리하는 유연한 방법을 제공합니다. 콜렉션 요소를 다른 콜렉션에 매핑하는 함수입니다. 결과적으로 `flatMap()`은 모든 요소의 값을 하나의 리스트로 반환합니다. 따라서 `flatMap()`은 `map()` \(매핑 결과를 콜렉션으로 사용\)과 `flatten()`의 다음 호출로 동작합니다.

```kotlin
data class StringContainer(val values: List<String>)

fun main() {
//sampleStart
    val containers = listOf(
        StringContainer(listOf("one", "two", "three")),
        StringContainer(listOf("four", "five", "six")),
        StringContainer(listOf("seven", "eight"))
    )
    println(containers.flatMap { it.values })
//sampleEnd
}
```

## 문자열 표현 \(String representation\)

콜렉션 컨텐츠를 읽을 수 있는 형태로 하려면 [`joinToString()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/join-to-string.html) 와 [`joinTo()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/join-to.html) 변환 함수를 사용하면 됩니다.

`joinToString()`은 콜렉션 요소를 하나의 `String`으로 만듭니다. `joinTo()`는 동일한 역할을 하지만 주어진 [`Appendable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-appendable/index.html) 객체에 결과를 추가합니다.

기본 인자로 호출하면 콜렉션에 `toString()` 호출과 비슷한 결과를 반환합니다: 요소를 콤마와 빈칸으로 구분한 문자열로 표현합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")

    println(numbers)         
    println(numbers.joinToString())

    val listString = StringBuffer("The list of numbers: ")
    numbers.joinTo(listString)
    println(listString)
//sampleEnd
}
```

커스텀 문자열 표현을 만드려면 `separator`, `prefix`, `postfix` 파라미터를 사용하면 가능합니다. 해당 파라미터를 사용하여 호출하여 반환 된 문자열은 `prefix`의 접두사를 `postfix`의 접미사를 사용하여 결과를 반환합니다. `separator`는 마지막 요소를 제외하고 각 요소의 구분자로 사용됩니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")    
    println(numbers.joinToString(separator = " | ", prefix = "start: ", postfix = ": end"))
//sampleEnd
}
```

큰 콜렉션에서 결과에 원하는 요소가 포함 되도록 일정 범위를 지정할 수 있는 `limit`가 있습니다. `limit`에 초과되는 콜렉션 크기의 값들은 `truncated` 인자의 하나의 값으로 변환 됩니다.

```kotlin
fun main() {
//sampleStart
    val numbers = (1..100).toList()
    println(numbers.joinToString(limit = 10, truncated = "<...>"))
//sampleEnd
}
```

마지막으로 요소 자신의 표현을 커스텀 할 수 있는 `transform` 함수가 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.joinToString { "Element: ${it.toUpperCase()}"})
//sampleEnd
}
```

