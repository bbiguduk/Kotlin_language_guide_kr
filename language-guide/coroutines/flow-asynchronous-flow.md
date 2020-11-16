# 비동기 플로우 \(Asynchronous Flow\)

중단 함수는 비동기적으로 하나의 값을 반환하지만 비동기적으로 여러개의 계산된 값을 반환하려면 어떻게 해야 할까요? 이를 위해 Kotlin 플로우가 존재합니다.

## 여러 값 표현 \(Representing multiple values\)

[collections](https://kotlinlang.org/docs/reference/collections-overview.html%20)을 이용하여 Kotlin에서 여러개의 값을 표현할 수 있습니다. 예를 들어 3개의 숫자를 가진 [List](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/index.html) 반환하고 [forEach](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/for-each.html) 를 이용하여 모든 값을 출력하는 `simple()` 을 가질 수 있습니다:

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)
 
fun main() {
    simple().forEach { value -> println(value) } 
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-01.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력값은 아래와 같습니다:

```text
1
2
3
```

### 시퀀스 \(Sequences\)

각 계산이 100ms 소요되어 CPU가 소모되는 코드는 [Sequence](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/index.html) 사용하여 숫자를 나타낼 수 있습니다:

```kotlin
fun simple(): Sequence<Int> = sequence { // sequence builder
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it
        yield(i) // yield next value
    }
}

fun main() {
    simple().forEach { value -> println(value) } 
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-02.kt)에서 전체 코드를 볼 수 있습니다.

이 코드 출력은 같은 숫자를 표기하지만 출력 전에 100ms를 기다린 후 출력합니다.

### 중단 함수 \(Suspending functions\)

그러나 위 코드의 계산은 코드를 실행하는 main 쓰레드를 차단합니다. 이러한 값들을 비동기적으로 계산하려면 함수 `simple` 에 `suspend` 수식어를 붙여 메인 쓰레드를 차단하지 않고 작업을 수행하고 그 결과를 리스트로 반환할 수 있습니다:

```kotlin
import kotlinx.coroutines.*                 

//sampleStart
suspend fun simple(): List<Int> {
    delay(1000) // pretend we are doing something asynchronous here
    return listOf(1, 2, 3)
}

fun main() = runBlocking<Unit> {
    simple().forEach { value -> println(value) } 
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-03.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 실행결과는 1초 간격으로 숫자를 출력합니다.

### 플로우 \(Flows\)

`List<Int>` 결과 타입을 사용하는 것은 모든 값을 한번에 반환할 수 있다는 의미입니다. 비동기적으로 계산되는 값을 스트림으로 표현하려면 동기적으로 계산된 값을 나타내는 `Sequence<Int>` 타입처럼 [`Flow<Int>`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html) 타입을 사용할 수 있습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart               
fun simple(): Flow<Int> = flow { // flow builder
    for (i in 1..3) {
        delay(100) // pretend we are doing something useful here
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    // Launch a concurrent coroutine to check if the main thread is blocked
    launch {
        for (k in 1..3) {
            println("I'm not blocked $k")
            delay(100)
        }
    }
    // Collect the flow
    simple().collect { value -> println(value) } 
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-04.kt)에서 전체 코드를 볼 수 있습니다.

이 코드는 main 쓰레드를 차단하지 않고 각 숫자를 출력하기 전에 100ms 기다립니다. 이는 100ms마다 main 쓰레드에서 실행중인 별도의 코루틴에서 "I'm not blocked"이 출력되는 것으로 학인할 수 있습니다:

```text
I'm not blocked 1
1
I'm not blocked 2
2
I'm not blocked 3
3
```

앞의 예제에서 [Flow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html) 코드에서의 차이점은 아래와 같습니다:

* [Flow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html) 타입에 대한 빌더 함수를 [flow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow.html)라고 합니다.
* `flow { ... }`의 빌더 블럭은 중단될 수 있습니다.
* `simple()` 함수는 더이상 `suspend` 수식어를 붙이지 않습니다.
* [emit](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/emit.html) 함수를 사용하여 플로우에서 배출됩니다.
* [collect](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect.html) 함수를 사용하여 플로우에서 값을 수집할 수 있습니다.

> [delay](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/delay.html)를 `simple`의 `flow { ... }` 본문에 `Thread.sleep`으로 대체할 수 있습니다. 이경우 메인 쓰레드가 차단되는지 확인해봅시다.

## 플로우는 콜드 스트림 \(Flows are cold\)

플로우는 시퀀스와 같이 _콜드 스트림 \(cold stream\)_ 입니다 - [flow](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow.html) 빌더에 코드는 플로우가 수집될 때까지 실행되지 않습니다. 이는 아래 예제에서 확인할 수 있습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart      
fun simple(): Flow<Int> = flow { 
    println("Flow started")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    println("Calling simple function...")
    val flow = simple()
    println("Calling collect...")
    flow.collect { value -> println(value) } 
    println("Calling collect again...")
    flow.collect { value -> println(value) } 
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-05.kt)에서 전체 코드를 볼 수 있습니다.

출력결과는 아래와 같습니다:

```text
Calling simple function...
Calling collect...
Flow started
1
2
3
Calling collect again...
Flow started
1
2
3
```

이것은 플로우를 반환하는 `simple` 함수에 `suspend` 수식어를 붙이지 않는 중요한 이유입니다. 저절로 `simple()`는 빠르게 반환하여 아무것도 기다리지 않습니다. 플로우는 수집될 때마다 실행되기 때문에 다시 `collect`를 호출할 때 "Flow started" 출력문을 볼 수 있습니다.

## 플로우 취소 \(Flow cancellation\)

플로우는 코루틴의 취소 메커니즘을 따릅니다. 그러나 플로우 구조에서는 추가적인 취소 포인트에 대한 기능은 도입되지 않았습니다. 그래서 취소를 위해선 완벽하게 투명해야 합니다. 일반적으로 [delay](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/delay.html)와 같은 취소가 가능한 중지 함수에서 플로우가 중지 상태일 때 플로우 수집을 취소할 수 있으며 그외에는 취소할 수 없습니다.

다음 예제는 [withTimeoutOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-timeout-or-null.html) 블럭 안에서 실행할 때 타임아웃 시 플로우가 취소되고 코드 실행이 중지되는 것을 보여줍니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart           
fun simple(): Flow<Int> = flow { 
    for (i in 1..3) {
        delay(100)          
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    withTimeoutOrNull(250) { // Timeout after 250ms 
        simple().collect { value -> println(value) } 
    }
    println("Done")
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-06.kt)에서 전체 코드를 볼 수 있습니다.

아래 출력 결과와 같이 `simple()` 함수에서 2개의 숫자만 배출되는 것을 주목하십시오:

```text
Emitting 1
1
Emitting 2
2
Done
```

## 플로우 빌더 \(Flow builders\)

이전 예제에서 `flow { ... }` 빌더는 가장 일반적인 것입니다. 플로우 선언을 위한 다른 빌더들도 존재합니다:

* 고정된 값의 셋을 정의하는 [flowOf](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-of.html) 빌더.
* 다양한 콜렉션과 시퀀스는 `.asFlow()` 확장 함수를 이용하여 플로우로 변경할 수 있습니다.

플로우로 부터 1부터 3까지의 숫자를 출력하는 예제는 아래와 같이 표기할 수 있습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
//sampleStart
    // Convert an integer range to a flow
    (1..3).asFlow().collect { value -> println(value) }
//sampleEnd 
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-07.kt)에서 전체 코드를 볼 수 있습니다.

## 중간 플로우 연산자 \(Intermediate flow operators\)

플로우는 콜렉션과 시퀀스처럼 연산자를 이용하여 변환할 수 있습니다. 중간 연산자는 업스트림 플로우에 적용되고 다운스트림 플로우를 반환합니다. 이 연산자는 플로우와 같이 빠릅니다 \(cold\). 이러한 연산자의 호출은 중단 함수가 아닙니다. 빠르게 동작하고 변환된 새로운 플로우의 정의를 반환합니다.

기본 연산자는 [map](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html) 과 [filter](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/filter.html)와 같은 이름을 가지고 있습니다. 시퀀스와 중요한 차이점은 연산자 내부의 코드 블럭이 중단 함수를 호출할 수 있다는 것입니다.

예를 들어 요청을 수행하는 것이 일시 중단 함수로 구현되는 긴 실행 작업인 경우에도 수신요청의 플로우를 [map](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html) 연산자로 결과에 매핑할 수 있습니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart           
suspend fun performRequest(request: Int): String {
    delay(1000) // imitate long-running asynchronous work
    return "response $request"
}

fun main() = runBlocking<Unit> {
    (1..3).asFlow() // a flow of requests
        .map { request -> performRequest(request) }
        .collect { response -> println(response) }
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-08.kt)에서 전체 코드를 볼 수 있습니다.

실행결과는 아래와 같으며, 각 라인은 1초 후에 나타납니다:

```text
response 1
response 2
response 3
```

### 변환 연산자 \(Transform operator\)

플로우 변환 연산자 중 가장 일반적인 연산자는 [transform](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/transform.html) 입니다. 이것은 [map](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html) 과 [filter](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/filter.html)와 같이 단순한 변환 뿐만 아니라 복잡한 변환을 구현하는데 사용합니다. `transform` 연산자를 사용하여 임의의 횟수로 임의의 숫자를 [emit](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/emit.html) 할 수 있습니다.

예를 들어 `transform`을 사용하여 긴 작업이 실행되기 전에 문자열을 배출할 수 있고 이어서 응답을 처리할 수 있습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

suspend fun performRequest(request: Int): String {
    delay(1000) // imitate long-running asynchronous work
    return "response $request"
}

fun main() = runBlocking<Unit> {
//sampleStart
    (1..3).asFlow() // a flow of requests
        .transform { request ->
            emit("Making request $request") 
            emit(performRequest(request)) 
        }
        .collect { response -> println(response) }
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-09.kt)에서 전체 코드를 볼 수 있습니다.

실행 결과는 아래와 같습니다:

```text
Making request 1
response 1
Making request 2
response 2
Making request 3
response 3
```

### 크기 제한 연산자 \(Size-limiting operators\)

[take](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/take.html)와 같은 크기 제한 연산자는 플로우의 끝에 도달하면 실행을 취소합니다. 코루틴의 취소는 항상 예외를 발생하기 때문에 `try { ... } finally { ... }` 블럭같은 리소스 관리 함수는 이러한 상황에서 정상적으로 동작합니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
fun numbers(): Flow<Int> = flow {
    try {                          
        emit(1)
        emit(2) 
        println("This line will not execute")
        emit(3)    
    } finally {
        println("Finally in numbers")
    }
}

fun main() = runBlocking<Unit> {
    numbers() 
        .take(2) // take only the first two
        .collect { value -> println(value) }
}            
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-10.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 2개의 숫자를 배출후에 중지되는 `numbers()` 함수 안에 `flow { ... }` 바디의 실행을 명확하게 보여줍니다:

```text
1
2
Finally in numbers
```

## 터미널 플로우 연산자 \(Terminal flow operators\)

플로우에서 터미널 연산자는 플로우 수집을 시작하는 _중단 함수_ 입니다. [collect](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect.html) 연산자는 가장 기본적인 연산자이지만 더 쉽게 만들 수 있는 다른 터미널 연산자가 존재합니다:

* [toList](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/to-list.html) 와 [toSet](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/to-set.html)과 같이 다양한 콜렉션으로 변환.
* [first](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/first.html) 값을 가져오고 플로우가 [single](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/single.html) 값을 배출하도록 보장.
* [reduce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/reduce.html) 와 [fold](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/fold.html)를 통해 플로우의 값을 줄일 수 있음.

예:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
//sampleStart         
    val sum = (1..5).asFlow()
        .map { it * it } // squares of numbers from 1 to 5                           
        .reduce { a, b -> a + b } // sum them (terminal operator)
    println(sum)
//sampleEnd     
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-11.kt)에서 전체 코드를 볼 수 있습니다.

하나의 숫자가 출력됩니다:

```text
55
```

## 순차적 플로우 \(Flows are sequential\)

플로우에 각각의 콜렉션은 여러 플로우에 사용하는 특수한 연산자를 사용하기 전까지는 순차적으로 동작합니다. 콜렉션은 터미널 연산자를 호출한 코루틴에서 직접적으로 동작합니다. 기본적으로 새로운 코루틴은 실행되지 않습니다. 각각의 배출된 값은 업스트림에서 다운스트림까지 모든 중간 연산자에 의해 처리되고 터미널 연산자에게 전달됩니다.

아래의 예제는 짝수를 필터링 하는 것과 그것을 문자열로 매핑하는 것을 보여줍니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> {
//sampleStart         
    (1..5).asFlow()
        .filter {
            println("Filter $it")
            it % 2 == 0              
        }              
        .map { 
            println("Map $it")
            "string $it"
        }.collect { 
            println("Collect $it")
        }    
//sampleEnd                  
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-12.kt)에서 전체 코드를 볼 수 있습니다.

출력 결과:

```text
Filter 1
Filter 2
Map 2
Collect string 2
Filter 3
Filter 4
Map 4
Collect string 4
Filter 5
```

## 플로우 컨텍스트 \(Flow context\)

플로우의 콜렉션은 항상 코루틴에서 호출한 컨텍스트에서 발생합니다. 예를 들어 `simple` 플로우가 있는 경우 `simple` 플로우의 상세한 구현과 상관없이 코드의 작성자에 의해 특정 컨텍스트에서 실행됩니다:

```kotlin
withContext(context) {
    simple().collect { value ->
        println(value) // run in the specified context 
    }
}
```

이 플로우의 프로퍼티는 _컨텍스트 보전_ 이라 부릅니다.

기본적으로 `flow { ... }` 빌더에 코드는 플로우의 콜렉터가 제공하는 컨텍스트에서 실행됩니다. 예를 들어 호출된 쓰레드를 출력하고 3개의 숫자를 배출하는 `simple`를 구현한다고 생각해봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

//sampleStart
fun simple(): Flow<Int> = flow {
    log("Started simple flow")
    for (i in 1..3) {
        emit(i)
    }
}  

fun main() = runBlocking<Unit> {
    simple().collect { value -> log("Collected $value") } 
}          
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-13.kt)에서 전체 코드를 볼 수 있습니다.

실행결과는 아래와 같습니다:

```text
[main @coroutine#1] Started simple flow
[main @coroutine#1] Collected 1
[main @coroutine#1] Collected 2
[main @coroutine#1] Collected 3
```

`simple().collect`는 메인 쓰레드에서 호출 되고 `simple` 플로우 바디도 메인 쓰레드에서 호출됩니다. 실행된 컨텍스트에 대해 상관하지 않고 호출자를 중단하지 않는 빠른 실행 또는 비동기 코드의 완벽한 기본입니다.

### withContext의 잘못된 배출 \(Wrong emission withContext\)

그러나 긴 시간 CPU를 소모하는 코드는 [Dispatchers.Default](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html)의 컨텍스트에서 실행되야 하며 UI 업데이트 코드는 [Dispatchers.Main](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html)의 컨텍스트에서 실행되어야 합니다. 대게, [withContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)은 Kotlin 코루틴을 사용하는 코드에서 컨텍스트를 바꾸기위해 사용하지만 `flow { ... }` 빌더의 코드는 컨텍스트 보존 특성을 준수해야 하며 다른 컨텍스트에서 [emit](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/emit.html) 할 수 없습니다.

다음 코드를 실행해 봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
fun simple(): Flow<Int> = flow {
    // The WRONG way to change context for CPU-consuming code in flow builder
    kotlinx.coroutines.withContext(Dispatchers.Default) {
        for (i in 1..3) {
            Thread.sleep(100) // pretend we are computing it in CPU-consuming way
            emit(i) // emit next value
        }
    }
}

fun main() = runBlocking<Unit> {
    simple().collect { value -> println(value) } 
}              
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-14.kt)에서 전체 코드를 볼 수 있습니다.

