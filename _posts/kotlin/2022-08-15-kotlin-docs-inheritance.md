---
title: "[Kotlin] 개념 정리 - Inheritance"
last_modified_at: 2022-08-16T21:00:00+09:00
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

> (참고) 의미 전달의 편의성을 위해 해당 장의 특정 단어에 대해 번역을 다음과 같이 했음을 참고하자. 
>  - `super, base` -> `부모`
>  - `derived` -> `자식`

코틀린 내의 모든 클래스는 `Any`클래스를 부모로 가지고 있다.

```kotlin
class Example // 암묵적으로 Any를 상속하고 있다.
```

`Any` 클래스는 아래와 같이 세개의 메서드를 가지고 있다.

- equals()
- hashCode()
- toString()

따라서, 위 메서드들은 코틀린 내 모든 클레스에서 정의되어있다고 보면 된다.

코틀린의 클래스는 기본적으로 `final`클래스 이고 이는 상속할 수 없다는 특징을 가지고 있다.

하지만 `open`이라는 키워드를 함께 표기하면 상속 가능한 클래스가 된다.

```kotlin
open class Base // 클래스 상속 가능하도록 공개
```

명시적으로 부모클래스를 선언하고 싶다면, 클래스의 헤더에서 콜론(:) 뒤에 타입을 배치시키면 된다. 

```kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

자식 클래스가 기본 생성자를 가지고 있을 경우, 부모 클래스는 매개변수에 따라 해당 기본 생성자에서 반드시 초기화 되어야 한다.

자식 클래스가 기본생성자를 가지고 있지 않을 경우, 각각의 보조 생성자에서 반드시 `super` 키워드를 이용해 부모 클래스를 초기화 하거나 이를 수행하는 다른 생성자에게 위임 해야한다.

아래와 같이 서로 다른 보조 생성자들은 부모 클래스 또한 다른 생성자를 호출할 수 있다.

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

<br>

# Overriding methods

코틀린은 오버라이딩 가능한 멤버와 오버라이딩 된 것에 대해 명시적인 수정자(modifier)를 표기해야 한다.

```kotlin
open class Shape {
    open fun draw() { /*...*/ }
    fun fill() { /*...*/ }
}

class Circle() : Shape() {
    override fun draw() { /*...*/ }
}
```

오버라이딩을 수행한 쪽에서는 `override`를 오버라이딩 대상에게는 `open`을 반드시 명시해야 한다.(명시 안하면 컴파일러가 잡음)

`open`을 명시하지 않으면 동일한 이름의 메서드를 선언할 수 없게 된다.

그리고, `open` 수정자는 final class의 맴버에는 추가해봤자 소용이 없으므로 유의하자.(클래스의 멤버를 override를 하기 위해서는 우선 클래스 부터 open 되어야 함.)

`override` 표기가 되어있는 멤버는 그 자체로 `open`를 내포하고 있다.(자식 클래스에서 override 가능)

만약 이에 대해 다시 override 되는 것을 막기 위해서는 `final`을 사용하면 된다.

```kotlin
open class Rectangle() : Shape() {
    final override fun draw() { /*...*/ }
}
```

<br>

# Overriding properties

오버라이딩 메커니즘이 메서드에서 적용된 것과 동일하게 속성에도 적용된다.

부모 클래스에 정의된 속성에 대해 자식 클래스에서 역시 `override`와 함께 재정의 할 수 있다.(단, 동일한 타입을 유지해야 함.)

```kotlin
open class Shape {
    open val vertexCount: Int = 0
}

class Rectangle : Shape() {
    override val vertexCount = 4
}
```

`val`를 `var` 속성으로 오버라이딩 할 수도 있다.(그 반대는 불가능)

- val -> var (o)
- var -> val (x)

이는 `val` 속성은 필수적으로 `get` 메서드를 가지고 있고, 오버라이딩 될 `var`에 대해 `set` 메서드를 자식 클래스에서 추가정의 할 수 있기 때문에 가능하다.

- val 에 대한 `get` 이미 존재
- `set`은 자식 클래스에서 추가 정의 가능
- 즉, 변경 불가했던 `val`를 변경 가능한 `var`로 변경 가능

기본 생성자의 속성 선언에 `override` 키워드를 사용할 수 있다는 점도 유의하자.

```kotlin
interface Shape {
    val vertexCount: Int
}

class Rectangle(override val vertexCount: Int = 4) : Shape // Always has 4 vertices

class Polygon : Shape {
    override var vertexCount: Int = 0  // Can be set to any number later
}
```

<br>

# Derived class initialization order

자식 클래스의 신규 인스턴스 생성시 부모 클래스의 초기화가 제일 먼저 수행된다.

```kotlin
open class Base(val name: String) {

    init { println("Initializing a base class") } // ... 3

    open val size: Int = 
        name.length.also { println("Initializing size in the base class: $it") } // ... 4
}

