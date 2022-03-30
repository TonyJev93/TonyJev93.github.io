---
title: "[Design Pattern] 구조 패턴 - 브릿지(Bridge)"
last_modified_at: 2022-03-31T02:00:00+09:00
categories:
    - Design Pattern
tags:
    - Design Pattern
    - 구조 패턴
    - 브릿지 패턴
toc: true
toc_sticky: true
toc_label: "목차"
---

디자인 패턴 : 구조 패턴 중 하나인 브릿지 패턴에 대해 알아보자.
{: .notice--info}

# 브릿지 패턴이란?

브리지 패턴은 구현부에서 추상층을 분리하여 각자 독립적으로 변형 및 확장이 가능하도록 만드는 패턴이다.

기능을 담당하는 클래스와 구현을 담당하는 클래스를 나누어 구현한다.

<br>

# 구성

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FckgUW9%2FbtqyYpsdGwp%2FegBh42Yg8HLkNBDeAz2MVk%2Fimg.png){: .align-center}
_- [그림 출처](https://ko.wikipedia.org/wiki/%EB%B8%8C%EB%A6%AC%EC%A7%80_%ED%8C%A8%ED%84%B4)_ -
{: .text-center}

- Abstraction : 기능 계층의 최상위 클래스, 추상 인터페이스
- RefinedAbstraction : 기능 계층에서 새로운 부분을 확장한 클래스
- Implementor : Abstraction 의 기능을 구현하기 위한 인터페이스
- ConcreteImplementor : 실제 기능을 구현하는 클래스

<br>

# 예제

## 다양한 방법으로 도형 그리기(_[소스 출처](https://ko.wikipedia.org/wiki/%EB%B8%8C%EB%A6%AC%EC%A7%80_%ED%8C%A8%ED%84%B4)_)


### Shape(Abstraction)

```java
/** "Abstraction" */
interface Shape {
    public void draw();   // low-level

    public void resizeByPercentage(double pct);     // high-level
}
```

- 기능 계층의 최상위 클래스
- 도형이라는 추상 클래스를 선언하고, 사용 가능한 기능에 대해 인터페이스 처리

도형의 공통적인 기능으로서 그림을 그리는 기능(draw())과 사이즈 조절(resizeByPercentage(double pct))이 선언되어 있다.<br>
앞으로 해당 인터페이스를 구현하는 모든 도형들은 해당 메서드를 포함하여 각자의 방식대로 구현되어야 한다.

### CircleShape(Refined Abstraction)

```java
/** "Refined Abstraction" */
class CircleShape implements Shape {
    private double x, y, radius;
    private DrawingAPI drawingAPI;

    public CircleShape(double x, double y, double radius, DrawingAPI drawingAPI) {
        this.x = x;
        this.y = y;
        this.radius = radius;
        this.drawingAPI = drawingAPI;
    }

    // low-level i.e. Implementation specific
    public void draw() {
        drawingAPI.drawCircle(x, y, radius);
    }

    // high-level i.e. Abstraction specific
    public void resizeByPercentage(double pct) {
        radius *= pct;
    }
}
```

- `Shape Interface`의 구현체
- Shape(Abstraction) 인터페이스에 추가적인 기능이 부여된 클래스
- 기능을 수행할 수 있는 실질적인 구현체

Shape 인터페이스를 구현한 원형 모양 도형에 대한 구현체이다.


원을 그리는 방법(`draw()`)에 대해 `drawingAPI.drawCircle(...)`로 구현 함으로써 앞으로 도형을 그리는 방법(`draw()`)은 아래에 나올 `DrawingAPI(Implementor)`에 달려있는 샘이다.

DrawingAPI 는 추상화 클래스이기 때문에, CircleShape 생성 당시 생성자의 파라미터로 넘어오는 DrawingAPI 에 의해 **다양한 그리기가 가능(다양한 기능 구현)해** 진다. (**Bridge Pattern 의 핵심 목적**)

### DrawingAPI(Implementor)

```java
/** "Implementor" */
interface DrawingAPI {
    public void drawCircle(double x, double y, double radius);
}
```

- Shape(Abstraction)의 기능을 구현하기 위한 인터페이스
- Shape(Abstraction)의 `draw()` 기능을 구현하기 위한 목적으로 생겨난 새로운 추상화 계층(Helper 역할 수행) 

도형의 그리기(`draw())`) 기능을 추상화하기 위해 필요한 추상화 계층이다.

도형에서는 DrawingAPI 를 통해서 그림 그리기의 기능을 다양하게 유연하게 확장할 수 있게 된다.(**Bridge Pattern 의 핵심 역할**) 

### DrawingAPI{n}(Concrete Implementor)

```java
/** "ConcreteImplementor" 1/2 */
class DrawingAPI1 implements DrawingAPI {
    public void drawCircle(double x, double y, double radius) {
        System.out.printf("API1.circle at %f:%f radius %f\n", x, y, radius);
    }
}

/** "ConcreteImplementor" 2/2 */
class DrawingAPI2 implements DrawingAPI {
    public void drawCircle(double x, double y, double radius) {
        System.out.printf("API2.circle at %f:%f radius %f\n", x, y, radius);
    }
}
```

- DrawingAPI(Implementor)를 구현한 구현체
- 추상화된 기능(draw)이 실질적으로 어떻게 수행되는지 알 수 있는 부분 
- CircleShape(Refined Abstraction)에 실제로 주입되는 구체화된 클래스

추상화 된 DrawingAPI의 drawCircle 메서드를 구체화 하고,<br>
클라이언트에 의해 선택되어 CircleShape 생성자 파라미터를 통해 주입된다. 어떤 구체 클래스를 주입하느냐에 따라 기능이 달라진다.

### Client

```java
/** "Client" */
class BridgePattern {
    public static void main(String[] args) {
        Shape[] shapes = new Shape[2];
        shapes[0] = new CircleShape(1, 2, 3, new DrawingAPI1());
        shapes[1] = new CircleShape(5, 7, 11, new DrawingAPI2());

        for (Shape shape : shapes) {
            shape.resizeByPercentage(2.5);
            shape.draw();
        }
    }
}
```

- 브릿지 패턴을 사용하는 역할
- 추상화된 기능 계층에 구체적인 기능을 부여하는 역할

도형을 그리는 방식이 새롭게 추가된다 하였을 때, 브릿지 패턴을 사용하지 않았더라면 CircleShape 내에 새로운 방식에 대한 소스가 반영되어야 할 것이다.

그러나 브릿지 패턴을 사용함으로써 기존 소스의 변경없이 새로운 방식을 구현한 구현체(Concrete Implementor)만 추가 해주면 클라이언트에서 이를 주입하여 사용할 수 있게 된다.

이는 기능을 담당하는 클래스(Shape)와 구현을 담당하는 클래스(DrawAPI)가 서로 영향을 주지 않고 독립적으로 동작할 수 있다는 면에서,

브릿지 패턴의 목적인 "기능과 구현을 분리한다."의 개념이 잘 적용된 것 같다.

<br>

# 고찰

처음 브릿지 패턴에 대한 설명을 글과 구성 다이어그램 으로만 봤을 때는 와닿는 부분이 없었다.<br><br>
근데 예제 소스를 통해 어떤 의도를 가지고 해당 패턴이 나오게 되었는지 확실히 깨달을 수 있었다.<br><br>
그리고 해당 패턴은 메서드 계의 팩토리 패턴이지 않나 생각이 들었다.<br>
같은 성질의 다양한 객체를 팩토리를 통해 제공하듯, 같은 성질의 다양한 메서드(기능)를 Bridge를 통해 제공한다는 점에서 큰 맥락에서는 유사하다는 생각을 해보았다.
{: .notice--primary}