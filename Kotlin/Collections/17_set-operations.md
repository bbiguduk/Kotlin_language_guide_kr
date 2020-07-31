# 셋 동작 (Set Specific Operations)

Kotlin 콜렉션 패키지는 셋 동작을 위한 확장 함수를 포함합니다: 각각의 콜렉션에 같은 부분 찾기, 병합 또는 빼기가 포함됩니다.

두개의 콜렉션을 하나의 콜렉션으로 병합하려면 [`union()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/union.html) 함수를 사용합니다. infix 형태인 `a union b`로도 사용할 수 있습니다.
정렬된 콜렉션의 경우 피연산자의 순서가 중요합니다: 결과 콜렉션에서 첫번째 피연산자 요소가 두번째 피연산자 요소보다 우선입니다.

두 콜렉션에 존재하는 같은 요소를 찾으려면 [`intersect()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/intersect.html)을 사용합니다.
다른 콜렉션에 없는 요소를 찾으려면 [`subtract()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/subtract.html)을 사용합니다.
두 함수 모두 infix 형태로 호출할 수 있습니다. 예를 들어 `a intersect b`.

```kotlin
fun main() {
//sampleStart
    val numbers = setOf("one", "two", "three")

    println(numbers union setOf("four", "five"))
    println(setOf("four", "five") union numbers)

    println(numbers intersect setOf("two", "one"))
    println(numbers subtract setOf("three", "four"))
    println(numbers subtract setOf("four", "three")) // same output
//sampleEnd
}
```

셋 동작은 `List`도 제공합니다.
그러나 리스트에 대한 셋 동작결과는 `Set` 이므로 중복된 요소는 모두 삭제됩니다.