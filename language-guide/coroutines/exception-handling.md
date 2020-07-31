# 예외 처리 \(Exception Handling\)

이번 섹션에서는 예외 처리와 예외 시 취소에 대해 다룰 예정입니다. 취소된 코루틴은 중지된 포인트에서 [CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellation-exception/index.html)을 발생하고 코루틴 동작에서 무시된다는 것을 알고 있습니다. 취소하는 동안 예외가 발생거나 동일한 코루틴의 여러 자식이 예외를 발생하는 경우에 어떠한 일이 생기는지 알아봅시다.

## 예외 전파 \(Exception propagation\)

코루틴 빌더는 자동으로 예외를 전파 \([launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) 와 [actor](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/actor.html)\) 또는 사용자에게 노출 \([async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 와 [produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html)\)하는 두가지의 특징이 있습니다. 이 빌더가 다른 코루틴의 _하위_ 가 아닌 _루트_ 코루틴을 생성하는데 사용되면 전자 빌더는 Java의 `Thread.uncaughtExceptionHandler`와 유사하게 포착되지 않은 예외로 처리하지만 후자는 예를 들어 [await](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/await.html) 또는 [receive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html) \([produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html) 와 [receive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html)는 [Channels](https://github.com/Kotlin/kotlinx.coroutines/blob/master/docs/channels.md) 섹션에서 다룹니다\) 통해 최종 예외를 소비하기 위해 사용자에 의존합니다.

[GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)를 사용하여 루트 코루틴을 생성하는 간단한 예제를 살펴봅시다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val job = GlobalScope.launch { // root coroutine with launch
        println("Throwing exception from launch")
        throw IndexOutOfBoundsException() // Will be printed to the console by Thread.defaultUncaughtExceptionHandler
    }
    job.join()
    println("Joined failed job")
    val deferred = GlobalScope.async { // root coroutine with async
        println("Throwing exception from async")
        throw ArithmeticException() // Nothing is printed, relying on user to call await
    }
    try {
        deferred.await()
        println("Unreached")
    } catch (e: ArithmeticException) {
        println("Caught ArithmeticException")
    }
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-exceptions-01.kt)에서 전체 코드를 볼 수 있습니다.

[debug](https://github.com/Kotlin/kotlinx.coroutines/blob/master/docs/coroutine-context-and-dispatchers.md#debugging-coroutines-and-threads)를 사용하여 출력된 결과는 아래와 같습니다:

```text
Throwing exception from launch
Exception in thread "DefaultDispatcher-worker-2 @coroutine#2" java.lang.IndexOutOfBoundsException
Joined failed job
Throwing exception from async
Caught ArithmeticException
```

## CoroutineExceptionHandler

**포착되지 않은** 예외를 콘솔에 출력하는 기본 동작을 커스터마이징 할 수 있습니다. _루트_ 코루틴에 [CoroutineExceptionHandler](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/index.html) 컨텍스트 요소는 이 루트 코루틴과 사용자 정의 예외 처리가 발생할 수 있는 모든 하위 요소에 대해 일반적인 `catch` 블럭을 사용할 수 있습니다. 이것은 \[`Thread.uncaughtExceptionHandler`\]\([https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html\#setUncaughtExceptionHandler\(java.lang.Thread.UncaughtExceptionHandler\)\)와](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#setUncaughtExceptionHandler%28java.lang.Thread.UncaughtExceptionHandler%29%29와) 비슷합니다. `CoroutineExceptionHandler`에서의 예외는 복구할 수 없습니다. 핸들러가 호출될 때 코루틴은 이미 해당 예외와 함께 완료되었습니다. 일반적으로 핸들러는 예외를 기록하고 약간의 에러 메세지를 보여주고 애플리케이션을 종료 및 재시작 하는데 사용합니다.

JVM에서는 [`ServiceLoader`](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html)를 통해 [CoroutineExceptionHandler](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/index.html)를 등록하고 모든 코루틴에 대한 전역 예외 핸들러를 재정의 하는 것이 가능합니다. 전역 예외 핸들러는 더이상 특정 핸들러가 등록되지 않을 때 사용되는 \[`Thread.defaultUncaughtExceptionHandler`\]\([https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html\#setDefaultUncaughtExceptionHandler\(java.lang.Thread.UncaughtExceptionHandler\)\)와](https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.html#setDefaultUncaughtExceptionHandler%28java.lang.Thread.UncaughtExceptionHandler%29%29와) 비슷합니다. Android에서 `uncaughtExceptionPreHandler`가 전역 코루틴 예외 핸들러로 있습니다.

`CoroutineExceptionHandler`는 다른 방법으로 처리되지 않는 예외에만 호출됩니다. 특히 다른 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)에 컨텍스트에서 생성된 코루틴인 모든 자식 코루틴은 예외 처리를 부모 코루틴에 위임하고 루트에 까지 위임되므로 컨텍스트에 있는 `CoroutineExceptionHandler`은 절대 사용되지 않습니다. 그 외에도 [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) 빌더는 항상 모든 예외를 캐치하고 결과 [Deferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/index.html) 객체에 표시하므로 `CoroutineExceptionHandler`는 아무런 영향이 없습니다.

> 슈퍼비전 스코프에서 실행되는 코루틴은 예외를 그들의 부모에게 전달하지 않으며 이 규칙에서 제외됩니다. 더 자세한 내용은 이 문서의 Supervision 섹션을 참고 바랍니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
    val job = GlobalScope.launch(handler) { // root coroutine, running in GlobalScope
        throw AssertionError()
    }
    val deferred = GlobalScope.async(handler) { // also root, but async instead of launch
        throw ArithmeticException() // Nothing will be printed, relying on user to call deferred.await()
    }
    joinAll(job, deferred)
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-exceptions-02.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 아래와 같습니다:

```text
CoroutineExceptionHandler got java.lang.AssertionError
```

## 취소와 예외 \(Cancellation and exceptions\)

취소는 예외와 밀접한 관련이 있습니다. 코루틴은 내부적으로 취소를 위해 `CancellationException`을 사용하고 이러한 예외는 모든 핸들러에서 무시되므로 추가 디버그 정보의 소스로만 사용해야 하며 `catch` 블럭으로 얻을 수 있습니다. 코루틴이 [Job.cancel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html)을 사용하여 취소되면 종료되지만 그것의 부모는 취소되지 않습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = launch {
        val child = launch {
            try {
                delay(Long.MAX_VALUE)
            } finally {
                println("Child is cancelled")
            }
        }
        yield()
        println("Cancelling child")
        child.cancel()
        child.join()
        yield()
        println("Parent is not cancelled")
    }
    job.join()
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-exceptions-03.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 아래와 같습니다:

```text
Cancelling child
Child is cancelled
Parent is not cancelled
```

코루틴에서 `CancellationException` 외에 다른 예외가 발생하면 해당 예외로 부모를 취소합니다. 이 동작은 재정의 될 수 없으며 [structured concurrency](https://github.com/Kotlin/kotlinx.coroutines/blob/master/docs/composing-suspending-functions.md#structured-concurrency-with-async)에 대해 안정적인 코루틴 계층을 제공하는데 사용됩니다. 자식 코루틴에는 [CoroutineExceptionHandler](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/index.html) 구현이 사용되지 않습니다.

> 예제에서 [CoroutineExceptionHandler](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/index.html)은 항상 [GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)에서 생성된 코루틴에 설치됩니다. 핸들러가 설치되었어도 자식 코루틴이 예외로 완료되면 main 코루틴은 항상 취소되기 때문에 main [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html)에 스코프에서 실행된 코루틴에 예외 핸들러를 설치하는 것은 의미가 없습니다.

원래 예외는 모든 자식이 종료될 때에만 부모에 의해 처리되며 다음 예제를 통해 살펴봅시다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
    val job = GlobalScope.launch(handler) {
        launch { // the first child
            try {
                delay(Long.MAX_VALUE)
            } finally {
                withContext(NonCancellable) {
                    println("Children are cancelled, but exception is not handled until all children terminate")
                    delay(100)
                    println("The first child finished its non cancellable block")
                }
            }
        }
        launch { // the second child
            delay(10)
            println("Second child throws an exception")
            throw ArithmeticException()
        }
    }
    job.join()
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-exceptions-04.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 아래와 같습니다:

```text
Second child throws an exception
Children are cancelled, but exception is not handled until all children terminate
The first child finished its non cancellable block
CoroutineExceptionHandler got java.lang.ArithmeticException
```

## 예외 집합 \(Exceptions aggregation\)

여러개의 자식 코루틴이 예외로 실패하면 일반적으로 첫번째 발생한 예외만 처리됩니다. 첫번째 예외 이후에 발생하는 모든 추가 예외는 첫번째 예외에 억제된 예외로 첨부됩니다.

```kotlin
import kotlinx.coroutines.*
import java.io.*

fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception with suppressed ${exception.suppressed.contentToString()}")
    }
    val job = GlobalScope.launch(handler) {
        launch {
            try {
                delay(Long.MAX_VALUE) // it gets cancelled when another sibling fails with IOException
            } finally {
                throw ArithmeticException() // the second exception
            }
        }
        launch {
            delay(100)
            throw IOException() // the first exception
        }
        delay(Long.MAX_VALUE)
    }
    job.join()  
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-exceptions-05.kt)에서 전체 코드를 볼 수 있습니다.
>
> 위 코드는 `suppressed` 예외를 지원하는 JDK7 이상에서 동작합니다.

이 코드의 출력은 아래와 같습니다:

```text
CoroutineExceptionHandler got java.io.IOException with suppressed [java.lang.ArithmeticException]
```

> 이 메커니즘은 현재 Java 1.7 이상에서만 동작합니다. JS 및 Native에 대한 제한은 일시적이고 향후 수정 예정입니다.

취소 예외는 투명하며 기본적으로 래핑되지 않습니다:

```kotlin
import kotlinx.coroutines.*
import java.io.*

fun main() = runBlocking {
//sampleStart
    val handler = CoroutineExceptionHandler { _, exception ->
        println("CoroutineExceptionHandler got $exception")
    }
    val job = GlobalScope.launch(handler) {
        val inner = launch { // all this stack of coroutines will get cancelled
            launch {
                launch {
                    throw IOException() // the original exception
                }
            }
        }
        try {
            inner.join()
        } catch (e: CancellationException) {
            println("Rethrowing CancellationException with original cause")
            throw e // cancellation exception is rethrown, yet the original IOException gets to the handler  
        }
    }
    job.join()
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-exceptions-06.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 아래와 같습니다:

```text
Rethrowing CancellationException with original cause
CoroutineExceptionHandler got java.io.IOException
```

## 슈퍼비전 \(Supervision\)

이전에 살펴봤듯이, 취소는 코루틴의 전체 계층을 통해 전파되는 양방향 관계입니다. 단방향 취소가 필요한 경우를 살펴봅시다.

단방향 취소의 좋은 예는 해당 범위의 작업이 정의된 UI 컴포넌트입니다. UI 하위 작업 중 하나라도 실패한 경우 항상 전체 UI 컴포넌트를 종료하거나 취소 할 필요는 없지만 UI 컴포넌트가 파괴되거나 해당 작업이 취소 되었을 때는 결과에 대한 필요성이 없어지므로 모든 하위 작업을 실패해야 합니다.

다른 예로는 여러 하위 작업을 생성하고 실행을 감독하고 실패를 추적하고 실패한 하위 작업만 다시 시작해야 하는 서버 프로세스 입니다.

### Supervision job

이러한 목적으로 [SupervisorJob](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-supervisor-job.html)을 사용할 수 있습니다. 취소가 아래쪽으로만 전파된다는 점만 제외하면 일반적인 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job.html)과 유사합니다. 예를 보면 이해하는데 도움이 될 것입니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val supervisor = SupervisorJob()
    with(CoroutineScope(coroutineContext + supervisor)) {
        // launch the first child -- its exception is ignored for this example (don't do this in practice!)
        val firstChild = launch(CoroutineExceptionHandler { _, _ ->  }) {
            println("First child is failing")
            throw AssertionError("First child is cancelled")
        }
        // launch the second child
        val secondChild = launch {
            firstChild.join()
            // Cancellation of the first child is not propagated to the second child
            println("First child is cancelled: ${firstChild.isCancelled}, but second one is still active")
            try {
                delay(Long.MAX_VALUE)
            } finally {
                // But cancellation of the supervisor is propagated
                println("Second child is cancelled because supervisor is cancelled")
            }
        }
        // wait until the first child fails & completes
        firstChild.join()
        println("Cancelling supervisor")
        supervisor.cancel()
        secondChild.join()
    }
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-supervision-01.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 아래와 같습니다:

```text
First child is failing
First child is cancelled: true, but second one is still active
Cancelling supervisor
Second child is cancelled because supervisor is cancelled
```

### 슈퍼비전 스코프 \(Supervision scope\)

_범위_ 가 동일한 동시성에 대해 동일한 목적으로 [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) 대신 [supervisorScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html)를 사용할 수 있습니다. 한 방향으로만 취소를 전파하고 실패했을 경우에만 모든 하위 작업을 취소합니다. 또한 [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html)와 마찬가지로 모든 하위 작업이 완료될 때까지 기다립니다.

```kotlin
import kotlin.coroutines.*
import kotlinx.coroutines.*

fun main() = runBlocking {
    try {
        supervisorScope {
            val child = launch {
                try {
                    println("Child is sleeping")
                    delay(Long.MAX_VALUE)
                } finally {
                    println("Child is cancelled")
                }
            }
            // Give our child a chance to execute and print using yield 
            yield()
            println("Throwing exception from scope")
            throw AssertionError()
        }
    } catch(e: AssertionError) {
        println("Caught assertion error")
    }
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-supervision-02.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력 결과는 아래와 같습니다:

```text
Child is sleeping
Throwing exception from scope
Child is cancelled
Caught assertion error
```

### Exceptions in supervised coroutines

일반과 슈퍼비저 작업 사이의 또다른 가장 큰 차이는 예외 처리에 있습니다. 모든 하위 작업은 예외 처리 메커니즘을 통해 스스로 예외를 처리해야합니다. 이 차이는 하위 작업의 실패가 상위 작업으로 전파되지 않는다는 사실에서 비롯됩니다. 이 의미는 [supervisorScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/supervisor-scope.html) 내에서 직접 실행된 코루틴은 루트 코루틴과 동일한 방식으로 해당 범위에 설치된 [CoroutineExceptionHandler](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-exception-handler/index.html)을 사용한다는 것입니다 \(자세한 내용은 CoroutineExceptionHandler 섹션을 참고 바랍니다\).

```kotlin
import kotlin.coroutines.*
import kotlinx.coroutines.*

fun main() = runBlocking {
    val handler = CoroutineExceptionHandler { _, exception -> 
        println("CoroutineExceptionHandler got $exception") 
    }
    supervisorScope {
        val child = launch(handler) {
            println("Child throws an exception")
            throw AssertionError()
        }
        println("Scope is completing")
    }
    println("Scope is completed")
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-supervision-03.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 아래와 같습니다:

```text
Scope is completing
Child throws an exception
CoroutineExceptionHandler got java.lang.AssertionError
Scope is completed
```

