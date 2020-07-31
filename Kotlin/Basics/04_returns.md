# 반환과 점프 (Returns and Jumps)

Kotlin은 세개의 점프 표현이 있습니다:

* *return*. 인접한 함수 또는 [anonymous function](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#anonymous-functions)로 부터 가장 기본적인 반환.
* *break*. 인접한 루프를 종료.
* *continue*. 인접한 루프의 다음 스텝 진행.

모든 표현은 더 큰 표현의 일부로 사용이 가능합니다:

```kotlin
val s = person.name ?: return
```

이 타입의 표현은 [Nothing type](https://kotlinlang.org/docs/reference/exceptions.html#the-nothing-type) 입니다.

## break와 continue 라벨 (Break and Continue Labels)

Kotlin에서는 어떠한 표현도 *label*로 표시될 수 있습니다.
라벨은 `@`를 추가하여 표시합니다. 예: `abc@`, `fooBar@` (자세한 내용은 [grammar](https://kotlinlang.org/docs/reference/grammar.html#label) 참고).
라벨 표현은 앞에 추가해주면 됩니다.

```kotlin
loop@ for (i in 1..100) {
    // ...
}
```

*break* 또는 *continue*에 라벨을 추가할 수 있습니다:

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

라벨이 붙여진 *break*는 라벨 포인트 바로 다음부터 실행 됩니다.
*continue*는 루프의 다음 반복으로 실행 됩니다.


## 라벨에서의 반환 (Return at Labels)

Kotlin에서는 중첩 된 함수 리터럴, 로컬 함수, 객체 표현식, 함수를 사용할 수 있습니다.
*return*을 이용해서 외부 함수에 값을 반환할 수 있습니다.
가장 중요한 부분은 람다 표현식에서 반환하는 것입니다. 아래와 같이 사용합니다:

```kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // non-local return directly to the caller of foo()
        print(it)
    }
    println("this point is unreachable")
}
//sampleEnd

fun main() {
    foo()
}
```

*return* 표현은 가장 인접한 함수에서 반환합니다. 위의 예에서 `foo`.
(이러한 로컬 반환은 오직 인라인 함수에서 전달 된 람다 표현식에서만 사용 가능합니다.)
람다 표현식에서 반환하려면 라벨이 필요하며, 아래와 같은 *return*이 있어야 합니다:

```kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with explicit label")
}
//sampleEnd

fun main() {
    foo()
}
```

이제, 람다 표현식에서만 반환 합니다. 종종 암시적 라벨이 더 편할 수 있습니다:
이런 경우 람다가 전달하는 함수는 라벨과 같은 이름을 갖습니다.

```kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // local return to the caller of the lambda, i.e. the forEach loop
        print(it)
    }
    print(" done with implicit label")
}
//sampleEnd

fun main() {
    foo()
}
```

또 다른 방법으로, 람다 표현식을 [anonymous function](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/higher-order-functions-and-lambdas#anonymous-functions)으로 대체 가능합니다.
익명 함수의 *return* 구문은 익명 함수 자체에서 반환합니다.

```kotlin
//sampleStart
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // local return to the caller of the anonymous fun, i.e. the forEach loop
        print(value)
    })
    print(" done with anonymous function")
}
//sampleEnd

fun main() {
    foo()
}
```

앞의 세 가지 예에서 로컬 반환을 사용하는 것은 일반 루프에서 *continue*와 유사하게 동작합니다. *break*와 직접적으로 동일한 것은 없지만 다른 중첩 람다를 추가하고 로컬에서 반환하지 않으면 시뮬레이트 가능합니다:

```kotlin
//sampleStart
fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}
//sampleEnd

fun main() {
    foo()
}
```

값을 반환 할 때, 파서는 반환 값에 우선순위를 줍니다. 예를 들어

```kotlin
return@a 1
```

이는 "라벨 식 `(@a 1)`을 반환" 하는게 아니라 "라벨 `@a`에 1을 반환" 하라는 의미입니다.