이 코드는 실행결과 아래와 같은 예외를 발생합니다:

```text
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
        Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@5511c7f8, BlockingEventLoop@2eac3323],
        but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@2dae0000, DefaultDispatcher].
        Please refer to 'flow' documentation or use 'flowOn' instead
    at ...
```

### flowOn 연산자 \(flowOn operator\)

위의 예외는 플로우의 배출 컨텍스트를 변경하기 위해선 [flowOn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-on.html) 함수를 사용하라고 제공하고 있습니다. 플로우의 컨텍스트를 변경하는 올바른 방법은 아래 예제에 있습니다. 이 예제는 쓰레드의 이름을 출력하여 어떻게 동작하는지 나타내줍니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

//sampleStart
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        Thread.sleep(100) // pretend we are computing it in CPU-consuming way
        log("Emitting $i")
        emit(i) // emit next value
    }
}.flowOn(Dispatchers.Default) // RIGHT way to change context for CPU-consuming code in flow builder

fun main() = runBlocking<Unit> {
    simple().collect { value ->
        log("Collected $value") 
    } 
}            
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-15.kt)에서 전체 코드를 볼 수 있습니다.

`flow { ... }`는 백그라운드 쓰레드에서 동작하는 반면에 콜렉션은 메인 쓰레드에서 동작합니다:

