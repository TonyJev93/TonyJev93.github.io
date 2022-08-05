---
title: "[Kotlin] 개념 정리 - Types"
last_modified_at: 2022-08-04T21:00:00+09:00
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

# Basic types

## Numbers

- 코틀린에서 자체적으로 제공하는 숫자 타입이 존재한다.
- `정수` 타입으로는 4가지 유형이 존재한다.

![image](https://user-images.githubusercontent.com/53864640/182738036-bdf9bc66-3773-42a6-87ab-4effd6fd0281.png)

- 별도로 타입을 명시하지 않는 이상 그리고 `Int`타입의 최대값을 넘지 않는 이상 default 타입은 `Int` 이다.
- `Int`의 최대값을 넘는 상태로 초기화 시 자동으로 `Long` 타입으로 선언된다. 
- `L`을 숫자 끝에 명시하여 `Long`타입으로 변환도 가능하다.

```kotlin
val one = 1 // Int
val threeBillion = 3000000000 // Long
val oneLong = 1L // Long
val oneByte: Byte = 1
```

<br>

## Floating-point

- 실수 영역을 표시하기 위해 코틀린은 `Float`과 `Double` 타입을 제공해준다.

![image](https://user-images.githubusercontent.com/53864640/182738383-063d35eb-2da9-4bdd-9117-9f8f83bbe036.png)

- 소수점을 가지는 숫자로 초기화를 할 경우 `Double`타입으로 선언된다.

```kotlin
val pi = 3.14 // Double
// val one: Double = 1 // Error: type mismatch
val oneDouble = 1.0 // Double
```

- 값 끝에 `f or F`를 명시적으로 표기할 경우 `Float`으로 선언할 수 있다.
- 만약 `Float`인데 소수점 자리수가 6-7자리를 넘는다면 경계에서 자동으로 반올림된다.

```kotlin
val e = 2.7182818284 // Double
val eFloat = 2.7182818284f // Float, actual value is 2.7182817
```

- 코틀린에서는 다른 언어들과 다르게 숫자의 자동 확장기능을 제공하지 않는다.
- 예를들면, `Double` 파라미터를 사용하는 함수는 오로지 `Double` 값에 의해서만 호출이 가능하다.(`Float, Int, ...` 사용시 오류)
- 그럼에도 타입 변경을 원한다면 [Explicit conversions](https://kotlinlang.org/docs/basic-types.html#explicit-conversions)을 사용하면 된다.

```kotlin
fun main() {
    fun printDouble(d: Double) {
        print(d)
    }

    val i = 1
    val d = 1.0
    val f = 1.0f

    printDouble(d)
//    printDouble(i) // Error: Type mismatch
//    printDouble(f) // Error: Type mismatch
}
```

<br>

## Literal Constants

다음과 같은 리터럴 상수 유형이 존재한다.

- Decimals: 123
  - Loangs are tagged by a captial `L`: 123L
- Hexadecimals: 0x0F
- Binaries: 0b00001011
- (Ocatal literals 는 지원되지 않는다.)
- Doubles by default: 123.5, 123.5e10
- Floats are tagged by `f or F`: 123.5f

또한, 밑줄을 통해 숫자 상수 값의 가독성을 높일 수 있다.

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

<br>

## Numbers representation on the JVM

JVM 안에서 숫자는 primitive types(int, double, ...)로 저장된다.

그치만, nullable한 숫자 참조(`Int`)나 제네릭을 통해 생성된 경우는 예외이다. 이 경우 `Integer, Double, ...`과 같은 자바 클래스로 박싱 된다.

nullable 숫자 참조는 값이 같더라도 다른 객체로 판단할 수 있다.

```kotlin
val a: Int = 100
val boxedA: Int? = a
val anotherBoxedA: Int? = a

val b: Int = 10000
val boxedB: Int? = b
val anotherBoxedB: Int? = b

println(boxedA === anotherBoxedA) // true
println(boxedB === anotherBoxedB) // false
```

위 예제에서 nullable한 `a`는 JVM가 Integer의 경우 -128 ~ 127 사이 값에 적용하는 메모리 최적화에 의해 같은 객체로 판단되었다.

그치만 메모리 최적화가 적용되지 않은 b 의 경우 다른 객체로 인식한다.

```kotlin
val b: Int = 10000
println(b == b) // Prints 'true'
val boxedB: Int? = b
val anotherBoxedB: Int? = b
println(boxedB == anotherBoxedB) // Prints 'true'
```

반면, 위와 같이 값 자체비교(`==`)는 동일한 것을 알 수 있다. 

<br/>

## Explicit conversions

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Types ](https://kotlinlang.org/docs/basic-types.html)