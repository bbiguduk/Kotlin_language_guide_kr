# 콜렉션 쓰기 동작 \(Collection Write Operations\)

[Mutable collections](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/kotlin-kotlin-collections-overview#collection-types)은 예를 들어 요소를 추가 또는 삭제를 위한 콜렉션 콘텐츠를 변경할 수 있는 동작을 지원합니다. 이 페이지에서 `MutableCollection`의 동작 가능한 모든 구현을 설명하도록 하겠습니다. `List` 와 `Map`에서 가능한 동작은 [List Specific Operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/list-specific-operations) 와 [Map Specific Operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/map-specific-operations)을 참고 바랍니다.

## 요소 추가 \(Adding elements\)

리스트 또는 셋에 하나의 요소를 추가하려면 [`add()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/add.html) 함수를 사용해야 합니다. 추가한 객체는 콜렉션 가장 마지막에 붙게 됩니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    numbers.add(5)
    println(numbers)
//sampleEnd
}
```

[`addAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/add-all.html)은 리스트 또는 셋에 인자로 받은 모든 요소를 추가합니다. 인자는 `Iterable`, `Sequence`, `Array` 일 수 있습니다. 수신자와 인자의 타입이 다를 수 있습니다. 예를 들어 `Set`에 모든 항목을 `List`에 추가할 수 있습니다.

리스트에서 호출될 때 `addAll()`은 인자와 같은 순서로 새로운 요소를 추가합니다. 첫번째 인자로 요소의 위치를 지정하여 `addAll()`을 호출 할 수 있습니다. 인자 콜렉션의 첫번째 요소가 해당 위치에 삽입됩니다. 인자 콜렉션의 다른 요소는 첫번째 요소에 뒤따라 붙게되며 수신자 요소는 그 뒤로 이동시킵니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 5, 6)
    numbers.addAll(arrayOf(7, 8))
    println(numbers)
    numbers.addAll(2, setOf(3, 4))
    println(numbers)
//sampleEnd
}
```

[`plus` operator](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/plus-minus-plus-and-minus-operators)의 내부에 있는 [`plusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus-assign.html) \(`+=`\)를 이용하여 요소를 추가 할 수 있습니다. 변경가능한 콜렉션에 적용하면 `+=`은 두번째 연산자 \(요소 또는 다른 콜렉션\)를 콜렉션 끝에 추가합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two")
    numbers += "three"
    println(numbers)
    numbers += listOf("four", "five")    
    println(numbers)
//sampleEnd
}
```

## 요소 삭제 \(Removing elements\)

변경가능한 콜렉션에서 요소를 삭제하려면 [`remove()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove.html) 함수를 사용합니다. `remove()`는 요소 값을 적용하고 값의 한 항목을 제거합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4, 3)
    numbers.remove(3)                    // removes the first `3`
    println(numbers)
    numbers.remove(5)                    // removes nothing
    println(numbers)
//sampleEnd
}
```

한번에 여러 요소를 삭제하려면 아래 함수를 참고바랍니다:

* [`removeAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-all.html)은 콜렉션에 있는 모든 요소를 삭제합니다.

   또는 인자로 조건을 추가하여 호출할 수 있으며 조건이 일치한 요소를 모두 삭제합니다.

* [`retainAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/retain-all.html)은 `removeAll()`의 반대입니다: 인자 콜렉션에서 요소를 제외한 모든 요소를 삭제합니다.

   조건을 사용하면 해당 조건에 맞는 요소를 제외한 나머지 요소를 삭제합니다.

* [`clear()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/clear.html)은 모든 요소를 삭제하고 빈상태로 만듭니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    println(numbers)
    numbers.retainAll { it >= 3 }
    println(numbers)
    numbers.clear()
    println(numbers)

    val numbersSet = mutableSetOf("one", "two", "three", "four")
    numbersSet.removeAll(setOf("one", "two"))
    println(numbersSet)
//sampleEnd
}
```

콜렉션에서 요소를 삭제하는 다른 방법은 [`minus`](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/plus-minus-plus-and-minus-operators)의 내부에 있는 [`minusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus-assign.html) \(`-=`\) 동작입니다. 두번째 인자는 요소 타입의 단일 인스턴스 또는 다른 콜렉션이 가능합니다. 하나의 요소와 함께 사용 시 `-=`은 첫 일치하는 항목을 삭제합니다. 반대로 콜렉션일 경우 모든 일치하는 항목을 삭제합니다. 예를 들어 리스트에 중복 요소가 있으면 한번에 삭제됩니다. 두번째 피연산자는 콜렉션에 없는 요소를 포함할 수 있습니다. 이러한 요소는 동작에 영향을 미치지 않습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "three", "four")
    numbers -= "three"
    println(numbers)
    numbers -= listOf("four", "five")    
    //numbers -= listOf("four")    // does the same as above
    println(numbers)    
//sampleEnd
}
```

## 요소 업데이트 \(Updating elements\)

리스트와 맵은 요소 업데이트를 제공합니다. [List Specific Operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/list-specific-operations) 와 [Map Specific Operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/map-specific-operations)에 자세히 설명되어 있습니다. 셋의 경우 업데이트는 실제로 요소를 삭제하고 다시 추가하기 때문에 의미가 없습니다.

