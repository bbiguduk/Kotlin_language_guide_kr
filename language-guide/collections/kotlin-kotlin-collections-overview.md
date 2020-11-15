# Kotlin 콜렉션 개요 \(Kotlin Collections Overview\)

Kotlin 표준 라이브러리는 _콜렉션 \(collections\)_ 관리를 위한 포괄적인 관리 툴을 제공합니다.

콜렉션은 Java나 Python 콜렉션처럼 대부분 언어에서 기본 형태입니다. 만약에 다른 언어에서의 콜렉션에 대해 알고 있다면 이번 장은 넘어가고 바로 상세 내용을 참고하면 됩니다.

콜렉션은 일반적으로 같은 타입의 객체의 모음입니다. 콜렉션 안의 객체는 _요소 \(elements\)_ 또는 _아이템 \(items\)_ 라 부릅니다. 예를 들어 한 학과의 모든 학생들은 그들의 평균 나이를 계산하는 데 사용될 수 있는 컬렉션을 형성합니다. 아래의 콜렉션 타입은 Kotlin에서 사용됩니다:

* _리스트 \(List\)_ 는 위치를 반영한 정수인 인덱스로 요소에 접근할 수 있는 정렬된 콜렉션입니다. 요소는 중복 될 수 있습니다. 리스트의 예로는 문장이 있습니다: 문장은 단어들의 집합이고 순서가 중요하며 단어가 중복 될 수 있습니다.
* _집합 \(Set\)_ 은 고유한 요소의 콜렉션입니다. 반복되지 않는 그룹의 객체 그룹인 수학적 추상화를 반영합니다. 일반적으로 요소의 순서는 중요하지 않습니다. 예를 들어 알파벳 문자 모음이 있습니다.
* _맵 \(Map\)_ \(또는 _dictionary_\)은 키-값 쌍의 집합입니다. 키는 고유한 값이며 각 키는 하나의 값과 매칭됩니다. 값은 중복이 가능합니다. 맵은 직원 ID와 직원의 직급과 같은 객체 간의 논리적 연결을 저장하는데 유용합니다.

Kotlin을 사용하면 콜렉션에 저장된 객체의 타입과 독립적으로 콜렉션을 조작 할 수 있습니다. 다시 말해 `Int` 또는 사용자 정의 클래스와 같이 `String` 리스트에 `String`을 추가할 수 있습니다. 따라서 Kotlin 표준 라이브러리는 모든 타입의 콜렉션을 만들고 채우기 그리고 관리를 위해 제너릭 인터페이스, 클래스와 함수를 제공합니다.

콜렉션 인터페이스와 관련된 함수는 kotlin.collections 패키지안에 존재합니다

## 콜렉션 타입 \(Collection types\)

Kotlin 표준 라이브러리는 집합 \(set\), 리스트 \(list\), 맵 \(map\)의 기본 콜렉션 타입을 위한 선언을 제공합니다. 한 쌍의 인터페이스는 각 콜렉션 타입을 나타냅니다:

* 콜렉션 요소에 접근하기 위한 _읽기-전용 \(read-only\)_ 인터페이스.
* 요소 추가, 삭제, 업데이트 인 쓰기 작업을 위한 _변경 가능한 \(mutable\)_ 인터페이스.

