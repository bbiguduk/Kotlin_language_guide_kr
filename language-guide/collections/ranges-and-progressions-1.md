# 범위와 진행 \(Ranges and Progressions\)

Kotlin은 `kotlin.ranges` 패키지에 있는 [`rangeTo()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/range-to.html) 함수를 사용하거나 `..` 수식어를 이용하여 값의 범위를 쉽게 생성할 수 있습니다. 일반적으로 `rangeTo()`는 `in` 또는 `!in` 함수로 보완합니다.

```kotlin
if (i in 1..4) {  // equivalent of 1 <= i && i <= 4
    print(i)
}
```

정수 타입의 범위 \([`IntRange`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/-int-range/index.html), [`LongRange`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/-long-range/index.html), [`CharRange`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/-char-range/index.html)\)는 반복할 수 있는 추가 기능이 있습니다. 이러한 범위는 정수 타입의 [progressions](https://en.wikipedia.org/wiki/Arithmetic_progression) 입니다. 이러한 범위는 일반적으로 `for` 루프 반복에 사용됩니다.

```kotlin
fun main() {
//sampleStart
    for (i in 1..4) print(i)
//sampleEnd
}
```

역순으로 숫자를 반복하기 위해선 `..` 대신 [`downTo`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/down-to.html) 함수를 사용합니다.

```kotlin
fun main() {
//sampleStart
    for (i in 4 downTo 1) print(i)
//sampleEnd
}
```

[`step`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/step.html) 함수를 사용하여 임의의 단계 \(1이 아닌\)로 반복하는 것도 가능합니다.

```kotlin
fun main() {
//sampleStart
    for (i in 1..8 step 2) print(i)
    println()
    for (i in 8 downTo 1 step 2) print(i)
//sampleEnd
}
```

[`until`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/until.html) 함수를 사용하면 범위의 마지막 요소는 포함하지 않고 반복할 수 있습니다:

```kotlin
fun main() {
//sampleStart
    for (i in 1 until 10) {       // i in [1, 10), 10 is excluded
        print(i)
    }
//sampleEnd
}
```

## 범위 \(Range\)

범위는 수학적 의미로 닫힌 간격으로 정의합니다: 범위에 포함 된 두 개의 끝점 값으로 정의합니다. 범위는 비교가능한 타입에 대해 정의됩니다: 순서가 있으면 임의의 인스턴스가 두 개의 지정된 인스턴스 사이의 범위에 있는지 여부를 정의할 수 있습니다. 범위에서 주요 연산자는 `contains` 이며 `in` 와 `!in` 연산자 형태로 사용됩니다.

class의 범위를 생성하려면 시작값에서 `rangeTo()` 함수를 호출하고 종료 값을 인자로 제공해야 합니다. `rangeTo()`는 `..` 형태 연산자로 호출되기도 합니다.

```kotlin
class Version(val major: Int, val minor: Int): Comparable<Version> {
    override fun compareTo(other: Version): Int {
        if (this.major != other.major) {
            return this.major - other.major
        }
        return this.minor - other.minor
    }
}

fun main() {
//sampleStart
    val versionRange = Version(1, 11)..Version(1, 30)
    println(Version(0, 9) in versionRange)
    println(Version(1, 20) in versionRange)
//sampleEnd
}
```

## 진행 \(Progression\)

위의 예와 같이 `Int`, `Long`, `Char` 와 같은 정수 타입의 범위는 [arithmetic progressions](https://en.wikipedia.org/wiki/Arithmetic_progression)으로 처리될 수 있습니다. Kotlin에서 진행은 특별한 타입으로 정의됩니다: [`IntProgression`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/-int-progression/index.html), [`LongProgression`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/-long-progression/index.html), [`CharProgression`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/-char-progression/index.html).

진행에서는 세 개의 프로퍼티가 있습니다: `first` 요소, `last` 요소, 0이 아닌 `step`. 첫 번째 요소는 `first`이고 다음 요소는 이전요소에 `step`을 더한 값입니다. 양수의 단계로 반복을 진행하는 것은 Java/JavaScript에서 인덱스 된 `for` 루프와 같습니다.

```java
for (int i = first; i <= last; i += step) {
  // ...
}
```

반복 범위로 만들어진 진행은 진행되는 `first` 와 `last` 요소는 범위는 양 끝 포인트이며 `step`은 1입니다.

```kotlin
fun main() {
//sampleStart
    for (i in 1..10) print(i)
//sampleEnd
}
```

커스텀한 step을 진행에서 정의할 때는 범위에 `step` 함수를 사용합니다.

```kotlin
fun main() {
//sampleStart
    for (i in 1..8 step 2) print(i)
//sampleEnd
}
```

진행에서 `last` 요소는 아래와 같이 계산됩니다:

* 양수의 step: 최대값은 최종값보다 크지 않으므로 `(last - first) % step == 0`.
* 음수의 step: 최소값은 최종값보다 작지 않으므로 `(last - first) % step == 0`.

따라서 `last` 요소는 항상 최종값과 같지 않습니다.

```kotlin
fun main() {
//sampleStart
    for (i in 1..9 step 3) print(i) // the last element is 7
//sampleEnd
}
```

역순으로 진행하는 반복을 만들기 위해선 `..` 대신에 `downTo`를 사용합니다.

```kotlin
fun main() {
//sampleStart
    for (i in 4 downTo 1) print(i)
//sampleEnd
}
```

진행은 `N` 이 `Int`, `Long`, `Char` 인 `Iterable<N>`을 구현합니다. 그래서 `map`, `filter`과 같은 [collection functions](https://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/collection-operations-overview)를 사용할 수 있습니다.

```kotlin
fun main() {
//sampleStart
    println((1..10).filter { it % 2 == 0 })
//sampleEnd
}
```

