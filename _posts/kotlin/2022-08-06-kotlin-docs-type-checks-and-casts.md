---
title: "[Kotlin] 개념 정리 - Type checks and casts"
last_modified_at: 2022-08-07T21:00:00+09:00
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

# is and !is operatores

객체가 주어진 타입과 일치 하는지 식별하는 런타임 검사를 위해 `is` 또는 그것의 부정형인 `!is` 연산자를 사용하자.

```kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // same as !(obj is String)
    print("Not a String")
} else {
    print(obj.length)
}
```

<br>

# Smart casts

코틀린에서 대부분의 경우 컴파일러가 불변 값들에 대해 `is-check` 및 `명시적 형변환`을 추적하기 때문에, 그리고 필요할 때 자동으로 (안전한) 형변환을 삽입하기 때문에 명시적인 형변환 연산자를 사용할 필요는 없다.

```kotlin
fun demo(x: Any) {
    if (x is String) {
        print(x.length) // x is automatically cast to String
    }
}
```

컴파일러는 부정 검사(if not)가 반환(return)을 유도하는 경우 형변환이 안전하다는 것을 알 만큼 충분히 똑똑합니다.

컴파일러는 (`if !is xxx` 같은)부정 검사로 인해 return을 수행할 경우, 그 이후 소스로에 대해 형변환으로부터 안전하다는 것을 인지할 수 있게 된다.

```kotlin
if (x !is String) return

print(x.length) // x is automatically cast to String
```

`&& or ||` 연산 기준으로 왼쪽 표현식에서 타입 체크가 정상적으로 이루어졌다면, 오른쪽 표현식의 변수 또한 자동 형변환이 이루어진다. 

```kotlin
// x is automatically cast to String on the right-hand side of `||`
if (x !is String || x.length == 0) return

// x is automatically cast to String on the right-hand side of `&&`
if (x is String && x.length > 0) {
    print(x.length) // x is automatically cast to String
}
```

`when` 그리고 `while` 표현식 또한 자동 형변환이 가능하다.

```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

변수가 검사 또는 사용 도중에 변하지 않는다는 보장이 있을 때만 컴파일러는 자동 형변환을 사용할 수 있다는 점을 유의해야 한다.

더 구체적으로 아래와 같은 조건에서 자동 형변환이 이루어질 수 있다.

- `val` 지역 변수 - 지역 위임 속성을 제외하고 항상.
- `val` 프로퍼티 - 프로퍼티가 private 또는 internal 이거나 property가 선언된 동일한 모듈에서 검사가 수행되는 경우. 공개된 프로퍼티나 사용자 지정 getter가 있는 프로퍼티에서는 동작 하지 않습니다.
- `var` 지역 변수 - 위임 속성이 아니며 이를 변경하는 람다가 존재하지 않고 타입 체크와 참조 사이에 변경이 일어나지 않는 경우 사용 가능 합니다.
- `var` 프로퍼티 - 다른 코드에서 언제든지 수정될 수 있기 때문에 절대 동작 되지 않음.

<br>

# "Unsafe" cast operator

일반적으로, 형변환 연산자는 형변환이 불가능한 경우 예외를 발생시키는데 이것을 `unsafe` 형변환이라고 부른다.

`unsafe` 형변환은 중위 연산자 `as`를 통해 사용할 수 있다.

```kotlin
val x: String = y as String
```

`null`은 `String(not nullable)`으로 형변환 될 수 없다는 것을 유의 하자. 만약 `y`가 null 이라면 위 코드는 예외를 던지게 될 것이다.

null 대입이 가능하기 위해서는 형변환 우측에 nullable 타입(`?`)을 추가해야 한다.

```kotlin
val x: String? = y as String?
```

<br>

# "Safe" (nullable) cast operator

예외를 피하기 위해 `safe` 형변환 연산자 `as?`를 사용한다. 이것을 사용하면 예외 발생 시 `null`을 반환한다.

```kotlin
val x: String? = y as? String
```

`as?` 우측에 `String`으로 nullable 하지 않은 형변환을 했음에도 불구하고 결과값의 형태는 nullable 하다는 점을 주목하자. 

<br>

# Type erasure and generic type checks

코틀린은 컴파일 타임에는 제네릭을 포함한 연산자에 대해 안정성을 보장한다. 반면, 런타임에 제네릭 타입의 인스턴스는 실제 타입 인자를 가지고 있지 않는다.

예를들면, `List<Foo>`는 런타임에서 `List<*>` 처럼 타입이 지워진다. 일반적으로, 런타임에 인스턴스가 특정 타입에 해당하는 제네릭 타입을 가지고 있는지 여부를 검증할 수 없다.

그렇기 때문에 컴파일러는 타입 지우개로 인해 런타임에 수행할 수 없는 `ints is List<Int> or list is T`와 같은`is-checks` 사용을 금지한다.

그러나, `*`이 적용된 인스턴스에 대해서는 확인 할 수 있다. 

```kotlin
if (something is List<*>) {
    something.forEach { println(it) } // The items are typed as `Any?`
}
```

마찬가지로, 컴파일 타임에 이미 확인된 타입 인자를 가지고 있는 인스턴스의 경우, 제네릭을 포함하지 않은 타입에 대한 `is-check` 또는 형변환이 가능하다.

이 경우 꺽쇠 괄호`<>`가 제거 됨을 유의하자.

```kotlin
fun handleStrings(list: List<String>) {
    if (list is ArrayList) {
        // `list` is smart-cast to `ArrayList<String>`
    }
}
```

The same syntax but with the type arguments omitted can be used for casts that do not take type arguments into account: list as ArrayList.

동일한 구문이지만 타입 인자가 생략된 경우 타입 인자를 고려하지 않는 형변환에 사용할 수 있다. 

```kotlin
list as ArrayList
```

`reified` 타입 인자를 지닌 인라인 함수는 해당 인라인 함수가 호출되는 부분에서 실제 타입 인자를 가질 수 있다.

이는 타입 인자에 대해 `arg is T` 검증을 가능하게 한다. 그치만 `arg` 자체가 제네릭 타입의 인스턴스라면, 그것의 타입 인자는 여전히 지워지게 된다.

```kotlin
inline fun <reified A, reified B> Pair<*, *>.asPairOf(): Pair<A, B>? {
    if (first !is A || second !is B) return null
    return first as A to second as B
}

