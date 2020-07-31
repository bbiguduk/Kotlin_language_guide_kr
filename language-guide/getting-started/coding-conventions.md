# 코딩 작성 스타일 \(Coding Conventions\)

이 페이지에선 Kotlin의 코딩 스타일 내용이 포함되어 있습니다.

### 스타일 가이드 적용

IntelliJ 에서 스타일 가이드를 설정하려면, Kotlin 플러그인 버전 1.2.20 또는 최신 버전을 설치하시기 바랍니다. 메뉴에서 **Settings \| Editor \| Code Style \| Kotlin \| Set from...** 눌러 드랍박스에서 **Predefined style \| Kotlin style guide** 을 선택합니다.

코드가 스타일 가이드에 맞게 작성되고 있는지 확인하려면, inspection settings 메뉴로 가서 **Kotlin \| Style issues \| File is not formatted according to project settings** 을 사용가능하도록 체크 해줍니다. 기본적으로 네이밍 가이드 같은 스타일 가이드들은 적용이 되어 있습니다.

## 소스 코드 구성

### 디렉토리 구조

Kotlin 프로젝트에서는 루트 패키지는 생략하고 작업하도록 추천하고 있습니다. 예를 들어, 프로젝트의 모든 소스 코드가 `org.example.kotlin`패키지와 하위패키지에 있는 경우, `org.example.kotlin` 패키지가 있는 파일은 루트 바로 아래에 배치해야 하며, `org.example.kotlin.network.socket`은 루트의 `network/socket` 하위 디렉토리에 있어야 합니다.

> **JVM에서의 구성**: Kotlin과 Java를 함께 쓰는 프로젝트에서는 Java 소스 파일과 같이 Kotlin 소스 파일은 같은 루트에 있어야 하며, 같은 디렉토리 구조를 갖습니다: 각 파일은 패키지명에 맞는 디렉토리에 저장 되어야 합니다.

### 소스 파일 이름

