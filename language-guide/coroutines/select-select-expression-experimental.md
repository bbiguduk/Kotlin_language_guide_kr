# select 표현 \(Select Expression\) \(experimental\)

select 표현을 사용하면 여러개의 일시 중단 함수를 동시에 대기하고 사용가능한 첫번째 결과를 _선택_ 합니다.

> select 표현은 `kotlinx.coroutines`에 실험적 기능입니다. `kotlinx.coroutines` 라이브러리에 업데이트를 통해 변경될 수 있습니다.

## 채널에서 선택 \(Selecting from channels\)

`fizz` 와 `buzz`라는 문자열의 2개의 프로듀서가 있다고 합시다. `fizz`는 "Fizz" 문자열을 매 300 ms 마다 생성합니다:

```kotlin
fun CoroutineScope.fizz() = produce<String> {
    while (true) { // sends "Fizz" every 300 ms
        delay(300)
        send("Fizz")
    }
}
```

그리고 `buzz`는 "Buzz!" 문자열을 매 500ms 마다 생성합니다:

```kotlin
fun CoroutineScope.buzz() = produce<String> {
    while (true) { // sends "Buzz!" every 500 ms
        delay(500)
        send("Buzz!")
    }
}
```

[receive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/receive.html) 중단 함수를 사용하면 한 채널에서 수신하거나 다른 채널에서 수신할 수 있습니다. 그러나 [select](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.selects/select.html) 표현을 사용하면 [onReceive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/on-receive.html) 절을 사용하여 둘다 동시에 수신할 수 있습니다:

```kotlin
suspend fun selectFizzBuzz(fizz: ReceiveChannel<String>, buzz: ReceiveChannel<String>) {
    select<Unit> { // <Unit> means that this select expression does not produce any result 
        fizz.onReceive { value ->  // this is the first select clause
            println("fizz -> '$value'")
        }
        buzz.onReceive { value ->  // this is the second select clause
            println("buzz -> '$value'")
        }
    }
}
```

7번 모두 실행해 봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlinx.coroutines.selects.*

fun CoroutineScope.fizz() = produce<String> {
    while (true) { // sends "Fizz" every 300 ms
        delay(300)
        send("Fizz")
    }
}

fun CoroutineScope.buzz() = produce<String> {
    while (true) { // sends "Buzz!" every 500 ms
        delay(500)
        send("Buzz!")
    }
}

suspend fun selectFizzBuzz(fizz: ReceiveChannel<String>, buzz: ReceiveChannel<String>) {
    select<Unit> { // <Unit> means that this select expression does not produce any result 
        fizz.onReceive { value ->  // this is the first select clause
            println("fizz -> '$value'")
        }
        buzz.onReceive { value ->  // this is the second select clause
            println("buzz -> '$value'")
        }
    }
}

fun main() = runBlocking<Unit> {
//sampleStart
    val fizz = fizz()
    val buzz = buzz()
    repeat(7) {
        selectFizzBuzz(fizz, buzz)
    }
    coroutineContext.cancelChildren() // cancel fizz & buzz coroutines
//sampleEnd        
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-select-01.kt)에서 전체 코드를 볼 수 있습니다.

이 코드 결과는 아래와 같습니다:

```text
fizz -> 'Fizz'
buzz -> 'Buzz!'
fizz -> 'Fizz'
fizz -> 'Fizz'
buzz -> 'Buzz!'
fizz -> 'Fizz'
buzz -> 'Buzz!'
```

## 닫힌 채널에서의 선택 \(Selecting on close\)

`select`에 [onReceive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-receive-channel/on-receive.html) 절은 `select`에서 예외를 발생하여 채널이 닫기면 실패하게 됩니다. 채널이 닫길 때 특정 행동을 하려면 [onReceiveOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/on-receive-or-null.html) 절을 사용할 수 있습니다. 다음 예제는 `select`가 선택된 절의 결과를 반환하는 표현식을 보여줍니다:

```kotlin
suspend fun selectAorB(a: ReceiveChannel<String>, b: ReceiveChannel<String>): String =
    select<String> {
        a.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'a' is closed" 
            else 
                "a -> '$value'"
        }
        b.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'b' is closed"
            else    
                "b -> '$value'"
        }
    }
```

[onReceiveOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/on-receive-or-null.html)은 null이 불가능한 요소의 채널에 대해서만 정의된 확장 함수 이므로 닫힌 채널과 null 값 사이에서 혼동되지 않습니다.

이것을 사용하여 "Hello" 문자열을 4차례 생성하는 `a` 채널과 "World" 문자열을 4차례 생성하는 `b` 채널을 만들어 봅시다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlinx.coroutines.selects.*

