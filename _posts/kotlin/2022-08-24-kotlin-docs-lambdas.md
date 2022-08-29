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

# 일급 객체

코틀린에서의 함수는 [일급 객체](https://medium.com/@lazysoul/functional-programming-%EC%97%90%EC%84%9C-1%EA%B8%89-%EA%B0%9D%EC%B2%B4%EB%9E%80-ba1aeb048059)이다.

**일급 객체란?**<br>
아래 3가지 조건을 충족한다면 일급 객체라고 할 수 있다.

1. 변수 또는 데이타에 할당할 수 있는 경우
2. 인자로 넘길 수 있는 경우
3. 리턴값으로서 반환할 수 있는 경우

따라서, **코틀린의 함수**는 함수임에도 불구하고 변수나 데이터에 할당 가능하며, 인자로 넘길 수 있고 리턴값으로 반환이 가능하다.

<br>

# 고차 함수(Higher-order functions)

고차 함수란 무엇인지 예제를 통해 알아보도록 하겠습니다.

**고차 함수**는 **함수를 파리머터 또는 리턴값으로 가지고 있는 함수**를 뜻한다.

고차 함수의 좋은 예시로 `fold`라는 Collection 함수가 있는데 이는 컬렉션의 요소들을 계속해서 결합하여 누적된 값을 반환하는 기능을 수행한다.

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

이때, `fold`함수를 호출하기 위해서는 정의된 함수 타입에 맞는 **함수 인스턴스**를 인자로 넘겨야 하는데,

이때 **람다 표현식**이 범용적으로 사용된다.

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

// 람다의 파라미터 타입은 유추할 수 있는 경우 생략이 가능하다.
val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })

// 함수 참조(Function reference)는 고차 함수 호출에도 사용할 수 있다.
val product = items.fold(1, Int::times)
```

<br>

# 함수 타입(Function types)

이번에는 인자 또는 반환값으로 함수가 사용되는 경우 타입 지정을 어떻게 해야 하는지 알아보도록 하겠습니다.

코틀린은 함수를 다루는 선언부에 `(Int) -> String`과 같은 함수 타입(function type)을 이용한다.

```kotlin
// ex)
val intToString: (Int) -> String = ...
val onClick: () -> Unit = ...
```

## 함수 타입 표기법

모든 함수 타입에는 `매개변수 및 반환 값`을 표현하기 위해 아래와 같은 표기법을 이용한다.

- 표기법: `(A, B) -> C`
    - `(A, B)`: 파라미터 타입
    - `C`: 리턴 타입
- 파라미터가 없는 경우: `() -> A`
- 리턴값이 없는 경우: `(A, B) -> Unit` 
  - `Unit`은 생략 불가능
- `nullable`한 함수 타입 표기: `((Int, Int) -> Int)?`
  - 전체를 괄호로 묶은 뒤 `?`를 붙인다.
- 함수 타입 결합: `(Int) -> ((Int) -> Unit)`

> 주의: 화살표 표기법은 **오른쪽 연관**이다.<br> 
> `(Int) -> (Int) -> Unit`은 `(Int) -> ((Int) -> Unit)`와 동일하고 `((Int) -> (Int)) -> Unit`과는 다르다.

- [Type alias](https://kotlinlang.org/docs/type-aliases.html) 사용: `typealias ClickHandler = (Button, ClickEvent) -> Unit`

### 수신기 타입(Receiver type) 표기법

그렇다면 수신기가 포함된 경우에는 어떻게 표기를 할까?

함수 타입으로 `receiver`가 포함된 타입을 가질 수 있는데 이 경우 점(`.`)을 이용하여 표기한다.

- 표기법: `A.(B) -> C`
  - receiver 객체 A에서 매개변수 B의 호출 가능

아래에서 설명할 내용인 [수신기가 있는 함수 리터럴(Function literals with receiver)](https://kotlinlang.org/docs/lambdas.html#function-literals-with-receiver)은 종종 이러한 타입을 이용한다고 한다.

### [일시 중단 함수(Suspending function)](https://kotlinlang.org/docs/coroutines-basics.html#extract-function-refactoring) 표기법

일시 중단 함수의 경우 어떻게 표기할까?

- 표기법: `suspend () -> Unit` 또는 `suspend A.(B) -> C`
  - `suspend` 수정자 사용

## 함수 타입 인스턴스화(Instantiating a function type)

위에서 선언부에 들어갈 함수 타입을 어떻게 표기하는지에 대해 알아보았다면,

이제 이를 실질적으로 사용하기 위해 함수 타입을 어떻게 인스턴스화 할것인지 살펴보도록 하겠습니다.

### 함수 리터럴에 코드 블록을 사용하는 경우

- 람다 표현식: `{a, b -> a + b}`
- 익명 함수: `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

