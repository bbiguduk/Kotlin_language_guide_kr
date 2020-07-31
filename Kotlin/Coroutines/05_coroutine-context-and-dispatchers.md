## 코루틴 컨택스트와 Dispatchers (Coroutine Context and Dispatchers)

코루틴은 Kotlin 표준 라이브러리에 정의된 [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/) 타입의 값으로 표현된 어떤 컨택스트에서 실행됩니다.

코루틴 컨택스트는 여러 요소의 집합입니다. 주요 요소는 이전에 본 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)과 이번 섹션에서 알아볼 dispatcher 입니다.

### Dispatcher와 쓰레드 (Dispatchers and threads)

코루틴 컨택스트는 코루틴 실행에 사용하는 쓰레드 또는 쓰레드를 결정하는 _coroutine dispatcher_ ([CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html)) 를 포함합니다. 코루틴 dispatcher는 코루틴 실행을 특정 쓰레드로 제한하거나 쓰레드 풀에 보내거나 또는 제한이 없는 상태로 실행할 수 있습니다.

[launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) 와 [async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html)와 같은 코루틴 빌더는 새로운 코루틴과 다른 컨택스트 요소에 대한 dispatcher를 명시적으로 사용할 수 있는 선택적으로 [CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines/-coroutine-context/) 파라미터를 허용합니다.

아래 예제를 살펴봅시다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
//sampleStart
    launch { // context of the parent, main runBlocking coroutine
        println("main runBlocking      : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
        println("Unconfined            : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Default) { // will get dispatched to DefaultDispatcher 
        println("Default               : I'm working in thread ${Thread.currentThread().name}")
    }
    launch(newSingleThreadContext("MyOwnThread")) { // will get its own new thread
        println("newSingleThreadContext: I'm working in thread ${Thread.currentThread().name}")
    }
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-01.kt)에서 전체 코드를 볼 수 있습니다.

실행결과는 아래와 같습니다 (출력 순서는 다를 수 있음):

```text
Unconfined            : I'm working in thread main
Default               : I'm working in thread DefaultDispatcher-worker-1
newSingleThreadContext: I'm working in thread MyOwnThread
main runBlocking      : I'm working in thread main
```

<!--- TEST LINES_START_UNORDERED -->

`launch { ... }`를 파라미터 없이 사용할 때 실행중인 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)에서 컨택스트 (dispatcher와 함께)를 상속 받습니다. 이 경우 `main` 쓰레드에서 실행되는 메인의 `runBlocking` 코루틴의 컨택스트를 상속 받습니다.

[Dispatchers.Unconfined](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-unconfined.html)는 `main` 쓰레드에서 실행되는 특수한 dispatcher 지만 다른 매커니즘입니다.

[GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)에서 코루틴이 실행될 때 기본 dispatcher는 [Dispatchers.Default](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html)와 백그라운드 쓰레드 풀에서 공유하여 사용합니다. 그래서 `launch(Dispatchers.Default) { ... }`는 `GlobalScope.launch { ... }`와 같은 dispatcher 입니다.

[newSingleThreadContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/new-single-thread-context.html)는 코루틴을 실행하기 위한 쓰레드를 생성합니다.
이 쓰레드는 무거운 리소스 입니다.
실제 애플리케이션에서는 더이상 필요치 않을경우 [close](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-executor-coroutine-dispatcher/close.html) 함수를 이용하거나 최상위 변수에 저장하고 애플리케이션 전체 걸쳐 사용해야 합니다.

### 제한되지 않은 vs 제한된 dispatcher (Unconfined vs confined dispatcher)
 
[Dispatchers.Unconfined](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-unconfined.html) 코루틴 dispatcher는 호출한 쓰레드에서 코루틴을 시작하지만 첫번째 중지 포인트까지만 시작합니다. 중지 이후에 호출된 중지 함수에 의해 다른 쓰레드로 코루틴을 재개합니다. 제한되지 않은 dispatcher는 CPU 시간을 소비하지 않고 특정 쓰레드에 제한된 공유 데이터 (UI)를 업데이트 하지 않는 코루틴에 적합합니다.