Kotlin 파일에 하나의 class만 가지고 있다면 \(대체로 최상단에 선언\), 파일이름은 class 명과 같게 하고 확장자명을 .kt로 해줍니다. 파일에 여러 class가 포함되어 있거나, 최상위 선언만 있다면 파일에 포함되어 있는 내용을 설명하는 이름을 사용합니다. 첫글자는 대문자로 작성하며 [camel case](https://en.wikipedia.org/wiki/Camel_case)에 맞게 파일명을 작성합니다 \(예를 들어, `ProcessDeclarations.kt`\).

파일 이름에는 어떠한 작업을 수행하는지 설명이 포함되어 있어야 합니다. 따라서, "Util"과 같은 의미없는 파일 이름은 피해야 합니다.

### 소스 파일 구성

서로 밀접한 관련이 있고 파일 사이즈가 적절한 경우 동일한 Kotlin 소스 파일에 여러 선언 \(class, 최상위 함수 또는 프로퍼티\)을 하는 것이 좋습니다 \(소스 라인 수가 수백라인을 넘지 않는 한\).

특히, class의 모든 클라이언트와 관련 된 확장 함수를 정의 할 때 같은 파일에 정의해야 합니다. 특정 클라이언트에만 적합한 확장 함수를 정의할 때는 클라이언트 코드 바로 다음에 정의합니다. "Foo의 모든 확장"을 위해 파일을 생성하지 마십시오.

### 클래스 레이아웃

일반적으로, class는 다음의 우선순위로 정렬 해야 합니다:

* 프로퍼티 선언과 초기화 블럭
* 두번째 생성자
* 메서드 선언
* Companion object

알파벳 순 또는 가시성 별로 메서드를 정렬 하지 마십시오. 그리고 확장 메서드에서 정규 메서드를 분리하지 마십시오. 대신에, 다른 누군가가 class를 보아도 이해할 수 있도록 관련된 것들을 한곳에 모아야 합니다. 정렬을 정하고 그 규칙을 지켜야 합니다.

중첩 class는 사용하는 코드 다음에 위치합니다. 만약에 class가 외부에서 사용되고 내부에서는 참조하지 않을 경우 companion object 다음, 코드 마지막에 위치합니다.

### 인터페이스 구현 레이아웃

인터페이스를 구현할 때는, 인터페이스 멤버는 같은 우선순의 입니다 \(필요한 경우, private 메서드 포함\)

### 오버로드 레이아웃

항상 class 안에서 오버로드를 차례로 위치하게 합니다.

## 네이밍 규칙

Kotlin에서 패키지와 class 네이밍 규칙은 매우 간단합니다:

* 패키지 명은 항상 소문자와 언더바를 포함하면 안됩니다 \(`org.example.project`\). 일반적으로 여러 단어는 패키지 명으로 사용이 안되지만, 사용해야 된다면 이어서 작성하거나 camel case를 사용해야 합니다 \(`org.example.myProject`\).
* class와 object 명은 첫글자는 대문자로 그리고 camel case를 사용합니다:

```kotlin
open class DeclarationProcessor { /*...*/ }

object EmptyDeclarationProcessor : DeclarationProcessor() { /*...*/ }
```

### 함수 이름

함수와 프로퍼티, 로컬 변수는 소문자로 시작하며 camel case를 사용합니다. 언더바는 사용하면 안됩니다:

```kotlin
fun processDeclarations() { /*...*/ }
var declarationCount = 1
```

예외: 클래스의 인스턴스를 만드는데 사용되는 팩토리 함수는 추상 리턴 타입과 같은 이름을 가질 수 있습니다:

```kotlin
interface Foo { /*...*/ }

class FooImpl : Foo { /*...*/ }

fun Foo(): Foo { return FooImpl() }
```

#### 테스트 메서드 명

테스트에 한해, 역 따옴표\(\`\)로 묶인 경우 공백도 사용이 가능합니다. \(현재 이러한 이름은 Android 런타임에서 지원하지 않습니다.\) 테스트 코드에서는 메서드 명에 언더바도 사용이 가능합니다.

```kotlin
class MyTestCase {
     @Test fun `ensure everything works`() { /*...*/ }

     @Test fun ensureEverythingWorks_onAndroid() { /*...*/ }
}
```

### 프로퍼티 명

상수 이름 \(`get` 기능이 없는 `const`로 표기된 프로퍼티, 또는 최상위 또는 object `val` 프로퍼티\) 언더바로 나뉘는 대문자 이름을 가집니다:

```kotlin
const val MAX_COUNT = 8
val USER_NAME_FIELD = "UserName"
```

가변 데이터나 실행이 가능한 최상위 또는 object 프로퍼티 명은 camel-case를 사용합니다:

```kotlin
val mutableCollection: MutableSet<String> = HashSet()
```

싱글톤을 참조하기 위한 프로퍼티 명은 `object`선언과 같은 네이밍 스타일을 사용할 수 있습니다:

```kotlin
val PersonComparator: Comparator<Person> = /*...*/
```

enum 상수는 대문자, 언더바를 사용한 이름 \(`enum class Color { RED, GREEN }`\) 또는 첫글자가 대문자이며, camel-case를 따르는 이름 설정이 가능합니다.

#### backing 프로퍼티 명

class에 동일하지만 하나는 public API의 일부이고 다른 하나는 상세 구현인 2개의 프로퍼티가 있는경우, private 프로퍼티에는 첫부분에 언더바를 붙여줍니다:

```kotlin
class C {
    private val _elementList = mutableListOf<Element>()

    val elementList: List<Element>
         get() = _elementList
}
```

### 좋은 이름 고르기

class 명은 대게 어떤한 class 인지 명사나 명사구를 사용합니다 _is_: `List`, `PersonReader`.

메서드 명은 대게 메서드 무엇을 실행하는 지에 대한 동사나 동사구를 사용합니다 _does_: `close`, `readPersons`. 메서드 명은 또한 object를 변형시키거나 새로운 것을 반환하는 것을 암시할 수 있어야 합니다. 예를 들어 `sort`는 콜렉션을 바로 정렬하는 것이며, `sorted`는 복사 된 정렬 콜렉션을 반환합니다.

좋은 이름은 목적을 명확하게 나타내야 하기 때문에 의미 없는 단어 \(`Manager`, `Wrapper` etc.\)는 피하는게 좋습니다.

선언 명에 약어가 두글자인 경우 대문자를 사용합니다 \(`IOStream`\); 더 긴 경우 첫글자만 대문자로 사용합니다 \(`XmlFormatter`, `HttpInputStream`\).

## 코드 포맷

들여쓰기 시 탭이 아닌 4칸의 공백을 사용합니다.

코드 블럭을 위한 괄호 \(```{``}```\)는 구문 끝 부분에 여는 괄호를 넣고 다른 줄에 구문 시작부분과 같은 위치 \(들여쓰기\)에 닫는 괄호를 넣어야 합니다.

```kotlin
if (elements != null) {
    for (element in elements) {
        // ...
    }
}
```

\(Note: Kotlin에서 세미콜론 \(`;`\)은 선택사항이기 때문에 라인의 끝이 명확해야 합니다. Kotlin 코드 디자인은 Java 디자인과 유사합니다. 만약에 새로운 포맷의 스타일을 적용 하면 예상치 못한 문제에 직면할 수 있습니다.\)

### 공백

바이너리 연산식 사이에 공백을 넣어야 합니다 \(`a + b`\). 예외: 범위 연산자에는 공백을 넣지 않습니다 \(`0..i`\).

단항 연산자에는 공백을 넣지 않습니다 \(`a++`\)

해당 키워드 \(`if`, `when`, `for` and `while`\) 사이와 여는 괄호 사이에 공백을 넣어야 합니다.

우선 생성자와 메서드 및 메서드 호출 하는 부분에서 여는 괄호 전에는 공백을 넣으면 안됩니다.

```kotlin
class A(val x: Int)

fun foo(x: Int) { ... }

fun bar() {
    foo(1)
}
```

절대 `(`, `[` 다음, `]`, `)` 전에 공백을 넣으면 안됩니다.

절대 `.` 또는 `?.` 사이에 공백을 넣으면 안됩니다: `foo.bar().filter { it > 2 }.joinToString()`, `foo?.bar()`

`//` 다음에는 공백을 넣습니다: `// 주석입니다.`

특별한 타임 파라미터를 나타내는 괄호에는 공백을 넣지 않습니다: `class Map<K, V> { ... }`

`::` 주변에 공백을 넣으면 안됩니다: `Foo::class`, `String::length`

null이 가능한 타입을 나타내는 `?` 전에 공백을 넣으면 안됩니다: `String?`

일반적으로, 어떠한 경우에도 수평 정렬은 피해야 합니다. 식별자의 이름을 다른 길이의 이름으로 변경하는 것은 선언이나 사용에 영향이 없어야 합니다.

### 콜론

아래와 같은 상황에서는 `:` 전에 공백을 넣어야 합니다:

* 어떤 타입과 슈퍼 타입을 분리할 때;
* 슈퍼 class의 생성사 또는 다른 동일한 class의 생성자에게 위임할 때;
* `object` 키워드 다음에.

선언과 타입을 구분할 때는 `:` 전에 공백을 넣지 않습니다.

`:` 다음에는 항상 공백을 넣습니다.

```kotlin
abstract class Foo<out T : Any> : IFoo {
    abstract fun foo(a: Int): T
}

class FooImpl : Foo() {
    constructor(x: String) : this(x) { /*...*/ }

    val x = object : IFoo { /*...*/ } 
}
```

### class 헤더 포맷

생성자 파라미터가 적을 경우 싱글 라인으로 class 를 작성할 수 있습니다:

```kotlin
class Person(id: Int, name: String)
```

class 헤더가 길 경우 각각의 생성자 파라미터를 다른 라인에 들여쓰기와 함께 작성할 수 있습니다. 또한, 닫는 괄호는 다른 줄에 작성되어야 합니다. 만약에 상속을 사용한다면 슈퍼 class 생성자 호출이나 인터페이스 구현에 대한 것은 닫는 괄호와 같은 라인에 작성해야 합니다.

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name) { /*...*/ }
```

슈퍼 class 생성자 호출과 여러개의 인터페이스를 구현할 경우 첫번째 부분은 닫는 괄호와 같은 라인에 작성하고 나머지 부분은 다른 라인에 작성합니다:

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker { /*...*/ }
```

슈퍼타입 리스트가 많은 class는 콜론 다음에 슈퍼타입을 작성하며 모든 슈퍼타입은 들여쓰기를 통해 수평 정렬을 맞춰 줍니다:

```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {

    fun foo() { /*...*/ }
}
```

class 헤더가 길 때 class 헤더와 바디를 명확하게 구분하기 위해 class 헤더에 공백 \(위의 샘플 코드 참조\)을 넣거나, 여는 괄호를 다른 라인에 작성합니다:

```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne 
{
    fun foo() { /*...*/ }
}
```

생성자 파라미터에는 4칸의 공백 들여쓰기를 사용합니다.

> Rationale: 생성자의 프로퍼티 선언부와 class 바디의 프로퍼티 선언부가 동일한 들여쓰기를 가질 수 있습니다.

### 제어자 \(Modifiers\)

여러개의 제어자를 선언할 경우 아래와 같은 우선순위를 따라야 합니다:

```kotlin
public / protected / private / internal
expect / actual
final / open / abstract / sealed / const
external
override
lateinit
tailrec
vararg
suspend
inner
enum / annotation
companion
inline
infix
operator
data
```

모든 annotation은 제어자 전에 위치 합니다:

```kotlin
@Named("Foo")
private val foo: Foo
```

라이브러리에서 작업하지 않는 한, 중복 제어자는 생략합니다 \(예. `public`\).

### Annotation 포맷

대체적으로 annotation은 선언부와 같은 들여쓰기로 선언부 전 라인에 위치합니다:

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```

argument가 없는 annotation이 여러개인 경우 경우 같은 라인에 위치합니다:

```kotlin
@JsonExclude @JvmField
var x: String
```

argument가 없는 annotation이 하나인 경우 선언부와 같은 라인에 위치합니다:

```kotlin
@Test fun foo() { /*...*/ }
```

### File annotations

File annotation은 file 코멘트가 있다면 그 뒤에 위치합니다. `package` 구문 전에 위치하며 중간에 빈 공백 라인으로 구분 해줍니다 \(file과 패키지의 구분을 명확하기 하기 위해\).

```kotlin
/** License, copyright and whatever */
@file:JvmName("FooBar")

package foo.bar
```

### 함수 포맷

함수 구문이 한줄에 표기가 안될 경우, 아래와 같은 포맷으로 작성해야 합니다:

```kotlin
fun longMethodName(
    argument: ArgumentType = defaultValue,
    argument2: AnotherArgumentType
): ReturnType {
    // body
}
```

함수 파라미터에 대한 들여쓰기는 4칸 공백을 사용합니다.

> Rationale: 생성자 파라미터와 동일한 포맷

리턴 타입 추론이 가능한 함수는 생략 함수로 표현하는 걸 추천합니다.

```kotlin
fun foo(): Int {     // bad
    return 1 
}

fun foo() = 1        // good
```

### Expression body 포맷

추론 가능한 리턴타입이 있는 함수가 한줄로 표현이 불가능한 경우 `=` 까지 첫번째 라인에 작성하며, 다음 라인에 들여쓰기 공백 4칸을 포함하여 나머지를 표기합니다.

```kotlin
fun f(x: String) =
    x.length
```

### 프로퍼티 포맷

간단한 읽기 전용 프로퍼티는 한줄포 표기합니다:

```kotlin
val isEmpty: Boolean get() = size == 0
```

더 복잡한 프로퍼티의 경우 `get` 과 `set` 키워드는 항상 다른 라인에 표기합니다:

```kotlin
val foo: String
    get() { /*...*/ }
```

프로퍼티 초기화 구문이 길 경우 `=` 까지 첫번째 라인에 작성하며, 다음 라인에 들여쓰기 공백 4칸을 포함하여 나머지를 표기합니다:

```kotlin
private val defaultCharset: Charset? =
    EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```

### 제어문 포맷

`if` 또는 `when`의 조건이 여러개라면, 구문의 바디에 괄호를 사용하여야 합니다. 각 조건의 라인에 들여쓰기 공백 4칸을 포함해야 합니다. 조건의 닫는 괄호와 바디의 열린 괄호를 다른 라인에 표기합니다:

```kotlin
if (!component.isSyncing &&
    !hasAnyKotlinRuntimeInScope(module)
) {
    return createKotlinNotConfiguredPanel(module)
}
```

> Rationale: 조건부와 바디 구문의 명확한 구분과 정렬

`else`, `catch`, `finally` 키워드와 do/while 루프의 `while` 키워드는 앞 조건의 닫는 괄호와 같은 라인에 표기합니다:

```kotlin
if (condition) {
    // body
} else {
    // else part
}

try {
    // body
} finally {
    // cleanup
}
```

`when` 구문에서 브랜치가 한줄보다 길 경우 공백라인과 블럭 괄호를 추가하여 구분시켜 줍니다:

```kotlin
private fun parsePropertyValue(propName: String, token: Token) {
    when (token) {
        is Token.ValueToken ->
            callback.visitValue(propName, token.value)

        Token.LBRACE -> { // ...
        }
    }
}
```

짧은 브랜치는 괄호 없이 조건을 같은 라인에 표기합니다.

```kotlin
when (foo) {
    true -> bar() // good
    false -> { baz() } // bad
}
```

### 메서드 호출 포맷

argument가 여러개인 경우, 열린 괄호 이후 다음 라인에 공백 4칸을 포함하여 표기합니다. 관련도가 높은 argument 들은 같은 라인에 표기합니다.

```kotlin
drawSquare(
    x = 10, y = 10,
    width = 100, height = 100,
    fill = true
)
```

이름과 value 구분을 위해 `=` 사이에 공백을 추가합니다.

### 체인드 호출 래핑 \(Chained call wrapping\)

체인드 호출시 `.` 또는 `?.` 연산자는 다음 라인에 들여쓰기 공백 4칸을 포함하여 표기합니다:

```kotlin
val anchor = owner
    ?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace }
