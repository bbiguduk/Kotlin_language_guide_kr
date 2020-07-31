# 일시 중단 함수 구성 \(Composing Suspending Functions\)

이번 섹션에서는 일시 중단 함수의 구성에 대해 살펴봅시다.

## 기본적으로 순차적 \(Sequential by default\)

원격 서비스 호출이나 계산과 같은 유용한 일시 중단 함수가 있다고 가정합시다. 유용하다고 가정하지만 실질적으로 각각의 예제의 목적을 위해 잠시 지연됩니다:

```kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

먼저 `doSomethingUsefulOne`를 호출하고 그 다음으로 `doSomethingUsefulTwo`를 호출하여 결과의 합을 구하려면 어떻게 해야할까요? 첫번째 함수의 결과를 사용하여 두번째 함수를 호출해야하는지 또는 호출 방법을 결정 여부를 정합니다.

일반적인 코드와 마찬가지로 코루틴 코드는 기본적으로 _순차적_ 이기 때문에 순차적으로 호출합니다. 다음 예는 두개의 일시 중단 함수의 걸리는 시간을 측정하여 보여줍니다:

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

fun main() = runBlocking<Unit> {
//sampleStart
    val time = measureTimeMillis {
        val one = doSomethingUsefulOne()
        val two = doSomethingUsefulTwo()
        println("The answer is ${one + two}")
    }
    println("Completed in $time ms")
//sampleEnd    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-compose-01.kt)에서 전체 코드를 볼 수 있습니다.

실행결과는 아래와 같습니다:

```text
The answer is 42
Completed in 2017 ms
```

## 동시에 async 사용 \(Concurrent using async\)

`doSomethingUsefulOne` 와 `doSomethingUsefulTwo`에 종속 없이 _동시_ 에 수행함으로써 더 빠르게 결과를 얻으려면 어떻게 해야할까요? 이러한 해결은 [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html)가 도와줍니다.

개념적으로 [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html)는 [launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html)와 같습니다. 다른 모든 코루틴과 동시에 동작하는 가벼운 쓰레드인 별도의 코루틴을 시작합니다. 차이점은 `launch`는 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)을 반환하고 어떠한 결과 값도 제공하지 않는 반면에 `async`는 나중에 결과를 제공하겠다는 약속을 나타내는 가볍고 블락되지 않는 [Deferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/index.html)를 반환합니다. `.await()`를 사용하여 연기된 값을 얻을 수 있지만 `Deferred`는 `Job` 이므로 필요에 따라 취소할 수 있습니다.

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

fun main() = runBlocking<Unit> {
//sampleStart
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
//sampleEnd    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-compose-02.kt)에서 전체 코드를 볼 수 있습니다.

실행결과는 아래와 같습니다:

```text
The answer is 42
Completed in 1017 ms
```

두개의 코루틴이 동시에 실행되므로 2배 더 빨라졌습니다. 코루틴의 동시성은 항상 명시적입니다.

## 느리게 시작하는 async \(Lazily started async\)

선택적으로 [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html)는 `start` 파라미터에 [CoroutineStart.LAZY](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-start/-l-a-z-y.html) 설정을 통해 느리게 만들 수 있습니다. 이 설정은 [await](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/await.html)로 부터 결과를 요구하거나 `Job`의 [start](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/start.html) 함수를 실행할 때 코루틴을 시작합니다. 아래 예제를 살펴봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

fun main() = runBlocking<Unit> {
//sampleStart
    val time = measureTimeMillis {
        val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
        val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
        // some computation
        one.start() // start the first one
        two.start() // start the second one
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
//sampleEnd    
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-compose-03.kt)에서 전체 코드를 볼 수 있습니다.

실행결과 아래와 같습니다:

```text
The answer is 42
Completed in 1017 ms
```

두개의 코루틴은 정의되었지만 이전 예제처럼 실행되지는 않지만 [start](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/start.html)를 호출하여 실행할 수 있게 프로그래머에게 제어권이 부여됩니다. 먼저 `one` 시작되고 `two`가 시작한 다음 개별 코루틴이 종료되길 기다립니다.

각각의 코루틴에서 [start](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/start.html) 호출 없이 `println`에서 [await](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/await.html)를 호출한다면 순차적으로 동작할 것입니다. 이것은 [await](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/await.html)는 코루틴을 실행하고 코루틴이 끝나기를 기다리기 때문이며 이는 느리게 실행되는 우리의 의도와 다릅니다. `async(start = CoroutineStart.LAZY)`은 값 계산시 일시 중단 함수를 동반하는 경우 일반적인 `lazy` 함수를 대체합니다.

## async 형태의 함수 \(Async-style functions\)

[GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)를 명시적으로 참조하는 [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 코루틴 빌더를 사용하여 `doSomethingUsefulOne` 와 `doSomethingUsefulTwo` _비동기적_ 으로 실행하는 비동기 형태의 함수를 정의할 수 있습니다. 이러한 함수의 이름 뒤에 "...Async"를 붙여 비동기 계산만 시작하고 결과를 얻기 위해 지연된 값을 사용해야한다는 것을 강조합니다.

```kotlin
// The result type of somethingUsefulOneAsync is Deferred<Int>
fun somethingUsefulOneAsync() = GlobalScope.async {
    doSomethingUsefulOne()
}