또 다른점은 [flowOn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-on.html) 연산자가 플로우의 기본 순차 특성을 변경 한 것입니다. 수집은 한 코루틴 \("coroutine\#1"\)에서 일어나고 배출은 수집 코루틴과 동시에 다른 코루틴 \("coroutine\#2"\)에서 일어납니다. [flowOn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-on.html) 연산자는 컨텍스트에서 [CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html)을 변경하려고 할 때 업스트림 플로우를 위한 다른 코루틴을 생성합니다.

## 버퍼링 \(Buffering\)

다른 코루틴에서 다른 플로우의 부분을 실행하는 것은 플로우를 수집하는 데 걸리는 전체 시간, 특히 긴 시간이 걸리는 비동기적 작업이 관련된 경우 전체 시간 관점에서 도움이 될 수 있습니다. 예를 들어 `simple()` 플로우에 의해 배출되는 작업이 느려서 하나의 요소를 처리하는데 100ms가 걸리고 콜렉터도 느려서 요소를 처리하는데 300ms가 걸린다고 생각해봅시다. 3개의 숫자를 수집하는데 얼마의 시간이 걸리는지 확인해 봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.*

//sampleStart
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
    val time = measureTimeMillis {
        simple().collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
    }   
    println("Collected in $time ms")
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-16.kt)에서 전체 코드를 볼 수 있습니다.