```

체인에서 첫 호출 시 다음 라인에 표기를 해주어야 하나, 간단한 구문이거나 이해하기 쉬운 구문일 경우 생략해도 됩니다 \(`{ it is PsiComment || it is PsiWhiteSpace }`\).

### 람다 포맷

람다 표현은 괄호와 활살표 주변에 공백을 포함해 줍니다. If a call takes a single lambda, it should be passed outside of parentheses whenever possible.

```kotlin
list.filter { it > 10 }
```

람다에 라벨링을 할 경우 괄호와 라벨링 사이에 공백을 표기하지 않습니다:

```kotlin
fun foo() {
    ints.forEach lit@{
        // ...
    }
}
```

여러 줄로 람다를 선언할 경우 네임과 화살표를 첫번째줄에 표기하고 화살표 다음은 다음 라인에 표기합니다:

```kotlin
appendCommaSeparated(properties) { prop ->
    val propertyValue = prop.get(obj)  // ...
}
```

싱글라인으로 작성하기에 파라미터가 길 경우, 화살표는 다른 라인에 표기합니다:

```kotlin
foo {
   context: Context,
   environment: Env
   ->
   context.configureEnv(environment)
}
```

## Doccument 주석

긴 내용의 주석을 사용할 경우 `/**` 표기 후 다음 라인에서 `*`와 함께 내용을 작성하면 됩니다:

```kotlin
/**
 * This is a documentation comment
 * on multiple lines.
 */