### 참조(Reference)를 사용하는 경우

참조(reference)를 사용하여 인스턴스화 수 있다.

ex) `::isOdd`, `String::toInt`, `List<Int>::size`, `::Regex`

### 사용자 정의 클래스를 통한 인스턴스 사용

인터페이스를 구현한 사용자 정의 클래스 또한 함수 타입 인스턴스로 사용 가능하다.

```kotlin
class IntTransformer: (Int) -> Int {
    override operator fun invoke(x: Int): Int = TODO()
}

val intFunction: (Int) -> Int = IntTransformer()
```

### 변수 타입 생략

컴파일러가 충분히 **변수**에 대한 함수 타입을 유추할 수 있는 경우 생략이 가능하다.

```kotlin
val a = { i: Int -> i + 1 } // 변수 a에 대해 추론된 함수 타입 = (Int) -> Int
```

### 매개변수-리시버 교체

첫 번째 매개변수와 리시버는 서로 교체가 가능하다. 

예를 들어, `(A, B) -> C`는 `A.(B) -> C`로 교체해서 사용 가능하다.(반대도 가능)

```kotlin
val repeatFun: String.(Int) -> String = { times -> this.repeat(times) }
val twoParameters: (String, Int) -> String = repeatFun // OK

fun runTransformation(f: (String, Int) -> String): String {
    return f("hello", 3)
}
val result = runTransformation(repeatFun) // OK
```

## 함수 타입 인스턴스 호출(Invoking a function type instance)

위에서 함수 타입에 대해 **인스턴스화 하는 방법**을 살펴보았으니까 이제는 인스턴스화 된 함수 타입을 **어떻게 호출하여 사용할 것인지** 알아보도록 하겠습니다.

함수 타입의 값을 호출하는 방법은 아래와 같다.

- `invoke(...)` 연산자 사용
  - ex) `f.invoke(x)` or `f(x) // invoke 생략`

receiver를 포함한 경우 호출하는 방법은 아래와 같다.

- 첫 번째 인자를 receiver 객체로 취급하여 사용 or 확장 함수를 호출하듯 사용 (ex. `1.foo(2)`)

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

<br>

# 함수 리터럴

람다 식과 익명 함수는 **함수 리터럴**이다.

**함수 리터럴**은 선언되지 않았지만 **즉시 표현식으로 전달되는 함수**를 말한다.

즉, 선언하지 않았음에도 불구하고 함수로서의 역할을 바로 수행한다고 보면 된다.

다음 예를 살펴보자.

```kotlin
max(strings, { a, b -> a.length < b.length })
```

`max`는 두 번째 인자로 함수 값을 취하고 있기 때문에 **고차 함수**이고,

이 두 번째 인자는 **함수 리터럴**이라고 하는 자체적으로 함수임을 뜻하는 표현식으로 다음 명명된 함수와 동일한 역할을 한다.

```kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

즉, 위의 `{ a, b -> a.length < b.length }`는 **명명 없이도 자체적으로 기능을 수행하기 때문에 함수 리터럴**이라고 할 수 있다.

<br>

# 람다 표현식(Lambda expressions)

## 람다 표현식 문법(Lambda expression syntax)

그럼 함수 리터럴인 람다 표현식의 문법에 대해 알아보도록 하겠습니다.

람다 식의 전체 문법 형식은 다음과 같다.

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
```

- 람다 식은 항상 **중괄호**(`{`)로 둘러싸여 있다.
- **매개변수**의 선언은 중괄호(`{`) 안에 들어가야 하며 타입에 대해서는 생략이 가능하다.
- 본문은 화살표(`->`) 우측에 정의된다.
- 람다의 유추된 리턴 타입이 `Unit`이 아니면, 람다 본문 내부의 **마지막(또는 단일) 식**이 곧 **리턴 값**으로 처리된다.

위 코드에서 생략 가능한 타입을 모두 제거하면 아래와 같다.

```kotlin
val sum = { x: Int, y: Int -> x + y } // return 타입이 유추가 가능하므로 생략 가능
```

## 후행 람다 통과(Passing trailing lambdas)

코틀린 컨벤션에 따르면 **함수의 마지막 매개변수가 함수**이면 해당 인자로 전달된 **람다 표현식을 괄호 밖에 배치**할 수 있다.

