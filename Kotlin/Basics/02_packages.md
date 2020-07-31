# 패키지 (Packages)

소스파일은 패키지 선언으로 시작합니다:

```kotlin
package org.example

fun printMessage() { /*...*/ }
class Message { /*...*/ }

// ...
```

소스파일의 모든 콘텐츠 (class와 함수) 선언된 패키지에 포함됩니다.
위 예제에서 `printMessage()`의 풀네임은 `org.example.printMessage` 이며, `Message` 의 풀네임은 `org.example.Message` 입니다.

패키지를 지정하지 않은경우, 기본 패키지에 해당합니다.

## 기본 import (Default Imports)

몇개의 패키지는 Kotlin에서 기본적으로 추가되어 있습니다:

- [kotlin.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/index.html)
- [kotlin.annotation.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/index.html)
- [kotlin.collections.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html)
- [kotlin.comparisons.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.comparisons/index.html)  (since 1.1)
- [kotlin.io.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/index.html)
- [kotlin.ranges.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/index.html)
- [kotlin.sequences.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/index.html)
- [kotlin.text.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/index.html)

플랫폼에 따라 추가적으로 더 import 된 패키지가 있습니다:

- JVM:
  - java.lang.*
  - [kotlin.jvm.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/index.html)

- JS:    
  - [kotlin.js.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.js/index.html)

## Imports

기본 적인 import 외에 각 파일은 직접적으로 import 할 수 있습니다.
import에 대한 설명은 [grammar](https://kotlinlang.org/docs/reference/grammar.html#importHeader)를 참고하시기 바랍니다.

단일 이름으로 import 할 수 있습니다.

```kotlin
import org.example.Message // Message is now accessible without qualification
```

또는 모든 콘텐츠에 접근 가능합니다 (패키지, class, object 등):

```kotlin
import org.example.* // everything in 'org.example' becomes accessible
```

만약 이름이 중복될 경우, *as* 키워드를 이용하여 로컬에서 사용할 이름을 재설정 할 수 있습니다:

```kotlin
import org.example.Message // Message is accessible
import org.test.Message as testMessage // testMessage stands for 'org.test.Message'
```

`import` 키워드는 class를 가져오는 것에 국한되지 않으며, 선언을 통해 다른 것을 import 할 수 있습니다:

  * 최상위 함수와 프로퍼티
  * 함수와 프로퍼티 선언은 [object declarations](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/object-expressions-and-declarations#object-declarations)를 참고
  * [enum constants](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-enum-classes).

## 최상위 선언의 가시성 (Visibility of Top-level Declarations)

최상위 선언이 *private*로 표기된 경우, 선언 된 파일에 국한합니다 ([Visibility Modifiers](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/visibility-modifiers) 참고).