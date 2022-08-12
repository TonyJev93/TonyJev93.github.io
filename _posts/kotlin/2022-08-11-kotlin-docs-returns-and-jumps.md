---
title: "[Kotlin] 개념 정리 - Returns and jumps"
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

코틀린에는 세 개의 점프 표현식이 존재한다.

- return: 가장 가까운 함수 또는 익명 함수에서의 반환
- break: 가장 가까운 루프 종료
- continue: 가장 가까운 루프 내에서 다음단계 진행

이러한 모든 표현식은 더 큰 표현식의 일부로서 사용될 수 있다.

```kotlin
val s = person.name ?: return
```

(해당 표현식의 타입은 `Nothing type`이다.)

Nothing type
: 리턴이라는 행위 자체를 하지 않음을 뜻함. 리턴 될 일이 없을 경우 or 예외를 던질 경우 사용.

<br>

# Break and continue labels

코틀린 내에 모든 표현식은 `label`로 표시할 수 있다. 라벨은 `@` 표시와 함께 식별자 형태를 갖는다.(`@abc` or `fooBar@`)

표현식에 레이블을 지정하려면, 앞에 표현식을 추가하기만 하면 된다.

```kotlin
loop@ for (i in 1..100) {
    // ...
}
```

이제, 라벨을 통해 `break or continue`를 사용할 수 있다.

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

라벨이 붙은 `break` 이후에는 즉시 표기된 라벨로 점프할 수 있게 된다.

`continue`의 경우 루프의 다음 반복문을 계속 진행하게 된다.

<br>

# Return to labels

코틀린에서 함수들은 함수 리터럴, 지역함수, 객체 표현식를 사용하면서 중첩이 될 수 있다.

`return`은 바깥 함수로 반환되도록 한다.

가장 중요한 사용 유형은 람다 표현식으로부터의 반환이다.

아래 예시에서 return 표현식은 `listOf(...).forEach`가 아닌 가장 가까운 함수 `foo`에서 반환된다.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // non-local return directly to the caller of foo()
        print(it)
    }
    println("this point is unreachable")
}

// output: 12
```

이러한 `non-local` 반환은 오직 인라인 함수를 통과하는 람다표현식에서만 유효하다는 것에 유의하자.

람다 표현식으로부터 반환되기 위해서는 라벨링을 하고 `return@xxx`을 사용하라.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // local return to the caller of the lambda - the forEach loop
        print(it)
    }
    print(" done with explicit label")
}

// output: 1245 done with explicit label
```

`implicit labels`를 사용하는 것이 람다가 통과하는 함수명과 라벨의 이름이 일치하도록 하면 되기 때문에 종종 더 편리하게 사용할 수 있다.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // local return to the caller of the lambda - the forEach loop
        print(it)
    }
    print(" done with implicit label")
}

// output: 1245 done with implicit label
```

또 다른 대안으로, 람다 표현식을 익명 함수로 대체할 수 있다.

익명 함수에서의 `return`은 익명 함수 그 자체로부터 반환될 것이다.

```kotlin
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // local return to the caller of the anonymous function - the forEach loop
        print(value)
    })
    print(" done with anonymous function")
}

// output: 1245 done with anonymous function
```

이전에 예시로 들었던 것들은 모두 local return을 사용하는 것을 볼 수 있다. 이는 마치 `continue`를 사용하는 것과 유사하다.

반면 `break`와 같은 기능을 제공하지는 않는데, 아래와 같이 다른 람다식으로 감싸는 방식을 통해 우회하여 구현할 수는 있다.

```kotlin
fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // non-local return from the lambda passed to run
            print(it)
        }
    }
    print(" done with nested loop")
}

// output: 12 done with nested loop
```

값을 반환할 때 파서는 정규화된 반환에 우선 순위를 부여한다.

```kotlin
return@a 1
```

이것은 `(@a 1)`이라고 라벨링된 곳으로 리턴하는 것이 아닌 `1`을 `@a label`에 반환하라는 것을 의미한다.

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Control flow > Returns and jumps](https://kotlinlang.org/docs/returns.html)