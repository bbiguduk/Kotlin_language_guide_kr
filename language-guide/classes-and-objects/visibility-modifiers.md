# 접근 제한자 \(Visibility Modifiers\)

클래, 객체, 인터페이스, 생성자, 함수, 프로퍼티와 setter는 _\(접근 제한자\)visibility modifiers_ 를 가질 수 있습니다. \(getter는 항상 프로퍼티와 같은 접근 제한자를 갖습니다.\) Kotlin에서는 4개의 접근 제한자가 있습니다: `private`, `protected`, `internal`, `public`. 아무런 표기 없이 사용하면 기본적으로 `public`으로 선언됩니다.

접근 제어자의 범위에 대해 자세히 배워보겠습니다.

## 패키지 \(Packages\)

함수, 프로퍼티, class, 객체, 인터페이스는 패키지내에서 최상위로 선언할 수 있습니다:

```kotlin
// file name: example.kt
package foo

fun baz() { ... }
class Bar { ... }
```

* `public`은 기본 접근 제한자로 어느 곳에서도 접근이 가능하다는 의미입니다;
* `private`은 오직 선언한 파일 내에서만 접근이 가능하다는 의미입니다;
* `internal`은 같은 [module](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/visibility-modifiers#modules)에서는 어느 곳에서도 접근이 가능하다는 의미입니다;
* `protected`은 최상위 선언이 불가능합니다.

참고: 다른 패키지에서 최상위 선언을 사용하려면 [import](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/basics/import-packages-and-imports#imports)를 해야 합니다.

예:

```kotlin
// file name: example.kt
package foo

private fun foo() { ... } // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt

internal val baz = 6    // visible inside the same module
```

## class와 인터페이스 \(Classes and Interfaces\)

class에 선언 된 멤버의 접근 제한자:

* `private`는 해당 class 내 \(모든 멤버 포함\)에서만 접근이 가능하다는 의미입니다;
* `protected`는 `private`와 같고 추가로 subclass 에서도 접근이 가능하다는 의미입니다;
* `internal`은 class의 정의를 볼 수 있는 _모듈 안 \(inside this module\)_ 모든 클라이언트는 `internal` 멤버에 접근이 가능하다는 의미입니다;
* `public`은 class의 정의를 볼 수 있는 모든 `public` 멤버에 접근이 가능하다는 의미입니다.

참고: Kotlin에서 inner class를 가지고 있는 외부 class는 inner class에 선언 된 private 멤버에 접근이 불가능 합니다.

`protected` 멤버를 오버라이드 하는 경우 아무런 표기를 하지 않으면 오버라이드 한 멤버도 `protected`의 접근 제한자를 가집니다.

예:

```kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // public by default

    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible
    // b, c and d are visible
    // Nested and e are visible

    override val b = 5   // 'b' is protected
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible
    // o.c and o.d are visible (same module)
    // Outer.Nested is not visible, and Nested::e is not visible either 
}
```

### 생성자 \(Constructors\)

주 생성자에 접근 제어자를 선언할 경우 아래와 같이 표기합니다 \(접근 제어자 명시시 반든시 _constructor_ 키워드를 추가해야 합니다\):

```kotlin
class C private constructor(a: Int) { ... }
```

위 예제에서는 생서자가 private 입니다. 기본적으로 모든 생성자는 `public` 이며, class가 보이는 곳이면 어디서든 접근 가능합니다 \(`internal` class의 생성자는 오직 같은 모듈에서만 접근이 가능합니다\).

### 지역 선언 \(Local declarations\)

지역 변수, 함수, class는 접근 제한자를 가질 수 없습니다.

## 모듈 \(Modules\)

`internal` 접근 제어자는 같은 모듈에서만 접근 가능하다는 의미입니다. 보다 구체적으로, 모듈은 함께 컴파일 된 Kotlin 파일들 입니다.

* IntelliJ IDEA 모듈;
* Maven 프로젝트;
* Gradle 소스 셋 \(`test` 소스 셋이 `main`의 internel 선언에 접근 할 수 있는 것을 제외\);
* `<kotlinc>` Ant task을 호출하여 컴파일 된 파일 셋.