```kotlin
val product = items.fold(1, { acc, e -> acc * e }) // before
val product1 = items.fold(1) { acc, e -> acc * e } // after - 람다식을 괄호 밖으로 뺌
```

이러한 문법을 **후행 람다**(`trailing lambda`)라고 한다.

람다가 해당 호출의 유일한 인수인 경우 괄호를 완전히 생략할 수 있다.

## `it` 단일 매개변수의 암시적 이름(implicit name of a single parameter)

람다 표현식에 매개변수가 하나만 있는 것은 매우 일반적인데,

컴파일러가 매개변수 없이도 파싱하는데 문제가 없다면 매개변수를 선언할 필요가 없고 `->`를 생략할 수 있다.

대신 매개변수는 `it(= iterator)`라는 이름으로 암시적으로 선언된다.

```kotlin
ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```

## 람다 식의 값 반환(Returning a value from a lambda expression)

[한정된 반환(qualified return)](https://kotlinlang.org/docs/returns.html#return-to-labels) 문법을 사용하여 람다에서 값을 명시적으로 반환할 수 있다. 

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

람다 표현식에서 사용되지 않는 매개변수에 대해서는 이름 대신 밑줄을 배치할 수 있다.

```kotlin
map.forEach { (_, value) -> println("$value!") }
```

## 람다에서 구조 분해(Destructuring in lambdas)

람다에서 구조 분해는 [구조 분해 선언](https://kotlinlang.org/docs/destructuring-declarations.html#destructuring-in-lambdas)를 참고하자.

<br>

# 익명 함수(Anonymous functions)

위의 람다 표현식은 대부분의 경우 리턴 타입이 자동으로 유추될 수 있어 함수의 리턴 타입을 지정하는 기능이 빠져 있는데

**명시적으로 지정**해야 하는 경우에는 대체 문법인 `익명 함수(anonymous function)`를 사용하면 된다.

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

익명 함수와 일반 함수 모두 파라미터와 반환값에 대해 둘 다 타입을 명시하지만

**익명 함수**의 경우 문맥상 매개변수에 대해 유추가 가능한 경우 **매개변수의 타입을 생략**할 수 있다는 점에서 차이가 있다.

이것을 제외하고는 일반 함수와 동일한 방식으로 사용된다.

```kotlin
ints.filter(fun(item) = item > 0)   // 매개변수 item에 대한 타입 생략
```

익명 함수에 대한 **리턴 타입 유추**는 일반 함수와 마찬가지로 작동한다. 

리턴 타입은 **표현식 으로 선언된 익명 함수**에 대해 **자동으로 유추**되지만, 

**블록 본문이 있는 익명 함수**에 대해서는 **명시적으로 지정** 하거나 그렇지 않을 경우 **`Unit`으로 가정**된다.

 
> 마지막 매개변수가 함수인 경우, 괄호 밖에 람다식 사용이 가능 했지만<br>
> 반면 익명 함수는 매개변수로 전달될 때 반드시 괄호 안에 존재해야 한다.<br>

람다 식과 익명 함수의 또 다른 차이점은 `non-local` 반환의 동작이다. 

`label`이 없는 return 문은 항상 `fun` 키워드로 선언된 함수에서 반환된다. 

즉, **람다식 내부 반환**은 **바깥쪽 (`fun` keyword가 표기된)함수에서 반환**되는 반면 **익명 함수 내부 반환**은 (`fun`을 자체적으로 가지고 있기 때문에) 익명 함수 본인에서 반환된다.

<br>

# 클로저(Closures)

람다 식 또는 익명 함수는 **외부 범위에서 선언된 변수**에 접근할 수 있습니다.

클로저에 캡처된 변수는 람다에서 수정할 수도 있습니다.

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum) // output: 6
```

<br>

# receiver가 있는 함수 리터럴(Function literals with receiver)

`A.(B) -> C`와 같이 receiver가 있는 함수 타입은 **특수한 형태의 함수 리터럴**로 인스턴스화할 수 있다.

함수 리터럴의 본문에서 receiver 객체는 암묵적으로 `this`가 된다.

그렇기 때문에 `this` 표현식을 사용하거나 심지어 `this`를 생략 하고도 해당 receiver 객체의 맴버에 접근이 가능하다.

다음은 receiver가 있는 함수 리터럴의 예시 이다. 여기서 `plus`는 receiver 객체에 의해 호출된 것이다.

```kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) } // plus 는 Int(receiver 객체)에 의해 호출 된 것임.(this 생략)
```

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Functions > Lambdas](https://kotlinlang.org/docs/lambdas.html)