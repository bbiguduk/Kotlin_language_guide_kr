# 콜렉션 동작 (Collection Operations Overview)

Kotlin 표준 라이브러리는 콜렉션에서 작업을 수행하기 위한 다양한 함수를 제공합니다. 요소를 불러오거나 추가하는 간단한 작업도 포함되며 검색, 정렬, 필터, 변환 등을 복잡한 작업도 포함됩니다.

## 확장과 멤버 함수 (Extension and member functions)

콜렉션 동작은 2가지 방법으로 표준 라이브러리에 구현됩니다:
콜렉션 인터페이스의 [member functions](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-classes-and-inheritance#class-members)와 [extension functions](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/extensions#extension-functions)입니다.

멤버 함수는 콜렉션 타입의 필수적인 동작을 정의합니다.
예를 들어 [`Collection`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-collection/index.html)은 콜렉션이 비어 있는지 체크하는 [`isEmpty()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-collection/is-empty.html) 함수를 포함합니다; [`List`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/index.html)는 요소에 접근을 위한 인덱스를 위한 [`get()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/get.html)을 포함합니다.

콜렉션 인터페이스를 작성할 때 반드시 멤버 함수를 구현해야 합니다.
구현을 쉽게 하기위해 표준 라이브러리에서 콜렉션 인터페이스 골격 구현을 사용하시기 바랍니다: [`AbstractCollection`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-abstract-collection/index.html), [`AbstractList`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-abstract-list/index.html), [`AbstractSet`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-abstract-set/index.html), [`AbstractMap`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-abstract-map/index.html) 그리고 가변성 구현도 포함됩니다.

다른 콜렉션 동작은 확장 함수로 선언 됩니다. 이것은 필터, 변환, 우선순위 등이 있습니다.

## 공통 동작 (Common operations)

[read-only and mutable collections](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/kotlin-kotlin-collections-overview#collection-types) 모두에 대해 공통 동작을 사용할 수 있습니다. 공통 동작은 아래에 속합니다:

* [Transformations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/collection-transformations)
* [Filtering](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/filtering)
* [`plus` and `minus` operators](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/plus-minus-plus-and-minus-operators)
* [Grouping](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/untitled)
* [Retrieving collection parts](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/retrieving-collection-parts)
* [Retrieving single elements](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/retrieving-single-elements)
* [Ordering](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/collection-ordering)
* [Aggregate operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/collection-aggregate-operations)

이 페이지에 설명 된 동작은 원본 콜렉션에 영향을 주지 않고 결과를 반환합니다. 예를 들어 필터링 조작은 일치하는 요소들의 _new collection_ 을 생성합니다. 이러한 동작의 결과는 변수에 저장되거나 다른 방법으로 전달됩니다 (예: 다른 함수에 전달).

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")  
    numbers.filter { it.length > 3 }  // nothing happens with `numbers`, result is lost
    println("numbers are still $numbers")
    val longerThan3 = numbers.filter { it.length > 3 } // result is stored in `longerThan3`
    println("numbers longer than 3 chars are $longerThan3")
//sampleEnd
}

```

콜렉션 동작은 특정 _대상 (destination)_ 객체를 지정하는 옵션이 있습니다.
대상은 새로운 객체로 결과를 반환하는 대신에 아이템을 추가할 수 있는 변환 가능한 콜렉션입니다.
대상에 대한 동작은 함수 이름에 `To` 접미사를 붙입니다. 예를 들어 [`filter()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter.html) 대신에 [`filterTo()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-to.html) 또는 [`associate()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/associate.html) 대신에 [`associateTo()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/associate-to.html)가 있습니다.
이러한 함수는 도착지 콜렉션을 파라미터로 받습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    val filterResults = mutableListOf<String>()  //destination object
    numbers.filterTo(filterResults) { it.length > 3 }
    numbers.filterIndexedTo(filterResults) { index, _ -> index == 0 }
    println(filterResults) // contains results of both operations
//sampleEnd
}
```

편의상 이러한 함수는 대상 콜렉션을 다시 반환하므로 함수 호출의 해당 인자에서 바로 만들 수 있습니다:

```kotlin
fun main() {
    val numbers = listOf("one", "two", "three", "four")
//sampleStart
    // filter numbers right into a new hash set, 
    // thus eliminating duplicates in the result
    val result = numbers.mapTo(HashSet()) { it.length }
    println("distinct item lengths are $result")
//sampleEnd
}
```

대상이 있는 함수는 필터링, 연결, 그룹화, 병합 등 기타 작업을 사용할 수 있습니다. 자세한 내용은 [Kotlin collections reference](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html) 을 참고 바랍니다.

## 쓰기 동작 (Write operations)

변경 가능한 콜렉션의 경우 콜렉션 상태를 변경하는 _쓰기 동작 (write operations)_ 도 있습니다. 이러한 동작은 요소의 추가, 제거 그리고 업데이트 동작을 포함합니다. 쓰기 동작은 [Write operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/collection-write-operations)와 [List specific operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/list-specific-operations#list-write-operations) 와 [Map specific operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/map-specific-operations#map-write-operations) 섹션에 나와 있습니다.

특정 동작에서 동일한 작업을 수행하는 함수 쌍이 있습니다: 하나는 해당 콜렉션에 결과가 반영이 되고 다른 하나는 새로운 콜렉션에 결과가 반영 됩니다.
예를 들어 변경 가능한 콜렉션에 바로 정렬하는 [`sort()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort.html)는 상태를 변경합니다; [`sorted()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sorted.html)은 동일한 요소를 정렬 된 순서로 새 콜렉션을 생성합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four")
    val sortedNumbers = numbers.sorted()
    println(numbers == sortedNumbers)  // false
    numbers.sort()
    println(numbers == sortedNumbers)  // true
//sampleEnd
}
```