# 취소와 타임아웃 \(Cancellation and Timeouts\)

이번 섹션에서는 코루틴의 취소와 타임아웃에 대해 다룹니다.

## 코루틴 실행 취소 \(Cancelling coroutine execution\)

오랜시간 실행되는 애플리케이션에서 백그라운드 코루틴에 대한 세밀한 제어가 필요할 수 있습니다. 예를 들어 사용자가 코루틴을 시작한 페이지를 닫았을 때 결과가 더이상 필요하지 않고 작업을 취소할 수 있습니다. [launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) 함수는 실행중인 코루틴을 취소하는데 사용할 수 있는 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)을 반환합니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = launch {
        repeat(1000) { i ->
            println("job: I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancel() // cancels the job
    job.join() // waits for job's completion 
    println("main: Now I can quit.")
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-01.kt)에서 전체 코드를 볼 수 있습니다.

실행결과 아래의 출력을 보입니다:

```text
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
main: Now I can quit.
```

메인이 `job.cancel`을 호출하자마자 다른 코루틴은 취소되었기 때문에 더이상 출력되지 않습니다. [cancel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html) 와 [join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html)이 결합된 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) 확장 함수 [cancelAndJoin](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel-and-join.html)도 있습니다.

## Cancellation is cooperative

코루틴 취소는 _협조적_ 입니다. 코루틴 코드는 취소가 가능하도록 협조적으로 작성해야 합니다. `kotlinx.coroutines`에 있는 모든 일시 중단 함수는 _취소 가능_ 합니다. 코루틴이 취소가능한 상태인지 체크하고 취소될 때 [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html) 발생시킵니다. 그러나 코루틴이 연산중이거나 취소가능 여부를 체크하지 않으면 취소할 수 없습니다. 아래의 예를 살펴봅시다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) { // computation loop, just wastes CPU
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-02.kt)에서 전체 코드를 볼 수 있습니다.

예제 코드 실행 시 코루틴은 취소 요청 후에도 "I'm sleeping"을 5번 반복하고 작업이 완료될 때까지 취소가 되지 않는 것을 확인할 수 있습니다.

## 연산 코드를 취소 가능하게 만들기 \(Making computation code cancellable\)

연산 코드가 취소가능하게 만드려면 두가지 방법이 있습니다. 첫번째는 취소를 확인하는 일시 중단 함수를 주기적으로 실행하는 것입니다. 이것에 좋은 선택은 [yield](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/yield.html) 함수입니다. 다른 하나는 취소 상태를 명시적으로 확인하는 것입니다. 후자의 방법을 시도해 봅시다.