반면에 dispatcher는 기본적으로 외부 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)로 부터 상속받습니다.
특히 [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 코루틴의 기본 dispatcher는 호출한 쓰레드에 제한되므로 이를 상속받으면 예측 가능한 FIFO 스케쥴링을 통해 이 쓰레드 실행을 제한할 수 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
//sampleStart
    launch(Dispatchers.Unconfined) { // not confined -- will work with main thread
        println("Unconfined      : I'm working in thread ${Thread.currentThread().name}")
        delay(500)
        println("Unconfined      : After delay in thread ${Thread.currentThread().name}")
    }
    launch { // context of the parent, main runBlocking coroutine
        println("main runBlocking: I'm working in thread ${Thread.currentThread().name}")
        delay(1000)
        println("main runBlocking: After delay in thread ${Thread.currentThread().name}")
    }
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-02.kt)에서 전체 코드를 볼 수 있습니다.

출력결과는 아래와 같습니다:
 
```text
Unconfined      : I'm working in thread main
main runBlocking: I'm working in thread main
Unconfined      : After delay in thread kotlinx.coroutines.DefaultExecutor
main runBlocking: After delay in thread main
```

<!--- TEST LINES_START -->

`runBlocking {...}`에서 컨텍스트를 상속받은 코루틴은 `main` 쓰레드에서 이어서 실행되는 반면에 제한된 쓰레드는 [delay](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/delay.html) 함수가 사용중인 기본 실행 쓰레드에서 이어서 실행되는 것을 확인할 수 있습니다.

> 제한된 dispatcher는 코루틴의 일부 연산을 즉시 수행해야 하기 때문에 나중에 실행하기 위해 코루틴을 디스패치 할 필요가 없거나 사이드 이펙트를 발생시키는 특정 경우에 도움이 됩니다.
제한된 dispatcher는 일반적인 코드에서 사용하지 않습니다.

### 디버깅 코루틴과 쓰레드 (Debugging coroutines and threads)

코루틴은 한 쓰레드에서 중지하고 다른 쓰레드에서 재개될 수 있습니다.
싱글 쓰레드 dispatcher로는 코루틴이 무엇을 어디서 언제 무엇을 하는지 알기 어려울 수 있습니다. 쓰레드를 사용하여 애플리케이션을 디버깅 하는 방법은 각 로그 문장에 쓰레드 이름을 출력하는 것입니다. 이 기능은 로깅 프레임워크로 부터 기본적으로 제공됩니다. 코루틴을 사용할 때 쓰레드의 이름만으로 컨텍스트를 판단하기 어렵기 때문에 `kotlinx.coroutines`에서는 디버깅을 쉽게 해주는 기능을 제공합니다.

아래 코드를 `-Dkotlinx.coroutines.debug` JVM 옵션으로 실행해봅시다:

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking<Unit> {
//sampleStart
    val a = async {
        log("I'm computing a piece of the answer")
        6
    }
    val b = async {
        log("I'm computing another piece of the answer")
        7
    }
    log("The answer is ${a.await() * b.await()}")
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-03.kt)에서 전체 코드를 볼 수 있습니다.

