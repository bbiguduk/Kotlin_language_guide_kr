# 제너릭 \(Generics\)

Java와 같이 Kotlin의 class는 타입 파라미터를 가질 수 있습니다:

```kotlin
class Box<T>(t: T) {
    var value = t
}
```

일반적으로 class 인스턴스를 생성하려면 타입 인자를 제공해야 합니다:

```kotlin
val box: Box<Int> = Box<Int>(1)
```

그러나 생성자 인자 등을 통해 추론 가능한 파라미터라면 타입 인자를 생략해도 됩니다:

```kotlin
val box = Box(1) // 1 has type Int, so the compiler figures out that we are talking about Box<Int>
```

## 가변 \(Variance\)

Java에서 와일드카드 타입은 가장 모호한 타입 중 하나 입니다 \(자세한 내용은 [Java 제너릭 FAQ \(Java Generics FAQ\)](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html) 참고 바랍니다\). Kotlin은 이러한 타입은 없지만, 2가지 다른게 있습니다: 선언위치 가변과 타입 추론 입니다.

먼저 Java가 왜 모호한 와일드카드 타입을 갖게 되었는지 생각해 봅시다. 이것은 [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html)에 Item 31: _Use bounded wildcards to increase API flexibility_에 설명 되어 있습니다. Java의 제너릭 타입은 **불변 \(invariant\)**입니다. 이 뜻은 `List<String>`은 `List<Object>`의 서브 타입이 아니라는 의미 입니다. 만약에 리스트가 **불변 \(invariant\)**가 아니라면 이것은 Java의 배열보다 좋은 점이 없습니다. 아래 코드는 컴파일은 되나 실행 시 런타임 예외가 발생하게 됩니다:

```java
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! A compile-time error here saves us from a runtime exception later
objs.add(1); // Here we put an Integer into a list of Strings
String s = strs.get(0); // !!! ClassCastException: Cannot cast Integer to String
```

그래서 Java는 런타임 안정성을 보장하기 위해 위와 같은 행위를 금지합니다. 그러나 다른 의미로 예를 들어 `Collection` 인터페이스로 부터 `addAll()` 메서드를 생각해 봅시다. 이 메서드는 어떻게 표현할까요? 아래와 같이 표현 할 수 있습니다:

```java
// Java
interface Collection<E> ... {
  void addAll(Collection<E> items);
}
```

그러나 아래와 같이 간단한 작업 \(완벽하게 안전한\)도 수행할 수 없습니다.

```java
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from);
  // !!! Would not compile with the naive declaration of addAll:
  // Collection<String> is not a subtype of Collection<Object>
}
```

