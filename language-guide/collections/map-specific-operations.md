# 맵 동작 \(Map Specific Operations\)

[maps](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/kotlin-kotlin-collections-overview#map)에서 키와 값의 타입은 사용자가 정의합니다. 맵 항목에 대한 키 기반 접근은 키로 값을 얻는 것부터 키와 값을 필터링하는 것까지 다양한 기능을 제공합니다. 이 페이지에서 표준 라이브러리의 맵 처리 함수에 대해 설명합니다.

## 키와 값 추출 \(Retrieving keys and values\)

맵에서 값을 추출하려면 [`get()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/get.html) 함수 인자로 키를 제공해야 합니다. 짧게 `[key]` 표현도 제공합니다. 주어진 키가 없을 경우 `null`을 반환합니다. [`getValue()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-value.html) 함수도 있으며 키가 없을 경우 예외를 발생하는 약간의 차이가 있습니다. 추가적으로 키가 없을경우에 대비해 두개의 함수가 더 있습니다:

* [`getOrElse()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-else.html)은 리스트와 동일하게 동작합니다: 키가 없을 경우 람다 함수로부터 반환합니다.
* [`getOrDefault()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-default.html)은 키가 없을 경우 특정 기본값을 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap.get("one"))
    println(numbersMap["one"])
    println(numbersMap.getOrDefault("four", 10))
    println(numbersMap["five"])               // null
    //numbersMap.getValue("six")      // exception!
//sampleEnd
}
```

맵에서 모든 키 또는 모든 값에 대해 동작을 수행하려면 `keys` 와 `values` 프로퍼티로 추출할 수 있습니다. `keys`는 맵에 모든 키 값을 `values`는 맵에 모든 값의 콜렉션 입니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap.keys)
    println(numbersMap.values)
//sampleEnd
}
```

## 필터링 \(Filtering\)

다른 콜렉션과 같이 [`filter()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter.html) 함수를 이용하여 맵 [filter](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/filtering)를 할 수 있습니다. 맵에서 `filter()`를 호출할 때 인자로 `Pair`를 조건에 전달합니다. 이를 통해 키와 값을 모두 조건에서 사용할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
    println(filteredMap)
//sampleEnd
}
```

맵을 필터링 하는 두개의 다른 방법이 있습니다: 키 기반 필터 와 값 기반 필터입니다. 이것의 함수는 [`filterKeys()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-keys.html) 와 [`filterValues()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-values.html) 입니다. 두 함수 모두 조건에 맞는 새로운 맵을 반환합니다. `filterKeys()`의 조건은 오직 요소의 키만 `filterValues()`의 조건은 오직 요소의 값만 체크합니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    val filteredKeysMap = numbersMap.filterKeys { it.endsWith("1") }
    val filteredValuesMap = numbersMap.filterValues { it < 10 }

    println(filteredKeysMap)
    println(filteredValuesMap)
//sampleEnd
}
```

## `plus` 와 `minus` \(`plus` and `minus` operators\)

요소의 키 접근으로 인해 [`plus`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus.html) \(`+`\) 와 [`minus`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus.html) \(`-`\) 동작은 맵에서 다른 콜렉션과 다르게 동작합니다. `plus`는 왼편에 `Map`에 오른편에 `Pair` 또는 다른 `Map`을 포함하는 하나의 `Map`을 반환합니다. 오른편에 있는 피연산자가 왼편에 있는 `Map`에 있는 키를 포함할 경우 해당 값은 더해져 반환됩니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap + Pair("four", 4))
    println(numbersMap + Pair("one", 10))
    println(numbersMap + mapOf("five" to 5, "one" to 11))
//sampleEnd
}
```

`minus`는 오른편에 있는 피연산자에 있는 키를 왼편에 있는 `Map`에서 제외하고 하나의 `Map`을 생성합니다. 그래서 오른편에는 하나의 키 또는 여러개의 키를 가진 콜렉션 \(리스트, 셋, 등\)이 올 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap - "one")
    println(numbersMap - listOf("two", "four"))
