# 코루틴 기본 \(Coroutine Basics\)

이 섹션은 코루틴 기본 컨셉에 대해 설명합니다.

## 코루틴 시작하기 \(Your first coroutine\)

아래 코드를 실행해봅시다:

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello,") // main thread continues while coroutine is delayed
    Thread.sleep(2000L) // block main thread for 2 seconds to keep JVM alive
}
```

> 전체 코드는 [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-01.kt)를 참고하시기 바랍니다.

실행결과 아래의 출력물을 얻을 수 있습니다:

```text
Hello,
World!
```

코루틴은 가벼운 쓰레드입니다. 일부 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)의 컨텍스트에서 [launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html) _코루틴 빌더_ 로 시작됩니다. 여기서는 [GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)에서 새로운 코루틴이 실행됩니다. 즉, 새로운 코루틴은 전체 애플리케이션 수명에 의해서만 제한됩니다.

`GlobalScope.launch { ... }`을 `thread { ... }`로 `delay(...)`를 `Thread.sleep(...)`로 변경하면 동일한 결과를 얻을 수 있습니다. 시도해봅시다 \(`kotlin.concurrent.thread`을 꼭 임포트해야 함\).

`GlobalScope.launch`을 `thread` 대체하고 시작한다면 컴파일러는 아래와 같은 에러를 발생시킵니다:

```text
Error: Kotlin: Suspend functions are only allowed to be called from a coroutine or another suspend function
```

[delay](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/delay.html)는 쓰레드를 차단하지 않고 코루틴을 일시 중지 시키는 특별한 _중단 함수 \(suspending function\)_ 이므로 코루틴에서만 사용할 수 있습니다.

## 브릿징 차단과 비차단의 세계 \(Bridging blocking and non-blocking worlds\)

첫번째 예는 _비차단 \(non-blocking\)_ `delay(...)`와 _차단 \(blocking\)_ `Thread.sleep(...)`이 혼합된 코드입니다. 이러한 코드는 어느 것이 차단되고 어느 것이 비차단되는지 알기 어렵습니다. [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 코루틴 빌더를 이용하여 차단되는 것을 명시적으로 나타내보겠습니다:

```kotlin
import kotlinx.coroutines.*

fun main() { 
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main thread continues here immediately
    runBlocking {     // but this expression blocks the main thread
        delay(2000L)  // ... while we delay for 2 seconds to keep JVM alive
    } 
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-02.kt)에서 전체 코드를 볼 수 있습니다.

결과는 같지만 이 코드는 오직 비차단 [delay](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/delay.html)를 사용하였습니다. `runBlocking`을 호출하는 메인 쓰레드는 코루틴 내부의 `runBlocking`이 완료될 때까지 _차단_ 합니다.

이 예제는 `runBlocking`으로 주 함수를 래핑하여 더 일반적인 형태로 작성 가능합니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking<Unit> { // start main coroutine
    GlobalScope.launch { // launch a new coroutine in background and continue
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main coroutine continues here immediately
    delay(2000L)      // delaying for 2 seconds to keep JVM alive
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-03.kt)에서 전체 코드를 볼 수 있습니다.

여기서 `runBlocking<Unit> { ... }`은 최상위 메인 코루틴을 시작하는데 사용되는 아답터로 작동합니다. Kotlin에서 `main` 함수는 `Unit`을 반환해야 하므로 `Unit` 반환 타입을 명시적으로 지정합니다.

이것은 중단 함수를 위한 유닛테스트 작성 방법이기도 합니다:

```kotlin
class MyTest {
    @Test
    fun testMySuspendingFunction() = runBlocking<Unit> {
        // here we can use suspending functions using any assertion style that we like
    }
}
```

## 작업을 위한 대기 \(Waiting for a job\)

다른 코루틴이 동작하는 동안 지연되는 것은 좋은 방법이 아닙니다. 시작한 백그라운드 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)이 완료될 때까지 명시적으로 비차단 방식으로 기다립니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    val job = GlobalScope.launch { // launch a new coroutine and keep a reference to its Job
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() // wait until child coroutine completes
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-04.kt)에서 전체 코드를 볼 수 있습니다.

결과는 동일하지만 메인 코루틴의 코드는 백그라운드 작업의 지속시간과 연관이 없습니다.

## 구조적 동시성 \(Structured concurrency\)

위의 예제처럼 사용하여도 코루틴을 사용하는데 문제가 없습니다. `GlobalScope.launch` 사용할 때 최상위 코루틴을 생성합니다. 비록 가볍지만 여전히 실행하는 동안 메모리를 약간 소비합니다. 만약 새로운 코루틴을 실행하는데 참조를 유지하는 것을 잊는다면 계속 실행된 상태로 유지됩니다. 코루틴이 중단 \(예를 들어 오랜시간 지연되는 경우\)되는 경우, 코루틴을 너무 많이 실행하여 메모리가 부족하면 어떻게 합니까? 실행된 모든 코루틴을 수동으로 유지하고 [join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html) 하려면 오류가 발생하기 쉽습니다.

이런 문제에 좋은 해결책이 있습니다. 코드에서 구조적 동시성을 사용할 수 있습니다. [GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)에서 코루틴을 시작하는 대신 일반적으로 쓰레드 \(쓰레드는 항상 전역으로 수행됨\)를 사용하는 것처럼 수행 중인 특정 작업에서 코루틴을 시작할 수 있습니다.

