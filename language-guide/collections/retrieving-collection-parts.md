# 콜렉션 부분화 \(Retrieving Collection Parts\)

Kotlin 표준 라이브러리는 콜렉션의 부분을 다루는 확장 함수가 포함되어 있습니다. 이 함수는 요소를 선택하고 결과 콜렉션을 다양한 방법으로 제공합니다: 요소의 위치를 명시적으로 리스트화 하거나 특별한 크기의 결과를 얻거나 다른 여러가지 방법이 있습니다.

## Slice

[`slice()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/slice.html)는 주어진 인덱스의 콜렉션 요소를 하나의 리스트로 반환합니다. 인덱스는 [range](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/ranges-and-progressions-1) 또는 콜렉션의 정수형 값으로 전달됩니다.

```kotlin
fun main() {
//sampleStart    
    val numbers = listOf("one", "two", "three", "four", "five", "six")    
    println(numbers.slice(1..3))
    println(numbers.slice(0..4 step 2))
    println(numbers.slice(setOf(3, 5, 0)))    
//sampleEnd
}
```

## Take 와 drop

일정 갯수의 요소를 처음부터 가져오려면 [`take()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/take.html) 함수를 사용합니다. 마지막부터 가져오려면 [`takeLast()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/take-last.html) 함수를 사용합니다. 두 함수 모두 콜렉션 크기보다 큰 수로 호출하면 콜렉션 전체를 반환합니다.

처음 또는 마지막에서 일정 갯수를 제외하고 가져오려면 [`drop()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/drop.html) 와 [`dropLast()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/drop-last.html) 함수를 사용합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.take(3))
    println(numbers.takeLast(3))
    println(numbers.drop(1))
    println(numbers.dropLast(5))
//sampleEnd
}
```

속성을 사용하여 요소를 가져오거나 제거할 수 있습니다. 위에서 설명한 비슷한 4개의 함수가 있습니다:

* [`takeWhile()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/take-while.html)은 `take()`에 조건을 추가한 것과 같습니다: 처음부터 조건과 판단하며 조건과 맞지 않는 첫번째 요소부터는 제외됩니다. 만약에 콜렉션 요소의 첫번째 부터 조건과 맞지 않으면 빈 결과 값을 반환합니다.
* [`takeLastWhile()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/take-last-while.html)은 `takeLast()`와 비슷합니다: 마지막부터 조건과 판단하며 조건과 맞지 않는 요소부터는 제외됩니다. 만약에 콜렉션 요소의 마지막 부터 조건과 맞지 않으면 빈 결과 값을 반환합니다.
* [`dropWhile()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/drop-while.html)은 같은 조건이 주어질 때 `takeWhile()`의 반대로 동작합니다:  조건과 맞지 않는 첫 요소부터 마지막 요소를 반환합니다.
* [`dropLastWhile()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/drop-last-while.html)은 같은 조건이 주어질 때 `takeLastWhile()`의 반대로 동작합니다: 조건과 맞지 않는 마지막 요소부터 첫 요소를 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.takeWhile { !it.startsWith('f') })
    println(numbers.takeLastWhile { it != "three" })
    println(numbers.dropWhile { it.length == 3 })
    println(numbers.dropLastWhile { it.contains('i') })
//sampleEnd
}
```

## Chunked

주어진 크기로 콜렉션을 쪼갤 때 [`chunked()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/chunked.html) 함수를 사용합니다. `chunked()`는 쪼갤 크기 인 하나의 인자를 받으며 주어진 크기의 `List`의 `List`를 반환합니다. 첫번째 요소부터 시작하여 `size` 만큼의 요소를 포함하고 그 다음 요소도 같은 `size`의 요소를 포함하여 쪼개집니다. 마지막 부분은 주어진 크기보다 작을 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = (0..13).toList()
    println(numbers.chunked(3))
//sampleEnd
}
```

쪼개진 결과에 바로 변환을 적용할 수 있습니다. `chunked()`를 호출 할 때 람다 함수로 변환을 제공합니다. 람다의 인자는 쪼개진 콜렉션입니다. 변환과 함께 `chunked()`가 호출될 때 쪼개진 콜렉션은 람다에서 바로 소모되어야 할 짧은 수명의 `List` 입니다.

```kotlin
fun main() {
//sampleStart
    val numbers = (0..13).toList() 
    println(numbers.chunked(3) { it.sum() })  // `it` is a chunk of the original collection
//sampleEnd
}
```

## Windowed

주어진 크기로 콜렉션 요소의 모든 가능한 범위를 나타낼 수 있습니다. [`windowed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/windowed.html) 함수를 사용할 수 있으며 주어진 크기를 슬라이딩 창문처럼 콜렉션의 가능한 범위를 반환합니다. `chunked()` 달리, `windowed()`는 각 콜렉션 요소에서 시작하여 요소 범위 \(_windows_\)를 반환합니다. 모든 window는 하나의 `List` 요소로 반환됩니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five")    
    println(numbers.windowed(3))
//sampleEnd
}
```

`windowed()`는 더 유연한 파라미터를 제공합니다:

* `step`은 두 window의 첫 요소의 거리를 정의합니다. 기본값은 1이므로 모든 요소에서 시작하는 window가 포함됩니다. step을 2로 증가시키면 첫번째, 세번째 등 홀수 요소에서 시작하는 window만 받게 됩니다.
* `partialWindows`은 콜렉션의 마지막에 주어진 크기보다 작은 window를 포함합니다. 예를 들어 세 요소의 window를 요청할 경우 마지막 두개의 요소는 생성할 수 없습니다. `partialWindows` 설정을 하게 되면 크기가 2인 하나의 리스트가 추가되게 됩니다.

마지막으로 반환 된 범위에 바로 변환을 적용할 수 있습니다. 이것은 `windowed()` 호출 시 람다 함수로 변환을 제공합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = (1..10).toList()
    println(numbers.windowed(3, step = 2, partialWindows = true))
    println(numbers.windowed(3) { it.sum() })
//sampleEnd
}
```

두 요소로 window 생성할 때 별도의 함수인 [`zipWithNext()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/zip-with-next.html)이 있습니다. 리시버 콜렉션의 인접한 요소의 쌍을 생성합니다. `zipWithNext()`은 콜렉션을 쌍으로 나누지 않습니다; 마지막 요소를 제외한 각 요소에 쌍을 생성하므로 `[1, 2, 3, 4]`의 결과는 `[[1, 2`\], `[3, 4]]`이 아닌 `[[1, 2], [2, 3], [3, 4]]` 입니다. `zipWithNext()`는 변한 함수를 호출 할 수 있습니다; 리시버 콜렉션의 두 요소를 인수로 사용해야 합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five")    
    println(numbers.zipWithNext())
    println(numbers.zipWithNext() { s1, s2 -> s1.length > s2.length})
//sampleEnd
}
```

