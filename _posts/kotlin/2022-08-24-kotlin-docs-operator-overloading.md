---
title: "[Kotlin] 개념 정리 - Operator overloading"
last_modified_at: 2022-08-25T21:00:00+09:00
categories:
    - Kotlin
tags:
    - Kotlin
toc: true
toc_sticky: true
toc_label: "목차"
---

Kotlin : 코틀린의 기본 지식을 쌓아보자.
{: .notice--info}

# 연산자 오버로딩(Operator overloading)

코틀린에는 타입에 대해 사전 정의된 연산자들을 커스텀하게 구현할 수 있도록 지원해준다.

이러한 연산자에는 사전 정의된 기호 표현(예: + 또는 )과 우선 순위가 존재한다.

연산자를 `implements`하려면 해당 타입에 대한 특정 이름을 가진 멤버 함수 또는 확장 함수를 제공해야한다.

[//]: # (이 유형은 이진 연산의 경우 왼쪽 타입이 되고 단항 연산의 경우 인자 타입이 된다.&#40;?&#41;)

연산자를 오버로드하려면 해당 함수를 `operator` 수정자로 표시하면 된다.

```kotlin
interface IndexedContainer {
    operator fun get(index: Int)
}
```

오버로드된 연산자를 오버라이딩할 때는 `operator`를 생략해도 된다.

```kotlin
class OrdersList: IndexedContainer {
    override fun get(index: Int) { /*...*/ }
}
```

<br>

# 단항 연산(Unary operations)

## 단항 접두사 연산자(Unary prefix operators)

| Expression | Translated to  |  
|------------|----------------|
| +a         | a.unaryPlus()  |
| -a         | a.unaryMinus() |
| !a         | a.not()        |

위 테이블을 토대로 했을때 컴파일러는 `+a`에 대해 다음과 같은 단계로 처리한다.

1. `a`의 타입을 결정한다. (그것이 `T`라고 가정하자.)
2. `operator` 수정자가 있으면서 `T` 수신자에 대한 매개변수가 없는 `unaryPlus()` 함수를 찾는다. 이것은 멤버 함수 또는 확장 함수를 의미한다.
3. 만약 함수가 없거나 모호한 경우 컴파일 에러가 발생한다. 
4. 만약 함수가 존재하고 `R` 타입을 리턴한다면 표현식 `+a`의 타입은 `R`이다.

예를들어 아래는 단항 minus 연산자를 어떻게 오버로드 할 수 있는지를 보여준다.

```kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(10, 20)

fun main() {
   println(-point)  // prints "Point(x=-10, y=-20)"
}
```

## Increments and decrements

| Expression | Translated to |  
|-----------|---------|
| `a++`       | a.inc() |
| `a--`       | a.dec() |

`inc()`와 `dec()` 함수는 반드시 값을 리턴하고 이 값은 `++` 또는 `--` 연산자가 사용된 변수에 대입되게 된다.

그리고 `inc` 또는 `dec`가 호출된 객체는 변경돼서는 안된다.(본인의 값이 증가 또는 감소되면 안되고 그대로여야 함)

컴파일러는 `a++(or a--)`와 같은 `postfix` 형태의 연산자를 수행하기 위해 아래와 같은 과정을 진행한다.

1. `a`의 타입을 결정한다. (그것이 `T`라고 가정하자.)
2. T 유형의 수신자에 적용할 수 있는 `operator` 수정자가 있고 매개변수가 없는 `inc()` 함수를 찾는다.
3. 리턴 타입이 `T`의 서브타입인지 확인한다.

표현식 계산은 아래와 같이 진행된다.

- `a`의 초기값을 임시저장소 `a0`에 저장 (본인 값은 그대로 유지하기 위해 임시저장소를 사용)
- `a0.inc()`의 결과를 `a`에 할당한다.
- 표현식의 결과로 `a0`를 반환한다.

`++a`, `--a`에 대해서도 유사하게 수행된다.

- `a.inc()`를 `a`에 할당한다. (본인에게 변경된 값을 할당)
- 새로운 값 `a`를 표현식의 결과로 리턴한다.

<br>

# 이항 연산자(Binary operations)

## 산술 연산자(Arithmetic operators)

| Expression | Translated to |  
|------------|---------------|
| a + b      | a.plus(b)     |
| a - b      | a.minus(b)    |
| a * b      | a.times(b)    |
| a / b      | a.div(b)      |
| a % b      | a.rem(b)      |
| a .. b     | a.rangeTo(b)  |

위 테이블의 연산자들을 위해 컴파일러는 `Translated to` 컬럼에 있는 표현식을 처리한다.

아래는 주어진 값에서 시작하고 오버로드 된 `+`연산자에 의해 값의 증가할 수 있는 `Counter` 클래스에 대한 예제이다.

```kotlin
data class Counter(val dayIndex: Int) {
    operator fun plus(increment: Int): Counter {
        return Counter(dayIndex + increment)
    }
}
```

## in 연산자(in operator)

| Expression | Translated to    |  
|------------|------------------|
| a in b     | b.contains(a)    |
| a !in b    | !b.contains(a)   |

`in`과 `!in`의 절차는 동일하지만 위에서 봤던 산술 연산자와는 달리 인자의 순서가 반대이다.(`a`, `b` 순서가 바뀜)

## 인덱싱된 액세스 연산자(Indexed access operator)

| Expression            | Translated to             |  
|-----------------------|---------------------------|
| a[i]                  | a.get(i)                  |
| a[i, j]               | a.get(i, j)               |
| a[i_1, ..., i_n]      | a.get(i_1, ..., i_n)      |
| a[i] = b              | a.set(i, b)               |
| a[i, j] = b           | a.set(i, j, b)            |
| a[i_1, ..., i_n] = b  | a.set(i_1, ..., i_n, b)   |

대괄호(`[`)에 대해서는 적절한 수의 인자들을 포함한 `get`과 `set`으로 변환된다.

## 호출 연산자(invoke operator)

| Expression           | Translated to            |  
|----------------------|--------------------------|
| a()                  | a.invoke()               |
| a(i)                 | a.invoke(i)              |
| a(i, j)              | a.invoke(i, j)           |
| a(i_1, ..., i_n)     | a.invoke(i_1, ..., i_n)  |

괄호(`(`)는 적절한 수의 인자들을 포함한 `invoke`를 호출하도록 변환된다.

## 증강 할당(Augmented assignments)

?? 뭐라는건지 모르겠음...

| Expression | Translated to    |  
|------------|------------------|
| a += b     | a.plusAssign(b)  |
| a -= b     | a.minusAssign(b) |
| a *= b     | a.timesAssign(b) |
| a /= b     | a.divAssign(b)   |
| a %= b     | a.remAssign(b)   |

`a += b`를 예를들어, 할당 연산자에서 컴파일러는 아래와 같은 단계를 수행한다.

- 오른쪽 열(컬럼)의 기능을 사용할 수 있는 경우:
  - 해당하는 이진 함수(`plusAssign()`의 경우 `plus()`를 의미함)도 사용할 수 있고 `a`는 변경 가능한 변수이고 `plus`의 반환 타입이 `a` 타입의 하위 타입인 경우, 모호성에 의해 오류가 발생한다.
  - 리턴 타입이 `Unit(= void in java)`인지 확인하자. 그렇지 않으면 오류가 발생한다.
  - `a.plusAssign(b)`에 대한 코드를 작성해라.
- 그렇지 않으면, `a = a + b`에 대한 코드를 만들자.(단, 여기서 `a + b`의 반환 타입이 `a`의 하위 타입이어야 한다.)

코틀린에서 `Assignments`는 표현식이 아니다.(즉, 반환값이 없다.)

## 등식 및 부등식 연산자(Equality and inequality operators)

| Expression | Translated to                   |  
|------------|---------------------------------|
| a == b     | a?.equals(b) ?: (b === null)    |
| a != b     | !(a?.equals(b) ?: (b === null)) |

해당 연산자는 오로지 `eqauls(other: Any?): Boolean` 함수와 함께 동작한다. 이는 커스텀한 동등성 체크를 제공하기 위해 오버라이딩 될 수 있다.

`equals(other: Foo)`와 같은 동일한 이름의 다른 함수들은 호출되지 않는다.

> 정체성(identity) 체크(`=== 과 !==`)는 오버로딩할 수 없다. 그래서 이들을 위한 컨밴션은 별도로 존재하지 않는다. 

`==` 연산자는 특별하게 `null`을 체크하는 복잡한 표현식으로 변환된다. `null == null`은 항상 true 이고, null이 아닌 `x`에 대해 `x == null`은 항상 false 이며 `x.equals()`를 호출하지 않는다.(eqauls를 호출하지 않고도 false를 반환 한다는 뜻)

## 비교 연산자(Comparison operators)

| Expression | Translated to        |  
|------------|----------------------|
| a > b      | a.compareTo(b) > 0   |
| a < b      | a.compareTo(b) < 0   |
| a >= b      | a.compareTo(b) >= 0  |
| a <= b      | a.compareTo(b) <= 0  |

모든 비교는 `Int`를 반환하는데 필요한 `comapreTo`에 대한 호출로 변환된다.

## 속성 위임 연산자(Property delegation operators)

`provideDelegate`, `getValue` 그리고 `setValue` 연산자 함수는 [Delegated properties](https://kotlinlang.org/docs/delegated-properties.html)에 설명되어 있다.

<br>

# 명명된 함수에 대한 중위 호출(Infix calls for named functions)

[`infix function calls`](https://kotlinlang.org/docs/functions.html#infix-notation)을 사용하여 사용자 정의 중위 조작을 시뮬레이션할 수 있다.

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Functions > Operator overloading](https://kotlinlang.org/docs/operator-overloading.html)