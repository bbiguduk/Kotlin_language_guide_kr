# 채널 \(Channels\)

거치된 값은 코루틴 간에 단일 값을 전달하기 위해 편한 방법을 제공합니다. 채널은 값의 스트림을 전달하기 위한 방법을 제공합니다.

## 채널 기본 \(Channel basics\)

[Channel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel/index.html)은 컨셉적으로 `BlockingQueue`와 아주 유사합니다. 중요한 한가지 차이점은 블럭킹 `put` 연산자 대신에 서스펜딩 [send](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/send.html)을 그리고 블럭킹 `take` 연산자 대신에 서스펜딩 [receive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html)가 있습니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
//sampleStart
    val channel = Channel<Int>()
    launch {
        // this might be heavy CPU-consuming computation or async logic, we'll just send five squares
        for (x in 1..5) channel.send(x * x)
    }
    // here we print five received integers:
    repeat(5) { println(channel.receive()) }
    println("Done!")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-01.kt)에서 전체 코드를 볼 수 있습니다..

이 코드의 출력은 아래와 같습니다:

```text
1
4
9
16
25
Done!
```

## 채널을 통한 닫기 및 반복 \(Closing and iteration over channels\)

큐와 다르게 채널을 닫아 더이상 요소가 오지 않음을 알 수 있습니다. 수신부 측에서는 일반적인 `for` 루프를 사용하여 채널로부터 요소를 받는것이 편리합니다.

개념적으로 [close](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/close.html)는 채널에 특별한 닫기 토근을 보내는 것과 같습니다. 닫기 토큰이 수신되자마자 반복은 중지되므로 닫기를 수신하기 전에 받은 모든 요소는 보장됩니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
//sampleStart
    val channel = Channel<Int>()
    launch {
        for (x in 1..5) channel.send(x * x)
        channel.close() // we're done sending
    }
    // here we print received values using `for` loop (until the channel is closed)
    for (y in channel) println(y)
    println("Done!")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-02.kt)에서 전체 코드를 볼 수 있습니다.

## 채널 생성 \(Building channel producers\)

코루틴에서 요소를 생성하는 패턴은 흔한 패턴입니다. 이것은 종종 동시 코드에서 발견되는 _생산자-소비자 \(producer-consumer\)_ 패턴의 일부분입니다. 생산자를 채널을 파라미터로 사용하는 함수로 추상화 할 수 있지만 결과가 반드시 반환되어야 한다는 점에서 어긋납니다.

[produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html)라는 코루틴 빌더가 생산자 측에서 쉽게 수행할 수 있도록 하고 확장 함수 [consumeEach](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/consume-each.html)은 소비자 측에서 `for` 루프를 대신합니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceSquares(): ReceiveChannel<Int> = produce {
    for (x in 1..5) send(x * x)
}

fun main() = runBlocking {
//sampleStart
    val squares = produceSquares()
    squares.consumeEach { println(it) }
    println("Done!")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-03.kt)에서 전체 코드를 볼 수 있습니다.

## 파이프라인 \(Pipelines\)