```

짧은 내용은 한줄로 작성 가능합니다:

```kotlin
/** This is a short documentation comment. */
```

일반적으로 `@param` 와 `@return` 태그는 지양해야 합니다. 대신에, 파라미터에 대한 링크 생성이 가능하므로 본문 주석에 해당 내용을 추가합니다. 본문 내용과 맞지 않게 설명이 필요한 경우 `@param` 와 `@return` 태그를 사용할 수 있습니다.

```kotlin
// Avoid doing this:

/**
 * Returns the absolute value of the given number.
 * @param number The number to return the absolute value for.
 * @return The absolute value.
 */
fun abs(number: Int) { /*...*/ }

// Do this instead:

/**
 * Returns the absolute value of the given [number].
 */
fun abs(number: Int) { /*...*/ }
```

## 중복 구문 피하기

일반적으로 IDE에서 특정 구문이 강조 되는 경우 명확하게 표현하기 위해 생략 해야 합니다.

### Unit

함수의 리턴 타입이 Unit일 경우 생략해야 합니다:

```kotlin
fun foo() { // ": Unit" is omitted here

}
```

### 세미콜론

세미콜론은 언제나 생략 가능합니다.

### 문자열 템플릿

간단한 변수를 문자열 템플릿에 넣으려면 괄호 \(`{` `}`\)가 필요하지 않습니다. 긴 구문으로 작성 시 괄호를 표기합니다.

```kotlin
println("$name has ${children.size} children")
```

## 관용구 사용 \(Idiomatic use of language features\)

### 불변성 \(Immutability\)

변하지 않는 데이터 사용을 추천합니다. 만약 초기화 후 값이 변하지 않는다면 로컬 변수와 프로퍼티는 `var`보다 `val`로 선언해야 합니다.

불변의 콜렉션을 선언할 때는 항상 불변 콜렉션 인터페이스 \(`Collection`, `List`, `Set`, `Map`\)를 사용하기 바랍니다. 콜렉션 인스턴스를 생성하기 위해 팩토리 함수를 사용할 때 가능하면 항상 불변의 콜렉션 타입을 반환하는 함수를 사용해야 합니다:

```kotlin
// Bad: use of mutable collection type for value which will not be mutated
fun validateValue(actualValue: String, allowedValues: HashSet<String>) { ... }

