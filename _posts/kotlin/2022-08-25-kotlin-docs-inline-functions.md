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

고차 함수를 사용하면 특정 런타임 패널티가 부과된다.

각 함수는 객체이고 클로저를 캡처한다.

클로저는 함수 본문에서 접근할 수 있는 변수의 범위이다.

메모리 할당(함수 객체 및 클래스 모두에 대해) 및 가상 호출은 런타임 오버헤드를 유발한다.

그러나 많은 경우에 이러한 종류의 오버헤드는 람다 식을 인라인하여 제거할 수 있다. 

아래에 표시된 함수는 이러한 상황의 좋은 예시이다. `lock()` 함수는 호출부에서 쉽게 인라인될 수 있다. 다음 경우를 고려해보자.

```kotlin
lock(l) { foo() }
```

매개 변수에 대한 함수 객체를 만들고 호출을 생성하는 대신 컴파일러에서 다음 코드를 생성할 수 있다.

```kotlin
l.lock()
try {
    foo()
} finally {
    l.unlock()
}
```

컴파일러가 이를 수행하도록 하려면 `inline` 수정자로 `lock()` 함수를 표시해야 한다.

```kotlin
inline fun <T> lock(lock: Lock, body: () -> T): T { ... }
```

`inline` 수정자는 함수 자체와 함수에 전달된 람다 모두에 영향을 미친다. 이 모든 것이 호출부에 인라인될 것이다.

인라인으로 인해 생성된 코드가 커질 수 있다. 

그러나 합리적인 방법으로 수행하면(큰 함수를 인라인하지 않고) 특히 루프 내부의 "메가모픽" 호출부에서 성능이 향상된다.

<br>

# noinline

인라인 함수에 전달된 모든 람다가 인라인되지 않도록 하려면 함수 매개변수 중 일부를 `noinline` 수정자로 표시하면 된다.

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) { ... }
```

인라인 가능 람다는 인라인 함수 내에서만 호출하거나 인라인 가능한 인수로 전달할 수 있다.

그러나 `noinline` 람다는 필드에 저장되거나 전달되는 것을 포함하여 원하는 방식으로 조작할 수 있다.

> 인라인 함수에 인라인 가능 함수 매개변수가 없고 구체화된 유형 매개변수가 없는 경우 컴파일러는 경고를 발생시킨다.<br> 
> 이러한 함수를 인라인하는 것은 이점이 거의 없기 때문이다.<br>
> 그럼에도 정말 인라이닝이 필요하다면 @Suppress("NOTHING_TO_INLINE") 주석을 사용하여 경고메시지를 억제할 수는 있다. 

# Non-local returns

코틀린에서는 명명된 함수(named function)나 익명의 함수를 종료하기 위해 `qualified return`이 아닌 일반 `return`만 사용할 수 있다. 

람다를 종료하기 위해서는 `label`을 사용해야 한다.

람다는 동봉된(enclosing) 함수를 `return`으로 만들 수 없기 때문에 람다 내에서 빈 `return` 사용은 금지됩니다.

```kotlin
fun ordinaryFunction(block: () -> Unit) {
    println("hi!")
}

fun foo() {
    ordinaryFunction {
        return // ERROR: cannot make `foo` return here
    }
}

fun main() {
    foo()
}

// Error Message: 'return' is not allowed here
```

그러나 람다를 인자로 갖는 함수가 인라인되면 `return`도 인라인될 수 있다. 따라서 다음이 허용된다.

```kotlin
inline fun inlined(block: () -> Unit) {
    println("hi!")
}
fun foo() {
    inlined {
        return // OK: the lambda is inlined
    }
}
fun main() {
    foo()
}

