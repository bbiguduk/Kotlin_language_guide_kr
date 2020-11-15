# 콜렉션 구성 \(Constructing Collections\)

## 요소로 구성 \(Constructing from elements\)

콜렉션을 생성하는 가장 일반적인 방법은 [`listOf<T>()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/list-of.html), [`setOf<T>()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/set-of.html), [`mutableListOf<T>()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/mutable-list-of.html), [`mutableSetOf<T>()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/mutable-set-of.html)인 표준 라이브러리 함수를 사용하는 것입니다. 콜렉션 요소를 콤마로 구분하여 제공할 경우 컴파일러는 요소의 타입을 자동으로 탐색합니다. 빈 콜렉션 생성시에는 반드시 타입을 명시적으로 표시해줘야 합니다.

```kotlin
val numbersSet = setOf("one", "two", "three", "four")
val emptySet = mutableSetOf<String>()
```

같은 형식으로 맵도 함수 [`mapOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-of.html) 와 [`mutableMapOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/mutable-map-of.html)을 이용하여 생성 가능합니다. 맵의 키와 값은 `Pair` 객체 \(일반적으로 `to` 중위 \(infix\) 함수로 생성\)로 전달됩니다.

```kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
```

`to` 표기법은 수명이 짧은 `Pair` 객체를 생성하므로 성능이 중요하지 않을 경우에 사용하는 것이 좋습니다. 과도한 메모리 사용을 피하려면 다른 방법을 사용해야 합니다. 예를 들어 변경 가능한 맵을 만들고 쓰기 작업을 통해 데이터를 채울 수 있습니다. [`apply()`](https://kotlinlang.org/docs/reference/scope-functions.html#apply) 함수는 이러한 상황에 유용합니다.

```kotlin
val numbersMap = mutableMapOf<String, String>().apply { this["one"] = "1"; this["two"] = "2" }
```

## 빈 콜렉션 \(Empty collections\)

[`emptyList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/empty-list.html), [`emptySet()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/empty-set.html), [`emptyMap()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/empty-map.html) 함수는 어떠한 요소도 없는 콜렉션을 만들 수 있습니다. 빈 콜렉션 생성 시 반드시 요소의 타입을 명시해 줘야 합니다.

```kotlin
val empty = emptyList<String>()
```

## 리스트 초기화 함수 \(Initializer functions for lists\)

리스트는 크기와 인덱스를 기반으로 요소 값을 정의하는 초기화 함수가 있습니다.

```kotlin
fun main() {
//sampleStart
    val doubled = List(3, { it * 2 })  // or MutableList if you want to change its content later
    println(doubled)
//sampleEnd
}
```

## 구체적 타입 생성자 \(Concrete type constructors\)

`ArrayList` 또는 `LinkedList`와 같은 구체적 타입 콜렉션을 생성하기 위해선 이러한 타입의 생성자를 사용할 수 있습니다. `Set` 과 `Map`의 구현과 비슷한 생성자가 있습니다.

```kotlin
val linkedList = LinkedList<String>(listOf("one", "two", "three"))
val presizedSet = HashSet<Int>(32)
```

## 복사 \(Copying\)

이미 존재하는 콜렉션과 같은 콜렉션을 생성하기 위해 복사 작업을 사용할 수 있습니다. 표준 라이브러리로 콜렉션 복사 작업은 같은 요소에 대한 참조로 _얕은 \(shallow\)_ 사본 콜렉션을 생성합니다. 따라서 콜렉션 요소는 모든 복사본에 반영됩니다.

[`toList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-list.html), [`toMutableList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-mutable-list.html), [`toSet()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-set.html)와 같은 콜렉션 복사 함수는 특정 순간의 콜렉션을 생성합니다. 결과는 동일한 요소의 새로운 콜렉션 입니다. 원본 콜렉션에 요소를 추가하거나 삭제를 해도 복사본에는 반영이 되지 않습니다. 사본은 원본과 독립적으로 변경 될 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val sourceList = mutableListOf(1, 2, 3)
    val copyList = sourceList.toMutableList()
    val readOnlyCopyList = sourceList.toList()
    sourceList.add(4)
    println("Copy size: ${copyList.size}")   

    //readOnlyCopyList.add(4)             // compilation error
    println("Read-only copy size: ${readOnlyCopyList.size}")
//sampleEnd
}
```

이러한 함수는 다른 타입의 콜렉션으로 변경할 수 있습니다. 예를 들어 리스트에서 집합으로 변경하거나 그 반대도 가능합니다.

```kotlin
fun main() {
//sampleStart
    val sourceList = mutableListOf(1, 2, 3)    
    val copySet = sourceList.toMutableSet()
    copySet.add(3)
    copySet.add(4)    
    println(copySet)
//sampleEnd
}
```

또한 동일한 콜렉션 인스턴스에 새로운 참조를 생성할 수 있습니다. 기존 콜렉션으로 콜렉션 변수를 초기화 하면 새로운 참조가 생성됩니다. 따라서 콜렉션 인스턴스가 참조를 통해 변경되면 모든 참조에 반영 됩니다.

```kotlin
fun main() {
//sampleStart
    val sourceList = mutableListOf(1, 2, 3)
    val referenceList = sourceList
    referenceList.add(4)
    println("Source size: ${sourceList.size}")
//sampleEnd
}
```

변경을 제한하기 위해 콜렉션 초기화를 사용할 수 있습니다. 예를 들어 `MutableList`에 대한 `List` 참조를 생성하고 그 참조를 통해 변경하려고 하면 컴파일러는 에러를 발생시킵니다.

```kotlin
fun main() {
//sampleStart 
    val sourceList = mutableListOf(1, 2, 3)
    val referenceList: List<Int> = sourceList
    //referenceList.add(4)            //compilation error
    sourceList.add(4)
    println(referenceList) // shows the current state of sourceList
//sampleEnd
}
```

## 다른 콜렉션에서 함수 호출 \(Invoking functions on other collections\)

다른 콜렉션에 다양한 작업을 통해 콜렉션을 생성할 수 있습니다. 예를 들어 [필터링 \(filtering\)](filtering.md) 은 조건에 맞는 요소의 리스트를 생성합니다:

```kotlin
fun main() {
//sampleStart 
    val numbers = listOf("one", "two", "three", "four")  
    val longerThan3 = numbers.filter { it.length > 3 }
    println(longerThan3)
//sampleEnd
}
```

[매핑 \(Mapping\) ](collection-transformations.md#mapping)은 변환 결과의 리스트를 생성합니다:

```kotlin
fun main() {
//sampleStart 
    val numbers = setOf(1, 2, 3)
    println(numbers.map { it * 3 })
    println(numbers.mapIndexed { idx, value -> value * idx })
//sampleEnd
}
```

[조합 \(Association\)](collection-transformations.md#association) 은 맵을 생성합니다:

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.associateWith { it.length })
//sampleEnd
}
```

Kotlin에서 콜렉션 작업의 자세한 내용은 [콜렉션 동작 개요 \(Collection Operations Overview\) ](collection-operations-overview.md)을 참고 바랍니다.

