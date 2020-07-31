## 공유된 변경 가능한 상태와 동시성 (Shared mutable state and concurrency)

코루틴은 [Dispatchers.Default](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html)와 같은 멀티 쓰레드 dispatcher를 이용하여 동시에 실행할 수 있습니다. 이것은 모든 일반적인 동시성 문제를 가지고 있습니다. 주요 문제는 **공유된 변경 가능한 상태**에 대한 접근 동기화입니다.
코루틴에서 이 문제에 대한 해결책은 멀티 쓰레드 해결책과 유사하지만 다른 솔루션은 독특합니다.

### 문제 (The problem)

같은 행동을 천번 하는 코루틴을 100개 실행해 봅시다.
추가 비교를 위해 완료 시간을 측정해 봅시다:

```kotlin
suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}
```

멀티 쓰레드 [Dispatchers.Default](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html)를 사용하여 공유된 가변 변수를 증가시키는 간단한 작업으로 시작합니다.

<!--- CLEAR -->

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*    

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

//sampleStart
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
//sampleEnd    
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-sync-01.kt)에서 전체 코드를 볼 수 있습니다.

<!--- TEST LINES_START
Completed 100000 actions in
Counter =
-->

마지막에 무엇이 출력됩니까? 100개의 코루틴은 동기화 없이 여러 쓰레드에서 동시에 `counter`를 증가시키므로 "Counter = 100000"을 출력할 가능성이 거의 없습니다.

### Volatiles는 도움이 되지 않음 (Volatiles are of no help)

변수를 `volatile`로 만드는 것이 동시성 문제를 해결한다는 일반적인 오해가 있습니다. 직접 적용해 봅시다:

<!--- CLEAR -->

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

//sampleStart
@Volatile // in Kotlin `volatile` is an annotation 
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
//sampleEnd    
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-sync-02.kt)에서 전체 코드를 볼 수 있습니다.

<!--- TEST LINES_START
Completed 100000 actions in
Counter =
-->

이 코드는 느리게 동작하지만 휘발성 변수가 해당 변수에 선형화  가능 (linearizable - "atomic"에 대한 기술적 용어)한 읽기 및 쓰기를 보장하지만 더 큰 동작 (여기에서는 증가)의 원자성을 제공하지 않기 때문에 끝에 "Counter = 100000" 출력을 얻을 수 없습니다.

### 안전한 쓰레드 데이터 구조 (Thread-safe data structures)

쓰레드와 코루틴에 모두 동작하는 일반적인 솔루션은 공유 상태에서 수행해야하는 해당 작업에 대한 모든 동기화를 제공하는 안전한 쓰레드 (일명 동기화, 선형화 또는 atomic) 데이터 구조를 사용하는 것입니다.
간단한 카운터의 경우 atomic `incrementAndGet` 동작을 가진 `AtomicInteger` class를 사용할 수 있습니다:

<!--- CLEAR -->

```kotlin
import kotlinx.coroutines.*
import java.util.concurrent.atomic.*
import kotlin.system.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

//sampleStart
val counter = AtomicInteger()

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.incrementAndGet()
        }
    }
    println("Counter = $counter")
}
//sampleEnd    
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-sync-03.kt)에서 전체 코드를 볼 수 있습니다.

<!--- TEST ARBITRARY_TIME
Completed 100000 actions in xxx ms
Counter = 100000
-->

이것이 이 특정 문제에 대한 가장 빠른 해결책입니다. 이것은 일반 카운터, 콜렉션, 큐 및 기타 표준 데이터 구조 및 기본 작업에 사용됩니다. 그러나 복잡한 상태나 즉시 사용 가능한 안전한 쓰레드 구현이 없는 복잡한 작업으로 쉽게 확장할 수 없습니다.

### 세밀한 쓰레드 제한 (Thread confinement fine-grained)

_쓰레드 제한_ 은 특정 공유 상태에 대한 모든 접근이 단일 쓰레드로 제한되는 공유된 가변 상태의 문제를 접근하는 방식입니다. 일반적으로 모든 UI 상태가 단일 이벤트 dispatch/애플리케이션 쓰레드로 제한되는 UI 애플리케이션에서 사용됩니다. 단일 쓰레드 컨텍스트를 사용하면 코루틴을 쉽게 적용할 수 있습니다.

<!--- CLEAR -->

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

//sampleStart
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            // confine each increment to a single-threaded context
            withContext(counterContext) {
                counter++
            }
        }
    }
    println("Counter = $counter")
}
//sampleEnd      
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-sync-04.kt)에서 전체 코드를 볼 수 있습니다.

<!--- TEST ARBITRARY_TIME
Completed 100000 actions in xxx ms
Counter = 100000
-->

이 코드는 _세밀하게_ 쓰레드 제한을 하기 때문에 매우 느리게 동작합니다. 각 증가 연산은 멀티 쓰레드 [Dispatchers.Default](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html) 컨텍스트에서 [withContext(counterContext)](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html) 블럭을 사용한 단일 쓰레드로 변환됩니다.

### 쓰레드 제한 (Thread confinement coarse-grained)

실제로 쓰레드 제한은 더 큰 범위에서 수행됩니다. 예를 들어 상태를 업데이트하는 비지니스 로직의 큰 조각은 단일 쓰레드로 한정합니다. 다음 예제는 단일 쓰레드 컨텍스트에서 각 코루틴을 실행하여 시작합니다.

<!--- CLEAR -->

```kotlin
import kotlinx.coroutines.*
import kotlin.system.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