각 400ms의 시간이 걸리므로 3개의 숫자를 수집하는데 대략 1200ms의 시간이 소요되는 것을 확인할 수 있습니다:

```text
1
2
3
Collected in 1220 ms
```

플로우에서 [buffer](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/buffer.html) 연산자를 사용하여 수집 코드와 동시에 `simple()`의 배출 코드를 실행할 수 있습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
//sampleStart
    val time = measureTimeMillis {
    simple()
        .buffer() // buffer emissions, don't wait
        .collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
    }   
    println("Collected in $time ms")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-17.kt)에서 전체 코드를 볼 수 있습니다.

실행결과 효과적인 파이프라인을 생성하여 출력결과는 같지만 더 빠릅니다. 첫번째 숫자를 위해 100ms만 기다리고 다음 진행을 위해 300ms만 기다리고 각 숫자를 출력합니다. 이렇게 하면 약 1000ms가 소요되는 것을 확인할 수 있습니다:

```text
1
2
3
Collected in 1071 ms
```

> [flowOn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-on.html) 연산자는 [CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-dispatcher/index.html)을 변경하는데 같은 버퍼링 메커니즘을 사용하지만 여기서 실행하는 컨텍스트를 변경하지 않고 명시적으로 버퍼링을 요청한 것을 유의해야 합니다.

### 융합 \(Conflation\)

플로우가 동작 또는 동작 상태의 업데이트 결과를 부분적으로 나타낼 때 각 단계의 값을 처리할 필요가 없고 가장 최근의 값만 처리하면 될 경우가 있습니다. 이러한 경우 [conflate](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/conflate.html) 연산자는 수집하는 처리가 너무 느릴 때 중간 값을 건너뛸 수 있습니다. 이전 예제를 변경하여 다시 살펴봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
//sampleStart
    val time = measureTimeMillis {
    simple()
        .conflate() // conflate emissions, don't process each one
        .collect { value -> 
            delay(300) // pretend we are processing it for 300 ms
            println(value) 
        } 
    }   
    println("Collected in $time ms")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-18.kt)에서 전체 코드를 볼 수 있습니다.

첫번째 숫자가 처리되는 동안 두번째 숫자는 이미 처리되었으며 세번째 숫자는 이미 생성되었으므로 두번째 숫자는 _융합 \(conflated\)_ 되고 가장 최근 숫자인 세번째 숫자만 수집기에 전달됩니다:

```text
1
3
Collected in 758 ms
```

### 최근 값 처리 \(Processing the latest value\)

융합은 배출과 수집이 느릴 때 처리속도를 높이는 방법중에 하나입니다. 융합은 배출된 값을 드랍하면서 동작합니다. 다른 방법으로는 느린 수집기를 취소하고 새로운 값이 배출될 때마다 재시작하는 것입니다. `xxx` 연산자와 동일한 필수 논리를 수행하지만 새로운 값으로 블록의 코드를 취소하는 `xxxLatest` 연산자가 있습니다. 이전 예제에서 [conflate](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/conflate.html)을 [collectLatest](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect-latest.html)으로 변경해봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*
import kotlin.system.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100) // pretend we are asynchronously waiting 100 ms
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> { 
//sampleStart
    val time = measureTimeMillis {
    simple()
        .collectLatest { value -> // cancel & restart on the latest value
            println("Collecting $value") 
            delay(300) // pretend we are processing it for 300 ms
            println("Done $value") 
        } 
    }   
    println("Collected in $time ms")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-19.kt)에서 전체 코드를 볼 수 있습니다.

