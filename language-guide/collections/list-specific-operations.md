# 리스트 동작 \(List Specific Operations\)

[`List`](kotlin-kotlin-collections-overview.md#list)는 Kotlin에서 가장 유명한 내장 콜렉션입니다. 리스트 요소에 대한 인덱스 접근은 강력한 동작들을 제공합니다.

## 인덱스로 요소 추출 \(Retrieving elements by index\)

리스트는 요소 검색을 위한 모든 공통 연산을 지원합니다: `elementAt()`, `first()`, `last()`, 와 [요소 추출 \(Retrieving Single Elements\)](retrieving-single-elements.md) 에 나와있는 연산. 리스트 요소에 인덱스 접근이 가장 특별한 것이므로 요소를 읽는 가장 간단한 방법은 인덱스로 요소를 검색하는 것입니다. 인자로 인덱스를 전달하는 [`get()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/get.html) 함수 또는 짧은 표현으로 `[index]`로 수행합니다.

요청한 인덱스보다 리스트 크기가 작으면 예외를 발생합니다. 이러한 예외를 우회할 수 있는 두개의 함수가 있습니다:

* [`getOrElse()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-else.html)는 콜렉션에 해당 인덱스 값이 없을 경우 기본 값을 계산하여 반환합니다.
* [`getOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-null.html)은 콜렉션에 해당 인덱스 값이 없을 경우 `null`을 기본 값으로 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.get(0))
    println(numbers[0])
    //numbers.get(5)                         // exception!
    println(numbers.getOrNull(5))             // null
    println(numbers.getOrElse(5, {it}))        // 5
//sampleEnd
}
```

## 리스트 부분 추출 \(Retrieving list parts\)

[콜렉션 부분화 \(Retrieving Collection Parts\) ](retrieving-collection-parts.md)를 위한 기본 동작으로 리스트는 리스트의 특정 요소의 범위를 반환하는 [`subList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/sub-list.html) 함수를 제공합니다. 따라서 기존 콜렉션이 변경되면 이전에 생성된 서브리스트도 변경되고 그 반대 동작도 동일합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = (0..13).toList()
    println(numbers.subList(3, 6))
//sampleEnd
}
```

## 요소 위치 검색 \(Finding element positions\)

### 선형 검색 \(Linear search\)

어떠한 리스트 안에서 [`indexOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index-of.html) 와 [`lastIndexOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/last-index-of.html) 함수를 이용해 요소의 위치를 찾을 수 있습니다. 리스트에서 주어진 인자와 같은 요소의 첫번째와 마지막 위치를 반환합니다. 해당 요소를 찾지 못하면 두 함수 모두 `-1`을 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(1, 2, 3, 4, 2, 5)
    println(numbers.indexOf(2))
    println(numbers.lastIndexOf(2))
//sampleEnd
}
```

조건을 가지는 함수도 존재합니다:

* [`indexOfFirst()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index-of-first.html)은 조건에 맞는 첫번째 요소를 반환하거나 조건에 맞는 요소가 없으면 `-1`을 반환합니다.
* [`indexOfLast()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index-of-last.html)은 조건에 맞는 마지막 요소를 반환하거나 맞는 요소가 없으면 `-1`을 반한합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    println(numbers.indexOfFirst { it > 2})
    println(numbers.indexOfLast { it % 2 == 1})
//sampleEnd
}
```

### 정렬된 리스트에서의 이진 검색 \(Binary search in sorted lists\)

리스트에서 요소를 검색하는 다른 방법이 있습니다 – [이진 검색 \(binary search\)](https://en.wikipedia.org/wiki/Binary_search_algorithm). 다른 검색 함수보다 빠르게 동작하지만 _리스트가 오름차순으로_ [_정렬 \(sorted\)_](collection-ordering.md) _되어 있어야 합니다._ 그렇지 않으면 결과가 정의되지 않습니다.

정렬된 리스트에서 요소를 검색하려면 인자로 값을 [`binarySearch()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/binary-search.html) 함수에 전달하여 호출합니다. 해당 요소가 존재한다면 해당 인덱스를 반환하고 그렇지 않으면 `(-insertionPoint - 1)`을 반환합니다. 여기서 `insertionPoint`은 정렬된 리스트에 적합한 위치의 인덱스 입니다. 주어진 값과 함께 하나 이상의 요소가 있을경우 검색은 그들의 인덱스를 반환할 수 있습니다.

인덱스 범위로 검색할 수도 있습니다: 이러한 경우 함수는 주어진 인덱스 사이만 검색합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four")
    numbers.sort()
    println(numbers)
    println(numbers.binarySearch("two"))  // 3
    println(numbers.binarySearch("z")) // -5
    println(numbers.binarySearch("two", 0, 2))  // -3
//sampleEnd
}
```

#### 이진 검색 비교기 \(Comparator binary search\)

리스트 요소가 `Comparable` 아닌경우 이진 검색을 사용하기 위해 [`Comparator`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparator.html)를 제공해야 합니다. 리스트는 `Comparator`에 따라 오름차순으로 정렬되어야 합니다. 예제를 살펴봅시다:

```kotlin
data class Product(val name: String, val price: Double)

fun main() {
//sampleStart
    val productList = listOf(
        Product("WebStorm", 49.0),
        Product("AppCode", 99.0),
        Product("DotTrace", 129.0),
        Product("ReSharper", 149.0))

    println(productList.binarySearch(Product("AppCode", 99.0), compareBy<Product> { it.price }.thenBy { it.name }))
//sampleEnd
}
```

여기 `Comparable`가 아닌 `Product` 인스턴스 리스트가 있고 `p1`의 가격이 `p2`의 가격보다 낮은 경우 제품 `p1`이 제품 `p2`보다 앞에 위치하도록 정의한 `Comparator`가 있습니다. 따라서 오름차순으로 정렬된 리스트를 가지려면 지정된 `Product`의 인덱스를 찾는 `binarySearch()`를 사용해야 합니다.

커스텀 비교기는 자연적인 순차가 아닌 정렬을 사용할 때도 편리합니다. 예를 들어 `String` 요소의 대소문자를 구분하지 않는 순서가 있습니다.

```kotlin
fun main() {
//sampleStart
    val colors = listOf("Blue", "green", "ORANGE", "Red", "yellow")
    println(colors.binarySearch("RED", String.CASE_INSENSITIVE_ORDER)) // 3
//sampleEnd
}
```

#### 이진 검색 비교 \(Comparison binary search\)

_comparison_ 함수를 이용한 이진 검색은 명시적인 검색 값이 없어도 요소를 찾을 수 있습니다. 대신 비교 함수 매핑 요소를 `Int` 값으로 하고 함수가 0을 반환하는 요소를 검색합니다. 리스트는 반드시 오름차순으로 정렬되어 있어야 합니다. 다시 말해 비교에 반환 값은 하나의 리스트 요소에서 다음 요소로 증가해야 합니다.

```kotlin
import kotlin.math.sign
//sampleStart
data class Product(val name: String, val price: Double)

fun priceComparison(product: Product, price: Double) = sign(product.price - price).toInt()

fun main() {
    val productList = listOf(
        Product("WebStorm", 49.0),
        Product("AppCode", 99.0),
        Product("DotTrace", 129.0),
        Product("ReSharper", 149.0))

    println(productList.binarySearch { priceComparison(it, 99.0) })
}
//sampleEnd
```

비교기와 비교 이진 검색은 리스트 범위에서도 동작할 수 있습니다.

## 리스트 쓰기 동작 \(List write operations\)

콜렉션 수정 동작은 [콜렉션 쓰기 동작 \(Collection Write Operations\) ](collection-write-operations.md)에 자세히 나와있습니다. [변경 가능한 \(mutable\)](kotlin-kotlin-collections-overview.md#collection-types) 리스트는 쓰기 동작을 제공합니다. 이러한 동작은 인덱스를 통해 요소에 접근하여 리스트 수정 기능을 확장합니다.

### 추가 \(Adding\)

리스트에서 특정 위치에 요소를 추가하려면 [`add()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/add.html)를 사용하고 추가할 요소의 위치를 지정하려면 [`addAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/add-all.html)에 위치 인자를 받아 사용합니다. 해당 위치 뒤에 오는 요소는 오른쪽으로 밀리게 됩니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "five", "six")
    numbers.add(1, "two")
    numbers.addAll(2, listOf("three", "four"))
    println(numbers)
//sampleEnd
}
```

### 업데이트 \(Updating\)

리스트는 주어진 위치의 요소를 대체하는 함수를 제공합니다 - [`set()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/set.html) 과 다르게 표현하면 `[]` 입니다. `set()`은 다른 요소의 인덱스를 변경하지 않습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "five", "three")
    numbers[1] =  "two"
    println(numbers)
//sampleEnd
}
```

[`fill()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/fill.html)은 간단하게 모든 콜렉션의 요소를 특정 값으로 대체할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    numbers.fill(3)
    println(numbers)
