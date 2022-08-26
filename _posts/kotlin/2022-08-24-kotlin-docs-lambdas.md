---
title: "[Kotlin] 개념 정리 - Lambdas"
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

코틀린에서 함수는 [일급 객체](https://medium.com/@lazysoul/functional-programming-%EC%97%90%EC%84%9C-1%EA%B8%89-%EA%B0%9D%EC%B2%B4%EB%9E%80-ba1aeb048059)이다.

**일급 객체란?**<br>
아래 3가지 조건을 충족한다면 1급 객체라고 할 수 있다.

1. 변수로 할당 가능
2. 인자로 넘기기 가능
3. 리턴값으로서 리턴 가능

정적으로 타입이 지정된 프로그래밍 언어인 Kotlin은 [function types](https://kotlinlang.org/docs/lambdas.html#function-types)을 사용하여 함수를 나타내고 람다 표현식과 같은 전문화된 언어 구성 세트를 제공한다.

<br>

# 고차 함수(Higher-order functions)

고차 함수는 함수를 파리머터 또는 리턴값으로 가지고 있는 함수를 뜻한다.

고차 함수의 좋은 예시로 `fold`가 있다. 컬렉션의 요소들을 계속해서 결합하여 누적된 값을 반환하는 함수이다.

해당 함수는 파라미터로 초기값(`initial`)과 결합 함수(`combine`)를 사용한다. 

```kotlin
fun <T, R> Collection<T>.fold(
    initial: R,
    combine: (acc: R, nextElement: T) -> R  // 파라미터로 함수를 사용(= 고차 함수)
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}
```

위 코드에서 `combine` 파라미터로 함수 타입(`(R, T) -> R`)을 가지고 있고, `for` 루프 내에서 호출되고 있으며 리턴된 값을 `accumulator`에 할당한다.

`fold`를 호출하기 위해 `function type`의 인스턴스를 인자로 넘겨야 하는데 람다 표현식이 이런 상황에서 범용적으로 사용된다. 

```kotlin
val items = listOf(1, 2, 3, 4, 5)

// 람다는 중괄호로 묶인 코드 블록이다.
items.fold(0, { 
    // 람다에 파라미터가 있으면 파라미터가 먼저 쓰고 '->' 를 붙인다.
    acc: Int, i: Int -> 
    print("acc = $acc, i = $i, ") 
    val result = acc + i
    println("result = $result")
    result // 람다의 마지막 표현식이 리턴값으로 사용 됨.
})

// 람다의 파라미터 타입은 유추할 수 있는 경우 선택사항이다.
val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })

// 함수 참조(Function reference)는 고차 함수 호출에도 사용할 수 있다.
val product = items.fold(1, Int::times)
```

<br>

# 함수 타입(Function types)

코틀린은 함수를 다루는 선언에 `(Int) -> String`과 같은 함수 타입(function type)을 이용한다.

```kotlin
// ex)
val onClick: () -> Unit = ...
```

## 함수 타입 표기법

모든 함수 타입에는 `매개변수 및 반환 값`을 표현하기 위해 아래와 같은 특수한 표기법을 이용한다.

- 표기법: `(A, B) -> C`
    - `(A, B)`: 파라미터 타입
    - `C`: 리턴 타입
- 파라미터가 없는 경우: `() -> A`
- 리턴값이 없는 경우: `(A, B) -> Unit` 
  - `Unit`은 생략 불가능
- `nullable`한 함수 타입 표기: `((Int, Int) -> Int)?`
  - 전체를 괄호로 묶은 뒤 `?`를 붙인다.
- 함수 타입 결합: `(Int) -> ((Int) -> Unit)`
- [Type alias](https://kotlinlang.org/docs/type-aliases.html) 사용: `typealias ClickHandler = (Button, ClickEvent) -> Unit`

> 화살표 표기법은 **오른쪽 연관**이다.<br> 
> `(Int) -> (Int) -> Unit`은 `(Int) -> ((Int) -> Unit)`와 동일하고 `((Int) -> (Int)) -> Unit`과는 다르다.

### 수신기 타입(Receiver type) 표기법

함수 타입은 선택적으로 추가적인 `receiver` 타입을 가질 수 있다. 이 타입은 표기법에서 점(`.`) 앞에 지정된다.

- 표기법: `A.(B) -> C`
  - 매개변수 B를 사용하여 receiver 객체 A에서 호출
  - 값 C를 반환

[수신기가 있는 함수 리터럴(Function literals with receiver)](https://kotlinlang.org/docs/lambdas.html#function-literals-with-receiver)은 종종 이러한 타입과 함께 사용된다.

### [일시 중단 함수(Suspending function)](https://kotlinlang.org/docs/coroutines-basics.html#extract-function-refactoring) 표기법

- 표기법: `suspend () -> Unit` 또는 `suspend A.(B) -> C`
  - `suspend` 수정자 사용

## 함수 타입 인스턴스화(Instantiating a function type)

### 함수 리터럴에 코드 블록 사용

- 람다 표현식: `{a, b -> a + b}`
- 익명 함수: `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

### 참조(Reference) 사용

기존 선언에 대한 호출 가능한 참조(reference)를 사용한다.

- top-level, 로컬, 멤버 또는 확장 함수: `::isOdd`, `String::toInt`
- top-level, 멤버 또는 확장 속성: `List<Int>::size`
- 생성자: `::Regex`

### 사용자 정의 클래스 인스턴스 사용

함수 타입을 인터페이스로 구현하는 사용자 정의 클래스 인스턴스를 사용한다.

```kotlin
class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
}

val intFunction: (Int) -> Int = IntTransformer()
```

컴파일러는 충분한 정보가 있는 경우 변수에 대한 함수 유형을 유추할 수 있다.

```kotlin
val a = { i: Int -> i + 1 } // 추론된 함수 타입: (Int) -> Int
```

리시버가 있거나 없는 함수 타입의 문자가 아닌 값은 서로 교환할 수 있으므로 리시버가 첫 번째 매개변수를 대신할 수 있으며 그 반대의 경우도 마찬가지이다. 

예를 들어, `(A, B) -> C` 타입의 값은 `A.(B) -> C` 타입 대신에 사용 가능하다.(반대 경우도 가능)

```kotlin
val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
val twoParameters: (String, Int) -> String = repeatFun // OK

fun runTransformation(f: (String, Int) -> String): String {
    return f("hello", 3)
}
val result = runTransformation(repeatFun) // OK
```

## 함수 타입 인스턴스 호출(Invoking a function type instance)

함수 타입의 값을 호출하는 방법은 아래와 같다.

- `invoke(...)` 연산자 사용
- ex) `f.invoke(x)` or `f(x)`

receiver 타입이 포함된 경우 호출하는 방법은 아래와 같다.

- receiver 객체를 첫 번째 인자로 넘긴다.
- 확장 함수를 호출하듯 사용한다. (ex. `1.foo(2)`)

```kotlin
fun main() {
    val stringPlus: (String, String) -> String = String::plus
    val intPlus: Int.(Int) -> Int = Int::plus

    println(stringPlus.invoke("<-", "->"))  // invoke 사용
    println(stringPlus("Hello, ", "world!")) // invoke 생략

    println(intPlus.invoke(1, 1))   // invoke 사용 + receiver 첫 번째 인자로 넘기기
    println(intPlus(1, 2))  // invoke 생략 + receiver 첫 번째 인자로 넘기기
    println(2.intPlus(3)) // receiver 객체를 확장 함수 처럼 호출
}
```

## Inline functions

때로는 고차 함수에 대해 유연한 제어 흐름을 제공하는 인라인 함수를 사용하는 것이 좋다.

<br>

# 람다 표현식 및 익명 함수(Lambda expressions and anonymous functions)

람다 식과 익명 함수는 **함수 리터럴**이다.

**함수 리터럴**은 선언되지 않았지만 **즉시 표현식으로 전달되는 함수**를 말한다. 

다음 예를 살펴보자.

```kotlin
max(strings, { a, b -> a.length < b.length })
```

`max`는 두 번째 인자로 함수 값을 취하고 있기 때문에 고차 함수이고,

이 두 번째 인자는 **함수 리터럴**이라고 하는 **자체 함수인 표현식**으로, 다음 명명된 함수와 동일한 역할을 한다.

```kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

즉, 위와 같이(`{ a, b -> a.length < b.length }`) **명명 없이 바로 기능을 수행할 수 있는 함수**를 **함수 리터럴**이라고 한다.

## 람다 표현식 문법(Lambda expression syntax)

람다 식의 전체 문법 형식은 다음과 같다.

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
```

- 람다 식은 항상 **중괄호**(`{`)로 둘러싸여 있다.
- 전체 문법 형식의 **매개변수 선언**은 중괄호(`{`) 안에 들어가며 옵셔널한 타입 주석이 있다.
- 본문은 화살표(`->`) 뒤에 나온다.
- 람다의 유추된 리턴 타입이 `Unit`이 아니면, 람다 본문 내부의 마지막(또는 단일) 식이 곧 리턴 값으로 처리된다.

위 코드에서 옵셔널한 주석을 모두 생략하면 아래와 같다.

```kotlin
val sum = { x: Int, y: Int -> x + y }
```

## 후행 람다 통과(Passing trailing lambdas)

코틀린 컨벤션에 따르면 **함수의 마지막 매개변수가 함수**이면 해당 인자로 전달된 **람다 표현식을 괄호 밖에 배치**할 수 있다.

```kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

이러한 문법을 후행 람다(`trailing lambda`)라고 한다.

람다가 해당 호출의 유일한 인수인 경우 괄호를 완전히 생략할 수 있다.

```kotlin
run { println("...") }
```

## it: 단일 매개변수의 암시적 이름(implicit name of a single parameter)

람다 표현식에 매개변수가 하나만 있는 것은 매우 일반적이다.

컴파일러가 매개변수 없이도 파싱하는데 문제가 없다면 매개변수를 선언할 필요가 없으며 `->`를 생략할 수 있다.

대신 매개변수는 `it`라는 이름으로 암시적으로 선언된다.

```kotlin
ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```

## 람다 식의 값 반환(Returning a value from a lambda expression)

[qualified return](https://kotlinlang.org/docs/returns.html#return-to-labels) 문법을 사용하여 람다에서 값을 명시적으로 반환할 수 있다. 

그렇지 않으면 마지막 표현식의 값이 암시적으로 반환된다.

```kotlin
ints.filter {
    val shouldFilter = it > 0
    shouldFilter // 마지막 줄 표현식이 리턴
}

ints.filter {
    val shouldFilter = it > 0
    return@filter shouldFilter  // qualified return 문법
}
```

이 규칙은 괄호 외부에 람다 식을 전달하는 것을 이용해서 아래와 같은 LINQ 스타일([LINQ-style](https://docs.microsoft.com/en-us/previous-versions/dotnet/articles/bb308959(v=msdn.10)))의 코드도 허용한다.

LINQ(Language-Integrated Query)
: C# 언어에서 쿼리 기능을 사용하는 것

```kotlin
strings.filter { it.length == 5 }.sortedBy { it }.map { it.uppercase() }
```

## 사용하지 않는 변수에 대한 밑줄(Underscore for unused variables)

람다 매개변수를 사용하지 않는 경우 이름 대신 밑줄을 배치할 수 있다.

```kotlin
map.forEach { (_, value) -> println("$value!") }
```

## 람다에서 구조 분해(Destructuring in lambdas)

람다에서 구조 분해는 [구조 분해 선언](https://kotlinlang.org/docs/destructuring-declarations.html#destructuring-in-lambdas)를 참고하자.

## 익명 함수(Anonymous functions)

위의 람다 식 문법에는 함수의 리턴 타입을 지정하는 기능이 빠졌다.

대부분의 경우 리턴 타입이 자동으로 유추될 수 있으므로 필요하지 않겠지만,

명시적으로 지정해야 하는 경우를 위해 대체 문법인 `익명 함수(anonymous function)`를 사용하면 된다.

```kotlin
fun(x: Int, y: Int): Int = x + y
```

익명 함수는 이름이 생략된다는 점을 제외하면 일반 함수 선언과 매우 유사하다. 

본문은 표현식(위에 표시된 대로) 뿐만 아니라 블록일 수도 있다.

```kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

익명 함수와 일반 함수 사이에는 매개변수와 리턴 타입이 문맥상 유추할 수 있는 경우 **매개변수 타입을 생략**할 수 있다는 **차이점**이 있다.

이것을 제외하고는 일반 함수와 동일한 방식으로 사용된다.

```kotlin
ints.filter(fun(item) = item > 0)   // 매개변수 item에 대한 타입 생략
```

익명 함수에 대한 **리턴 타입 유추**는 일반 함수와 마찬가지로 작동한다. 

리턴 타입은 **표현식 으로 선언된 익명 함수**에 대해 **자동으로 유추**되지만, 

**블록 본문이 있는 익명 함수**에 대해서는 **명시적으로 지정** 하거나 **`Unit`으로 가정**된다.

> 익명 함수를 매개변수로 전달할 때 괄호 안에 넣어야 한다.<br> 
> 함수를 괄호 외부에 둘 수 있도록 하는 약식 구문은 람다 식에만 적용된다.

람다 식과 익명 함수의 또 다른 차이점은 `non-local` 반환의 동작이다. 

`label`이 없는 return 문은 항상 `fun` 키워드로 선언된 함수에서 반환된다. 

즉, **람다 식 내부의 반환**은 **바깥쪽 함수에서 반환**되는 반면 **익명 함수 내부의 반환**은 익명 함수 자체에서 반환된다.

## Closures

람다 식 또는 익명 함수(로컬 함수 및 객체 식 포함)는 외부 범위에서 선언된 변수를 포함하는 클로저에 접근할 수 있다.

클로저에 캡처된 변수는 람다에서 수정할 수 있다.

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum) // output: 6
```

## receiver가 있는 함수 리터럴(Function literals with receiver)

`A.(B) -> C`와 같이 receiver가 있는 함수 타입은 특수한 형태의 함수 리터럴(receiver가 있는 함수 리터럴)로 인스턴스화할 수 있다.

위에서 언급했듯이 코틀린은 `receiver 객체`를 제공하면서 receiver로 함수 타입의 인스턴스를 호출하는 기능을 제공한다.

함수 리터럴의 본문 내부에서 호출에 전달 된 receiver 객체는 암시적 `this`가 되어 추가 한정자 없이 해당 수신자 객체의 멤버에 액세스 하거나 `this` 표현식을 사용하여 receiver 객체에 액세스 할 수 있다. (?)

이 동작은 확장 함수의 동작과 유사하며, 이를 통해 함수 본문 내부의 receiver 객체 멤버에 접근할 수도 있다.

다음은 해당 유형과 함께 receiver가 있는 함수 리터럴의 예시 이다. 여기서 `plus`는 receiver 객체에서 호출된다.

```kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) }
```

익명 함수 문법을 사용하면 함수 리터럴의 receiver 타입을 직접 지정할 수 있다. 

이것은 receiver를 사용하여 함수 타입의 변수를 선언한 다음 나중에 사용해야 하는 경우에 유용할 수 있다.

```kotlin
val sum = fun Int.(other: Int): Int = this + other
```

문맥상 receiver 타입을 유추할 수 있는 경우 Lambda 표현식을 receiver와 함께 함수 리터럴로 사용할 수 있다. 

사용의 가장 중요한 예 중 하나는 `type-saf builder`이다.

```kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // create the receiver object
    html.init()        // pass the receiver object to the lambda
    return html
}

html {       // lambda with receiver begins here
    body()   // calling a method on the receiver object
}
```

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Functions > Lambdas](https://kotlinlang.org/docs/lambdas.html)