[collectLatest](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect-latest.html)의 바디는 300ms가 걸리지만 100ms마다 새로운 값을 배출하는데 모든 값에 대해 실행되지만 마지막 값에 대해서만 완료가 되는 것을 알 수 있습니다:

```text
Collecting 1
Collecting 2
Collecting 3
Done 3
Collected in 741 ms
```

## 다중 플로우 구성 \(Composing multiple flows\)

다중 플로우를 구성하는 여러가지 방법이 있습니다.

### Zip

Kotlin 표준 라이브러리에 있는 [Sequence.zip](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/zip.html) 확장 함수와 같이 플로우는 2개의 플로우의 값을 합치는 [zip](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/zip.html) 연산자를 가지고 있습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> { 
//sampleStart                                                                           
    val nums = (1..3).asFlow() // numbers 1..3
    val strs = flowOf("one", "two", "three") // strings 
    nums.zip(strs) { a, b -> "$a -> $b" } // compose a single string
        .collect { println(it) } // collect and print
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-20.kt)에서 전체 코드를 볼 수 있습니다.

이 예제의 출력값은 아래와 같습니다:

```text
1 -> one
2 -> two
3 -> three
```

### Combine

플로우가 변수나 연산의 가장 최근의 값을 나타낼 때 해당 플로우의 가장 최근 값에 따라 계산을 수행하고 업스트림 플로우가 값을 배출할 때마다 다시 계산해야 합니다. 이러한 연산자를 [combine](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html)이라 합니다.

예를 들어 이전 예제의 숫자가 300ms 마다 업데이트 되지만 문자열이 400ms 마다 업데이트 되는 경우 [zip](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/zip.html) 연산자를 사용하여 압축하면 400ms 마다 출력되지만 동일한 결과를 보여줍니다:

> 예제에서 [onEach](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-each.html) 중간 연산자를 사용하여 각 요소를 지연시키고 플로우를 배출하는 코드를 더 짧게 나타낼 수 있습니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> { 
//sampleStart                                                                           
    val nums = (1..3).asFlow().onEach { delay(300) } // numbers 1..3 every 300 ms
    val strs = flowOf("one", "two", "three").onEach { delay(400) } // strings every 400 ms
    val startTime = System.currentTimeMillis() // remember the start time 
    nums.zip(strs) { a, b -> "$a -> $b" } // compose a single string with "zip"
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-21.kt)에서 전체 코드를 볼 수 있습니다.

그러나 [zip](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/zip.html) 대신 [combine](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html) 연산자를 사용하는 경우는 아래와 같습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking<Unit> { 
//sampleStart                                                                           
    val nums = (1..3).asFlow().onEach { delay(300) } // numbers 1..3 every 300 ms
    val strs = flowOf("one", "two", "three").onEach { delay(400) } // strings every 400 ms          
    val startTime = System.currentTimeMillis() // remember the start time 
    nums.combine(strs) { a, b -> "$a -> $b" } // compose a single string with "combine"
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-22.kt)에서 전체 코드를 볼 수 있습니다.

실행결과 다른 결과값이 나오는 것을 확인할 수 있습니다. `nums` 또는 `strs` 플로우의 각 배출마다 출력됩니다:

```text
1 -> one at 452 ms from start
2 -> one at 651 ms from start
2 -> two at 854 ms from start
3 -> two at 952 ms from start
3 -> three at 1256 ms from start
```

## Flattening flows

플로우는 비동기적으로 수신된 값의 시퀀스를 표기합니다. 그래서 각 값이 다른 값의 시퀀스에 대한 요청을 트리거 하는 것은 쉽습니다. 예를 들어 500ms 간격으로 2개의 문자열의 플로우를 반환하는 함수가 있다고 가정해 봅시다:

```kotlin
fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}
```

3개의 정수를 가지고 있는 플로우가 있고 각각 `requestFlow`를 호출할 경우를 살펴 봅시다:

```kotlin
(1..3).asFlow().map { requestFlow(it) }
```

그런 다음 추가 처리를 위해 하나의 플로우로 _병합 \(flattened\)_ 되어야 할 플로우를 가집니다. 콜렉션과 시퀀스는 이것을 위해 [flatten](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/flatten.html) 와 [flatMap](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/flat-map.html) 연산자가 있습니다. 그러나 플로우의 비동기 특성으로 인해 서로 다른 병함 모드가 필요합니다. 아래에서 플로우에 적합한 병합 연산자를 알아봅시다.

### flatMapConcat

연속 모드는 [flatMapConcat](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-concat.html) 와 [flattenConcat](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flatten-concat.html) 연산자로 구현됩니다. 이것은 시퀀스 연산자와 가장 직접적으로 유사성을 가집니다. 다음 예에서 알 수 있듯이 수집 작업을 시작하기 전에 내부 플로우가 완료될 때까지 기다립니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
//sampleStart
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapConcat { requestFlow(it) }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-23.kt)에서 전체 코드를 볼 수 있습니다.

[flatMapConcat](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-concat.html)의 순차적인 특성을 출력에서 확인할 수 있습니다:

