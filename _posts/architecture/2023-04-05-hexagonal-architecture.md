---
title: "[Architecture] Hexagonal Architecture"
last_modified_at: 2023-04-06T14:00:00+09:00
categories:
   - Architecture
   - Hexagonal Architecture
tags:
   - Architecture
   - Hexagonal Architecture
toc: true
toc_sticky: true
toc_label: "목차"
---

Architecture : MSA, DDD와 어울리는 헥사고날 아키테처에 대해 알아보자. 
{: .notice--info}

# 아키텍처

* 건축물의 구조물

![image](https://user-images.githubusercontent.com/53864640/230626523-d1cebb79-c068-4155-9f88-f6e835a1a42b.png)


## 소프트웨어 아키텍처

* **소프트웨어의 구조와 구성 요소 간의 상호관계를 설계**하는 것
* **개발 초기 단계**에 이루어지면서 **유지보수와 확장 및 변경에 용이하게 만드는 것**이 목적
* 개발 및 운영 과정에서 **유지보수되는 방식을 결정**하는 중요한 역할

![image](https://user-images.githubusercontent.com/53864640/230626629-7eea6739-c0c5-49da-979e-7165edab1d3f.png)

## 등장 배경

* **초기 소프트웨어는 단순**하여 명시적인 **아키텍처 불필요**
* 소프트웨어 **복잡도 증가**
* 아키텍처가 없는 설계에서 문제가 발생
* 개발자들이 연구를 통해 **다양한 아키텍처 패턴이 탄생함**
* 개발자는 **문제해결을 위한 적절한 아키텍처를 선택**하여 사용하게 됨

![image](https://user-images.githubusercontent.com/53864640/230626688-87a2564e-6dcd-4322-8342-b693bd40532e.png)

## 최근 서비스들의 특징

서비스 규모가 점점 커지고 있음. 사용자 수, 트래픽, 처리해야할 데이터의 양 등이 계속해서 커지는 중.

### 대용량 데이터

* 데이터 양이 많아짐
* 대규모 데이터를 저장하고 처리하는 기술 필요

### 다량의 트래픽

* 동시 접속자 수가 많아지고 있음
* 대규모 트래픽을 처리할 수 있는 기술이 필요

### 빠른 응답성 및 고가용성

* 응답이 빨라야 함
* 연중무휴 서버가 다운되면 안 됨

### 서비스의 다양성 및 유연성

* 바쁘다 바뻐 현대사회 변화무쌍한 고객의 니즈
* 수 많은 변화를 받아들일 수 있는 유연성

## MSA, DDD

위 서비스 특징을 다루기 위해 **하나의 큰 서비스**를 **작은 단위의 독립적인 서비스**들로 나누며

각각의 서비스에 최적화된 환경을 구축하며 변화에 빠른 대응을 하기 시작.

이러한 소프트웨어 구조를 **MSA**(마이크로서비스 아키텍처)라고 부르게 되었고,

MSA를 진행하는데 있어 핵심 비즈니스 로직을 담당하는 '**도메인**'이라는 개념이 중심을 잡기 시작하면서 **DDD**(도메인 주도 설계)가 중요하게 여겨짐.

이러한 변화 속에서 **헥사고날 아키텍처**는 **도메인 중심의 모듈화 개발과 MSA와의 높은 연계성을 가지는 매우 효과적인 아키텍처로 평가**되고 있음

---
<br/>
<br/>


# 헥사고날 아키텍처

* **클린 아키텍처**의 한 종류
* DDD와 MSA에 어울리는 아키텍처

## 클린 아키텍처

* `로버트 C. 마틴(클린 코드, 클린 아키텍처 저자)`이 제안한 아키텍처 디자인 원칙을 기반
* 특징
    * **프레임워크 독립적이다.** \- 프레임워크 종류 상관없이 사용 가능한 아키텍처
    * **테스트 용이하다.** \- UI\, DB\, Web 등 외부 요소 없이도 비즈니스 로직의 **테스트 가능**
    * **UI 독립적이다.** \- UI의 변경이 비즈니스 로직에 영향을 끼치지 않음
    * **DB 독립적이다.** \- DB의 변경이 비즈니스 로직에 영향을 끼치지 않음
    * **외부 에이전시 독립적이다.** \- 외부 서비스의 변경이 비즈니스 로직에 영향을 끼치지 않음

\>\> 외부의 어떠한 것도 비즈니스 로직에 영향을 끼쳐서는 안된다\.

## 클린 아키텍처 구조

![image](https://user-images.githubusercontent.com/53864640/230626848-bc6ab641-398f-49b9-b4e8-297cabd2b6ea.png)

* 계층형 아키텍처의 의존성은 항상 다음 계층(아래 방향)을 향함
* 클린 아키텍처의 의존성은 원 안쪽으로 향함
    * **저수준 정책**(기술적인 세부 사항. ex. Web, DB, ...) -> **고수준 정책**(업무 로직)

### 결론 : 내부는 외부가 어떻게 생겼는지 전혀 모름...

![image](https://user-images.githubusercontent.com/53864640/230626958-87bdccc0-a339-4da9-91be-39f036a4232c.png)

**클린 아키텍처는 하나의 사상**이라고 보면 되고 **이를 구현한 다양한 구현체중 하나가 바로 헥사고날 아키텍처**

## 헥사고날 아키텍처 구조

![image](https://user-images.githubusercontent.com/53864640/230627012-1ef1b475-7b87-4f2d-ab81-ca828a61d682.png)

**비즈니스 로직을 다루는 내부(Port, Application Core)**와 **기술적인 부분을 다루는 외부(Adapter)**로 나뉨.

그래서 헥사고날 아키텍처는 **Port & Adapter 아키텍처라고도 불린다**.

### \# 어댑터\(Adapter\)

기술적인 부분을 다루는 외부 계층
언제든지 갈아 끼워질 수 있는 구체적이고 세부적인 기술 계층

![image](https://user-images.githubusercontent.com/53864640/230627050-99038942-b1c6-4ad8-8008-7091ca53e073.png)

* driving adapter (= incoming adapter)
    * 외부 세계의 요청을 통해 **애플리케이션 코어를 호출하는 역할**(외부 세계가 내부 세계를 호출)
    * UI, Controller, EventListener ...
    * **Presentation Layer**(in Layered Architecture)
* driven adapter (= outgoing adapter)
    * **애플리케이션 코어에 의해 호출되는 역할**(내부 세계가 외부 세계 호출)
    * Repository(DB, Redis, ...), EventPublisher, Email, 외부 API ...
    * **Persistence / Infrastructure Layer**(in Layered Architecture)

### \# 포트\(Port\)

**인터페이스로 구성**되어 `어댑터`와 `애플리케이션 코어` 간의 통신을 담당

![image](https://user-images.githubusercontent.com/53864640/230627481-b78fd52f-598c-4a01-bf58-092150f18290.png)

* Input Port
    * 어댑터(driving adapter) -> **Input Port** -> 애플리케이션 코어
    * Input Port 구현체 = UseCase
* Output Port
    * 애플리케이션 코어 -> **Output Port** -> 어댑터(driven adapter)
    * Output Port 구현체 = driven adapter

### \# 애플리케이션 코어\(Application Core\)

비즈니스 핵심 로직 담당

* UseCase
    * driving adapter에 의해 호출되는 Input Port의 구현체이다.
    * 도메인과 외부 서비스를 이용하여 서비스 로직을 구현
    * **Application Service Layer**(in Layered Architecture)
* Entity(= Domain)
    * 비즈니스의 핵심 로직을 다루는 도메인
    * **Domain Model Layer**(in Layered Architecture)

<br/>
<br/>

# 요약

![image](https://user-images.githubusercontent.com/53864640/230627532-b457ee40-4fcd-4acf-a956-bd922a410e47.png)

언제 갈아 치워질 지 모르는 **외부 세계** 불안...

언제나 평화로운 **내부 세계** 편안~