// The result type of somethingUsefulTwoAsync is Deferred<Int>
fun somethingUsefulTwoAsync() = GlobalScope.async {
    doSomethingUsefulTwo()
}
```

`xxxAsync` 함수는 _일시 중단_ 함수가 **아닙니다**. 어디서나 사용 가능합니다. 그러나 이는 실행 코드와 함께 사용될 때 항상 동시다발 적으로 실행됩니다.

다음 예는 코루틴 밖에서 사용되는 것을 보여줍니다:

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

//sampleStart
// note that we don't have `runBlocking` to the right of `main` in this example
fun main() {
    val time = measureTimeMillis {
        // we can initiate async actions outside of a coroutine
        val one = somethingUsefulOneAsync()
        val two = somethingUsefulTwoAsync()
        // but waiting for a result must involve either suspending or blocking.
        // here we use `runBlocking { ... }` to block the main thread while waiting for the result
        runBlocking {
            println("The answer is ${one.await() + two.await()}")
        }
    }
    println("Completed in $time ms")
}
//sampleEnd

fun somethingUsefulOneAsync() = GlobalScope.async {
    doSomethingUsefulOne()
}

fun somethingUsefulTwoAsync() = GlobalScope.async {
    doSomethingUsefulTwo()
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-compose-04.kt)에서 전체 코드를 볼 수 있습니다.

> 비동기 함수가 있는 형태의 프로그래밍은 다른 언어에서도 유명한 형태이므로 여기서는 예제로만 제공합니다. Kotlin 코루틴에서 이러한 형태로 사용하는 것은 아래 설명을 토대로 **강하게 추천하지 않습니다.**

`val one = somethingUsefulOneAsync()` 라인과 `one.await()` 표현 사이에 논리 오류가 있고 예외를 발생시킨 후 프로그램이 중단되면 어떠한 일이 일어날지 생각해봅시다. 일반적으로 전역 에러 핸들러는 개발자에게 에러를 기록하고 리포트 할 수 있지만 그렇지 않으면 프로그램은 계속해서 다른 작업을 할 수 있습니다. 그러나 `somethingUsefulOneAsync`은 중지되었지만 여전히 백그라운드에서 실행 중입니다. 이러한 문제는 아래 섹션에서와 같이 구조적으로 동시에 발생하지 않습니다.

## 비동기를 이용한 구조적 동시성 \(Structured concurrency with async\)

Concurrent using async에서의 예제의 함수를 사용하여 `doSomethingUsefulOne` 와 `doSomethingUsefulTwo`을 동시에 수행하여 결과 값을 더하고 반환해봅시다. [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 코루틴 빌더는 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)의 확장으로 정의되기 때문에 [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) 함수가 제공하는 것을 포함해야 합니다:

```kotlin
suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}
```

`concurrentSum` 함수의 코드에 문제가 있어 예외를 발생하면 모든 실행된 코루틴은 취소됩니다.

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

fun main() = runBlocking<Unit> {
//sampleStart
    val time = measureTimeMillis {
        println("The answer is ${concurrentSum()}")
    }
    println("Completed in $time ms")
//sampleEnd    
}

suspend fun concurrentSum(): Int = coroutineScope {
    val one = async { doSomethingUsefulOne() }
    val two = async { doSomethingUsefulTwo() }
    one.await() + two.await()
}

suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // pretend we are doing something useful here
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // pretend we are doing something useful here, too
    return 29
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-compose-05.kt)에서 전체 코드를 볼 수 있습니다.

위의 `main` 함수의 출력에서 알 수 있듯이 두 작업을 동시에 실행할 수 있습니다:

```text
The answer is 42
Completed in 1017 ms
```

취소는 항상 코루틴의 계층 구조에 따라 전파됩니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
    try {
        failedConcurrentSum()
    } catch(e: ArithmeticException) {
        println("Computation failed with ArithmeticException")
    }
}

suspend fun failedConcurrentSum(): Int = coroutineScope {
    val one = async<Int> { 
        try {
            delay(Long.MAX_VALUE) // Emulates very long computation
            42
        } finally {
            println("First child was cancelled")
        }
    }
    val two = async<Int> { 
        println("Second child throws an exception")
        throw ArithmeticException()
    }
    one.await() + two.await()
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-compose-06.kt)에서 전체 코드를 볼 수 있습니다.

`two`가 실패하면 첫번째 `async`와 대기중인 부모가 모두 최소되는 것을 유의해야 합니다:

```text
Second child throws an exception
First child was cancelled
Computation failed with ArithmeticException
```

