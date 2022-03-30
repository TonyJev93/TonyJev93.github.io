---
title: "[Design Pattern] 디자인 패턴 Overview"
last_modified_at: 2022-03-17T02:33:00+09:00
categories:
    - Design Pattern
tags:
    - Design Pattern
toc: true
toc_sticky: true
toc_label: "목차"
---

디자인 패턴 : 디자인 패턴의 전체적인 개요를 살펴보자.
{: .notice--info}

# 디자인 패턴이란?

**디자인 패턴**은 그동안 소프트웨어를 개발해오면서 마주했던 고질적인 문제들을 해결하기 위해 고착화된 해결 방법들을 뜻한다.

> Don't reinvent the wheel
> : 바퀴를 다시 발명하지 마라 

이미 문제를 해결해줄 바퀴는 만들어져 있으니 처음부터 다시 만들지 말고 있는 것을 사용하라는 의미이다.<br>
각종 풍파를 겪고 디자인 패턴을 남겨주신 선배님들께 감사를 표하자. (_ _)

<br>

# 디자인 패턴 종류

## GoF 디자인 패턴

GoF 란?
: GoF(Gang of Fout)라 불리는 사람들 : Erich Gamma, Richard Helm, Ralph Johnson, John Vissides<br>
소프트웨어 개발 영역에 있어 디자인 패턴을 구체화하고 체계화한 사람들이다.

## 분류

### 생성 패턴
**객체 생성**과 관련된 패턴으로 객체의 생성과 조합을 캡슐화하여 기존 구조에 영향을 주지 않고도 특정 객체의 생성과 변경을 자유롭게 할 수 있도록 유연성을 제공한다.

### 구조 패턴
**구조 패턴**은 클래스 또는 객체를 조합하여 더 큰 구조를 만드는 패턴이다. 서로 다른 인터페이스 또는 객체를 묶어 새로운 기능을 제공하도록 하는 패턴이다.

### 행위 패턴
**행위 패턴**은 객체나 클래스 사이의 알고리즘이나 책임 분배에 관련된 패턴이다. 한 객체가 혼자 수행할 수 없는 작업을 여러개의 객체로 어떻게 분배하는지, 또 그렇게 하면서도 객체 사이의 결합도를 최소화 하는것에 중점을 두는 방식이다.

## 종류

|                                                                                                                     생성 패턴(Creational Patterns) | 구조 패턴(Structural Patterns)                                                                             | 행위 패턴(Behavioral Pattern)                |
|-----------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|------------------------------------------|
|                                                                                                                      싱글톤 패턴(Singleton Pattern) | [어댑터 패턴(Adapter Pattern)](/design%20pattern/design-pattern-adapter/)        | 커맨드 패턴(Command Pattern)                  |
|                            [추상팩토리 패턴(Abstract Factory Pattern)](/design%20pattern/design-pattern-abstract-factory/) | [브리지 패턴(Bridge Pattern)](/design%20pattern/design-pattern-bridge/)                                     | 인터프리터 패턴(Interpreter Pattern)            |
|                                                                                                                         빌더 패턴(Builder Pattern) | [컴퍼지트 패턴(Composite Pattern)](/design%20pattern/design-pattern-composition/) | 이터레이터 패턴(Iterator Pattern)               |
|                              [팩토리 메서드 패턴(Factory Method Pattern)](/design%20pattern/design-pattern-factory-method/) | 데코레이터 패턴(Decoratior Pattern)                                                                           | 미디에이터 패턴(Mediator Pattern)               |
|                                          [프로토타입 패턴(Prototype Pattern)](/design%20pattern/design-pattern-prototype/) | [퍼사드 패턴(Facade Pattern)](/design%20pattern/design-pattern-facade/)          | 메멘토 패턴(Memento Pattern)                  |
|                                                                                                                                                | 플라이웨이트 패턴(Flyweight Pattern)                                                                           | 옵저버 패턴(Observer Pattern)                 |
|                                                                                                                                                | 프록시 패턴(Proxy Pattern)                                                                                  | 스테이트 패턴(State Pattern)                   |
|                                                                                                                                                |                                                                                                        | 책임 연쇄 패턴(Chain of Responsibility Pattern) | 
|                                                                                                                                                |                                                                                                        | 스트래티지 패턴(Strategy Pattern)               | 
|                                                                                                                                                |                                                                                                        | 템플릿 메서드 패턴(Template Method Pattern)      | 
|                                                                                                                                                |                                                                                                        | 비지터 패턴(Visitor Pattern)                  |


## ETC

[Marker interface pattern](/design%20pattern/design-pattern-marker-interface)
: Serializable, Cloneable - 클래스 명만으로 파악가능하도록 하는 패턴 