예에서 [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 코루틴 빌더를 사용하여 코루틴으로 변환하는 `main` 함수가 있습니다. `runBlocking` 포함한 모든 코루틴 빌더는 코드 블럭의 스코프에 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html) 인스턴스를 추가합니다. 스코프에서 실행 된 모든 코루틴이 완료될 때까지 외부 코루틴 \(이 예에서는 `runBlocking`\)이 완료되지 않으므로 명시적으로 `join` 없이 스코프에서 코루틴을 실행할 수 있습니다. 따라서 예제를 더 간단하게 만들 수 있습니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine in the scope of runBlocking
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-05.kt)에서 전체 코드를 볼 수 있습니다.

## 스코프 빌더 \(Scope builder\)

다른 빌더가 제공하는 코루틴 스코프 외에도 [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html)를 사용하여 고유한 스코프를 선언할 수 있습니다. 코루틴 스코프를 생성하고 실행된 모든 하위 항목이 완료되기 전까지 완료되지 않습니다.

[runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 와 [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html)은 모두 바디와 하위 항목이 완료될 때까지 기다리기 때문에 비슷해 보입니다. 가장 큰 차이점은 [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 메서드가 현재 쓰레드를 대기중으로 _차단_ 하고 [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html)은 일시 중단되는 동안 다른 용도를 위해 기본 쓰레드를 해제합니다. 이러한 차이점 때문에 [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html)은 기본 함수이고 [coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html) 중단 함수입니다.

다음 예제를 통해 살펴 봅시다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking { // this: CoroutineScope
    launch { 
        delay(200L)
        println("Task from runBlocking")
    }

    coroutineScope { // Creates a coroutine scope
        launch {
            delay(500L) 
            println("Task from nested launch")
        }

        delay(100L)
        println("Task from coroutine scope") // This line will be printed before the nested launch
    }

    println("Coroutine scope is over") // This line is not printed until the nested launch completes
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-06.kt)에서 전체 코드를 볼 수 있습니다.

[coroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/coroutine-scope.html)가 완료되지 않았지만 "Task from coroutine scope" 메세지 \(중첩된 launch를 기다리는 동안\)가 출력되고 바로 "Task from runBlocking" 메세지가 출력됩니다.

## 함수 리팩토링 추출 \(Extract function refactoring\)

`launch { ... }` 안에 있는 코드를 별도의 함수로 추출 해봅시다. 이 코드에서 "추출 함수"를 수행하면 `suspend`가 붙은 새로운 함수를 얻게됩니다. 이것은 첫번째 _중단 함수 \(suspending function\)_ 입니다. 중단 함수는 일반 함수와 같이 코루틴 내에서 사용할 수 있지만 추가 기능은 다른 중단 함수 \(이 예에서는 `delay`\)를 사용하여 코루틴 실행을 _일시 중단_ 할 수 있습니다.

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    launch { doWorld() }
    println("Hello,")
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-07.kt)에서 전체 코드를 볼 수 있습니다.

그러나 추출된 함수에 현재 스코프에서 호출되는 코루틴 빌더가 포함되어 있으면 어떨까요? 이러한 경우 추출된 함수의 `suspend` 수정자는 충분치 않습니다. `CoroutineScope`에서 `doWorld`를 확장 메서드로 만드는 것이 해결책 중 하나이지만 API가 명확하지 않기 때문에 항상 적용 가능하지 않습니다. 일반적인 해결책은 대상 함수를 포함하는 class의 필드로 `CoroutineScope` 가지거나 외부 class가 `CoroutineScope` 구현할 때 암묵적으로 가지는 방법이 있습니다. 마지막으로 [CoroutineScope\(coroutineContext\)](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope.html)를 사용할 수 있지만 이러한 접근은 메서드의 실행 범위를 제어할 수 없으므로 구조적으로 안전하지 않습니다. 이런 빌더는 오직 private API에서만 사용할 수 있습니다.

## 코루틴은 가볍습니다 \(Coroutines ARE light-weight\)

아래의 코드를 실행해 봅시다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    repeat(100_000) { // launch a lot of coroutines
        launch {
            delay(1000L)
            print(".")
        }
    }
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-08.kt)에서 전체 코드를 볼 수 있습니다.

이 예제는 10만개의 코루틴을 실행하고 1초 후에 각 코루틴은 점 \(.\)을 출력합니다.

이제 쓰레드로 같은 구현을 실행 해봅시다. 어떤 일이 벌어질까요? \(아마 대부분 메모리 부족 오류가 발생할 것입니다\)

## 글로벌 코루틴은 데몬 쓰레드와 같습니다 \(Global coroutines are like daemon threads\)

아래의 코드는 [GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)에서 동작하는 코루틴으로써 1초에 두번 "I'm sleeping" 문자열을 출력하고 일정 시간 지연 후에 메인 함수로 돌아옵니다:

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
//sampleStart
    GlobalScope.launch {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // just quit after delay
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-basic-09.kt)에서 전체 코드를 볼 수 있습니다.

실행하면 아래와 같이 세줄을 출력하고 종료되는 것을 볼 수 있습니다:

```text
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
```

[GlobalScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/index.html)에서 실행된 코루틴은 프로세스를 유지하지 않습니다. 이것은 데몬 쓰레드와 같습니다.

