---
title: "[Design Pattern] 구조 패턴 - 컴포지션(Composition)"
last_modified_at: 2022-03-23T01:00:00+09:00
categories:
    - Design Pattern
tags:
    - Design Pattern
    - 구조 패턴
    - 컴포지션 패턴
toc: true
toc_sticky: true
toc_label: "목차"
---

디자인 패턴 : 구조 패턴 중 하나인 컴포지션 패턴에 대해 알아보자.
{: .notice--info}

# Composition Pattern

## 개념

Composition 단어 뜻?
: 구성...

**그렇다면 구성(Composition) 패턴이란 무엇일까..?**

- **구조 패턴** 중 하나
- 객체들의 관계를 트리 구조로 구성하여 부분-전체 **계층을 표현하는 패턴**
- 사용자가 단일 객체와 복합 객체 모두 클라이언트에서 동일하게 다루도록 하는 패턴

참고로 구조 패턴이란..?
: 클래스나 객체를 조합해 더 큰 구조를 만드는 패턴

복합 객체는 뭘까..?
: 복합 객체 = 여러 개의 객체들로 구성

**그럼 언제 쓰면 좋은걸까?**<br>
: **전체-부분의 관계(EX. Directory-File)를 갖는 객체들 사이의 관계를 정의**할 때 유용하다.

## 구조

![img](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcW7HqG%2FbtqyV5IV2PM%2FVW9kw0xyX36XdeEksfD6o1%2Fimg.png){: .align-center}

### Component
  - `Composite`이 관리할 요소
  - `Leaf`와 `Composite`에서 사용되는 공통 메서드를 갖는 **인터페이스 또는 추상 클래스**

### Leaf
  - `Composite`의 **부품 역할**
  - 다른 객체에 대한 참조는 포함되어 있지 않음
  
### Composite
  - 전체 클래스 (`Composite도` `Component`의 하위 클래스로 선언됨)
  - `Leaf`를 요소로 가지고 있음
  - 복수개의 `Leaf`와 심지어 복수개의 `Composite`객체를 부분으로 가질 수 있음


# 예제

## ex) 컴퓨터

### Component : ComputerDevice
- `getPrice()`, `getPower()` 메서드를 포함하는 인터페이스

### Leaf : Keyboard, Body, Monitor
- `ComputerDevice(= Component)` 를 구현한 실제 클래스들
- 추 후, `Composite`의 요소가 될 대상들

### Composite : Computer
- `addComponent(ComputerDevice component)`를 통해 `구체적인 ComputerDevice(= Leaf)`를 복수개 포함시킬 수 있음 (반대로, `removeComponent(ComputerDevice component)`를 통해 제거도 가능)
- `Computer(= Composite)` 또한 `ComputerDevice`를 구현한 구현체임.
- `Computer`내에 존재하는 복수개의 `ComputerDevice` 마다의 함수를 iterator를 통해 호출하여 원하는 기능을 구현한다. 
```java
    // 전체 부품의 가격 합산
    public int getPrice() {
        int price = 0;
        for(ComputerDevice component : components) {
          price += component.getPrice();
        }
        return price;
    }
``` 
- 앞으로 새로운 `ComputerDevice`가 추가되었을 때 이를 구현한 새로운 클래스(부품)만 만들어주면 되고, 클라이언트에서의 `Computer(= Composite)` 사용 코드는 변경없이 유지가 된다.

# 고찰

사실 와닿지 않는 패턴이다. 실질적으로 사용을 할 수 있는 상황이 안주어져서 그런것 같은데, 만약 **계층적인 관계를 가지고 있으면서** **클라이언트에서 모두 동일하게 다루도록 하는 상황**이 생긴다면, 사용을 고려해볼만 할 것 같다.
{: .notice--primary}

---

# 참고

- [[Design Pattern] Composition Pattern](https://beomseok95.tistory.com/258)
- [[Design Pattern] 컴퍼지트 패턴이란](https://gmlwjd9405.github.io/2018/08/10/composite-pattern.html)

[⬅️ 디자인패턴 목차보기](/design%20pattern/design-pattern-overview/)