파이프라인은 하나의 코루틴이 무한한 값의 스트림을 생성하는 패턴입니다:

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++) // infinite stream of integers starting from 1
}
```

그리고 다른 코루틴이나 코루틴들은 스트림을 소비하고 어떠한 처리를 하고 다른 결과를 만들어 냅니다. 숫자를 제곱하는 아래 예를 참고해 봅시다:

```kotlin
fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
    for (x in numbers) send(x * x)
}
```

main 코드는 시작하고 전체 파이프라인을 연결합니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
//sampleStart
    val numbers = produceNumbers() // produces integers from 1 and on
    val squares = square(numbers) // squares integers
    repeat(5) {
        println(squares.receive()) // print first five
    }
    println("Done!") // we are done
    coroutineContext.cancelChildren() // cancel children coroutines
//sampleEnd
}

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++) // infinite stream of integers starting from 1
}

fun CoroutineScope.square(numbers: ReceiveChannel<Int>): ReceiveChannel<Int> = produce {
    for (x in numbers) send(x * x)
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-04.kt)에서 전체 코드를 볼 수 있습니다.

> 코루틴을 생성하는 모든 함수는 [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html)에서 확장으로 정의되므로 [structured concurrency](https://kotlinlang.org/docs/reference/coroutines/composing-suspending-functions.html#structured-concurrency-with-async)을 사용하여 애플리케이션에 남아있는 글로벌 코루틴이 없는지 확인할 수 있습니다.

## 파이프라인이 있는 소수 \(Prime numbers with pipeline\)

코루틴 파이프라인을 이용하여 소수를 생성하는 예제를 통해 파이프라인을 최대한 활용해 봅시다. 무한한 숫자 시퀀스로 시작합니다.

```kotlin
fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start
    while (true) send(x++) // infinite stream of integers from start
}
```

다음 파이프라인 단계는 들어오는 숫자 스트림을 필터링하여 주어진 소수로 나눌 수 있는 모든 숫자를 제거합니다:

```kotlin
fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int) = produce<Int> {
    for (x in numbers) if (x % prime != 0) send(x)
}
```

숫자 2부터 숫자 스트림을 시작하고 현재 채널에서 소수를 가져오고 각각의 소수에 대해 새로운 파이프라인을 실행함으로써 파이프라인을 빌드할 수 있습니다:

```text
numbersFrom(2) -> filter(2) -> filter(3) -> filter(5) -> filter(7) ...
```

아래 예제는 10개의 소수를 출력하고 main 쓰레드에 컨텍스트에서 전체 파이프라인을 실행합니다. 모든 코루틴이 main [runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/run-blocking.html) 코루틴의 스코프에서 실행되기 때문에 시작된 모든 코루틴의 리스트를 명시적으로 가지고 있을 필요가 없습니다. 10개의 소수를 출력하고 모든 자식 코루틴을 취소하기 위해 [cancelChildren](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/kotlin.coroutines.-coroutine-context/cancel-children.html) 확장 함수를 사용합니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
//sampleStart
    var cur = numbersFrom(2)
    repeat(10) {
        val prime = cur.receive()
        println(prime)
        cur = filter(cur, prime)
    }
    coroutineContext.cancelChildren() // cancel all children to let main finish
//sampleEnd    
}

fun CoroutineScope.numbersFrom(start: Int) = produce<Int> {
    var x = start
    while (true) send(x++) // infinite stream of integers from start
}

fun CoroutineScope.filter(numbers: ReceiveChannel<Int>, prime: Int) = produce<Int> {
    for (x in numbers) if (x % prime != 0) send(x)
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-05.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 아래와 같습니다:

```text
2
3
5
7
11
13
17
19
23
29
```

표준 라이브러리에 [`iterator`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/iterator.html) 코루틴 빌더를 이용하여 동일한 파이프라인을 만들 수 있습니다. `produce`는 `iterator`로 `send`는 `yield`로 `receive`는 `next`로 `ReceiveChannel`을 `Iterator`로 변경하고 코루틴 스코프를 제거하십시오. `runBlocking`도 필요하지 않습니다. 그러나 위에 보여진대로 채널을 사용하는 파이프라인의 이점은 [Dispatchers.Default](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-default.html) 컨텍스트에서 실행한다면 여러개의 CPU 코어를 사용할 수 있습니다.

어찌 됐든 이렇게 소수를 찾는 방법은 비효율적입니다. 실제로 파이프라인에는 다른 일시 중단 호출 \(원격 서비스를 위한 비동기적 호출\)이 포함되며 이러한 파이프라인은 완전 비동기적인 `produce`와 달리 임의의 서스펜션을 허용하지 않기 때문에 `sequence`/`iterator`을 이용하여 만들 수 없습니다.

## Fan-out

여러 코루틴이 동일한 채널에서 수신되어 서로간에 작업을 분배할 수 있습니다. 주기적으로 정수 \(초당 10개의 숫자\)를 생성하는 코루틴으로 시작합니다:

```kotlin
fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1 // start from 1
    while (true) {
        send(x++) // produce next
        delay(100) // wait 0.1s
    }
}
```

그러면 몇개의 프로세서 코루틴을 얻을 수 있습니다. 여기 예제에서 ID와 수신된 숫자를 출력합니다:

```kotlin
fun CoroutineScope.launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    for (msg in channel) {
        println("Processor #$id received $msg")
    }    
}
```

이제 5개의 프로세서를 실행하고 1초 동안 동작하도록 하겠습니다. 어떤일이 일어나는지 확인해보세요:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
//sampleStart
    val producer = produceNumbers()
    repeat(5) { launchProcessor(it, producer) }
    delay(950)
    producer.cancel() // cancel producer coroutine and thus kill them all
//sampleEnd
}

fun CoroutineScope.produceNumbers() = produce<Int> {
    var x = 1 // start from 1
    while (true) {
        send(x++) // produce next
        delay(100) // wait 0.1s
    }
}

fun CoroutineScope.launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    for (msg in channel) {
        println("Processor #$id received $msg")
    }    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-06.kt)에서 전체 코드를 볼 수 있습니다.

각 특정 정수를 받는 프로세서 ID는 다를 수 있지만 출력은 다음과 비슷합니다:

```text
Processor #2 received 1
Processor #4 received 2
Processor #0 received 3
Processor #1 received 4
Processor #3 received 5
Processor #2 received 6
Processor #4 received 7
Processor #0 received 8
Processor #1 received 9
Processor #3 received 10
```

생산자 코루틴을 취소하면 채널이 닫히므로 프로세서 코루틴이 수행하는 채널에 대한 반복이 종료됩니다.

또한 `for` 루프로 채널을 명시적으로 반복하여 `launchProcessor` 코드에서 팬-아웃을 어떻게 수행하는지 주의하십시오. `consumeEach`와 달리 `for` 루프 패턴은 여러 코루틴에서 사용하기에 완벽하게 안전합니다. 프로세서 코루틴 중 하나가 실패하더라도 다른 프로세서는 여전히 채널을 처리하는 반면에 `consumeEach`를 통해 작성 된 프로세서는 항상 정상 또는 비정상 완료 시 기본 채널을 소비 \(취소\) 합니다.

## Fan-in

여러 코루틴은 같은 채널로 전송될 수 있습니다. 예를 들어 문자열 타입의 채널에 특정 지연과 함께 특정 문자열을 반복적으로 보내는 중단 함수를 사용해 보겠습니다:

```kotlin
suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
    while (true) {
        delay(time)
        channel.send(s)
    }
}
```

이제 문자열을 보내는 몇개의 코루틴을 실행하면 어떻게 되는지 확인해 봅시다 \(이 예제에서는 main 쓰레드의 컨텍스트에서 main 코루틴의 자식으로 실행합니다\):

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking {
//sampleStart
    val channel = Channel<String>()
    launch { sendString(channel, "foo", 200L) }
    launch { sendString(channel, "BAR!", 500L) }
    repeat(6) { // receive first six
        println(channel.receive())
    }
    coroutineContext.cancelChildren() // cancel all children to let main finish
//sampleEnd
}

suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
    while (true) {
        delay(time)
        channel.send(s)
    }
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-07.kt)에서 전체 코드를 볼 수 있습니다.

이 출력은 아래와 같습니다:

```text
foo
foo
BAR!
foo
foo
BAR!
```

## 채널 버퍼 \(Buffered channels\)

지금까지 살펴본 채널은 버퍼를 가지고 있지 않았습니다. 송신자와 수신자가 서로 만나면 버퍼되지 않은 채널이 요소를 전송합니다 \(일명 랑데뷰\). 송신이 먼저 호출되면 수신이 호출될 때까지 중단되고 수신이 먼저 호출되면 송신이 호출될 때까지 일시 중단됩니다.

[Channel\(\)](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-channel.html) 팩토리 함수와 [produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html) 빌더는 특정 _버퍼 크기_ 를 결정하는 `capacity` 파라미터를 선택적으로 지정할 수 있습니다. 버퍼는 버퍼가 가득 찼을 때 차단하는 특정 크기의 `BlockingQueue`와 유사하게 전송하기 전에 여러 요소를 보낼 수 있습니다.

다음 코드가 어떻게 동작하는지 살펴봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
//sampleStart
    val channel = Channel<Int>(4) // create buffered channel
    val sender = launch { // launch sender coroutine
        repeat(10) {
            println("Sending $it") // print before sending each element
            channel.send(it) // will suspend when buffer is full
        }
    }
    // don't receive anything... just wait....
    delay(1000)
    sender.cancel() // cancel sender coroutine
//sampleEnd    
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-08.kt)에서 전체 코드를 볼 수 있습니다.

_4개_ 의 크기를 가지는 버퍼된 채널을 사용하기 때문에 "sending"은 _5번_ 출력됩니다:

```text
Sending 0
Sending 1
Sending 2
Sending 3
Sending 4
```

처음 4개의 요소는 버퍼에 추가되고 다섯번째 요소를 송신하려고 할 때 송신자는 중단합니다.

## 채널의 공정성 \(Channels are fair\)

여러 코루틴의 호출 순서와 관련하여 채널로의 송신과 수신 동작은 _공정_ 합니다. 즉 선착순으로 제공됩니다. 예를 들어 처음 `receive`를 호출한 코루틴이 요소를 수신합니다. 다음 예제에서 2개의 코루틴 "ping" 과 "pong"은 공유된 "table" 채널에서 "ball" 객체를 수신합니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

//sampleStart
data class Ball(var hits: Int)

fun main() = runBlocking {
    val table = Channel<Ball>() // a shared table
    launch { player("ping", table) }
    launch { player("pong", table) }
    table.send(Ball(0)) // serve the ball
    delay(1000) // delay 1 second
    coroutineContext.cancelChildren() // game over, cancel them
}

suspend fun player(name: String, table: Channel<Ball>) {
    for (ball in table) { // receive the ball in a loop
        ball.hits++
        println("$name $ball")
        delay(300) // wait a bit
        table.send(ball) // send the ball back
    }
}
//sampleEnd
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-09.kt)에서 전체 코드를 볼 수 있습니다.

"ping" 코루틴은 먼저 실행되므로 볼을 처음으로 받습니다. "ping" 코루틴은 테이블로 보내자마자 즉시 볼을 수신하기 시작하지만 "pong" 코루틴이 먼저 기다리고 있었기 때문에 "pong" 코루틴이 볼을 받습니다:

```text
ping Ball(hits=1)
pong Ball(hits=2)
ping Ball(hits=3)
pong Ball(hits=4)
```

때때로 사용중인 실행 프로그램의 특성으로 인해 채널에서 실행이 불공정 해보일 수도 있습니다. 자세한 내용은 [this issue](https://github.com/Kotlin/kotlinx.coroutines/issues/111)을 참고 바랍니다.

## 티커 채널 \(Ticker channels\)

티커 채널은 이 채널의 마지막 소비 이후 딜레이가 주어질 때마다 `Unit` 생성하는 특별한 약속 같은 채널입니다. 일반적으로 필요 없는 기능으로 보여질 수 있지만 복잡한 시간 기반 [produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/produce.html) 파이프라인과 윈도잉과 다른 시간 종속 처리를 수행하는 곳에 유용한 빌딩 블럭입니다. 티커 채널은 "on tick" 액션을 수행하기 위해 [select](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.selects/select.html)를 사용합니다.

팩토리 메서드 [ticker](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/ticker.html)를 사용하여 티커 채널을 생성할 수 있습니다. [ReceiveChannel.cancel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/cancel.html) 메서드를 사용하여 더이상 요소가 필요치 않다고 알려줄 수 있습니다.

이제 예제를 통해 어떻게 동작하는지 확인해 봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun main() = runBlocking<Unit> {
    val tickerChannel = ticker(delayMillis = 100, initialDelayMillis = 0) // create ticker channel
    var nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
    println("Initial element is available immediately: $nextElement") // no initial delay

    nextElement = withTimeoutOrNull(50) { tickerChannel.receive() } // all subsequent elements have 100ms delay
    println("Next element is not ready in 50 ms: $nextElement")

    nextElement = withTimeoutOrNull(60) { tickerChannel.receive() }
    println("Next element is ready in 100 ms: $nextElement")

    // Emulate large consumption delays
    println("Consumer pauses for 150ms")
    delay(150)
    // Next element is available immediately
    nextElement = withTimeoutOrNull(1) { tickerChannel.receive() }
    println("Next element is available immediately after large consumer delay: $nextElement")
    // Note that the pause between `receive` calls is taken into account and next element arrives faster
    nextElement = withTimeoutOrNull(60) { tickerChannel.receive() } 
    println("Next element is ready in 50ms after consumer pause in 150ms: $nextElement")

    tickerChannel.cancel() // indicate that no more elements are needed
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-channel-10.kt)에서 전체 코드를 볼 수 있습니다.

출력결과는 아래와 같습니다:

```text
Initial element is available immediately: kotlin.Unit
Next element is not ready in 50 ms: null
Next element is ready in 100 ms: kotlin.Unit
Consumer pauses for 150ms
Next element is available immediately after large consumer delay: kotlin.Unit
Next element is ready in 50ms after consumer pause in 150ms: kotlin.Unit
```

기본적으로 [ticker](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/ticker.html)는 소비자 일시정지를 인식하고 일시정지가 발생하면 다음에 생성된 요소 지연을 조정하여 고정 비율의 생산 요소를 유지하려고 합니다.

선택적으로 [TickerMode.FIXED\_DELAY](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-ticker-mode/-f-i-x-e-d_-d-e-l-a-y.html)와 동일한 `mode` 파라미터를 통해 요소 간의 고정 지연을 유지할 수 있습니다.

