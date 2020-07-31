# 관용구 (Idioms)

Kotlin에서 자주 사용하는 관용구입니다. 원하는 관용구가 있다면 사용하시기 바랍니다.

### DTO(Data Transfer Object) 생성 (POJO/POCO - Plain Old Java/CLR Object)

```kotlin
data class Customer(val name: String, val email: String)
```

`Customer` class는 아래의 기능을 제공합니다:

* 모든 프로퍼티에 대한 getters (setters의 경우 *var*)
* `equals()`
* `hashCode()`
* `toString()`
* `copy()`
* `component1()`, `component2()`, ..., for all properties (see [Data classes](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/classes-and-objects/class-data-classes))


### 함수 파라미터에 대한 기본 값

```kotlin
fun foo(a: Int = 0, b: String = "") { ... }
```

### 리스트 필터링

```kotlin
val positives = list.filter { x -> x > 0 }
```

또는 더 짧게 표현이 가능합니다:

```kotlin
val positives = list.filter { it > 0 }
```

### 콜렉션 안에 엘리먼트 존재여부 체크

```kotlin
if ("john@example.com" in emailsList) { ... }

if ("jane@example.com" !in emailsList) { ... }
```

### 문자열 보간

```kotlin
println("Name $name")
```

### 인스턴스 체크

```kotlin
when (x) {
    is Foo -> ...
    is Bar -> ...
    else   -> ...
}
```

### 쌍으로 이루어진 map/list 구분

```kotlin
for ((k, v) in map) {
    println("$k -> $v")
}
```

`k`, `v` 다른 명칭으로 지정가능합니다.

### 범위 사용

```kotlin
for (i in 1..100) { ... }  // closed range: includes 100
for (i in 1 until 100) { ... } // half-open range: does not include 100
for (x in 2..10 step 2) { ... }
for (x in 10 downTo 1) { ... }
if (x in 1..10) { ... }
```

### 읽기 전용 list

```kotlin
val list = listOf("a", "b", "c")
```

### 읽기 전용 map

```kotlin
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
```

### map 접근

```kotlin
println(map["key"])
map["key"] = value
```

### Lazy 프로퍼티

```kotlin
val p: String by lazy {
    // compute the string
}
```

### 함수 확장

```kotlin
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```

### Singleton 생성

```kotlin
object Resource {
    val name = "Name"
}
```

### if not null 짧은 표현

```kotlin
val files = File("Test").listFiles()

println(files?.size)
```

### if not null과 else 짧은 표현

```kotlin
val files = File("Test").listFiles()

println(files?.size ?: "empty")
```

### if null 실행 구문

```kotlin
val values = ...
val email = values["email"] ?: throw IllegalStateException("Email is missing!")
```

### 콜렉션이 비어있을 수 있는 상태에서 첫번째 item 가져오기

```kotlin
val emails = ... // might be empty
val mainEmail = emails.firstOrNull() ?: ""
```

### if not null 실행 구문

```kotlin
val value = ...

value?.let {
    ... // execute this block if not null
}
```

### null이 가능한 map value에 대한 if not null

```kotlin
val value = ...

val mapped = value?.let { transformValue(it) } ?: defaultValue 
// defaultValue is returned if the value or the transform result is null.
```

### when 구문 리턴

```kotlin
fun transform(color: String): Int {
    return when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
}
```

### 'try/catch' 표현

```kotlin
fun test() {
    val result = try {
        count()
    } catch (e: ArithmeticException) {
        throw IllegalStateException(e)
    }

    // Working with result
}
```

### 'if' 표현

```kotlin
fun foo(param: Int) {
    val result = if (param == 1) {
        "one"
    } else if (param == 2) {
        "two"
    } else {
        "three"
    }
}
```

### `Unit`을 리턴하는 빌더-스타일 메서드 사용방법

```kotlin
fun arrayOfMinusOnes(size: Int): IntArray {
    return IntArray(size).apply { fill(-1) }
}
```


### 싱글 표현 함수

```kotlin
fun theAnswer() = 42
```

아래와 같이 표현도 가능합니다.

```kotlin
fun theAnswer(): Int {
    return 42
}
```

다른 관용구와 섞어서 표현이 가능합니다. 예를 들어, *when*-표현과 같이 사용 가능합니다::

```kotlin
fun transform(color: String): Int = when (color) {
    "Red" -> 0
    "Green" -> 1
    "Blue" -> 2
    else -> throw IllegalArgumentException("Invalid color param value")
}
```

### 객체 인스턴스에서 여러 메서드 호출 (`with`)

```kotlin
class Turtle {
    fun penDown()
    fun penUp()
    fun turn(degrees: Double)
    fun forward(pixels: Double)
}

val myTurtle = Turtle()
with(myTurtle) { //draw a 100 pix square
    penDown()
    for (i in 1..4) {
        forward(100.0)
        turn(90.0)
    }
    penUp()
}
```

### 객체의 프로퍼티 구성 (`apply`)
```kotlin
val myRectangle = Rectangle().apply {
    length = 4
    breadth = 5
    color = 0xFAFAFA
}
```

이 방법은 프로퍼티를 구성하는데 유용합니다. 객체의 생성자는 접근할 수 없습니다.

### Java 7의 리소스 접근

```kotlin
val stream = Files.newInputStream(Paths.get("/some/file.txt"))
stream.buffered().reader().use { reader ->
    println(reader.readText())
}
```

### 제너릭 타입 정보를 요구하는 제너릭 함수 표현

```kotlin
//  public final class Gson {
//     ...
//     public <T> T fromJson(JsonElement json, Class<T> classOfT) throws JsonSyntaxException {
//     ...

inline fun <reified T: Any> Gson.fromJson(json: JsonElement): T = this.fromJson(json, T::class.java)
```

### null이 가능한 Boolean 타입

```kotlin
val b: Boolean? = ...
if (b == true) {
    ...
} else {
    // `b` is false or null
}
```

### 두 변수 스와핑

```kotlin
var a = 1
var b = 2
a = b.also { b = a }
```

### TODO(): 미완성 코드에 대한 표기
 
Kotlin's standard library has a `TODO()` function that will always throw a `NotImplementedError`.
Its return type is `Nothing` so it can be used regardless of expected type.
There's also an overload that accepts a reason parameter:
```kotlin
fun calcTaxes(): BigDecimal = TODO("Waiting for feedback from accounting")
```

IntelliJ IDEA's kotlin plugin understands the semantics of `TODO()` and automatically adds a code pointer in the TODO toolwindow. 