변경 가능한 콜렉션을 변경한다고 해서 [`var`](../getting-started/basic-syntax.md#variables)일 필요가 없습니다: 쓰기 작업은 동일한 변경 가능한 콜렉션 객체를 수정하므로 참조가 변경되지 않습니다. `val` 콜렉션을 다시 할당하려고 하면 컴파일 오류가 발생합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four")
    numbers.add("five")   // this is OK    
    //numbers = mutableListOf("six", "seven")      // compilation error
//sampleEnd
}
```

읽기 전용 콜렉션 타입은 [covariant](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/generics#variance) 입니다. 이것은 `Rectangle` 클래스가 `Shape` 클래스를 상속한다면 `List<Shape>`이 필요한 어디에서든 `List<Rectangle>`을 사용할 수 있습니다. 콜렉션 타입은 요소 타입과 동일한 하위 타입 관계를 가집니다. 맵은 값 타입에 대해 공변이지만 키 타입에는 없습니다.

결론적으로 변경 가능한 콜렉션은 공변이 아닙니다. 그렇지 않으면 런타임 오류가 발생합니다. `MutableList<Rectangle>`이 `MutableList<Shape>`의 하위 타입이면 `Shape`을 상속 \(예를 들어 `Circle`\)받은 다른 타입을 넣어 `Rectangle` 타입 인자를 위반할 수 있습니다.

아래는 Kotlin 콜렉션 인터페이스의 다이어그램을 나타냅니다:

![Collections Diagram](../../.gitbook/assets/collections-diagram.png)

인터페이스와 구현을 살펴 보도록 하겠습니다.

### 콜렉션 \(Collection\)

[`Collection<T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-collection/)은 콜렉션 계층의 최상위 입니다. 이 인터페이스는 읽기 전용 콜렉션의 크기 검색, 아이템 체크 등 일반적인 동작을 나타냅니다. `Collection`은 요소 반복 작업을 정의하는 `Iterable<T>` 인터페이스를 상속 합니다. 다른 콜렉션 타입에 적용 되는 함수의 파라미터로 `Collection`을 사용할 수 있습니다. 더 다양한 사용은 `Collection`의 상속자를 사용 하시기 바랍니다: [`List`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/) 와 [`Set`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-set/).

```kotlin
fun printAll(strings: Collection<String>) {
        for(s in strings) print("$s ")
        println()
    }

fun main() {
    val stringList = listOf("one", "two", "one")
    printAll(stringList)

    val stringSet = setOf("one", "two", "three")
    printAll(stringSet)
}
```

[`MutableCollection`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-collection/)은 `add` 와 `remove` 같은 쓰기 작업이 있는 `Collection` 입니다.

```kotlin
fun List<String>.getShortWordsTo(shortWords: MutableList<String>, maxLength: Int) {
    this.filterTo(shortWords) { it.length <= maxLength }
    // throwing away the articles
    val articles = setOf("a", "A", "an", "An", "the", "The")
    shortWords -= articles
}

fun main() {
    val words = "A long time ago in a galaxy far far away".split(" ")
    val shortWords = mutableListOf<String>()
    words.getShortWordsTo(shortWords, 3)
    println(shortWords)
}
```

### 리스트 \(List\)