`runBlocking` 안에 메인 코루틴 (#1)과 지연된 값 `a` (#2) 와 `b` (#3) 를 연산하는 코루틴 2개로 3개의 코루틴이 있습니다.
모두 `runBlocking`에 컨텍스트에서 실행되고 모두 메인 쓰레드로 제한됩니다.
이 코드의 출력은 아래와 같습니다:

```text
[main @coroutine#2] I'm computing a piece of the answer
[main @coroutine#3] I'm computing another piece of the answer
[main @coroutine#1] The answer is 42
```

<!--- TEST FLEXIBLE_THREAD -->

`log` 함수는 쓰레드의 이름을 대괄호 안에 표기해주고 현재 실행중인 코루틴이 `main` 쓰레드임을 알게 해줍니다. 이 식별자는 디버깅 모드가 켜져있을 때 생성되는 모든 코루틴에 연속적으로 할당됩니다.

> 디버깅 모드는 JVM이 `-ea` 옵션과 함께 실행될 때도 켜집니다.
[DEBUG_PROPERTY_NAME](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-d-e-b-u-g_-p-r-o-p-e-r-t-y_-n-a-m-e.html)에서 자세한 내용을 확인하세요.

### 쓰레드 사이에서 점핑 (Jumping between threads)

아래 예제를 `-Dkotlinx.coroutines.debug` 옵션으로 실행해 봅시다:

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() {
//sampleStart
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                withContext(ctx2) {
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-04.kt)에서 전체 코드를 볼 수 있습니다.

이것은 몇가지 새로운 기술을 보여줍니다. 하나는 명시적으로 지정된 컨텍스트의 [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html)을 사용하는 것이고 다른 하나는 같은 코루틴에 머물지만 다른 코루틴의 컨텍스트로 변경하기 위해 [withContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html) 함수를 사용하는 것입니다.
따라서 아래와 같은 출력을 볼 수 있습니다:

```text
[Ctx1 @coroutine#1] Started in ctx1
[Ctx2 @coroutine#1] Working in ctx2
[Ctx1 @coroutine#1] Back to ctx1
```

<!--- TEST -->

이 예제에서 더이상 필요하지 않을 때 [newSingleThreadContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/new-single-thread-context.html)로 만든 쓰레드를 해제하기 위해 Kotlin 표준 라이브러리에 정의 된 `use` 함수를 사용한 것에 주목합시다.

### 컨텍스트에서 작업 (Job in the context)

코루틴의 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)은 컨텍스트의 일부이고 `coroutineContext[Job]` 표현을 이용하여 가져올 수 있습니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
//sampleStart
    println("My job is ${coroutineContext[Job]}")
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-05.kt)에서 전체 코드를 볼 수 있습니다.

디버그 모드에서 출력은 아래와 같습니다:

```
My job is "coroutine#1":BlockingCoroutine{Active}@6d311334
```

<!--- TEST lines.size == 1 && lines[0].startsWith("My job is \"coroutine#1\":BlockingCoroutine{Active}@") -->

[CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) 안에 [isActive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/is-active.html)는 `coroutineContext[Job]?.isActive == true`을 짧게 나타낸 것입니다.

### 코루틴의 자식 (Children of a coroutine)

다른 코루틴의 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)에서 코루틴이 실행될 때 [CoroutineScope.coroutineContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/coroutine-context.html)로부터 컨텍스트를 상속받고 새로운 코루틴의 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)은 부모 코루틴의 작업의 _자식_ 이 됩니다. 부모 코루틴이 취소되면 자식 코루틴도 재귀적으로 취소됩니다.

그러나 [GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)가 코루틴을 실행하기 위해 사용되면 새로운 코루틴의 작업을 위한 부모는 존재하지 않습니다.
따라서 실행된 범위와 별개로 실행합니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
//sampleStart
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        // it spawns two other jobs, one with GlobalScope
        GlobalScope.launch {
            println("job1: I run in GlobalScope and execute independently!")
            delay(1000)
            println("job1: I am not affected by cancellation of the request")
        }
        // and the other inherits the parent context
        launch {
            delay(100)
            println("job2: I am a child of the request coroutine")
            delay(1000)
            println("job2: I will not execute this line if my parent request is cancelled")
        }
    }
    delay(500)
    request.cancel() // cancel processing of the request
    delay(1000) // delay a second to see what happens
    println("main: Who has survived request cancellation?")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-06.kt)에서 전체 코드를 볼 수 있습니다.

출력은 아래와 같습니다:

```text
job1: I run in GlobalScope and execute independently!
job2: I am a child of the request coroutine
job1: I am not affected by cancellation of the request
main: Who has survived request cancellation?
```

<!--- TEST -->

### 부모 코루틴의 영향 (Parental responsibilities)

부모 코루틴은 모든 자식 코루틴이 완료될 때까지 기다립니다. 부모 코루틴은 명시적으로 자식 코루틴을 알 필요가 없으며 [Job.join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html)을 사용하여 자식 코루틴이 끝까지 기다릴 필요도 없습니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
//sampleStart
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        repeat(3) { i -> // launch a few children jobs
            launch  {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, 600ms
                println("Coroutine $i is done")
            }
        }
        println("request: I'm done and I don't explicitly join my children that are still active")
    }
    request.join() // wait for completion of the request, including all its children
    println("Now processing of the request is complete")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-07.kt)에서 전체 코드를 볼 수 있습니다.

