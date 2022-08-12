---
title: "[Kotlin] 개념 정리 - Conditions and loops"
last_modified_at: 2022-08-10T21:00:00+09:00
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

# If expression

코틀린에서 `if`는 값을 리턴하는 표현식이다.

`if`가 `삼항 연산자(condition ? then : else)`를 대체하기 때문에 삼항 연산자는 더이상 존재하지 않는다.

```kotlin
var max = a
if (a < b) max = b

// With else
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}

// As expression
val max = if (a > b) a else b
```

`if`표현식을 블록으로도 표현할 수 있는데 이 경우 블록의 마지막 표현식이 리턴 값이 된다.

```kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

이 경우 만약 값을 리턴하거나 변수에 할당하기 위해서는 else 문은 필수가 된다.

<br>

# When expression

`when`은 조건문이 여러개인 경우 사용된다. 이것은 마치 `switch` 문법과 유사하다.

```kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> {
        print("x is neither 1 nor 2")
    }
}
```

`when`의 인자값을 모든 조건문에 대해 어느 하나 일치할 때까지 순차적으로 매칭시켜본다.

`when` 사용 시 표현식을 사용할 경우 첫 번째 매칭된 조건문이 전체 표현식의 값이 된다. 만약 블록을 통해 감싸는 경우 해당 블록의 마지막 표현식이 값이 된다.

아무 조건문에도 일치하는 결과를 얻지 못할 경우 `else`를 체택하게 된다.

`when`이 표현식으로 사용될 경우 컴파일러가 판단에 의해 선언된 조건문이 모든 경우 의 수를 커버(`enum class` 목록 검증 또는 `sealed class` 서브타입 검증)할 수 있는지 판단되지 않는한 `else`는 필수로 선언되어야 한다.

```kotlin
enum class Bit {
  ZERO, ONE
}

val numericValue = when (getRandomBit()) {
    Bit.ZERO -> 0
    Bit.ONE -> 1
    // 'else' is not required because all cases are covered
}
```

`when` statements 일 때, 다음과 같은 상황에서 `else`는 필수이다.

- Boolean, enum, sealed type, nullable 한 것들
- 조건문들이 모든 경우의 수를 커버하지 못하는 경우

```kotlin
enum class Color {
  RED, GREEN, BLUE
}

when (getColor()) {
    Color.RED -> println("red")
    Color.GREEN -> println("green")
    Color.BLUE -> println("blue")
    // 'else' is not required because all cases are covered
}

when (getColor()) {
  Color.RED -> println("red") // no branches for GREEN and BLUE
  else -> println("not red") // 'else' is required
}
```

다중 조건문을 단일 라인에 선언하기 위해서는 `콤마(,)`를 사용하면 된다.

```kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

조건문으로서 상수 뿐만 아니라 임의의 표현식을 사용할 수도 있다.

```kotlin
when (x) {
    s.toInt() -> print("s encodes x")
    else -> print("s does not encode x")
}
```

범위나 콜랙션에 대해 `in or !in`을 통해 조건문을 사용할 수도 있다.

```kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

`is or !is`를 통해 변수가 특정 타입에 해당하는지 여부도 조건문으로 사용할 수 있다.

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

`when`은 또한 `if-else or if` 문을 대체할 수 있다. 만약 `when`에 별도의 인자가 제공되지 않은 경우 각각의 조건문은 단순히 `boolean` 표현식이 되어 조건문이 `true`인 경우 수행하게 된다.

```kotlin
when {
    x.isOdd() -> print("x is odd")
    y.isEven() -> print("y is even")
    else -> print("x+y is odd")
}
```

다음과 같은 문법을 이용하여 `when`의 인자값을 캡처하는 기능도 가능하다.

```kotlin
fun Request.getBody() =
    when (val response = executeRequest()) {
        is Success -> response.body
        is HttpError -> throw HttpException(response.status)
    }
```

이때 변수의 범위는 `when`내에서만 유효하다.

<br>

# For loops

iterator를 제공하는 경우 `for` 문을 사용할 수 있는데 이는 C#에서의 `foreach`와 동일하다.

```kotlin
for (item in collection) print(item)

for (item: Int in ints) {
    // ...
}
```

숫자 범위를 순회하기 위해서는 범위 표현식을 사용하면 된다.

```kotlin
for (i in 1..3) {
    println(i)
}

for (i in 6 downTo 0 step 2) {
    println(i)
}
```

범위 또는 배열에 대해서는 iterator 객체를 생성하지 않는 인덱스 기반의 루프로 컴파일 된다.

만약 배열 또는 리스트를 인덱스와 함께 iterate 하고 싶다면 아래와 같은 방법으로 구현 할 수 있다.

```kotlin
for (i in array.indices) {
    println(array[i])
}
```

위와 같이 하는 방법 대신에 `withIndex` 라이브러리 함수를 이용할 수도 있다.

```kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

<br>

# While loops

`while`과 `do-while` 문은 조건문이 만족될 때까지 계속해서 실행된다.

위 두가지는 조건문을 검증하는 시기에 있어 차이를 갖는다.

- `while`: 조건을 확인하고 만약 만족되면 본문을 수행한다.
- `do-while`: 본문을 수행하고 조건문을 확인한다. 만약 조건문이 만족되면 본문을 반복하여 수행한다. 따라서 조건문에 상관없이 본문이 무조건 1회 이상 수행된다.

```kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // y is visible here!
```

<br>

# Break and continue in loops

코틀린은 전통적인 `break`와 `continune` 연산자를 지원한다. [[참고]](https://kotlinlang.org/docs/returns.html)

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Control flow > Conditions and loops](https://kotlinlang.org/docs/control-flow.html)