// print: hi!
```

람다에 있지만 enclosing function를 종료하는 `return`을 `non-local return`이라고 한다. 

이러한 종류의 구성은 일반적으로 인라인 함수가 자주 사용되는 반복문(loop)에서 발생한다.

```kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // returns from hasZeros
    }
    return false
}
```

일부 인라인 함수는 함수 본문에서 직접 전달되는 것이 아니라 로컬 객체 또는 중첩 함수와 같은 다른 실행 컨텍스트에서 매개변수로 전달된 람다를 호출할 수 있다. 

이러한 경우 람다에서 `non-local` 제어 흐름도 허용되지 않는다. 

인라인 함수의 람다 매개변수가 `non-local return`을 사용할 수 없음을 나타내려면 람다 매개변수를 `crossinline` 수정자로 표시해야 한다.

```kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ...
}
```

[//]: # (&#40;????? 뭔말이지...&#41;)

<br>

# 구체화된 타입 매개변수(Reified type parameters)

때로는 매개변수로 전달된 타입에 접근해야 할 때가 있다.

```kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```

여기에서 `tree`를 타고 올라가 리플렉션을 사용하여 노드에 특정 타입이 있는지 확인한다.

다 좋은데 호출부가 별로 안 예쁘다.

```kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```

더 나은 솔루션은 단순히 이 함수에 타입을 전달하는 것이다. 다음과 같이 호출할 수 있다.

```kotlin
treeNode.findParentOfType<MyTreeNode>()
```

이를 위해 인라인 함수는 구체화된 타입 파라미터(reified type parameters)를 지원하므로 다음과 같이 작성할 수 있다.

```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

위의 코드는 거의 일반 클래스인 것처럼 함수 내부에서 액세스할 수 있도록 타입 매개변수를 `reified` 수정자로 한정하였다. 

함수가 인라인되기 때문에 리플렉션이 필요하지 않으며 이제 `!is` 및 `as`와 같은 일반 연산자를 사용할 수 있다. 

또한, 원하던 대로 아래와 같이 단순히 클래스만 넘겨주는 것으로 호출이 가능하게 되었다. 

```kotlin
myTree.findParentOfType<MyTreeNodeType>()
```

많은 경우 리플렉션이 필요하지 않을 수 있지만 구체화된 타입 매개변수(reified type parameter)와 함께 사용할 수 있다.

```kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}

```

인라인으로 표시되지 않은 일반 함수는 구체화된(reified) 매개변수를 가질 수 없다. 

런타임 표현이 없는 타입(예: non-reified 타입 매개변수 또는 `Nothing`과 같은 가상의 타입)은 구체화된(reified) 타입 매개변수에 대한 인수로 사용할 수 없다.

<br>

# Inline properties

`inline` 수정자는 backing 필드가 없는 프로퍼티 접근자에 사용할 수 있고

개별 프로퍼티 접근자에 사용할 수 있다.

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

호출부에서 인라인 접근자는 일반 인라인 함수 처럼 인라인된다.

<br>

# 공개 API 인라인 함수에 대한 제한사항(Restrictions for public API inline functions)

인라인 함수가 `public` 또는 `protected` 이지만 `private` 이나 `internal`에 포함되어 있지 않는 경우 모듈의 공개 API로 간주된다. 

다른 모듈에서 호출할 수 있으며 이러한 호출부에서도 인라인된다.

이는 특정 위험성을 내포하는데 인라인 함수를 **선언한 모듈에서는 변화**가 발생 하였지만 이를 **호출한 모듈에서는 재컴파일을 하지 않게** 되어 **이진 비호환성의 위험**을 가질수 있기 때문이다.

모듈의 비공개 API 변경으로 인해 발생하는 이러한 비호환성 문제를 제거하기 위해

공개 API 인라인 함수는 `private` 및 `internal` 선언과 같은 비공개 API 선언을 그들의 본문에서 사용할 수 없다.

`internal` 선언은 `@PublishedApi`로 주석을 달 수 있으며, 이를 통해 공개 API 인라인 함수에서 사용할 수 있다. `internal` 인라인 함수가 `@PublishedApi`로 표시되면 해당 본문도 공개된 것처럼 확인된다.

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Functions > Inline functions](https://kotlinlang.org/docs/inline-functions.html)