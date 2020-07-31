# 반복자 \(Iterators\)

콜렉션 요소를 탐색하기 위해 Kotlin 표준 라이브러리는 일반적으로 사용되는 _iterators_ 매커니즘을 지원합니다 - 객체는 콜렉션의 구조를 노출시키지 않고 요소에 순차적으로 접근할 수 있습니다. 반복자는 콜렉션 요소 하나하나 작업 진행이 필요할 때 유용합니다.

반복자는 [`iterator()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-iterable/iterator.html) 함수 호출을 통해 `Set` 과 `List`를 포함하여 [`Iterable<T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-iterable/) 인터페이스의 상속자를 얻을 수 있습니다. 반복자를 얻으면 콜렉션의 첫번째 요소를 가리킵니다; [`next()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-iterator/next.html) 함수를 호출하면 다음 요소가 존재한다면 다음 위치의 요소를 반환합니다. 반복자가 마지막 요소를 전달하면 더이상 요소를 검색하는데 사용할 수 없습니다. 또한 이전 위치로 재설정 할 수도 없습니다. 콜렉션을 다시 반복하려면 새로운 반복자를 생성해야 합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    val numbersIterator = numbers.iterator()
    while (numbersIterator.hasNext()) {
        println(numbersIterator.next())
    }
//sampleEnd
}
```

`Iterable` 콜렉션을 통한 다른 방법은 잘 알려진 `for` 루프입니다. 콜렉션에서 `for`를 사용할 때 암묵적인 반복자를 얻을 수 있습니다. 그래서 아래의 예제는 위의 예제와 같습니다:

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    for (item in numbers) {
        println(item)
    }
//sampleEnd
}
```

마지막으로 콜렉션을 자동으로 반복하고 요소에 대해 주어진 코드를 실행할 수 있는 `forEach()` 함수가 있습니다. 그래서 위의 예제들을 아래와 같이 표현이 가능합니다:

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    numbers.forEach {
        println(it)
    }
//sampleEnd
}
```

## List 반복자 \(List iterators\)

List는 특별한 반복자 구현이 있습니다: [`ListIterator`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list-iterator/index.html). 이것은 List 반복의 앞뒤 양쪽 방향을 모두 지원합니다. 뒤 반복은 [`hasPrevious()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list-iterator/has-previous.html) 와 [`previous()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list-iterator/previous.html) 함수로 구현되어 있습니다. 추가적으로 `ListIterator`은 [`nextIndex()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list-iterator/next-index.html) 와 [`previousIndex()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list-iterator/previous-index.html) 함수로 요소의 인덱스 정보도 제공합니다.

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    val listIterator = numbers.listIterator()
    while (listIterator.hasNext()) listIterator.next()
    println("Iterating backwards:")
    while (listIterator.hasPrevious()) {
        print("Index: ${listIterator.previousIndex()}")
        println(", value: ${listIterator.previous()}")
    }
//sampleEnd
}
```

양방향을 모두 지원한다는 말은 `ListIterator`이 가장 마지막 요소에 도달해도 계속 사용할 수 있다는 의미입니다.

## 변경 가능한 반복자 \(Mutable iterators\)

변경가능한 콜렉션을 반복하기 위해 [`remove()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-iterator/remove.html) 함수로 요소를 삭제할 수 있는 `Iterator`를 확장한 [`MutableIterator`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-iterator/index.html)가 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four") 
    val mutableIterator = numbers.iterator()

    mutableIterator.next()
    mutableIterator.remove()    
    println("After removal: $numbers")
//sampleEnd
}
```

요소를 삭제하는 것 외에도 [`MutableListIterator`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list-iterator/index.html)은 요소를 추가하거나 대체할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "four", "four") 
    val mutableListIterator = numbers.listIterator()

    mutableListIterator.next()
    mutableListIterator.add("two")
    mutableListIterator.next()
    mutableListIterator.set("three")   
    println(numbers)
//sampleEnd
}
```