val somePair: Pair<Any?, Any?> = "items" to listOf(1, 2, 3)


val stringToSomething = somePair.asPairOf<String, Any>() // (items, [1, 2, 3])
val stringToInt = somePair.asPairOf<String, Int>() // null
val stringToList = somePair.asPairOf<String, List<*>>() // (items, [1, 2, 3])
val stringToStringList = somePair.asPairOf<String, List<String>>() // (items, [1, 2, 3]), Compiles but breaks type safety!
// Expand the sample for more details
```

<br>

# Unchecked casts

위에서 확인한 바와 같이, 타입 지우개는 제네릭 타입 인스턴스의 실제 타입 인자를 런타임에 체크하는 것을 불가능하게 한다.

또한, 제네릭 타입은 컴파일러가 타입 안전성을 보장할 수 없게 만든다.

대신 우리는 타입 안전성을 암시하는 높은 수준의 프로그램 로직을 가지고 있다.

```kotlin
fun readDictionary(file: File): Map<String, *> = file.inputStream().use {
   TODO("Read a mapping of strings to arbitrary elements.")
}

// We saved a map with `Int`s into this file
val intsFile = File("ints.dictionary")

// Warning: Unchecked cast: `Map<String, *>` to `Map<String, Int>`
val intsDictionary: Map<String, Int> = readDictionary(intsFile) as Map<String, Int>
```

마지막 줄에 형변환 관련하여 경고가 나타난다. 컴파일러가 런타임에 형변환에 대해 완전히 검증할 수 없고 Map 안에 값이 `Int`라는 것도 보장 할 수 없다.

서로 다른 타입에 대한 타입 세이프한 구현으로 DictionaryReader<T> 및 DictionaryWriter<T> 인터페이스를 사용하거나 unchecked cast를 호출 지점에서 구현 세부 사항으로 이동하기 위해 합리적인 추상화를 도입할 수 있다. 제네릭의 가변성을 적절히 사용하는 것도 도움이 될 수 있다.

제네릭 함수에서 `reified` 타입 인자를 사용하면 검증된 `arg as T`과 같은 형변환을 할 수 있다.

그렇지 않으면 `arg`는 이미 지워진 자체적인 타입 인자를 갖게 될 것이다.

`unchecked` 형변환 경고는 해당 문장 또는 선언에 `@Suppress("UNCCHECKED_CAST")` 주석을 추가 하여 감출 수 있다.

```kotlin
inline fun <reified T> List<*>.asListOfType(): List<T>? =
    if (all { it is T })
        @Suppress("UNCHECKED_CAST")
        this as List<T> else
        null
```

<br>

# 고찰

`Unchecked casts` 부분은 솔직히 무슨 말인지 하나도 모르겠다. 나중에 다시 한번 공부 해보는 것이 좋을 것 같다...
{: .notice--info}

# 참고

- [코틀린 공식 문서 - Concepts > Types > Type checks and casts](https://kotlinlang.org/docs/typecasts.html)