//sampleStart
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main() = runBlocking {
    // confine everything to a single-threaded context
    withContext(counterContext) {
        massiveRun {
            counter++
        }
    }
    println("Counter = $counter")
}
//sampleEnd     
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-sync-05.kt)에서 전체 코드를 볼 수 있습니다.

<!--- TEST ARBITRARY_TIME
Completed 100000 actions in xxx ms
Counter = 100000
-->

확인결과 더 빠르고 정확한 결과를 얻을 수 있습니다.

### 상호 배제 (Mutual exclusion)

문제에 대한 상호 배제 해결책은 동시에 실행되지 않는 _중요한 섹션_ 으로 공유 상태의 모든 수정을 보호하는 것입니다. 일반적으로 `synchronized` 또는 `ReentrantLock`을 사용합니다.
코루틴의 대안은 [Mutex](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/-mutex/index.html) 입니다. 이것은 중요한 섹션을 구분하는 [lock](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/-mutex/lock.html) 과 [unlock](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/-mutex/unlock.html) 함수를 가지고 있습니다. 주요 차이점은 `Mutex.lock()`은 일시 중단 함수입니다. 이것은 쓰레드를 차단하지 않습니다.

`mutex.lock(); try { ... } finally { mutex.unlock() }` 패턴을 편하게 표현하는 [withLock](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/with-lock.html) 확장 함수도 있습니다:

<!--- CLEAR -->

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.sync.*
import kotlin.system.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

//sampleStart
val mutex = Mutex()
var counter = 0

fun main() = runBlocking {
    withContext(Dispatchers.Default) {
        massiveRun {
            // protect each increment with lock
            mutex.withLock {
                counter++
            }
        }
    }
    println("Counter = $counter")
}
//sampleEnd    
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-sync-06.kt)에서 전체 코드를 볼 수 있습니다.

<!--- TEST ARBITRARY_TIME
Completed 100000 actions in xxx ms
Counter = 100000
-->

예제에서 lock은 세밀하기 때문에 성능면에서 느리게 동작합니다. 그러나 어떠한 공유된 상태를 주기적으로 수정해야 하지만 상태가 제한적이어서 쓰레드 적용이 어려운 경우에 좋은 해결책 입니다.

### 액터 (Actors)

