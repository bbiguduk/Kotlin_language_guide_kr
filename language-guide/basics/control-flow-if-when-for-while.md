# 제어 흐름 \(Control Flow\): if, when, for, while

## If Expression

Kotlin에서 _if_는 표현식입니다. 즉, 어떠한 값을 반환한다는 의미입니다. _if_는 기본적으로 잘 수행이 되므로, 삼항 연산자 \(조건 ? 그러면 : 아니면 \(condition ? then : else\)\)가 필요없습니다.

```kotlin
// Traditional usage 
var max = a 
if (a < b) max = b

// With else 
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}

// As expression 
val max = if (a > b) a else b
```

_if_는 블럭을 가질 수 있으며, 블럭의 마지막 표현식이 반환값이 됩니다:

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

_if_를 문장보다 표현식으로 사용하는 경우 \(예를 들어, 값을 반환하거나 변수에 할당하는 경우\) 표현식은 `else`를 가지고 있어야 합니다.

자세한 내용은 [grammar for _if_{: .keyword }](https://kotlinlang.org/docs/reference/grammar.html#ifExpression) 참고바랍니다.

## When Expression

_when_은 C언어에서 switch 연산자를 대신 합니다. 아래 간단한 예문을 참고하시기 바랍니다.

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

_when_은 조건이 만족할 때 까지 모든 브랜치 \(케이스\)를 순차적으로 확인합니다. _when_은 표현식 또는 문장으로 사용 가능합니다. 표현식으로 사용하면 조건이 맞는 브랜치의 값은 전체 표현식의 값이 됩니다. 문장으로 사용되면 개별 브랜치는 무시됩니다. \(_if_ 처럼 각 브랜치는 블럭을 가질 수 있으며, 블럭의 마지막 표현식이 반환값이 됩니다.\)

_else_ 브랜치는 모든 조건을 만족하지 않을 경우 해당합니다. _when_을 표현석으로 사용할 경우 컴파일러가 모든 케이스가 존재함 \(예를 들어, [_enum_ class](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-enum-classes) 의 엔트리와 [_sealed_ class](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-enum-classes) 서브 타입\)을 증명할 수 없는 한 _else_ 브랜치는 반드시 포함되어야 합니다.

만약 많은 케이스를 같은 방식으로 처리 된다면, 각각의 조건들을 콤마로 연결할 수 있습니다:

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

조건 브랜치는 상수 뿐만 아니라 표현식으로도 사용이 가능합니다.

```kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

[range](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/ranges-and-progressions-1) 또는 콜렉션에서 범위 조건으로 _in_ 또는 _!in_을 사용할 수 있습니다:

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

_is_ 또는 _!is_을 이용하여 특정 타입에 대한 체크가 가능합니다. [smart casts](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts)로 인해 별도의 체크 없이 메서드나 프로퍼티에 접근할 수 있습니다.

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

_when_은 _if_-_else if_ 체인으로 대체 가능합니다. 만약에 인수를 제공하지 않는다면, 브랜치 조건은 단순히 불린 표현이며, 브랜치는 조건이 참 \(true\) 일 경우에만 실행됩니다.

```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

Kotlin 1.3 부터 _when_의 subject를 변수로 캡쳐할 수 있습니다:

```kotlin
fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
```

_when_의 subject를 변수로 캡쳐 시 해당 변수는 _when_의 바디에서만 유효합니다.

자세한 내용은 [grammar for _when_](https://kotlinlang.org/docs/reference/grammar.html#whenExpression)를 참고 바랍니다.

## For Loops

_for_ 루프는 반복자를 제공하는 모든 것을 반복합니다. 이것은 C\#에서 `foreach` 루프와 동일합니다. 구문은 아래와 같습니다:

```kotlin
for (item in collection) print(item)
```

바디는 블럭을 사용 할 수 있습니다.

```kotlin
for (item: Int in ints) {
    // ...
}
```

아까도 언급했듯이, _for_ 루프는 반복자를 제공하는 모든 것을 반복합니다. 즉,

* 멤버 함수 또는 확장 함수는 `iterator()` 가지며, 이 자체가 반환 타입
  * 멤버 함수 또는 확장 함수는 `next()` 가지며,
  * 멤버 함수 또는 확장 함수는 `hasNext()` 가지며, `Boolean`을 반환합니다.

위 세가지 함수는 `operator`로 표시해야 합니다.

숫자 범위를 반복하려면, [range expression](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/collections/ranges-and-progressions-1)을 사용하시기 바랍니다:

```kotlin
fun main() {
//sampleStart
    for (i in 1..3) {
        println(i)
    }
    for (i in 6 downTo 0 step 2) {
        println(i)
    }
//sampleEnd
}
```

범위 또는 배열에 대한 `for` 루프는 반복 객체를 생성하지 않고 인덱스 기반 루프로 컴파일 됩니다.

배열이나 인덱스가 있는 리스트를 반복하려면 아래와 같이 하시기 바랍니다:

```kotlin
fun main() {
val array = arrayOf("a", "b", "c")
//sampleStart
    for (i in array.indices) {
        println(array[i])
    }
//sampleEnd
}
```

또다른 방법으로는 `withIndex` 함수를 이용하시기 바랍니다:

```kotlin
fun main() {
    val array = arrayOf("a", "b", "c")
//sampleStart
    for ((index, value) in array.withIndex()) {
        println("the element at $index is $value")
    }
//sampleEnd
}
```

자세한 내용은 [grammar for _for_](https://kotlinlang.org/docs/reference/grammar.html#forStatement)를 참고바랍니다.

## While Loops

_while_과 _do_.._while_은 다른 언어처럼 동작합니다.

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```

자세한 내용은 [grammar for _while_](https://kotlinlang.org/docs/reference/grammar.html#whileStatement)를 참고바랍니다.

## 루프에서의 break와 continue \(Break and continue in loops\)

Kotlin은 다른언어와 동일하게 루프에서 _break_ 와 _continue_를 지원합니다. 자세한 내용은 [Returns and jumps](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/returns-and-jumps)를 참고바랍니다.