suspend fun selectAorB(a: ReceiveChannel<String>, b: ReceiveChannel<String>): String =
    select<String> {
        a.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'a' is closed" 
            else 
                "a -> '$value'"
        }
        b.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'b' is closed"
            else    
                "b -> '$value'"
        }
    }

fun main() = runBlocking<Unit> {
//sampleStart
    val a = produce<String> {
        repeat(4) { send("Hello $it") }
    }
    val b = produce<String> {
        repeat(4) { send("World $it") }
    }
    repeat(8) { // print first eight results
        println(selectAorB(a, b))
    }
    coroutineContext.cancelChildren()  
//sampleEnd      
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-select-02.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 결과를 보면 흥미로운 점이 있습니다. 이 점에 대해 더 자세히 검토해 봅시다:

```text
a -> 'Hello 0'
a -> 'Hello 1'
b -> 'World 0'
a -> 'Hello 2'
a -> 'Hello 3'
b -> 'World 1'
Channel 'a' is closed
Channel 'a' is closed
```

이것을 확인하려면 몇가지 관찰을 해야 합니다.

우선 `select`는 첫번째 절에 _편향_ 되어 있습니다. 여러 절을 동시에 선택할 수 있으면 그 중에 첫번째 절을 선택합니다. 여기서 두 채널 모두 지속적으로 문자열을 생성하므로 첫번째 절인 `a` 채널이 선택됩니다. 그러나 버퍼링되지 않은 채널을 사용하기 때문에 `a`는 [send](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/send.html) 호출 시 때때로 일시 중단되며 `b`도 보낼 기회를 줍니다.

두번째 관찰해야 할 부분은 채널이 이미 닫혀 있으면 [onReceiveOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/on-receive-or-null.html)이 즉시 선택된다는 것입니다.

## 전송을 위한 선택 \(Selecting to send\)

select 표현은 편향 특성을 결합하기 좋은 [onSend](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/-send-channel/on-send.html) 절을 가지고 있습니다.

정수를 생성하고 이 값을 주 채널의 소비자가 유지할 수 없을 때 `side` 채널에 보내는 예제를 작성해 봅시다:

```kotlin
fun CoroutineScope.produceNumbers(side: SendChannel<Int>) = produce<Int> {
    for (num in 1..10) { // produce 10 numbers from 1 to 10
        delay(100) // every 100 ms
        select<Unit> {
            onSend(num) {} // Send to the primary channel
            side.onSend(num) {} // or to the side channel     
        }
    }
}
```

각 숫자를 처리하는데 250 ms 가 걸리고 소비자는 느릴 것입니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlinx.coroutines.selects.*

fun CoroutineScope.produceNumbers(side: SendChannel<Int>) = produce<Int> {
    for (num in 1..10) { // produce 10 numbers from 1 to 10
        delay(100) // every 100 ms
        select<Unit> {
            onSend(num) {} // Send to the primary channel
            side.onSend(num) {} // or to the side channel     
        }
    }
}

fun main() = runBlocking<Unit> {
//sampleStart
    val side = Channel<Int>() // allocate side channel
    launch { // this is a very fast consumer for the side channel
        side.consumeEach { println("Side channel has $it") }
    }
    produceNumbers(side).consumeEach { 
        println("Consuming $it")
        delay(250) // let us digest the consumed number properly, do not hurry
    }
    println("Done consuming")
    coroutineContext.cancelChildren()  
//sampleEnd      
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-select-03.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 실행 결과는 아래와 같습니다:

```text
Consuming 1
Side channel has 2
Side channel has 3
Consuming 4
Side channel has 5
Side channel has 6
Consuming 7
Side channel has 8
Side channel has 9
Consuming 10
Done consuming
```

## 지연된 값 선택 \(Selecting deferred values\)

지연된 값은 [onAwait](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/on-await.html) 절을 사용하여 선택될 수 있습니다. 랜덤한 딜레이 후에 지연된 문자열을 반환하는 비동기 함수를 살펴봅시다:

```kotlin
fun CoroutineScope.asyncString(time: Int) = async {
    delay(time.toLong())
    "Waited for $time ms"
}
```

랜덤한 딜레이와 함께 12번 실행해 봅시다.

```kotlin
fun CoroutineScope.asyncStringsList(): List<Deferred<String>> {
    val random = Random(3)
    return List(12) { asyncString(random.nextInt(1000)) }
}
```

main 함수는 첫번째 함수가 완료되길 기다리고 여전히 활성화 상태인 지연된 값의 숫자를 카운트 합니다. `select` 표현식이 Kotlin DSL 이라는 사실을 이용했으므로 임의의 코드를 사용하여 절을 제공할 수 있습니다. 이 경우 지연된 값의 리스트를 반복하여 각 지연된 값의 `onAwait` 절을 제공합니다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.selects.*
import java.util.*

fun CoroutineScope.asyncString(time: Int) = async {
    delay(time.toLong())
    "Waited for $time ms"
}

fun CoroutineScope.asyncStringsList(): List<Deferred<String>> {
    val random = Random(3)
    return List(12) { asyncString(random.nextInt(1000)) }
}

fun main() = runBlocking<Unit> {
//sampleStart
    val list = asyncStringsList()
    val result = select<String> {
        list.withIndex().forEach { (index, deferred) ->
            deferred.onAwait { answer ->
                "Deferred $index produced answer '$answer'"
            }
        }
    }
    println(result)
    val countActive = list.count { it.isActive }
    println("$countActive coroutines are still active")
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-select-04.kt)에서 전체 코드를 볼 수 있습니다.

출력은 아래와 같습니다:

```text
Deferred 4 produced answer 'Waited for 128 ms'
11 coroutines are still active
```

## 지연된 값 채널 간 전환 \(Switch over a channel of deferred values\)

지연된 문자열 값의 채널을 소비하고 각 수신된 지연된 값을 기다리지만 다음 지연된 값이 오거나 채널이 닫힐때까지만 기다리는 채널 프로듀서 함수를 작성해 봅시다. 이 예제는 같은 `select` 안에 [onReceiveOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.channels/on-receive-or-null.html) 와 [onAwait](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/on-await.html) 절을 함께 사용합니다:

```kotlin
fun CoroutineScope.switchMapDeferreds(input: ReceiveChannel<Deferred<String>>) = produce<String> {
    var current = input.receive() // start with first received deferred value
    while (isActive) { // loop while not cancelled/closed
        val next = select<Deferred<String>?> { // return next deferred value from this select or null
            input.onReceiveOrNull { update ->
                update // replaces next value to wait
            }
            current.onAwait { value ->  
                send(value) // send value that current deferred has produced
                input.receiveOrNull() // and use the next deferred from the input channel
            }
        }
        if (next == null) {
            println("Channel was closed")
            break // out of loop
        } else {
            current = next
        }
    }
}
```

테스트하기 위해 특정 시간 이후에 특정 문자열로 해석되는 간단한 비동기 함수를 사용할 것입니다:

```kotlin
fun CoroutineScope.asyncString(str: String, time: Long) = async {
    delay(time)
    str
}
```

main 함수는 `switchMapDeferreds`의 결과를 출력하고 어떤 테스트 데이터를 송신하기 위해 코루틴을 실행합니다:

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*
import kotlinx.coroutines.selects.*

fun CoroutineScope.switchMapDeferreds(input: ReceiveChannel<Deferred<String>>) = produce<String> {
    var current = input.receive() // start with first received deferred value
    while (isActive) { // loop while not cancelled/closed
        val next = select<Deferred<String>?> { // return next deferred value from this select or null
            input.onReceiveOrNull { update ->
                update // replaces next value to wait
            }
            current.onAwait { value ->  
                send(value) // send value that current deferred has produced
                input.receiveOrNull() // and use the next deferred from the input channel
            }
        }
        if (next == null) {
            println("Channel was closed")
            break // out of loop
        } else {
            current = next
        }
    }
}

fun CoroutineScope.asyncString(str: String, time: Long) = async {
    delay(time)
    str
}

fun main() = runBlocking<Unit> {
//sampleStart
    val chan = Channel<Deferred<String>>() // the channel for test
    launch { // launch printing coroutine
        for (s in switchMapDeferreds(chan)) 
            println(s) // print each received string
    }
    chan.send(asyncString("BEGIN", 100))
    delay(200) // enough time for "BEGIN" to be produced
    chan.send(asyncString("Slow", 500))
    delay(100) // not enough time to produce slow
    chan.send(asyncString("Replace", 100))
    delay(500) // give it time before the last one
    chan.send(asyncString("END", 500))
    delay(1000) // give it time to process
    chan.close() // close the channel ... 
    delay(500) // and wait some time to let it finish
//sampleEnd
}
```

> [여기](https://github.com/kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/test/guide/example-select-05.kt)에서 전체 코드를 볼 수 있습니다.

이 코드의 출력은 아래와 같습니다:

```text
BEGIN
Replace
END
Channel was closed
```