//sampleEnd
}
```

### 삭제 \(Removing\)

리스트에서 특정 위치의 요소를 삭제하려면 인자로 위치를 받는 [`removeAt()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/remove-at.html) 함수를 사용해야 합니다. 요소가 삭제된 위치의 다음 요소들은 인덱스가 하나씩 줄어듭니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4, 3)    
    numbers.removeAt(1)
    println(numbers)
//sampleEnd
}
```

첫번째와 마지막 요소를 삭제하기 위해 짧은 [`removeFirst()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-first.html) 와 [`removeLast()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-last.html) 가 있습니다. 빈 리스트에선 예외를 발생한다는 것을 명심해야 합니다. 대신에 null 을 받으려면 [`removeFirstOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-first-or-null.html) 과 [`removeLastOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-last-or-null.html) 을 사용해야 합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4, 3)    
    numbers.removeFirst()
    numbers.removeLast()
    println(numbers)
    
    val empty = mutableListOf<Int>()
    // empty.removeFirst() // NoSuchElementException: List is empty.
    empty.removeFirstOrNull() //null
//sampleEnd
}
```

### 정렬 \(Sorting\)

[콜렉션 정렬 \(Collection Ordering\) ](collection-ordering.md)에서 특정 순서로 콜렉션 요소를 추출하는 내용을 설명하였습니다. 변경 가능한 리스트의 경우 표준 라이브러리는 동일한 순서의 동작을 수행하는 유사한 확장 함수를 제공합니다. 리스트 인스턴스에 해당 동작을 적용하면 요소의 순서가 변경됩니다.

이러한 정렬 함수는 읽기 전용 리스트에 적용한 함수와 유사한 이름을 가지지만 접미사 `ed/d`가 포함되어 있지 않습니다:

* 모든 정렬 함수 이름에 `sorted*` 대신에 `sort*` 로 표기합니다: [`sort()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort.html), [`sortDescending()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort-descending.html), [`sortBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort-by.html), 등.
* `shuffled()` 대신에 [`shuffle()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/shuffle.html).
* `reversed()` 대신에 [`reverse()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reverse.html).

변경 가능한 리스트에서 호출 된 [`asReversed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/as-reversed.html)은 다른 변경 가능한 역정렬 된 리스트를 반환합니다. 변경사항은 기존 리스트에 반영 됩니다. 아래 예제는 읽기 가능한 리스트의 정렬 함수를 나타냅니다:

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four")

    numbers.sort()
    println("Sort into ascending: $numbers")
    numbers.sortDescending()
    println("Sort into descending: $numbers")

    numbers.sortBy { it.length }
    println("Sort into ascending by length: $numbers")
    numbers.sortByDescending { it.last() }
    println("Sort into descending by the last letter: $numbers")

    numbers.sortWith(compareBy<String> { it.length }.thenBy { it })
    println("Sort by Comparator: $numbers")

    numbers.shuffle()
    println("Shuffle: $numbers")

    numbers.reverse()
    println("Reverse: $numbers")
//sampleEnd
}
```