class Derived(
    name: String, // hello
    val lastName: String, // world
) : Base(name.replaceFirstChar { it.uppercase() }.also { println("Argument for the base class: $it") }) { // ... 2

    init { println("Initializing a derived class") } // ... 5

    override val size: Int =
        (super.size + lastName.length).also { println("Initializing size in the derived class: $it") } // ... 6
}

fun main() {
    println("Constructing the derived class(\"hello\", \"world\")") // ... 1
    Derived("hello", "world")
}

// Output:
// 1 Constructing the derived class("hello", "world")
// 2 Argument for the base class: Hello
// 3 Initializing a base class
// 4 Initializing size in the base class: 5
// 5 Initializing a derived class
// 6 Initializing size in the derived class: 10
```

위 예제를 통해 부모 클래스의 생성자가 실행될 당시에는 아직 자식 클래스의 초기화가 이루어지지 않았음을 알 수 있다.

따라서, 자식 클래스에 선언되었거나 재정의된 속성을 부모의 `생성자, 속성 초기화, init` 과 같은 곳에서 사용하게 되면 잘못된 동작과 런타임 오류를 초래할 수 있다.

- 원래 의도: `자식 클래스 속성 초기화` -> `해당 속성을 부모 클래스의 초기화에 사용`
- 실제 동작: `초기화 되지 않은 자식 클래스의 속성을 부모 클래스가 사용` -> `이후 자식 클래스 초기화`

위와 같은 상황 때문에 의도치 않은 행동 또는 runtime 에러가 발생할 수 있으므로, 부모 클래스를 설계할 때는 `생성자, 속성 초기화, init`에서는 `open` 맴버를 사용하지 않는 것이 좋다.

<br>

# Calling the superclass implementation

자식 클래스의 코드는 `super` 키워드를 통해 부모 클래스의 함수와 속성 접근 구현체(get or set)를 호출할 수 있다.

```kotlin
open class Rectangle {
    open fun draw() { println("Drawing a rectangle") }
    val borderColor: String get() = "black"
}

class FilledRectangle : Rectangle() {
    override fun draw() {
        super.draw()
        println("Filling the rectangle")
    }

    val fillColor: String get() = super.borderColor
}
```

`내부 클래스(inner class)`에서 `외부 클래스(outer class)`의 `부모 클래스(super class)`로의 접근은 `외부 클래스(outer class)`의 이름으로 제한된(qualified) `super` 키워드를 사용하므로써 가능하다.

```kotlin
open class Rectangle {
    open fun draw() { println("Drawing a rectangle") } // ... 1
    val borderColor: String get() = "black"
}

class FilledRectangle: Rectangle() {
    override fun draw() {
        val filler = Filler()
        filler.drawAndFill()
    }

    inner class Filler {
        fun fill() { println("Filling") } // ... 2
        fun drawAndFill() {
            super@FilledRectangle.draw() // Calls Rectangle's implementation of draw()
            fill()
            println("Drawn a filled rectangle with color ${super@FilledRectangle.borderColor}") // Uses Rectangle's implementation of borderColor's get() ... 3
        }
    }
}

fun main() {
    val fr = FilledRectangle()
        fr.draw()
}

// Output:
// 1 Drawing a rectangle
// 2 Filling
// 3 Drawn a filled rectangle with color black
```

<br>

# Overriding rules

코틀린에서의 구현(implementation) 상속은 아래와 같은 규칙을 따른다.

만약 다중 부모 클래스를 상속 받고 있는데 부모간에 이름이 겹치는 맴버를 가지고 있다면, 해당 맴버에 대해서는 반드시 본인 고유의 구현을 갖도록 오버라이딩 해야 한다.(대부분 상속받은 부모 클래스 중 하나를 그대로 가져다 사용한다.)

상속된 구현을 선택적으로 가져오기 위해서는 `super<Base>`와 같이 사용하면 된다.

```kotlin
open class Rectangle {
    open fun draw() { /* ... */ }
}

interface Polygon {
    fun draw() { /* ... */ } // interface members are 'open' by default
}

class Square() : Rectangle(), Polygon {
    // The compiler requires draw() to be overridden:
    override fun draw() {
        super<Rectangle>.draw() // call to Rectangle.draw()
        super<Polygon>.draw() // call to Polygon.draw()
    }
}
```

위 예제를 살펴보면 `Rectangle, Polygon` 둘 다 각자의 `draw()` 구현을 가지고 있다.

따라서, 구현 상속 규칙에 의해 반드시 두 클래스를 상속하는 `Square`는 `draw()`를 오버라이딩 하고 모호함을 제거하기 위해 별도의 구현을 제공해야 한다.

<br>

# 참고

- [코틀린 공식 문서 - Concepts > Classes and objects > Inheritance](https://kotlinlang.org/docs/inheritance.html)