위 예제의 `while (i < 5)`을 `while (isActive)`으로 변경하고 다시 실행해 봅시다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val startTime = System.currentTimeMillis()
    val job = launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (isActive) { // cancellable computation loop
            // print a message twice a second
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("job: I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-03.kt)에서 전체 코드를 볼 수 있습니다.

실행시켜보면 이제 이 루프는 취소됩니다. [isActive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/is-active.html)는 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) 객체를 통해 코루틴 안에서 사용 가능한 확장 프로퍼티 입니다.

## `finally`로 리소스 닫기 \(Closing resources with `finally`\)

취소가능한 일시 중단 함수는 일반적인 방법으로 취소 시 [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html)을 발생합니다. 예를 들어 `try {...} finally {...}` 표현과 Kotlin `use` 함수는 코루틴이 취소되면 정상적으로 종료 작업을 실행합니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = launch {
        try {
            repeat(1000) { i ->
                println("job: I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            println("job: I'm running finally")
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-04.kt)에서 전체 코드를 볼 수 있습니다.

[join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html) 와 [cancelAndJoin](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel-and-join.html)은 모든 액션이 종료될 때까지 기다립니다. 그래서 위의 예제의 출력은 아래와 같습니다:

```text
job: I'm sleeping 0 ...
job: I'm sleeping 1 ...
job: I'm sleeping 2 ...
main: I'm tired of waiting!
job: I'm running finally
main: Now I can quit.
```

## 취소 불가능한 블럭 실행하기 \(Run non-cancellable block\)

이 코드의 코루틴이 취소되었으므로 이전 예제에서 `finally` 블럭에서 일시 중단 함수를 사용하려고 하면 [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html)이 발생합니다. 파일을 닫거나, 작업을 취소하거나 모든 종류의 커뮤니케이션 채널을 닫는 작업은 일반적으로 차단되지 않으며 어떠한 일시 중단 함수와 관련이 없으므로 일반적으로 문제가 되지 않습니다. 그러나 취소된 코루틴에서 일시 중단이 필요한 경우 [withContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html) 함수와 [NonCancellable](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-non-cancellable.html) 컨텍스트를 이용하여 해당 코드를 `withContext(NonCancellable) {...}`으로 래핑할 수 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = launch {
        try {
            repeat(1000) { i ->
                println("job: I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            withContext(NonCancellable) {
                println("job: I'm running finally")
                delay(1000L)
                println("job: And I've just delayed for 1 sec because I'm non-cancellable")
            }
        }
    }
    delay(1300L) // delay a bit
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // cancels the job and waits for its completion
    println("main: Now I can quit.")
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-05.kt)에서 전체 코드를 볼 수 있습니다.

## 타임아웃 \(Timeout\)

코루틴 실행을 취소하는 가장 확실한 이유는 실행 시간이 타임아웃을 초과했기 때문입니다. 해당 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)에 대한 참조를 직접적으로 추적하고 지연 후에 추적한 작업에 대한 취소를 위해 별도의 코루틴을 실행하는 것이 가능하지만 [withTimeout](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout.html) 함수를 사용할 수 있습니다. 아래 예제를 참고해 봅시다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    withTimeout(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-06.kt)에서 전체 코드를 볼 수 있습니다.

출력문은 아래와 같습니다:

```text
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Exception in thread "main" kotlinx.coroutines.TimeoutCancellationException: Timed out waiting for 1300 ms
```

[withTimeout](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout.html)에서 발생된 `TimeoutCancellationException`은 [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html)의 하위 class 입니다. 콘솔에 스택 추적이 출력 된 것을 본 적이 없습니다. 취소된 코루틴 `CancellationException` 내부는 코루틴 완료의 일반적인 원인으로 간주하기 때문입니다. 그러나 이 예제에서 `main` 함수 내에서 `withTimeout`을 사용했습니다.

취소는 예외일 뿐이며 모든 리소스는 일반적인 방식으로 닫깁니다. 타임아웃에 대한 특별한 작업을 하려면 타임아웃 코드에 `try {...} catch (e: TimeoutCancellationException) {...}` 블럭을 래핑하거나 [withTimeout](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout.html)와 비슷하지만 예외를 발생하는 대신 `null`을 반환하는 [withTimeoutOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout-or-null.html) 함수를 사용하면 됩니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val result = withTimeoutOrNull(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
        "Done" // will get cancelled before it produces this result
    }
    println("Result is $result")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-07.kt)에서 전체 코드를 볼 수 있습니다.

더이상 예외가 발생하지 않습니다:

```text
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Result is null
```

## 비동기 타임아웃과 리소스 \(Asynchronous timeout and resources\)

[withTimeout](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout.html) 의 타임아웃 이벤트는 해당 블럭에서 실행중인 코드와 관련하여 비동기적이며 타임아웃 블럭 내에서 반환되기 직전에 언제든지 발생할 수 있습니다. 블럭의 외부에서 닫거나 해제해야 하는 블럭 내부의 리소스를 열거나 획득하는 경우 이를 염두에 두어야 합니다.

예를 들어 여기서는 획득한 카운터를 증가시키고 `close` 함수에서 이 카운터를 감소시켜 생성된 횟수를 추적하는 `Resource` 클래스를 사용하여 닫기 가능한 리소스를 모방합니다. 작은 타임아웃으로 많은 코루틴을 실행하면 약간의 지연 후 `withTimeout` 블럭 내에서 이 리소스를 획득하고 외부에서 해제해 보십시오.

```kotlin
import kotlinx.coroutines.*

//sampleStart
var acquired = 0

class Resource {
    init { acquired++ } // Acquire the resource
    fun close() { acquired-- } // Release the resource
}

fun main() {
    runBlocking {
        repeat(100_000) { // Launch 100K coroutines
            launch { 
                val resource = withTimeout(60) { // Timeout of 60 ms
                    delay(50) // Delay for 50 ms
                    Resource() // Acquire a resource and return it from withTimeout block     
                }
                resource.close() // Release the resource
            }
        }
    }
    // Outside of runBlocking all coroutines have completed
    println(acquired) // Print the number of resources still acquired
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-08.kt) 에서 전체 코드를 볼 수 있습니다.

위의 코드를 실행하면 항상 0을 출력하는 것은 아니지만 실제로 0이 아닌 값을 보려면 이 예제에서 타임아웃을 변경해야 할 수 있습니다.

> 여기서 100K 코루틴에서 `acquired` 카운터를 증가 및 감소시키는 것은 항상 동일한 메인 스레드에서 발생하기 때문에 완전히 안전합니다. 이에 대한 자세한 내용은 코루틴 컨텍스트에 대한 다음 장에서 설명합니다.

이 문제를 해결하려면 `withTimeout` 블럭에서 반환하는 것과 반대로 리소스에 대한 참조를 변수에 저장할 수 있습니다.

```kotlin
import kotlinx.coroutines.*

var acquired = 0

class Resource {
    init { acquired++ } // Acquire the resource
    fun close() { acquired-- } // Release the resource
}

fun main() {
//sampleStart
    runBlocking {
        repeat(100_000) { // Launch 100K coroutines
            launch { 
                var resource: Resource? = null // Not acquired yet
                try {
                    withTimeout(60) { // Timeout of 60 ms
                        delay(50) // Delay for 50 ms
                        resource = Resource() // Store a resource to the variable if acquired      
                    }
                    // We can do something else with the resource here
                } finally {  
                    resource?.close() // Release the resource if it was acquired
                }
            }
        }
    }
    // Outside of runBlocking all coroutines have completed
    println(acquired) // Print the number of resources still acquired
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-cancel-09.kt) 에서 전체 코드를 볼 수 있습니다.

이 예제는 항상 0을 출력합니다. 리소스는 누출되지 않습니다.