결과는 아래와 같습니다:

```text
request: I'm done and I don't explicitly join my children that are still active
Coroutine 0 is done
Coroutine 1 is done
Coroutine 2 is done
Now processing of the request is complete
```

<!--- TEST -->

### 디버깅을 위한 코루틴 네이밍 (Naming coroutines for debugging)

자동으로 할당 된 아이디는 코루틴이 로그에 기록될 때 유용하고 동일한 코루틴에 로그 기록을 통해 연관성이 있음을 알려줍니다. 그러나 코루틴이 특정 요청을 처리하거나 특정 백그라운드 작업을 수행하는 경우에는 명시적으로 이름을 지정해주는 것이 좋습니다.
[CoroutineName](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-name/index.html) 컨텍스트 요소는 쓰레드 이름과 같은 목적으로 제공됩니다. 디버깅 모드를 켰을 때 코루틴을 실행하는 쓰레드의 이름에 포함됩니다.

예제를 통해 확인해 봅시다:

```kotlin
import kotlinx.coroutines.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking(CoroutineName("main")) {
//sampleStart
    log("Started main coroutine")
    // run two background value computations
    val v1 = async(CoroutineName("v1coroutine")) {
        delay(500)
        log("Computing v1")
        252
    }
    val v2 = async(CoroutineName("v2coroutine")) {
        delay(1000)
        log("Computing v2")
        6
    }
    log("The answer for v1 / v2 = ${v1.await() / v2.await()}")
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-08.kt)에서 전체 코드를 볼 수 있습니다.

`-Dkotlinx.coroutines.debug` JVM 옵션과 함께 실행하면 아래와 같습니다:
 
```text
[main @main#1] Started main coroutine
[main @v1coroutine#2] Computing v1
[main @v2coroutine#3] Computing v2
[main @main#1] The answer for v1 / v2 = 42
```

<!--- TEST FLEXIBLE_THREAD -->

### 컨텍스트 요소 결합 (Combining context elements)

코루틴 컨텍스트에 대한 여러개의 요소를 정의할 필요가 있습니다. 이럴 경우 `+` 연산자를 사용합니다.
예를 들어 명시적으로 지정된 dispatcher와 명시적으로 지정된 이름을 가진 코루틴을 동시에 실행할 수 있습니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> {
//sampleStart
    launch(Dispatchers.Default + CoroutineName("test")) {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-09.kt)에서 전체 코드를 볼 수 있습니다.

`-Dkotlinx.coroutines.debug` JVM 옵션으로 실행한 결과는 아래와 같습니다:

```text
I'm working in thread DefaultDispatcher-worker-1 @test#2
```

<!--- TEST FLEXIBLE_THREAD -->

### 코루틴 스코프 (Coroutine scope)

컨텍스트, 자식 코루틴과 작업에 대해 지식을 가지게 되었습니다. 애플리케이션에 라이프사이클이 있는 객체가 있지만 그 객체가 코루틴이 아니라고 가정해봅시다. 예를 들어 Android 애플리케이션을 만들고 데이터를 업데이트하거나 애니메이션 등 비동기적인 작업을 수행하기 위해 Android 액티비티의 컨텍스트에서 다양한 코루틴을 실행한다면 메모리 누수를 방지하기위해 액티비티가 종료될 때 모든 코루틴은 반드시 취소되야 합니다. 물론 액티비티와 코루틴의 라이프사이클을 연결하기위해 수동적으로 컨텍스트와 작업을 다룰 수 있지만 `kotlinx.coroutines`은 추상적인 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)을 제공합니다.
모든 코루틴 빌더가 확장으로 선언되므로 코루틴 스코프는 이미 앞의 예제를 통해 친숙할 것입니다.

액티비티의 라이프사이클과 연결된 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) 인스턴스를 생성하여 코루틴의 라이프사이클을 관리할 수 있습니다. `CoroutineScope` 인스턴스는 [CoroutineScope()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope.html) 또는 [MainScope()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-main-scope.html) 팩토리 함수로 생성될 수 있습니다. 전자는 범용적인 스코프를 생성하는 반면에 후자는 UI 애플리케이션을 위한 스코프를 만들고 기본 dispatcher인 [Dispatchers.Main](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html)을 사용합니다:

```kotlin
class Activity {
    private val mainScope = MainScope()
    
    fun destroy() {
        mainScope.cancel()
    }
    // to be continued ...
```

정의된 `scope`를 사용하여 `Activity`의 스코프 안에서 코루틴을 실행할 수 있습니다.
데모를 위해 다른 시간의 딜레이를 가지는 10개의 코루틴을 실행합니다:

```kotlin
    // class Activity continues
    fun doSomething() {
        // launch ten coroutines for a demo, each working for a different time
        repeat(10) { i ->
            mainScope.launch {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, ... etc
                println("Coroutine $i is done")
            }
        }
    }
} // class Activity ends
``` 

main 함수 안에서 액티비티를 만들고 `doSomething` 함수를 호출하고 500ms 후에 액티비티를 종료합니다.
`doSomething`에서 실행된 코루틴을 모두 취소합니다. 액티비티가 종료된 후 더이상 메세지가 출력되지 않으므로 취소되었다는 것을 알 수 있습니다.

<!--- CLEAR -->

```kotlin
import kotlinx.coroutines.*

class Activity {
    private val mainScope = CoroutineScope(Dispatchers.Default) // use Default for test purposes
    
    fun destroy() {
        mainScope.cancel()
    }

    fun doSomething() {
        // launch ten coroutines for a demo, each working for a different time
        repeat(10) { i ->
            mainScope.launch {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, ... etc
                println("Coroutine $i is done")
            }
        }
    }
} // class Activity ends

fun main() = runBlocking<Unit> {
//sampleStart
    val activity = Activity()
    activity.doSomething() // run test function
    println("Launched coroutines")
    delay(500L) // delay for half a second
    println("Destroying activity!")
    activity.destroy() // cancels all coroutines
    delay(1000) // visually confirm that they don't work
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-10.kt)에서 전체 코드를 볼 수 있습니다.

이 예제의 출력은 아래와 같습니다:

```text
Launched coroutines
Coroutine 0 is done
Coroutine 1 is done
Destroying activity!
```

<!--- TEST -->

결과를 봤듯이 처음 두 코루틴만 메세지가 출력되고 `Activity.destroy()`안에 `job.cancel()`의 호출로 나머지는 취소됩니다.

> Android는 라이프사이클이 있는 모든 엔티티에서 코루틴 스코프에 대한 first-party를 지원합니다.
자세한 설명은 [the corresponding documentation](https://developer.android.com/topic/libraries/architecture/coroutines#lifecyclescope)를 참고 바랍니다.

### 쓰레드 로컬 데이터 (Thread-local data)

일부 쓰레드 로컬 데이터를 코루틴으로 또는 코루틴 간에 전달하는 기능이 편리할 때가 있습니다.
그러나 특정 쓰레드에 구속되지 않으므로 수동으로 수행하면 상용구로 이어질 수 있습니다.

[`ThreadLocal`](https://docs.oracle.com/javase/8/docs/api/java/lang/ThreadLocal.html)에서는 이를 해결하기 위해 [asContextElement](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/java.lang.-thread-local/as-context-element.html)라는 확장 함수가 존재합니다. 주어진 `ThreadLocal` 값을 유지하는 추가적인 컨텍스트 요소를 생성하고 코루틴이 컨텍스트를 전환할 때마다 이 요소를 복원합니다.

이것을 증명하는 것은 매우 간단합니다:

```kotlin
import kotlinx.coroutines.*

val threadLocal = ThreadLocal<String?>() // declare thread-local variable

fun main() = runBlocking<Unit> {
//sampleStart
    threadLocal.set("main")
    println("Pre-main, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
    val job = launch(Dispatchers.Default + threadLocal.asContextElement(value = "launch")) {
        println("Launch start, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
        yield()
        println("After yield, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
    }
    job.join()
    println("Post-main, current thread: ${Thread.currentThread()}, thread local value: '${threadLocal.get()}'")
//sampleEnd    
}
```                                                                                      

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-context-11.kt)에서 전체 코드를 볼 수 있습니다.

이 예제에서 [Dispatchers.Default](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html)을 사용하여 백그라운드 쓰레드 풀에 새로운 코루틴을 실행합니다. 그래서 쓰레드 풀과는 다른 쓰레드에서 작동하지만 코루틴이 실행되는 쓰레드와 상관없이 `threadLocal.asContextElement(value = "launch")`을 사용하여 지정한 쓰레드 로컬 변수의 값을 가지고 있습니다.
따라서 이 예제의 출력은 아래와 같습니다:

```text
Pre-main, current thread: Thread[main @coroutine#1,5,main], thread local value: 'main'
Launch start, current thread: Thread[DefaultDispatcher-worker-1 @coroutine#2,5,main], thread local value: 'launch'
After yield, current thread: Thread[DefaultDispatcher-worker-2 @coroutine#2,5,main], thread local value: 'launch'
Post-main, current thread: Thread[main @coroutine#1,5,main], thread local value: 'main'
```

<!--- TEST FLEXIBLE_THREAD -->

해당 컨텍스트 요소를 설정하는 것을 잊어버리기 쉽습니다. 코루틴을 실행하는 쓰레드가 다를경우 코루틴에서 접근하는 쓰레드 로컬 변수는 예상하지 못한 값을 가질 수 있습니다.
이러한 상황을 피하려면 [ensurePresent](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/java.lang.-thread-local/ensure-present.html) 메서드나 부적절한 방법인 fail-fast를 사용하는 것이 좋습니다.

`ThreadLocal`은 first-class를 지원하고 `kotlinx.coroutines`에서 제공하는 기본기능을 함께 사용할 수 있습니다.
한가지 중요한 제약사항이 존재합니다: 쓰레드 로컬이 변경될 경우 새로운 값은 컨텍스트 요소가 모든 `ThreadLocal` 객체 접근을 추적할 수 없으므로 코루틴 호출자에게 전달되지 않고 업데이트 된 값이 다음 중지에서 손실된다는 것입니다.
코루틴에서 쓰레드 로컬의 값을 업데이트하려면 [withContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)을 사용해야 합니다. 자세한 내용은 [asContextElement](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/java.lang.-thread-local/as-context-element.html)을 참고 바랍니다.

또한 값을 `class Counter(var i: Int)`와 같은 변경가능한 박스에 저장할 수 있고 이같은 경우 쓰레드 로컬 변수에 저장됩니다. 그러나 이러한 경우 변경가능한 박스 안에 변수의 수정사항을 동기화를 반영해 주어야 합니다.

예를 들어 MDC 로깅, 트랜젝션 컨텍스트 또는 데이터 전송을 위한 쓰레드 로컬을 사용하는 다른 라이브러리와의 통합과 같은 내용은 [ThreadContextElement](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-thread-context-element/index.html) 인터페이스 설명을 참고 바랍니다.

<!--- MODULE kotlinx-coroutines-core -->
<!--- INDEX kotlinx.coroutines -->
[Job]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html
[CoroutineDispatcher]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html
[launch]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html
[async]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html
[CoroutineScope]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html
[Dispatchers.Unconfined]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-unconfined.html
[GlobalScope]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html
[Dispatchers.Default]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html
[newSingleThreadContext]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/new-single-thread-context.html
[ExecutorCoroutineDispatcher.close]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-executor-coroutine-dispatcher/close.html
[runBlocking]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html
[delay]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/delay.html
[DEBUG_PROPERTY_NAME]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-d-e-b-u-g_-p-r-o-p-e-r-t-y_-n-a-m-e.html
[withContext]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html
[isActive]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/is-active.html
[CoroutineScope.coroutineContext]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/coroutine-context.html
[Job.join]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html
[CoroutineName]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-name/index.html
[CoroutineScope()]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope.html
[MainScope()]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-main-scope.html
[Dispatchers.Main]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html
[asContextElement]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/java.lang.-thread-local/as-context-element.html
[ensurePresent]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/java.lang.-thread-local/ensure-present.html
[ThreadContextElement]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-thread-context-element/index.html
<!--- END -->