\(Java에서 [Effective Java, 3rd Edition](http://www.oracle.com/technetwork/java/effectivejava-136174.html)의 Item 28: _Prefer lists to arrays_에 자세한 내용 참고 바랍니다\)

실제 `addAll()`은 아래와 같이 표현합니다:

```java
// Java
interface Collection<E> ... {
  void addAll(Collection<? extends E> items);
}
```

**와일드카드 타입 인자 \(wildcard type argument\)** `? extends E`은 `E`의 객체의 콜렉션 또는 `E`의 서브타입의 콜렉션을 받아 들인다는 의미입니다. 이것은 아이템 \(콜렉션의 요소는 E의 서브 클래스의 인스턴스 입니다\)으로 부터 안전하게 **읽기**는 가능하지만 `E`의 알수없는 서브타입에는 **쓰기가 불가능** 합니다. 이 제한으로 인해 원하는 것을 수행할 수 있습니다: `Collection<String>`은 `Collection<? extends Object>`의 서브타입입니다. 어려운 말로 **extends**-bound \(**upper** bound\)로 와일드카드는 타입을 **공변 \(covariant\)** 하게 만든다 라고 합니다.

왜 이러한 방법이 가능한지는 간단합니다: 콜렉션에서 아이템을 **꺼내기**만 한다면 `String` 콜렉션에서 `Object`를 읽는 것은 괜찮습니다. 반대로 콜렉션에 아이템을 **넣기**만 한다면 `Object` 콜렉션에 `String`을 넣어도 괜찮습니다: Java에서 `List<? super String>`은 `List<Object>`의 **슈퍼타입**입니다.

후자를 **반변성 \(contravariance\)** 이라고 부르고 `List<? super String>`은 String을 인수로 취하는 메서드만 호출 할 수 있습니다 \(`add(String)` 또는 `set(int, String)`은 호출 가능\). 반면에 `List<T>`을 호출하여 반환 된 `T`는 `String`이 아니라 `Object` 형태 입니다.

Joshua Bloch는 객체는 **생산자 \(Producers\)**에서만 읽고 **소비자 \(Consumers\)**에서만 쓴다고 말했습니다. 그는 "_유연성을 위해 생산자 또는 소비자에 입력 파라미터는 와일드카드 타입을 사용하라_"고 말했습니다. 그리고 그는 아래의 방법으로 기억하라고 말했습니다:

_PECS는 Producer-Extends, Consumer-Super를 말합니다._

_참고_: `List<? extends Foo>`인 생산자 객체를 사용한다면 `add()` 또는 `set()` 사용이 불가하지만 객체가 **불변 \(immutable\)**이라는 뜻은 아닙니다: 예를 들어 파라미터를 사용하지 않는 `clear()`는 리스트의 모든 항목을 삭제하기 위해 `clear()` 호출하는 것을 막을 수 없습니다. 와일드카드가 보장하는 것은 **타입 안정성 \(type safety\)** 입니다. 불변성은 완벽히 다른 이야기 입니다.

### 선언 위치 가변 \(Declaration-site variance\)

`T`를 파라미터로 쓰는 메서드가 없고 `T`를 반환하는 메서드만 있는 `Source<T>` 인터페이스가 있다고 가정합시다:

```java
// Java
interface Source<T> {
  T nextT();
}
```

`Source<Object>` 타입의 변수가 `Source<String>`의 인스턴스를 참조하는 것은 완벽하게 안전합니다 -- 소비자 메서드가 없기 때문입니다. 그러나 Java는 이를 알 수 없기 때문에 이러한 방식은 금지합니다:

```java
// Java
void demo(Source<String> strs) {
  Source<Object> objects = strs; // !!! Not allowed in Java
  // ...
}
```

이 문제를 해결하려면 의미없는 `Source<? extends Object>` 타입의 객체를 선언해야 합니다. 와일드카드가 없어도 이전과 똑같이 모든 메서드를 호출할 수 있기 때문입니다.

Kotlin은 컴파일러에게 이러한 것을 설명할 수 있습니다. 이것을 **선언 위치 가변 \(declaration-site variance\)**이라고 부릅니다: `Source<T>`에 멤버는 오직 **반환 \(returned\)** \(produced\)만 되고 절대 소비되지 않는면 **out** 을 제공해 나타낼 수 있습니다:

```kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```

기본적인 규칙: class `C`에 파라미터 `T`를 **out**으로 선언하면 `C`의 멤버는 오직 반환 할 때만 사용되기 때문에 `C<Base>`은 안전하게 `C<Derived>`의 슈퍼타입이 될 수 있습니다.

어려운 말로 `C`class는 파라미터 `T`에서 **공변 \(covariant\)**이거나 `T`가 **공변 \(covariant\)** 타입 파라미터 입니다. `C`는 `T`의 **소비자 \(consumer\)**가 아닌 **생산자 \(producer\)**입니다.

**out**은 **가변 annotation \(variance annotation\)**이라 부르며, 타입 파라미터 선언하는 곳에 추가 하기 때문에 **선언 위치 가변 \(declaration-site variance\)**이라 말합니다. 타입에 와일드카드를 사용하는 Java의 **사용 위치 가변 \(use-site variance\)**과 대조됩니다.

Kotlin은 **out**외에 추가로 **in** 가변 annotation을 제공합니다. **in**은 타입 파라미터를 **반공변 \(contravariant\)**으로 만듭니다: 이것은 오직 소비만 가능합니다. `Comparable`은 반공변의 아주 좋은 예입니다:

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, we can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}
```

**in** 과 **out**은 단어 그대로 설명이 되어지는 수식어 입니다 \(C\#에서도 이미 사용 중\) 따라서 반드시 필요한 것은 아니지만 더 고차원 적으로 표현할 수 있습니다:

[**실존적 \(The Existential\)**](http://en.wikipedia.org/wiki/Existentialism) **변환: 소비자 \(Consumer\) 안으로, 생산자 \(Producer\) 밖으로!** :-\)

## 타입 추론 \(Type projections\)

### 사용 위치 가변: 타입 추론 \(Use-site variance: Type projections\)

타입 파라미터 T를 _out_으로 선언하면 서브 타입 문제를 피할 수 있지만 항상 `T`를 반환하도록 강제로 수행 할 수는 없습니다! 가장 좋은 예는 배열 입니다:

```kotlin
class Array<T>(val size: Int) {
    fun get(index: Int): T { ... }
    fun set(index: Int, value: T) { ... }
}
```

클래스의 `T`는 공변도 반공변도 아니기 때문에 제약이 있습니다. 아래 함수를 참고 하십시오:

```kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```

이 함수는 하나의 배열에서 다른 배열로의 복사를 수행합니다. 아래 연습 코드를 확인해 보십시오:

```kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" } 
copy(ints, any)
//   ^ type is Array<Int> but Array<Any> was expected
```

아까와 비슷한 문제가 존재합니다: `Array<T>`은 `T`에 대해 불변이기 때문에 `Array<Int>` 와 `Array<Any>`은 서로의 서브타입이 아닙니다. 왜냐하면 복사가 잘못 동작 할 수도 있기 때문입니다. 예를 들어 `from`에 문자열을 쓴다고 가정합시다. 만약 `Int`를 전달하게 된다면 `ClassCastException`가 발생하게 됩니다.

따라서 `copy()`의 잘못 동작하는 것을 방지하기 위해 `from`에 쓰려는 행동을 금지할 수 있습니다:

```kotlin
fun copy(from: Array<out Any>, to: Array<Any>) { ... }
```

이러한 **타입 추론 \(type projection\)**은 `from`이 일반 배열이 아닌 제한 된 배열 \(**추론 된 \(projected\)**\) 입니다. `from`이 타입 파라미터 `T`를 반환하는 메서드만 호출 할 수 있도록 제한합니다. 이러한 경우 `get()`만 호출 할 수 있습니다. Java의 `Array<? extends Object>`와 비슷한 **사용 위치 가변 \(use-site variance\)**이지만 더 간결합니다.

**in**으로도 사용 가능합니다:

```kotlin
fun fill(dest: Array<in String>, value: String) { ... }
```

`Array<in String>`은 Java의 `Array<? super String>`와 비슷합니다. `CharSequence` 또는 `Object` 배열을 `fill()` 함수에 전달할 수 있습니다.

### 별 추론 \(Star-projections\)

타입 인자에 대해 아무것도 모르지만 안전한 방법으로 사용하고 싶은 경우가 있습니다. 안전한 방법은 제너릭의 타입 추론을 정의하고 제너릭 타입을 초기화 시 추론 된 서브타입으로 진행하는 것입니다.

Kotlin은 **별 추론 \(star-projection\)**을 제공합니다:

* `T`가 `TUpper`이며 공변 타입 파라미터 인 `Foo<out T : TUpper>`가 있으면, `Foo<*>`와 `Foo<out TUpper>`은 같습니다. 이것은 `T`를 몰라도 `Foo<*>`으로 부터 `TUpper`의 값을 읽을 수 있다는 의미입니다.
* `T`가 반공변 타입 파라미터 인 `Foo<in T>`가 있으면, `Foo<*>`와 `Foo<in Nothing>`은 같습니다. 이것은 `T`를 모르면 `Foo<*>`에 쓸 수 없다는 것을 의미합니다.
* `T`가 `TUpper`이며 불변 타입 파라미터 인 `Foo<T : TUpper>`가 있으면, `Foo<*>`은 읽을 땐 `Foo<out TUpper>`와 같고 쓸 때는 `Foo<in Nothing>`와 같습니다.

제너릭 타입이 여러개의 타입 파라미터를 가지면 각각 따로 추론할 수 있습니다. 예를 들어 `interface Function<in T, out U>`로 선언하면 다음의 별 추론이 가능합니다:

* `Function<*, String>` 은 `Function<in Nothing, String>` 을 의미합니다.
* `Function<Int, *>` 은 `Function<Int, out Any?>` 을 의미합니다.
* `Function<*, *>` 은 `Function<in Nothing, out Any?>` 을 의미합니다.

_참고_: 별 추론은 Java의 원시 타입과 유사하지만 안전합니다.

## 제너릭 함수 \(Generic functions\)

함수도 클래스 처럼 타입 파라미터를 가질 수 있습니다. 타입 파라미터는 함수 이름 **전**에 위치합니다:

```kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

fun <T> T.basicToString(): String {  // extension function
    // ...
}
```

제너릭 함수를 호출하려면 함수 이름 **뒤**에 타입 인자를 명시해야 합니다:

```kotlin
val l = singletonList<Int>(1)
```

타입 인자는 아래 예제와 같이 타입 추론이 가능한 경우 생략이 가능합니다:

```kotlin
val l = singletonList(1)
```

## 제너릭 제약 \(Generic constraints\)

주어진 타입 파라미터로 대체될 수 있는 모든 가능한 타입은 **제너릭 제약 \(generic constraints\)**로 인해 제한 될 수 있습니다.

### 상한 \(Upper bounds\)

Java의 _확장_ 같상\(은 **upper bou\)nd**은 가장 많이 쓰이는 제약입니다:

```kotlin
fun <T : Comparable<T>> sort(list: List<T>) {  ... }
```

콜론뒤에 명시 된 타입이 **upper bound** 입니다: `Comparable<T>`의 서브타입만 `T`로 대체 가능합니다. 예를 들어:

```kotlin
sort(listOf(1, 2, 3)) // OK. Int is a subtype of Comparable<Int>
sort(listOf(HashMap<Int, String>())) // Error: HashMap<Int, String> is not a subtype of Comparable<HashMap<Int, String>>
```

따로 명시하지 않으면 upper bound의 기본값은 `Any?`입니다. 꺽쇠 \(**&lt;&gt;**\)에는 오직 하나의 upper bound만 표기합니다. 같은 타입 파라미터에 하나 이상의 upper bound가 필요한 경우 **where**로 분리해줍니다:

```kotlin
fun <T> copyWhenGreater(list: List<T>, threshold: T): List<String>
    where T : CharSequence,
          T : Comparable<T> {
    return list.filter { it > threshold }.map { it.toString() }
}
```

전달 된 타입은 반드시 `where`절의 모든 조건을 만족시켜야 합니다. 위의 예에서 `T` 타입은 `CharSequence` 와 `Comparable` 모두 구현 되어야 합니다.

## 타입 삭제 \(Type erasure\)

Kotlin은 제너릭 선언에 대한 안정성 확인을 컴파일 시간에만 수행합니다. runtime시에는 제너릭 타입의 인스턴스는 타입 인자 정보를 가지고 있지 않습니다. 타입 정보는 지워집니다. 예를 들어 `Foo<Bar>` 와 `Foo<Baz?>`의 인스턴스는 `Foo<*>`에 의해 지워집니다.

따라서 런타임 시 어떤 타입 인자에 의해 제너릭 타입의 인스턴스가 생성되었는지 확인 할 수 없으며 컴파일러는 [_is_ 키워드 사용을 금지합니다](https://kotlinlang.org/docs/reference/typecasts.html#type-erasure-and-generic-type-checks).

타입 인자를 가지고 있는 제너릭 타입의 타입 캐스트는 런타임 시 체크 할 수 없습니다. 예: `foo as List<String>` [unchecked casts](https://kotlinlang.org/docs/reference/typecasts.html#unchecked-casts)은 더 상위 레벨 프로그램 로직에 의해 안정성이 암시 되지만 컴파일러가 직접 유추할 수 없는 경우에 사용할 수 있습니다. 컴파일러는 이러한 unchecked casts에 경고하고, 런타임시에는 제너릭이 아닌 부분만 검사합니다 \(`foo as List<*>`\).

제너릭 함수의 타입 인자는 컴파일 시에 체크합니다. 함수 안에서는 타입 파라미터는 타입 체크로 사용이 불가능하고 타입 파라미터에 의한 타입 캐스트도 체크하지 않습니다. 그러나 인라인 함수의 [reified type parameters](http://app.gitbook.com/@bbiguduk/s/kotlin/language-guide/functions-and-lambdas/inline-functions#reified-type-parameters)은 호출 위치의 인라인 함수 안에서 실제 타입 인자로 대체되기 때문에 위에서 말한 제너릭 타입 인스턴스의 제약 안에서 타입 검사와 캐스트를 할 수 있습니다.