[actor](https://en.wikipedia.org/wiki/Actor_model)는 코루틴에 한정되어 캡슐화 된 상태 및 다른 코루틴과 통신을 위한 채널의 조합으로 구성되어있습니다. 간단한 액터는 함수로 작성할 수 있지만 복잡한 액터는 class에 더 적합합니다.

[actor](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/actor.html) 코루틴 빌더는 메세지를 수신하기위해  그것의 스코프로 액터의 메일박스 채널을 편리하게 결합하고 액터에 대한 단일 참조를 핸들로써 전달할 수 있는 결과 작업 객체로 송신 채널을 결합합니다.

액터를 사용하는 첫 단계는 액터가 처리할 메세지 class를 정의하는 것입니다.
Kotlin의 [sealed classes](https://kotlinlang.org/docs/reference/sealed-classes.html)는 이러한 목적에 적합합니다.
카운터를 증가시키는 `IncCounter` 메세지와 값을 가져오는 `GetCounter` 메세지로 이뤄진 `CounterMsg` sealed class를 정의합니다. 그리고 나서 응답을 보내야 합니다. 앞으로 알려질 (통신될) 단일 값을 나타내는 [CompletableDeferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-deferred/index.html) 통신 프리미티브가 여기에서 해당 목적으로 사용됩니다.

```kotlin
// Message types for counterActor
sealed class CounterMsg
object IncCounter : CounterMsg() // one-way message to increment counter
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg() // a request with reply
```

[actor](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/actor.html) 코루틴 빌더를 사용하여 액터를 실행하는 함수를 정의합니다:

```kotlin
// This function launches a new counter actor
fun CoroutineScope.counterActor() = actor<CounterMsg> {
    var counter = 0 // actor state
    for (msg in channel) { // iterate over incoming messages
        when (msg) {
            is IncCounter -> counter++
            is GetCounter -> msg.response.complete(counter)
        }
    }
}
```

주요 코드는 간단합니다:

<!--- CLEAR -->

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlin.system.*

suspend fun massiveRun(action: suspend () -> Unit) {
    val n = 100  // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        coroutineScope { // scope for coroutines 
            repeat(n) {
                launch {
                    repeat(k) { action() }
                }
            }
        }
    }
    println("Completed ${n * k} actions in $time ms")    
}

// Message types for counterActor
sealed class CounterMsg
object IncCounter : CounterMsg() // one-way message to increment counter
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg() // a request with reply

// This function launches a new counter actor
fun CoroutineScope.counterActor() = actor<CounterMsg> {
    var counter = 0 // actor state
    for (msg in channel) { // iterate over incoming messages
        when (msg) {
            is IncCounter -> counter++
            is GetCounter -> msg.response.complete(counter)
        }
    }
}

//sampleStart
fun main() = runBlocking<Unit> {
    val counter = counterActor() // create the actor
    withContext(Dispatchers.Default) {
        massiveRun {
            counter.send(IncCounter)
        }
    }
    // send a message to get a counter value from an actor
    val response = CompletableDeferred<Int>()
    counter.send(GetCounter(response))
    println("Counter = ${response.await()}")
    counter.close() // shutdown the actor
}
//sampleEnd    
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-sync-07.kt)에서 전체 코드를 볼 수 있습니다.

<!--- TEST ARBITRARY_TIME
Completed 100000 actions in xxx ms
Counter = 100000
-->

액터 자체가 어떤 컨텍스트에서 실행되는지는 중요하지 (정확성을 위해) 않습니다. 액터는 코루틴이고 코루틴은 순차적으로 실행되므로 특정 코루틴의 상태 제한은 공유 가능한 가변 상태의 문제의 해결책으로 동작합니다. 실제로 액터는 자신의 private 상태를 수정할 수 있지만 메세지 (lock이 필요치 않음)를 통해서만 서로에 영향을 줄 수 있습니다.

액터는 수행해야 할 작업이 항상 있고 다른 컨텍스트로 전환할 필요가 없기 때문에 잠금보다 효율적입니다.

> [actor](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/actor.html) 코루틴 빌더는 [produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html) 코루틴 빌더와 비슷합니다. 액터는 수신 메세지 채널과 관련이 있고 프로듀서는 송신 요소 채널과 관련이 있습니다.

<!--- MODULE kotlinx-coroutines-core -->
<!--- INDEX kotlinx.coroutines -->
[Dispatchers.Default]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html
[withContext]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html
[CompletableDeferred]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-completable-deferred/index.html
<!--- INDEX kotlinx.coroutines.sync -->
[Mutex]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/-mutex/index.html
[Mutex.lock]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/-mutex/lock.html
[Mutex.unlock]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/-mutex/unlock.html
[withLock]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/with-lock.html
<!--- INDEX kotlinx.coroutines.channels -->
[actor]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/actor.html
[produce]: https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html
<!--- END -->