// Good: immutable collection type used instead
fun validateValue(actualValue: String, allowedValues: Set<String>) { ... }

// Bad: arrayListOf() returns ArrayList<T>, which is a mutable collection type
val allowedValues = arrayListOf("a", "b", "c")

// Good: listOf() returns List<T>
val allowedValues = listOf("a", "b", "c")
```

### 파라미터의 기본 값

오버로드 함수를 선언 할 때는 파라미터에 기본 값을 선언해 주시기 바랍니다.

```kotlin
// Bad
fun foo() = foo("a")
fun foo(a: String) { /*...*/ }

// Good
fun foo(a: String = "a") { /*...*/ }
```

### typealias \(Type aliases\)

함수형 타입이거나 여러번 사용되어지는 파라미터에 대해 type alias를 선언하시기 바랍니다:

```kotlin
typealias MouseClickHandler = (Any, MouseEvent) -> Unit
typealias PersonIndex = Map<String, Person>
```

### 람다 파라미터

짧고 중첩이 되지 않은 람다에서는, 파라미터 선언을 하지 않고 `it`을 사용하도록 권장합니다. 파라미터가 있는 중첩된 람다는 반드시 파라미터를 명시적으로 선언해 주어야 합니다.

### 람다 리턴

람다에서는 하나의 반환 포인트만 고려하여 설계되기 때문에 여러개의 레이블 된 반환값을 피해야 합니다. 만약에 불가능하거나 명확하지 않다면 람다에서 익명 함수로 변환을 고려해 봐야 합니다.

람다에서 마지막 구문에 레이블 된 반환값을 사용하면 안됩니다.

### 명시적 인자 \(Named arguments\)

여러 파라미터의 타입이 같거나 `Boolean` 타입, 모든 파라미터의 의미가 명확하지 않은 경우 명시적으로 파라미터명을 표기할 수 있습니다.

```kotlin
drawSquare(x = 10, y = 10, width = 100, height = 100, fill = true)
```

### 조건문 사용 \(Using conditional statements\)

`try`, `if` 와 `when`의 표현은 아래 예제를 참고하세요:

```kotlin
return if (x) foo() else bar()