```text
1: First at 121 ms from start
1: Second at 622 ms from start
2: First at 727 ms from start
2: Second at 1227 ms from start
3: First at 1328 ms from start
3: Second at 1829 ms from start
```

### flatMapMerge

다른 병합 모드는 유입되는 플로우를 수집하고 값들을 하나의 플로우로 합친 후 가능한 빨리 값을 배출하는 방식입니다. 이것은 [flatMapMerge](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-merge.html) 와 [flattenMerge](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flatten-merge.html) 연산자로 구현되어 있습니다. 이 2개의 연산자는 같은 시간에 수집하는 플로우를 제한할 수 있는 `concurrency` 파라미터 \(기본값은 [DEFAULT\_CONCURRENCY](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-d-e-f-a-u-l-t_-c-o-n-c-u-r-r-e-n-c-y.html)\)를 옵션으로 가지고 있습니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
//sampleStart
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapMerge { requestFlow(it) }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-24.kt)에서 전체 코드를 볼 수 있습니다.

[flatMapMerge](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-merge.html)의 동시성은 명확하게 나타납니다:

```text
1: First at 136 ms from start
2: First at 231 ms from start
3: First at 333 ms from start
1: Second at 639 ms from start
2: Second at 732 ms from start
3: Second at 833 ms from start
```

> [flatMapMerge](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-merge.html)은 코드의 블럭 \(예제에서 `{ requestFlow(it) }`\)을 순차적으로 호출하지만 결과 플로우를 동시에 수집하고 이것은 `map { requestFlow(it) }`을 먼저 호출하고 [flattenMerge](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flatten-merge.html)을 호출한 것과 같은 결과를 가집니다.

### flatMapLatest

\["Processing the latest value"\] 섹션에서 봤던 [collectLatest](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect-latest.html) 연산자와 비슷한 방식으로 "최신" 병합 모드가 있으며 새로운 플로우가 배출되자마자 이전 플로우 수집이 취소됩니다. 이것은 [flatMapLatest](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-latest.html) 연산자로 구현되어 있습니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First") 
    delay(500) // wait 500 ms
    emit("$i: Second")    
}

fun main() = runBlocking<Unit> { 
//sampleStart
    val startTime = System.currentTimeMillis() // remember the start time 
    (1..3).asFlow().onEach { delay(100) } // a number every 100 ms 
        .flatMapLatest { requestFlow(it) }                                                                           
        .collect { value -> // collect and print 
            println("$value at ${System.currentTimeMillis() - startTime} ms from start") 
        } 
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-25.kt)에서 전체 코드를 볼 수 있습니다.

이 예제의 출력은 [flatMapLatest](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-latest.html)이 어떻게 동작하는지 잘 보여줍니다:

```text
1: First at 142 ms from start
2: First at 322 ms from start
3: First at 425 ms from start
3: Second at 931 ms from start
```

> [flatMapLatest](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-latest.html)은 새로운 값에 대한 모든 블럭에 코드 \(이 예제에서는 `{ requestFlow(it) }`\)를 취소합니다. 이 예제에서는 `requestFlow` 호출이 빠르고 중단되지 않으며 취소할 수 없으므로 차이점이 나타나진 않지만 `delay`와 같은 중단 함수를 쓰면 차이점이 나타납니다.

## Flow exceptions

플로우 수집은 연산자 내부의 배출 또는 코드에서 예외가 발생하면 예외를 발생해 완료할 수 있습니다. 예외를 다루는 몇가지 방법이 있습니다.

### Collector try and catch

수집기는 예외를 다루기 위해 Kotlin의 [`try/catch`](https://kotlinlang.org/docs/reference/exceptions.html) 블럭을 사용할 수 있습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i) // emit next value
    }
}

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value ->         
            println(value)
            check(value <= 1) { "Collected $value" }
        }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}        
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-26.kt)에서 전체 코드를 볼 수 있습니다.

이 코드는 [collect](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect.html) 터미널 연산자에서 예외를 성공적으로 처리하고 더이상 값을 배출하지 않습니다:

```text
Emitting 1
1
Emitting 2
2
Caught java.lang.IllegalStateException: Collected 2
```

### Everything is caught

앞의 예제는 실질적으로 배출 또는 어떠한 중간 연산 또는 터미널 연산에서 발생한 예외를 모두 처리합니다. 예를 들어 배출된 값을 문자열로 [mapped](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html) 변경되도록 바꿀 수 있지만 예외를 발생합니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } catch (e: Throwable) {
        println("Caught $e")
    } 
}            
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-27.kt)에서 전체 코드를 볼 수 있습니다.

이 예외는 여전히 처리되고 수집은 중지됩니다:

```text
Emitting 1
string 1
Emitting 2
Caught java.lang.IllegalStateException: Crashed on 2
```

## 예외의 투명성 \(Exception transparency\)

그러나 배출 코드를 어떻게 예외 처리 동작을 캡슐화 할 수 있을까요?

플로우는 _예외에 대해 투명_ 해야 하고 `try/catch` 블럭 안에서 `flow { ... }` 빌더에 값을 [emit](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/emit.html) 하는 것은 예외 투명성에 위배됩니다. 이는 예외를 발생하는 수집기는 이전 예제에서 처럼 `try/catch`를 사용하여 항상 예외를 핸들링 할 수 있는걸 보장합니다.

배출은 예외 투명성을 보존하고 예외 처리를 캡슐화 할 수 있는 [catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html) 연산자를 사용할 수 있습니다. `catch` 연산자의 블럭은 예외를 분석할 수 있고 예외에 따라 다른 방식으로 대응할 수 있습니다:

* 예외는 `throw`를 사용하여 다시 던질 수 있습니다.
* 예외는 [catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html) 바디에서 [emit](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/emit.html)을 사용하여 값을 배출하는 것으로 전환할 수 있습니다.
* 일부 다른 코드에서 예외는 무시, 기록 또는 처리될 수 있습니다.

예를 들어 예외를 처리할 때 문자를 배출한다고 해봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<String> = 
    flow {
        for (i in 1..3) {
            println("Emitting $i")
            emit(i) // emit next value
        }
    }
    .map { value ->
        check(value <= 1) { "Crashed on $value" }                 
        "string $value"
    }

fun main() = runBlocking<Unit> {
//sampleStart
    simple()
        .catch { e -> emit("Caught $e") } // emit on exception
        .collect { value -> println(value) }
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-28.kt)에서 전체 코드를 볼 수 있습니다.

예제의 출력은 `try/catch`가 없음에도 같습니다.

### Transparent catch

예외의 투명성을 보장하는 [catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html) 중간 연산자는 업스트림 예외 \(`catch` 위의 모든 연산자에서의 예외\)만 포착합니다. `catch` 아래에 위치한 `collect { ... }` 블럭 에서 예외를 발생하면 이것은 `catch`의 영역을 벗어납니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    simple()
        .catch { e -> println("Caught $e") } // does not catch downstream exceptions
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}            
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-29.kt)에서 전체 코드를 볼 수 있습니다.

`catch` 연산자가 있음에도 "Caught ..." 메세지가 출력되지 않음을 확인할 수 있습니다:

```text
Emitting 1
1
Emitting 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
    at ...
```

### Catching declaratively

[collect](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect.html) 연산자의 바디를 [onEach](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-each.html)로 이동하고 `catch` 연산자 앞에 두어 [catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html) 연산자로 모든 예외를 처리할 수 있게 선언할 수 있습니다. 플로우의 콜렉션은 파라미터 없는 `collect()`를 호출해야 합니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = flow {
    for (i in 1..3) {
        println("Emitting $i")
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
//sampleStart
    simple()
        .onEach { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
        .catch { e -> println("Caught $e") }
        .collect()
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-30.kt)에서 전체 코드를 볼 수 있습니다.

이제 "Caught ..." 메세지가 출력되는 것을 확인할 수 있고 `try/catch` 블럭 없이 모든 예외를 처리할 수 있습니다:

```text
Emitting 1
1
Emitting 2
Caught java.lang.IllegalStateException: Collected 2
```

## 플로우 완료 \(Flow completion\)

일반적이거나 예외를 발생시켜 플로우 수집이 완료될 때 어떠한 액션을 실행해야 할 수도 있습니다. 이미 알고 있듯이 명령형 또는 선언형의 두가지 방법이 있습니다.

### 명령적 완료 블럭 \(Imperative finally block\)

`try`/`catch` 외에도 수집기는 `collect` 완료 시 `finally` 블럭을 사용하여 작업을 실행할 수 있습니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("Done")
    }
}            
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-31.kt)에서 전체 코드를 볼 수 있습니다.

이 코드는 `simple()`를 통해 1부터 3까지의 숫자를 출력하고 "Done" 문자열을 출력합니다:

```text
1
2
3
Done
```

### 선언적 처리 \(Declarative handling\)

선언적 접근방식의 경우 플로우는 플로우가 완전히 수집되었을 때 실행하는 [onCompletion](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html) 중간 연산자를 가지고 있습니다.

이전 예제는 [onCompletion](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html)을 사용하여 다시 작성할 수 있고 출력결과가 같음을 확인할 수 있습니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
//sampleStart
    simple()
        .onCompletion { println("Done") }
        .collect { value -> println(value) }
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-32.kt)에서 전체 코드를 볼 수 있습니다.

[onCompletion](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html)의 주요 장점은 플로우 수집이 정상적으로 완료되었는지 아니면 예외가 발생했는지를 나타내는 람다의 nullable `Throwable` 파라미터 입니다. 다음 예제의 `simple()` 플로우는 숫자 1을 배출하고 예외를 발생합니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
fun simple(): Flow<Int> = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> if (cause != null) println("Flow completed exceptionally") }
        .catch { cause -> println("Caught exception") }
        .collect { value -> println(value) }
}            
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-33.kt)에서 전체 코드를 볼 수 있습니다.

예상대로 아래와 같이 출력됩니다:

```text
1
Flow completed exceptionally
Caught exception
```

[onCompletion](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html) 연산자는 [catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html)와 다르게 예외를 처리하지 않습니다. 위 예제 코드에서 봤듯이 예외는 여전히 다운 스트림으로 진행됩니다. 예외는 `onCompletion` 연산자에 전달되어 지고 `catch` 연산자로 처리할 수 있습니다.

### Successful completion

[catch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html) 연산자와 또다른 차이점은 [onCompletion](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html)은 모든 예외를 살펴보고 업스트림 플로우에 취소 또는 실패없이 성공한 완료에 대해서만 `null` 예외를 받습니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking<Unit> {
    simple()
        .onCompletion { cause -> println("Flow completed with $cause") }
        .collect { value ->
            check(value <= 1) { "Collected $value" }                 
            println(value) 
        }
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-34.kt)에서 전체 코드를 볼 수 있습니다.

다운스트림 예외로 인해 플로우가 중단되었기 때문에 null 이 아닌 완료가 발생된 것을 알 수 있습니다:

```text
1
Flow completed with java.lang.IllegalStateException: Collected 2
Exception in thread "main" java.lang.IllegalStateException: Collected 2
```

## 명령적 vs 선언적 \(Imperative versus declarative\)

플로우를 수집하는 방법을 알고 명령적인 방식과 선언적인 방식에서의 완료와 예외를 처리하는 방법을 알고 있습니다. 어떠한 방식이 더 선호하며 왜 그런가? 에 대한 질문을 던질 수 있습니다. 라이브러리로서 특정 접근법을 옹호하지 않으며 두 옵션이 모두 유효하며 자신의 선호와 코드 스타일에 따라 선택해야 한다고 생각합니다.

## 실행 플로우 \(Launching flow\)

이것은 플로우를 사용하여 어떠한 소스로부터 오는 비동기적 이벤트를 표기하기 쉽게 해줍니다. 이러한 경우 들어오는 이벤트의 반응을 코드에 등록하고 계속해서 작업을 진행하는 `addEventListener` 함수의 아날로그가 필요합니다. [onEach](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-each.html) 연산자가 이 역할을 수행할 수 있습니다. 그러나 `onEach`는 중간 연산자 입니다. 플로우를 수집하기위해 터미널 연산자가 필요합니다. 그렇지 않으면 `onEach`는 아무런 효과가 없습니다.

`onEach` 후에 [collect](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/collect.html) 터미널 연산자를 사용하면 플로우가 수집될 때까지 기다리게 됩니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

//sampleStart
// Imitate a flow of events
fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .collect() // <--- Collecting the flow waits
    println("Done")
}            
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-35.kt)에서 전체 코드를 볼 수 있습니다.

실행 시 아래와 같은 출력 결과를 볼 수 있습니다:

```text
Event: 1
Event: 2
Event: 3
Done
```

이럴 때 [launchIn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/launch-in.html) 터미널 연산자가 유용합니다. `collect`를 `launchIn`으로 대체를 통해 별도의 코루틴에서 플로우의 수집을 실행할 수 있기 때문에 추가적인 코드는 즉각적으로 실행됩니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

// Imitate a flow of events
fun events(): Flow<Int> = (1..3).asFlow().onEach { delay(100) }

//sampleStart
fun main() = runBlocking<Unit> {
    events()
        .onEach { event -> println("Event: $event") }
        .launchIn(this) // <--- Launching the flow in a separate coroutine
    println("Done")
}            
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-flow-36.kt)에서 전체 코드를 볼 수 있습니다.

실행결과 출력은 아래와 같습니다:

```text
Done
Event: 1
Event: 2
Event: 3
```

`launchIn`에 필요한 파라미터는 플로우를 수집하기 위해 코루틴이 시작되는 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)을 지정해야 합니다. 위의 예제에서 스코프는 [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 코루틴 빌더로 부터 시작되므로 플로우가 실행되는 동안 이 [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 스코프는 하위 코루틴이 완료되길 기다리며 main 함수가 반환하거나 종료되지 않도록 합니다.

실제 애플리케이션에서 스코프는 제한된 라이프타임을 가지는 엔티티에서 시작됩니다. 이 엔티티의 라이프타임이 종료되자마자 스코프는 취소되고 해당 플로우의 수집도 취소됩니다. 이렇게 되면 `onEach { ... }.launchIn(scope)`은 `addEventListener` 처럼 동작합니다. 그러나 취소와 구조적 동시성이 이 목적을 수행하므로 `removeEventListener` 함수는 필요하지 않습니다.

[launchIn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/launch-in.html)은 또한 전체 스코프나 [join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html) 없이 해당 플로우 수집 코루틴을 [cancel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/cancel.html) 하는데 사용할 수 있는 [Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html)도 반환합니다.

## 플로우와 리액티브 스트림 \(Flow and Reactive Streams\)

[Reactive Streams](https://www.reactive-streams.org/) 또는 RxJava와 Reactor 프로젝트와 같은 반응성 프레임워크에 익숙한 사용자에게는 플로우 디자인이 매우 친숙하게 다가올 것입니다.

실제로 이것의 디자인은 Reactive Streams와 다양한 구현에서 영감을 얻었습니다. 그러나 플로우의 주요 목표는 가능한 단순한 디자인을 유지하고 Kotlin과 suspension 친화적이며 구조적 동시성을 유지하는 것입니다. 이러한 목표는 적극적으로 반영하고 끊임없는 노력 없이는 불가능합니다. [Reactive Streams and Kotlin Flows](https://medium.com/@elizarov/reactive-streams-and-kotlin-flows-bfd12772cda4) 기사에서 더 자세한 이야기를 살펴 볼 수 있습니다.

개념적으로 다르지만 플로우는 반응형 스트림이며 반응형 \(스펙 및 TCK 호환\) Publisher로 변환하거나 그 반대로 변환할 수 있습니다. 이러한 변환기는 `kotlinx.coroutines`에서 제공되고 해당되는 반응형 모듈 \(반응형 스트림을 위한 `kotlinx-coroutines-reactive`, Project Reactor를 위한 `kotlinx-coroutines-reactor`, RxJava2/RxJava3를 위한 `kotlinx-coroutines-rx2`/`kotlinx-coroutines-rx3`\)을 찾을 수 있습니다. 통합 모듈은 `Flow`에서 `Flow`로의 변환, 반응형 `Context`와 다양한 반응형 엔티티를 가지고 작업할 수 있는 suspension 친화적 통합이 포함되어 있습니다.

