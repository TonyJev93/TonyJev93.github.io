---
title: "[Kotlin] 개념 정리 - Inline functions"
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

고차 함수를 사용하면 함수 객체의 **메모리 할당 및 가상 호출**로 인해 **런타임 오버헤드가 발생**할 수 있어 패널티가 부과될 수 있다.

그러나 많은 경우에 이러한 종류의 오버헤드는 람다 표현식을 **인라인을 이용하여 제거**할 수 있다. 

아래에 표시된 함수는 이러한 상황의 좋은 예시이다. `lock()` 함수는 호출부에서 쉽게 인라인될 수 있다. 다음 경우를 고려해보자.

```kotlin
lock(l) { foo() } // lambda 표현식, 런타임 시 오버헤드 발생
```

고차 함수(lambda)를 생성하고 호출하는 대신 컴파일러에서 아래와 같은 코드를 직접 생성할 수도 있다.

```kotlin
l.lock()
try {
    foo()
} finally {
    l.unlock()
}
```

컴파일러가 이런 코드를 만들 수 있게 하려면 `lock()` 함수에 `inline` 수정자를 표시하면 된다.

```kotlin
inline fun <T> lock(lock: Lock, body: () -> T): T { ... }
```

`inline` 수정자는 **함수 자체**와 **함수에 전달된 람다식** 둘 다 호출부에서 인라인 되도록 영향을 끼친다.

인라인으로 인해 생성된 코드가 커질 수 있으나 큰 함수를 인라인하지 않는 합리적인 방법으로 사용하면 성능이 향상된다.

<br>

# noinline

인라인 함수에 전달된 람다 중 일부는 인라인되지 않도록 하고 싶다면 대상 매개변수 앞에 `noinline` 수정자를 표시하면 된다.

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { ... }
```

인라인으로 사용 가능한 람다식은 **인라인 함수 내에서 사용되거나 인라인 함수의 인수로 밖에 사용**할 수 없는 반면,

`noinline` 람다의 경우 필드로 저장한다던지 어디로든 전달이 가능하여 원하는 모든 방식으로 사용이 가능하다.

# Non-local returns

코틀린에서는 명명된 함수(named function)나 익명의 함수를 종료하기 위해 `한정된 반환(qualified return)`이 아닌 일반 `return`만 사용할 수 있다. 

근데 반면 람다를 종료하기 위해서는 `label`을 사용해야만 한다. 그러나 인라인 함수의 인자로 사용된 람다식은 `label`을 사용하지 않고도 `return`만을 사용하여 반환하는 것이 허용된다.

```kotlin
// 1. 람다에 return 사용한 경우
fun ordinaryFunction(block: () -> Unit) {
    println("hi!")
}

fun foo1() {
    ordinaryFunction {
        return // ERROR: cannot make `foo1` return here -> 람다식에 return만 사용 불가능.
    }
}

// 2. inline 함수 내 람다에 return 사용한 경우
inline fun inlined(block: () -> Unit) {
    println("hi!")
}

fun foo2() {
    inlined {
        return // OK: the lambda is inlined -> inline 된 함수에 람다 식이 사용된 경우. return 사용 가능
    }
}
```

위와 같이 `return`이 람다에 존재하지만 enclosing function를 종료하는 것을 `non-local return`이라고 한다. 

이러한 종류의 구성은 일반적으로 인라인 함수가 자주 사용되는 **반복문**(loop)에서 볼 수 있다.

```kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach { // forEach => inline 함수
        if (it == 0) return true // returns from hasZeros
    }
    return false
}
```

일부 인라인 함수는 **다른 실행 컨텍스트**에서 **매개변수로 전달된 람다를 호출**할 수 있다. 

이러한 경우 람다에서 `non-local` 제어 흐름이 허용 되지 않는다. 

이런 경우 인라인 함수의 람다 매개변수가 `non-local return`을 사용할 수 없음을 나타내기 위해 `crossinline` 수정자를 표시해주면 해결이 된다.

```kotlin
inline fun View.click(block: (View) -> Unit) {
    setOnClickListener { // 다른 실행 컨텍스트
            view -> block(view) // 컴파일 error: 매개변수로 전달된 람다(block)를 다른 실행 컨텍스트에서 호출하는 것이 허용되지 않음. 
    }
}