[`List<T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/) 저장된 순서대로 요소를 지정하고 접근하기 위한 인덱스를 제공합니다. 인덱스는 첫번째 요소의 인덱스인 0에서 시작하여 `(list.size - 1)` 인 `lastIndex`로 되어 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    println("Number of elements: ${numbers.size}")
    println("Third element: ${numbers.get(2)}")
    println("Fourth element: ${numbers[3]}")
    println("Index of element \"two\" ${numbers.indexOf("two")}")
//sampleEnd
}
```

리스트 요소 \(null 포함\)는 중복될 수 있습니다: 리스트는 여러개의 객체 또는 하나의 객체를 포함할 수 있습니다. 2개의 리스트가 같은 크기와 같은 위치의 요소 \([구조적으로 같음 \(structurally equal\)](https://kotlinlang.org/docs/reference/equality.html#structural-equality)\)를 가지고 있으면 같다고 합니다.

```kotlin
data class Person(var name: String, var age: Int)

fun main() {
//sampleStart
    val bob = Person("Bob", 31)
    val people = listOf(Person("Adam", 20), bob, bob)
    val people2 = listOf(Person("Adam", 20), Person("Bob", 31), bob)
    println(people == people2)
    bob.age = 32
    println(people == people2)
//sampleEnd
}
```

[`MutableList`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/)은 예를 들어 특정 위치에 요소를 추가하거나 삭제하는 쓰기 작업을 할 수 있는 `List` 입니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    numbers.add(5)
    numbers.removeAt(1)
    numbers[0] = 0
    numbers.shuffle()
    println(numbers)
//sampleEnd
}
```

리스트는 배열과 유사합니다. 그러나 중요한 다른점이 있습니다: 배열의 크기는 초기화 할 때 정의되고 절대 바뀌지 않으나 리스트는 요소의 추가, 업데이트, 삭제의 쓰기 작업을 통해 크기가 변경될 수 있습니다.

Kotlin에서 `List`의 기본 구현은 크기가 유동적인 배열인 [`ArrayList`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-array-list/) 입니다.

### 집합 \(Set\)

[`Set<T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-set/)은 순서는 상관이 없는 고유한 요소를 가지고 있습니다. `null` 또한 고유한 요소이므로 오직 하나의 `null`만 가질 수 있습니다. 2개의 집합이 같다는 것은 크기가 같고 각 요소가 같을 때 해당합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = setOf(1, 2, 3, 4)
    println("Number of elements: ${numbers.size}")
    if (numbers.contains(1)) println("1 is in the set")

    val numbersBackwards = setOf(4, 3, 2, 1)
    println("The sets are equal: ${numbers == numbersBackwards}")
//sampleEnd
}
```

[`MutableSet`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-set/)은 `MutableCollection`으로 부터 쓰기가 가능한 `Set` 입니다.

`Set`의 기본 구현은 요소의 저장 순서를 유지하는 [`LinkedHashSet`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-linked-hash-set/) 입니다. 그러므로 `first()` 또는 `last()` 같이 순서와 관련 된 함수는 예측 가능한 결과값을 반환합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = setOf(1, 2, 3, 4)  // LinkedHashSet is the default implementation
    val numbersBackwards = setOf(4, 3, 2, 1)

    println(numbers.first() == numbersBackwards.first())
    println(numbers.first() == numbersBackwards.last())
//sampleEnd
}
```

[`HashSet`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-hash-set/)은 `Set`의 대체 구현이며 순서는 가지고 있지 않아 순서와 관련 된 함수 호출 시 예측이 불가능합니다. 그러나 `HashSet`은 같은 수의 요소를 가지고 있을 때 더 적은 메모리 사용량을 필요로 합니다.

### 맵 \(Map\)

[`Map<K, V>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/)은 `Collection` 인터페이스를 상속하지 않습니다. 그러나 Kotlin 콜렉션 타입입니다. `Map`은 _키-값 \(key-value\)_ \(또는 _엔트리 \(etries\)_\) 쌍으로 가지고 있습니다. 키는 고유하나 다른 키는 같은 값을 가지고 있을 수 있습니다. `Map` 인터페이스는 키로부터 값에 접근하거나 키와 값을 찾는 등의 특별한 함수를 제공합니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)

    println("All keys: ${numbersMap.keys}")
    println("All values: ${numbersMap.values}")
    if ("key2" in numbersMap) println("Value by key \"key2\": ${numbersMap["key2"]}")    
    if (1 in numbersMap.values) println("The value 1 is in the map")
    if (numbersMap.containsValue(1)) println("The value 1 is in the map") // same as previous
//sampleEnd
}
```

2개의 맵은 쌍의 순서와 상관없이 같다면 서로 같다고 볼 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)    
    val anotherMap = mapOf("key2" to 2, "key1" to 1, "key4" to 1, "key3" to 3)

    println("The maps are equal: ${numbersMap == anotherMap}")
//sampleEnd
}
```

[`MutableMap`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-map/)은 예를 들어 키-값을 추가 또는 이미 있는 키에 대한 값을 업데이트를 할 수 있는 쓰기가 가능한 `Map` 입니다.

```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    numbersMap.put("three", 3)
    numbersMap["one"] = 11

    println(numbersMap)
//sampleEnd
}
```

`Map`의 기본 구현은 요소의 순서를 유지하는 [`LinkedHashMap`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-linked-hash-map/) 입니다. 다른 구현은 요소의 순서를 유지하지 않는 [`HashMap`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-hash-map/)이 있습니다.