//sampleEnd
}
```

변경가능한 맵에서 [`plusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus-assign.html) \(`+=`\) 와 [`minusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus-assign.html) \(`-=`\) 연산의 사용방법은 아래의 [Map write operations](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/map-specific-operations#map-write-operations)를 참고 바랍니다.

## 맵 쓰기 동작 \(Map write operations\)

[Mutable](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/kotlin-kotlin-collections-overview#collection-types) 맵은 맵에 특화된 쓰기 동작을 제공합니다. 이러한 동작은 맵에 키 기반 접근으로 값을 변경할 수 있습니다.

맵에서의 쓰기 동작은 아래의 규칙을 포함합니다:

* 값은 업데이트 될 수 있습니다. 반대로 키는 절대 변경되지 않습니다: Values can be updated. In turn, keys never change: 맵에 요소를 추가하면 그 키는 변경되지 않습니다.
* 각 키는 항상 하나의 값이 연결되어 있습니다. 전체 항목을 추가하고 제거할 수 있습니다.

아래는 변경 가능한 맵에서 사용 가능한 쓰기 작업을 위한 표준 라이브러리 함수에 대한 설명입니다.

### 항목 추가와 업데이트 \(Adding and updating entries\)

변경 가능한 맵에 새로운 키-값 쌍을 추가하려면 [`put()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-map/put.html)을 사용합니다. `LinkedHashMap` \(기본 맵 구현\)에 새로운 항목을 추가할 때 맵의 가장 마지막 순서에 추가됩니다. 정렬된 맵에서 새로운 요소의 위치는 키 순서에 따라 정의됩니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    numbersMap.put("three", 3)
    println(numbersMap)
//sampleEnd
}
```

한번에 여러 항목을 추가하려면 [`putAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/put-all.html)을 사용합니다. 이 함수의 인자는 `Map` 또는 `Pair`의 그룹 \(`Iterable`, `Sequence`, `Array`\)이 올 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap.putAll(setOf("four" to 4, "five" to 5))
    println(numbersMap)
//sampleEnd
}
```

`put()` 와 `putAll()` 둘다 맵에 이미 키가 존재할 경우 값을 해당 값으로 덮어 쓰게 됩니다. 따라서 이를 이용하여 값을 업데이트 할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    val previousValue = numbersMap.put("one", 11)
    println("value associated with 'one', before: $previousValue, after: ${numbersMap["one"]}")
    println(numbersMap)
//sampleEnd
}
```

맵에 새로운 항목을 추가할 때 짧은 표현으로도 사용이 가능하며 두가지 방법이 있습니다:

* [`plusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus-assign.html) \(`+=`\) 동작.
* `put()`에 대한 연산 alias: `[]`.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    numbersMap["three"] = 3     // calls numbersMap.put("three", 3)
    numbersMap += mapOf("four" to 4, "five" to 5)
    println(numbersMap)
//sampleEnd
}
```

맵에 있는 키를 이용하여 호출할 경우 연산자는 해당 항목의 값을 덮어 씁니다.

### 항목 삭제 \(Removing entries\)

변경 가능한 맵에 항목을 삭제하려면 [`remove()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-map/remove.html) 함수를 사용합니다. `remove()`를 호출할 때 키 또는 키-값 쌍을 전달해야 합니다. 키-값 쌍으로 삭제를 호출할 경우 이 키가 있는 요소는 해당 값이 일치해야 삭제됩니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap.remove("one")
    println(numbersMap)
    numbersMap.remove("three", 4)            //doesn't remove anything
    println(numbersMap)
//sampleEnd
}
```

키 또는 값으로 변경 가능한 맵의 항목을 삭제할 수 있습니다. 이를 위해서는 항목의 키 또는 값을 제공하는 맵의 키 또는 값에서 `remove()`를 호출해야 합니다. 값에서 호출 될 때 `remove()`는 주어진 값을 가진 첫 항목만 삭제합니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3, "threeAgain" to 3)
    numbersMap.keys.remove("one")
    println(numbersMap)
    numbersMap.values.remove(3)
    println(numbersMap)
//sampleEnd
}
```

변경 가능한 맵에서 [`minusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus-assign.html) \(`-=`\) 동작도 사용 가능합니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap -= "two"
    println(numbersMap)
    numbersMap -= "five"             //doesn't remove anything
    println(numbersMap)
//sampleEnd
}
```

