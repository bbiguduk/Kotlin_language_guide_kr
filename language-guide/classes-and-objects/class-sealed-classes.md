# 한정 클래스 \(Sealed Classes\)

한정 클래스 \(sealed class\)는 제한된 클래스 계층 구조를 나타내기 위해 사용되며, 제한된 집합의 타입 중 하나만 값으로 가질 수 있으며 다른 타입은 가질 수 없습니다. 열거형 클래스의 확장 개념을 가집니다: 열거형 값의 집합도 제한 되지만 열거형 인스턴스는 단일 인스턴스로만 존재합니다. 그러나 한정 클래스는 상태를 포함 한 여러개의 인스턴스를 가질 수 있습니다.

한정 클래스는 선언하려면 `sealed`를 클래스 이름 앞에 붙여야 합니다. 한정 클래스는 서브 클래스를 가질 수 있지만 모두 한정 클래스와 동일한 파일에 선언 되어야 합니다 \(Kotlin 1.1 이전에는 한정 클래스안에 클래스가 중첩되어 있어야 했습니다\).

```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

\(위 예에서는 Kotlin 1.1의 새로운 기능을 사용합니다: 데이터 클래스에 한정 클래스를 포함하여 다른 클래스로 확장 할 수 있습니다.\)

한정 클래스는 자체적으로 [추상적 \(abstract\)](class-classes-and-inheritance.md#abstract-classes) 이며, 직접적으로 인스턴스화 될 수 없고 _추상_ 멤버는 가질 수 있습니다.

한정 클래는 _private_이 아닌 생성자를 사용할 수 없습니다 \(한정 클래의 생성자는 기본적으로 _private_ 입니다\).

한정 클래스의 서브 클래스를 확장하는 클래스는 꼭 같은 파일에 위치할 필요는 없습니다.

한정 클래스를 사용으로 얻는 가장 큰 이득은 [`when` 표현 \(`when` expression\)](../basics/control-flow-if-when-for-while.md#when-when-expression) 에서 사용할 때 입니다. 모든 조건 \(case\)에 대해 동작을 구현한다면 `else` 조건을 추가하지 않아도 됩니다. 그러나 반드시 `when`을 표현구로 사용해야 하며 구문일 때는 사용이 불가합니다.

```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // the `else` clause is not required because we've covered all the cases
}
```