return when(x) {
    0 -> "zero"
    else -> "nonzero"
}
```

아래의 형식보단 위의 형식을 더 선호합니다:

```kotlin
if (x)
    return foo()
else
    return bar()

when(x) {
    0 -> return "zero"
    else -> return "nonzero"
}
```

### `if` versus `when`

바이너리 조건에 대해서는 `when` 보다는 `if`를 선호합니다. 아래의 예제보다는

```kotlin
when (x) {
    null -> // ...
    else -> // ...
}
```

이 구문을 더 선호 `if (x == null) ... else ...`

조건이 세개 또는 그 이상일 경우 `when`을 사용합니다.

### null이 가능한 `Boolean` 조건문 \(Using nullable `Boolean` values in conditions\)

null이 가능한 `Boolean` 타입을 조건으로 사용하려면 `if (value == true)` 또는 `if (value == false)` 체크가 필요합니다.

### 반복문 \(Using loops\)

반복문은 고차 함수 \(`filter`, `map` etc.\)를 사용하는 것을 선호합니다. 예외: `forEach` \(`forEach`가 nullable 한 값을 받지 않거나 긴 호출 체인에 일부분이 아닐경우 `for` 대체\)

복잡한 환경에서 고차 함수와 반복문 선택 시, 작업 비용과 성능을 비교하여 적용해야 합니다.

### 범위 반복문 \(Loops on ranges\)

범위 반복을 하려면 `until` 함수를 사용해야 합니다:

```kotlin
for (i in 0..n - 1) { /*...*/ }  // bad
for (i in 0 until n) { /*...*/ }  // good
```

### 문자열 사용 \(Using strings\)

문자열 연결은 문자열 템플릿을 사용해야 합니다.

여러줄 문자열을 사용하기 위해 `\n`을 포함시키기 보다 기본 문자열 기능을 사용해야 합니다.

여러줄 문자열을 사용할 때 들여쓰기가 필요한 경우 `trimMargin`을 사용하고 그렇지 않은 경우에는 `trimIndent`을 사용해야 합니다.

```kotlin
assertEquals(
    """
    Foo
    Bar
    """.trimIndent(), 
    value
)

