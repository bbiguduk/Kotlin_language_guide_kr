# Kotlin 코루틴 개요 \(Kotlin Collections Overview\)

Kotlin은 언어로서 표준 라이브러리에서 최소한의 저레벨 API 만 제공하여 다양한 다른 라이브러리가 코루틴을 활용할 수 있도록 합니다. 비슷한 기능을 가진 다른 언어들과 달리 `async` 와 `await`는 Kotlin의 키워드가 아니며 표준 라이브러리의 일부도 아닙니다. 또한 Kotlin의 _중단 함수 \(suspending function\)_ 의 개념은 미래와 약속보다 비동기 작업에 대한 더 안전한 추상화를 제공합니다.

`kotlinx.coroutines`은 JetBrains에서 개발한 다양한 라이브러리입니다. 여기에는 다양한 고차원 코루틴 지원 기능 \(`launch`, `async`, 등\)이 포함되어 있습니다.

이것은 `kotlinx.coroutines`의 핵심 기능에 대한 가이드이며 여러 예제로 구성되어 있으며 여러 주제로 나뉩니다.

코루틴을 사용하고 이 가이드의 예제를 따라하려면 [프로젝에서 README \(in the project README\)](https://github.com/kotlin/kotlinx.coroutines/blob/master/README.md#using-in-your-projects) 에 설명 된대로 `kotlinx-coroutines-core` 모듈을 추가해야 합니다.

## 콘텐츠 \(Table of contents\)

* [코루틴 기본 \(Basics\)](coroutine-basics.md)
* [취소와 타임아웃 \(Cancellation and Timeouts\)](cancellation-and-timeouts.md)
* [일시 중단 함수 구성 \(Composing Suspending Functions\)](composing-suspending-functions.md)
* [코루틴 컨텍스트와 디스패처 \(Coroutine Context and Dispatchers\)](dispatchers-coroutine-context-and-dispatchers.md)
* [비동기 플로우 \(Asynchronous Flow\)](flow-asynchronous-flow.md)
* [채널 \(Channels\)](channels.md)
* [예외 처리와 감독 \(Exception Handling and Supervision\)](exception-handling.md)
* [공유된 변경 가능한 상태와 동시성 \(Shared Mutable State and Concurrency\)](untitled.md)
* [Select 표현 \(Select Expression\) \(experimental\)](select-select-expression-experimental.md)

## 추가 참조 \(Additional references\)

* [Guide to UI programming with coroutines](https://github.com/kotlin/kotlinx.coroutines/blob/master/ui/coroutines-guide-ui.md)
* [Coroutines design document \(KEEP\)](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)
* [Full kotlinx.coroutines API reference](https://kotlin.github.io/kotlinx.coroutines)

