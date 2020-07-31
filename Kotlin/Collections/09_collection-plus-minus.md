# `plus` 와 `minus` 동작 (`plus` and `minus` Operators)

Kotlin에서 [`plus`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus.html) (`+`) 와 [`minus`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus.html) (`-`) 동작은 콜렉션에서 사용할 수 있게 정의되어 있습니다.
첫번째 피연산자로 콜렉션을 취하고 두번째 피연산자로 요소나 다른 콜렉션을 취할 수 있습니다.
반환된 값은 하나의 새로운 읽기 전용 콜렉션입니다:

* `plus`의 결과는 원본 콜렉션과 두번째 피연산자를 포함합니다.
* `minus`의 결과는 두번째 피연산자의 요소를 제외한 원본 콜렉션을 포함합니다.
   두번째 피연산자가 요소이면 `minus`는 일치하는 첫번째 요소를 삭제합니다; 두번째 피연산자가 콜렉션이면 일치하는 모든 요소를 삭제합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")

    val plusList = numbers + "five"
    val minusList = numbers - listOf("three", "four")
    println(plusList)
    println(minusList)
//sampleEnd
}
```

맵에서 `plus` 와 `minus`는 [Map Specific Operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/map-specific-operations)을 참고 바랍니다.
[augmented assignment operators](https://kotlinlang.org/docs/reference/operator-overloading.html#assignments) [`plusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus-assign.html) (`+=`) 와 [`minusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus-assign.html) (`-=`)는 콜렉션에서 사용할 수 있게 정의되어 있습니다.
그러나 읽기 전용 콜렉션의 경우 실제로 `plus` 또는 `minus` 연산을 사용하고 그 결과를 같은 변수에 저장하려고 할 것입니다.
따라서 `var` 읽기 전용 콜렉션에서만 사용할 수 있습니다.
변경 가능한 콜렉션의 경우 `val`인 경우 콜렉션을 수정합니다. 더 자세한 내용은 [Collection Write Operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/collection-write-operations)을 참고 바랍니다.