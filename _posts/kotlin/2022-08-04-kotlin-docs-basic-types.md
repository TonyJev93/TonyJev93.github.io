---
title: "[Kotlin] 개념 정리 - Basic Types"
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

코틀린에서는 어느 변수에서든 맴버 함수와 맴버 속성들을 호출할 수 있다는 점에서 전부 객체라고 할 수 있다.

numbers, characters, booleans와 같이 런타임에서는 primitive 로 표현 될 수 있지만, 사용자에게는 일반적인 클래스처럼 보여지는 것처럼 일부 타입들은 특별한 내부 표현방식을 가지고 있다.

이번 장에서는 코틀린에서 사용되는 기본 타입들에 대해 설명할 것이다.

- numbers
- booleans
- characters
- strings
- arrays

<br>

# Numbers

## Integer Types

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

## Floating-point Types

- 실수 영역을 표시하기 위해 코틀린은 `Float`과 `Double` 타입을 제공해준다.

![image](https://user-images.githubusercontent.com/53864640/182738383-063d35eb-2da9-4bdd-9117-9f8f83bbe036.png)

- 소수점을 가지는 숫자로 초기화를 할 경우 `Double`타입으로 선언된다.

```kotlin
val pi = 3.14 // Double
// val one: Double = 1 // Error: type mismatch
val oneDouble = 1.0 // Double
```

- 값 끝에 `f or F`를 명시적으로 표기할 경우 `Float`으로 선언할 수 있다.
- 만약 `Float`인데 소수점 자리수가 6-7자리를 넘는다면 경계에서 자동으로 2진수 반올림된다.

```kotlin
val e = 2.7182818284 // Double
val eFloat = 2.7182818284f // Float, actual value is 2.7182817
```

- 코틀린에서는 다른 언어들과 다르게 숫자의 암시적 확장 변환을 제공하지 않는다.
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

그치만, nullable한 숫자 참조(`Int?`)나 제네릭을 통해 생성된 경우는 예외이다. 이 경우 `Integer, Double, ...`과 같은 자바 클래스로 박싱 된다.

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

서로 다른 내부 표현 방식으로 인해, 작은 타입이 큰 타입의 서브 타입일 수 없다. 아래 예제를 살펴 보겠다. 

```kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(b == a) // Surprise! This prints "false" as Long's equals() checks whether the other is Long as well
```

작은 타입인 `Int?`를 큰 타입 `Long?`에 대입 하였고, `Int?`와 `Long?` 타입으로 선언된 값의 동등성을 비교해보았다.

둘의 정체성(클래스)는 물론이거니와 값의 동등성(==) 조차 일치하지 않는다.

결론적으로, 작은 타입은 큰 타입으로 암시적 변환이 이루어지지 않는다. 이를 위해서는 명시적 변환을 해야한다.

```kotlin
val b: Byte = 1 // OK, literals are checked statically
// val i: Int = b // ERROR
val i1: Int = b.toInt()
```

모든 숫자타입은 아래와 같은 변환 함수를 지원한다.

- toByte(): Byte
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble(): Double
- toChar(): Char

많은 경우 명시적 변환을 사용할 필요는 없다. 왜냐하면, 문법적으로 타입이 추론가능하기 때문이다. 그리고 산술적인 연산에는 적절한 타입 변환을 포함시키고 있기 때문이다.

```kotlin
val l = 1L + 3 // Long + Int => Long
```

## Operations

코틀린은 숫자타입에 대해 산술적인 표준 연산들을 지원한다.(+, -, *, /, %)

```kotlin
println(1 + 2)
println(2_500_000_000L - 1L)
println(3.14 * 2.71)
println(10.0 / 3)
```

이러한 연산 로직들은 클래스의 맴버로서 선언 되어 존재한다. 그렇기 때문에 이러한 연산 로직을 커스텀 클래스에서 `override` 하여 사용할 수 있다.([Operator overloading 참고](https://kotlinlang.org/docs/operator-overloading.html))

## Division of Integers

정수에서의 나누기는 항상 정수타입을 반환한다. 모든 소숫점은 삭제된다.

이는 서로 다른 정수 타입들 간에도 유효하다.

```kotlin
val x = 5 / 2
//println(x == 2.5) // ERROR: Operator '==' cannot be applied to 'Int' and 'Double'
println(x == 2)

val x = 5L / 2 // 서로다른 정수 타입간에 Division
println(x == 2L)
```

부동 소수점 타입을 리턴하기 위해서는 연산되는 값들 중 하나에 암시적 변환을 수행해야 한다.

```kotlin
val x = 5 / 2.toDouble()
println(x == 2.5)
```

## Bitwise operations

코틀린에서는 정수 타입에 대해 비트단위 연산을 제공한다.

이들은 숫자 표현의 비트를 이용하여 이진 수준에서 직접 연산된다. 비트 연산은 중위 형식으로 호출되는 함수에 의해 표현된다.

이들은 오직 `Int`와 `Long`에 대해서만 적용된다.

```kotlin
val x = (1 shl 2) and 0x000FF000
```

다음은 비트 연산에 대한 전체 목록이다.

- shl(bits) – signed shift left
- shr(bits) – signed shift right
- ushr(bits) – unsigned shift right
- and(bits) – bitwise and
- or(bits) – bitwise or
- xor(bits) – bitwise xor
- inv() – bitwise inversion

## Floating-point numbers comparison

- Equality checks: `a == b and a != b`
- Comparison operators: `a < b, a > b, a <= b, a >= b`
- Range instantiation and range checks: `a..b, x in a..b, x !in a..b`

피연산자 a와 b가 정적으로 Float 또는 Double 또는 nullable 대응하는 것으로 알려진 경우 숫자 및 이들이 형성하는 범위에 대한 연산은 [IEEE 754의 부동 소수점 산술 표준](https://en.wikipedia.org/wiki/IEEE_754)을 따른다.

그러나, 제네릭 사용이나 전체 순서를 지원하기 위해 피연산자가 정적으로 부동 소수점 숫자(예: Any, Comparable<...>, 제네릭 타입 매개변수)타입이 지정되지 않은 경우 Float 및 Double 연산은 표준을 사용 하지 않고 equals 및 compareTo 구현을 사용한다.

- `NaN`은 자신과 동일한 것으로 간주 된다.
- `NaN`은 `POSITIVE_INFINITY`을 포함하여 다른 어떤 것보다 더 큰 것으로 간주된다.
- `-0.0` 은 `0.0`보다 작은 것으로 간주 된다.

## Unsigned integers

코틀린에서는 아래와 같이 부호 없는 정수 타입을 제공해준다.

- UByte: 0에서 255 사이의 부호 없는 8비트 정수
- UShort: 0에서 65535 사이의 부호 없는 16비트 정수
- UInt: 0에서 2^32 - 1 사이의 부호 없는 32비트 정수
- ULong: 0에서 2^64 - 1 사이의 부호 없는 64비트 정수

부호 없는 정수 타입 또한 부호있는 타입에서 지원하는 연산들 대부분을 지원한다.

<br>

# Booleans

`Boolean` 타입은 `true & false` 에 해당하는 값을 표현한다.

내장된 연산으로는 아래와 같다.

- || – disjunction (logical OR)
- && – conjunction (logical AND)
- ! - negation (logical NOT)

`||`와 `&&`연산은 지연 동작한다.

```kotlin
val myTrue: Boolean = true
val myFalse: Boolean = false
val boolNull: Boolean? = null

println(myTrue || myFalse) // true
println(myTrue && myFalse) // false
println(!myTrue) // false
```

<br>

# Characters

문자는 `Char`로 표현된다. 문자 리터럴은 홑 따움표로 표시된다. ex) `'1'`

특수 문자는 백슬래쉬(`\`)와 함께 시작되어 사용된다. 지원되는 이스케이프 시퀀스는 아래와 같다. 

- `\t, \b, \n, \r, \', \", \\, \$.`

다른 문자로 인코딩하기 위해서는 유니코드 이스케이프 시퀀스 문법을 사용한다. `'\uFF00'`

```kotlin
val aChar: Char = 'a'

println(aChar) // a
println('\n') //prints an extra newline character
println('\uFF00') // ＀
```

만약 문자 변수의 값이 숫자인 경우, `digitToInt()` 함수를 이용하여 `Int` 타입으로 명시적 변환을 할 수 있다.

<br>

# String

코틀린에서의 문자열은 `String`으로 표현된다. 일반적으로, 문자열의 값은 쌍 따움표로 표시된다. ex) `"1"`

```kotlin
val str = "abcd 123"
```

문자열의 요소들은 인덱싱 연산자(`s[i]`)를 통해 접근할 수 있는 문자들로 이루어져 있다.

`for` 반복문을 이용하여 각 문자들에 대해 접근 가능하다.

```kotlin
for (c in str) {
    println(c)
}
```

문자열은 불변(immutable)이다. 문자열을 한 번 초기화 하면 해당 값을 바꾸거나 새로운 값을 할당할 수 없다. 문자열을 변경시키는 모든 연산들은 기존 문자열을 변경하지 않은 채 `new String`을 통해 새로운 문자열 객체를 생성한다.

```kotlin
val str = "abcd"
println(str.uppercase()) // ABCD, Create and print a new String object
println(str) // abcd, the original string remains the same
```

문자열을 붙이기 위해서 `+` 연산자를 사용한다. 이는 다른 타입의 값을 문자열로 붙이는 작업도 가능하다.(단, 첫 번째 요소가 문자열이여야 함)

```kotlin
val s = "abc" + 1
println(s + "def")
```

대부분의 경우 문자열 연결보다 [문자열 템플릿](https://kotlinlang.org/docs/basic-types.html#string-templates) 또는 원시 문자열을 사용하는 것이 좋습니다.

## String literals

두 가지 유형의 문자열 리터럴이 존재한다.

- escaped strings (escaped 문자를 포함)
- raw strings (개행, 임의의 문자 포함)

```kotlin
val s = "Hello, world!\n" // escaped 문자 포함 예시
```

raw string은 세 개의 쌍따움표(""")로 구분된다. escape 문자를 포함하고 있지 않고, 개행과 다른 문자들을 포함 할 수 있다.

```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

raw string 각 행 앞에 공백을 제거하기 위해 trimMargin() 함수를 이용한다.

```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

기본적으로, `|` 는 마진으로서 접미어로 사용된다. 그 외에 `trimMargin(">")`와 같이 trimMargin 함수의 파라미터로 마진을 위해 사용할 문자를 넘겨 사용자화 할 수도 있다.

## String templates

`$`로 시작하는 표현식이나 중괄호(`{}`)로 묶인 표현식으로 템플릿 표현식을 사용할 수 있다.

```kotlin
val i = 10
println("i = $i") // prints "i = 10"

val s = "abc"
println("$s.length is ${s.length}") // prints "abc.length is 3"
```

아래와 같이 raw strings와 escaped strings 에서도 템플릿 사용이 가능하다. 식별자의 시작으로 허용되는 기호 앞에 `$` 문자를 raw strings에 삽입하려면 아래와 같이 사용하면 된다.

```kotlin
val price = """
${'$'}_9.99
"""
```

<br>

# Arrays

코틀린에서 배열은 `Array` 클래스로 표현된다.

이는 다른 유용한 멤버 함수와 함께 연산자 오버로딩 규칙과 `size` 속성에 의해 `[]`로 바뀌는 `get, set` 함수를 포함하고 있다.

```kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

배열 생성을 위해 arrayOf() 함수를 이용한다. 만약 값을 세팅하고 싶으면 `arrayOf(1, 2, 3)`과 같이 item 을 넘기면 된다.

`arrayOfNulls()` 함수를 사용한다면 정해진 사이즈에 맞게 `null`로 채워진 배열을 생성할 수 있다.

다른 방식으로는 아래와 같이 `Array`의 생성자를 사용하여 초기화 할 수도 있다.

```kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5) { i -> (i * i).toString() }
asc.forEach { println(it) }
```

코틀린에서의 배열은 불변인다. 즉, `Array<String>`을 `Array<Any>`로 할당 할 수 없다. 이는 런타임에 발생할 수 있는 실수를 예방하기 위함이다. 

## Primitive type arrays

코틀린에는 `ByteArray, ShortArray, IntArray`와 같이 박싱 오버헤드 발생없이도 primitive type을 배열로 표현할 수 있는 클래스들이 있다.

이 클래스들은 `Array` 클래스와 상속 관계를 갖지는 않지만 메서드 및 속성 집합이 동일하다. 각각에는 팩토리 기능도 존재한다.

```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

```kotlin
// Array of int of size 5 with values [0, 0, 0, 0, 0]
val arr = IntArray(5)

// e.g. initialise the values in the array with a constant
// Array of int of size 5 with values [42, 42, 42, 42, 42]
val arr = IntArray(5) { 42 }

// e.g. initialise the values in the array using a lambda
// Array of int of size 5 with values [0, 1, 2, 3, 4] (values initialised to their index value)
var arr = IntArray(5) { it * 1 }
```

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Types > Basic types](https://kotlinlang.org/docs/basic-types.html)