val a = """if(a > 1) {
          |    return a
          |}""".trimMargin()
```

### 함수 vs 프로퍼티 \(Functions vs Properties\)

함수에 인자가 없을 경우에 읽기 전용 프로퍼티로 대체할 수 있습니다. 문법은 유사하지만 스타일 포맷이 다를 수 있습니다.

아래와 같은 상황에서는 함수보다는 프로퍼티를 선호합니다:

* throw 아닐 경우
* 계산 비용이 저렴할 경우 \(또는 첫 실행에서 cached 된 경우\)
* object 상태가 변경되지 않았을 때, 동일한 반환 결과를 가져올 경우

### 확장 함수 사용 \(Using extension functions\)

확장 함수를 자유롭게 사용해야 합니다. object에서 주로 동작하는 함수가 있을 때마다 확장 함수 생성에 대해 고민해 봐야 합니다. 가시성을 적절하게 제한하면 API 오염도를 줄일 수 있습니다. 필요에 따라 로컬 확장 함수, 멤버 확장 함수 또는 최상위 확장 함수를 private로 제한하시기 바랍니다.

### 고정 함수 사용 \(Using infix functions\)

유사한 역할의 두 object 에서 동작하는 함수의 경우 고정 함수로 선언이 가능합니다. 좋은 예: `and`, `to`, `zip`. 나쁜 예: `add`.

수신 object가 변경이 되는 경우 고정 함수로 선언하면 안됩니다.

### 팩토리 함수 \(Factory functions\)

팩토리 함수를 선언 할 때, class와 동일한 이름을 사용하면 안됩니다. 팩토리 함수의 기능을 명확하게 하는 이름을 사용하시기 바랍니다. 특별한 의미가 없을 경우에만 class와 동일한 이름을 사용할 수 있습니다.

예:

```kotlin
class Point(val x: Double, val y: Double) {
    companion object {
        fun fromPolar(angle: Double, radius: Double) = Point(...)
    }
}
```

다른 슈퍼 class 생성자를 호출하지 않는 여러개의 오버로드 생성자가 있고 기본 인자를 가지고 있는 하나의 생성자로 줄일 수 없는 경우, 오버로드 생성자들을 팩토리 함수로 대체하는 것을 선호합니다.

### 플랫폼 타입 \(Platform types\)

플랫폼 타입을 반환하는 public으로 선언 된 함수/메서드는 반드시 Kotlin 타입이 명시적으로 선언되어야 합니다:

```kotlin
fun apiCall(): String = MyJavaApi.getProperty("name")
```

플랫폼 타입을 초기화 하는 어떠한 프로퍼티 \(패키지 레벨 또는 class 레벨\)는 반드시 Kotlin 타입이 명시적을 선언되어야 합니다:

```kotlin
class Person {
    val name: String = MyJavaApi.getProperty("name")
}
```

플랫폼 타입을 초기화 하는 로컬 변수는 Kotlin 타입을 명시할 필요가 없습니다:

```kotlin
fun main() {
    val name = MyJavaApi.getProperty("name")
    println(name)
}
```

### 스코프 함수 사용 apply/with/run/also/let \(Using scope functions apply/with/run/also/let\)

Kotlin은 다양한 코드 블럭 함수를 지원합니다: `let`, `run`, `with`, `apply`, `also`. 스코프 함수에 대한 적절한 사용은 [Scope Functions](https://kotlinlang.org/docs/reference/scope-functions.html) 참고하시기 바랍니다.

## 라이브러리를 위한 코딩 작성 스타일 \(Coding conventions for libraries\)

라이브러리 작성 시 API의 안정성을 위해 다음 사항을 권장합니다:

* 항상 멤버 가시성을 명확하게 선언해 주시기 바랍니다 \(실수로 public API로 노출 되지 않기 위함\).
* 항상 함수의 반환 타입과 프로퍼티 타입을 명확하게 선언해 주시기 바랍니다 \(변경 될 때 반환 타입이 변경 되지 않기 위함\).
* 오버라이드 될 때 참고할 수 있도록 모든 public 멤버에 대해 KDoc을 제공해야 합니다 \(라이브러리를 위한 문서 생성을 지원하기 위해\).