inline fun View.click(crossinline block: (View) -> Unit) {
    setOnClickListener { 
            view -> block(view) // ok. crossinline 를 통해 허용해줌. 
    }
}
```

<br>

# 구체화된 타입 매개변수(Reified type parameters)

`reified`는 제네릭 타입 T를 runtime 시에 어떤 타입인지, 또는 type casting을 하는 경우 필요하다.

T는 컴파일 타임에는 존재하지만 runtime 시에는 Type erasure 때문에 접근할 수 없어 T가 어떤 타입인지 알 수 없기에 컴파일 오류가 발생한다.

```kotlin
private inline fun <T> whoAreYou(value: T, func: (String) -> Unit) {
    // T::class 에서 에러 발생. T의 타입을 알 수 없음
    func(when(T::class.java) { // Error: Cannot use 'T' as reified type parameter. Use a class instead
        String::class.java -> { "${value}는 글자 입니다." }
        Integer::class.java -> { "${value}는 숫자 입니다." }
        else -> { "${value}는 모르겠습니다." }
    })
}
```

위 처럼 `T::class.java`로 T의 타입을 알려고 할 때 에러가 발생한다.

따라서 만약 제네릭 타입에 접근하고 싶다면 명시적으로 타입을 파라미터로 전달해주여야 한다.

그런대 `reified`와 같이 inline 함수를 만들어 주면 추가적으로 `Class<T>`를 파라미터로 넘겨줄 필요 없이 런타임에 타입 `T`에 접근할 수 있게 된다.

`reified`와 함께 `inline` 함수가 호출되면 컴파일러는 인자로 사용된 타입을 미리 알고 바이트코드로 직접 클래스에 대응되도록 바꾸어 준다. (`myVar is T` in compile -> `myVar is String` in runtime) 

그래서 아래와 같이 `fun <reified T>`으로 선언해 놓으면 `value: T`의 타입을 확인하여 처리가 가능해진다.

```kotlin
private inline fun <reified T> whoAreYou(value: T, func: (String) -> Unit) {
    func(when(T::class.java) {
        String::class.java -> { "${value}는 글자 입니다." }
        Integer::class.java -> { "${value}는 숫자 입니다." }
        else -> { "${value}는 모르겠습니다." }
    })
}
```

주의해야 할 것은 `reified`는 `inline`과 반드시 함께 사용되어야 한다.

<br>

# Inline properties

`inline` 수정자는 backing 필드가 없는 프로퍼티 접근자에 사용할 수 있고 개별 프로퍼티 접근자에 사용할 수 있다.

```kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

전체 프로퍼티에도 적용할 수 있다.(두 접근자를 모두 `inline` 적용 됨)

```kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

<br>

# 공개 API 인라인 함수에 대한 제한사항(Restrictions for public API inline functions)

인라인 함수는 `public` 또는 `protected` 이지만 `private` 이나 `internal`에 소속되어 있지 않을 경우 **모듈의 공개 API**로 간주된다. 

모듈의 공개 API는 다른 모듈에서 호출할 수 있으며 해당 모듈 내에서도 호출된 곳에서 인라인된다.

이는 **인라인 함수를 선언한 모듈에서는 변화**가 발생 하였지만 이를 **호출한 모듈에서는 재컴파일을 하지 않게** 되면서 **이진 비호환성의 문제**를 발생시킬 수 있는 위험성을 갖게 만든다.

(요약: 인라인 함수에 대해 선언부는 변경했지만 사용부에서 재컴파일 하지 않아 발생하는 비호환 문제)

모듈의 비공개 API의 변경으로 인해 발생하는 이러한 비호환성 문제를 제거하기 위해

공개 API 인라인 함수는 `private` 및 `internal` 선언과 같은 비공개 API 선언을 그들의 본문에서 사용할 수 없도록 한다.

`internal` 선언은 `@PublishedApi`로 주석을 달 수 있으며, 이를 통해 공개 API 인라인 함수에서 사용할 수 있도록 해준다. 

`internal` 인라인 함수가 `@PublishedApi`로 표시되면 해당 본문도 공개 된다.

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Functions > Inline functions](https://kotlinlang.org/docs/inline-functions.html)