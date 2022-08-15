---
title: "[Kotlin] 개념 정리 - Exceptions"
last_modified_at: 2022-08-12T21:00:00+09:00
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

# Exception classes

코틀린의 모든 예외 클래스는 `Throwable` 클래스를 상속하고 있다.

모든 예외에는 `message, stack trace, optional cause`가 존재한다.

예외를 발생시키기 위해서는 `throw` 표현식을 이용한다.

```kotlin
throw Exception("Hi There!")

/**
 * Output:
 * Exception in thread "main" java.lang.Exception: Hi There!
 * at FileKt.main (File.kt:2)
 * at FileKt.main (File.kt:-1)
 * at jdk.internal.reflect.NativeMethodAccessorImpl.invoke0 (:-2) 
 */
```

예외를 catch 하기 위해 `try ... catch` 표현식을 사용할 수 있다.

```kotlin
try {
    // some code
} catch (e: SomeException) {
    // handler
} finally {
    // optional finally block
}
```

`catch`문은 0개 또는 그 이상 존재할 수 있으며 `finally`문은 생략가능하다.

그러나 `catch or finally` 중 적어도 하나는 필수적으로 존재해야한다.

## Try is an expression

`try`는 표현식이다. 이것은 return 값을 가질 수 있다는 것을 의미한다.

```kotlin
val a: Int? = try { input.toInt() } catch (e: NumberFormatException) { null }
```

`try`표현식의 반환값은 마지막에 명시된 값이거나 `catch`문에 명시된 표현식이 된다.

`finally`문의 내용은 반환값에 영향을 주지 않는다.

<br>

# Checked exceptions

코틀린에는 `Checked exception`이 존재하지 않는다.

많은 이유가 있지만 왜 그런지 간단하게 예시를 통해 알아보자.

아래는 `StringBuilder`에 의해 구현된 JDK 인터페이스 예시이다.

```java
Appendable append(CharSequence csq) throws IOException;
```

위 예제는 string에 무언가를 append 할 때 마다 IOException 을 catch 해야만 한다는 것을 알 수 있다.

왜냐하면 위 예제는 IO 연산자에 의해 수행될 것이기 때문이다.(`Writer` 또한 `Appeendable`의 구현체이다.)

그 결과 전역적으로 아래와 같은 코드 결과를 초래할 것이다.

```java
try {
    log.append(message)
} catch (IOException e) {
    // Must be safe
}
```

이것은 좋지 못하다. 이펙티브자바(3rd Edition)의 [`Item77: Don't ignore exceptions`](https://github.com/AlphaWang/alpha-effective-java-3e/blob/master/10_exceptions/item_77_dont_ignore_exceptions.md)를 참고하길 바란다.

[`Bruce Eckel`](https://en.wikipedia.org/wiki/Bruce_Eckel)는 `checked exception`에 대해 아래와 같이 언급한바 있다.

> **소규모 프로그램**에서는 필수적인 예외처리 스펙이 개발자의 생산성과 코드 품질을 향상에 도움이 되지만, <br>
> **대규모 소프트웨어 프로젝트**에서는 오히려 이들을 감소시킨다는 결론을 얻었다.

해당 문제에 대해 아래와 같은 추가적인 견해가 존재한다.

- [Java 의 `checked exception`은 실수다](https://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html) (by [Rod Waldhoff](http://heyrod.com/))
- [Checked Exception 의 문제점](https://www.artima.com/articles/the-trouble-with-checked-exceptions) (by [Anders Hejlsberg](https://ko.wikipedia.org/wiki/%EC%95%84%EB%84%A4%EB%A5%B4%EC%8A%A4_%ED%95%98%EC%9D%BC%EC%8A%A4%EB%B2%A0%EB%A5%B4))

만약 코틀린 코드가 `Java, Swift, Objective-C`에서 호출될 때 호출자에게 발생 가능한 예외를 알리기 원한다면 `@Throws` 어노테이션을 사용하면 된다.

<br>

# The Nothing type

코틀린에서의 `throw`는 표현식이다. 따라서, 이를 `?:`로 표기되는 `Elvis 표현식`의 일부로써 사용할 수 있다.

```kotlin
val s = person.name ?: throw IllegalArgumentException("Name required")
```

`thorw` 표현식은 `Nothing` 타입을 갖는다. 이 타입은 아무 값도 가지지 않는다. 또한, 이는 **절대로 접근될 일 없는 코드**임을 알리기 위해 표기 목적으로 사용된다.

실제 코드를 작성할 때 아무 값도 반환하지 않는 함수에 대해 `Nothing` 타입을 사용하면 된다.

```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

해당 함수가 호출될 때, 컴파일러는 더이상 해당 호출부분을 넘어서서는 실행되지 않는다는 것을 알게 된다.

```kotlin
val s = person.name ?: fail("Name required")
println(s)     // 's' 는 이 때 초기화가 되는것으로 알려져 있다.
```

`Nothing`은 또한 타입 인터페이스를 다룰 때 마주할 수 있는데, 해당 타입의 nullable 함을 나타내는 `Nothing?`은 오로지 `null` 값만을 가질 수 있다.
만약 타입 추론에 의해 선언된 값을 `null`로 초기화 하고 특정 타입으로 추론할만한 정보가 더 이상 존재하지 않는다면, 컴파일러는 자동적으로 해당 변수를 `Nothing?` 타입으로 추론하게 된다.

```kotlin
val x = null           // 'x' has type `Nothing?`
val l = listOf(null)   // 'l' has type `List<Nothing?>
```

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Control flow > Exceptions](https://kotlinlang.org/docs/exceptions.html)