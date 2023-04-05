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

MSA : 회사 프로젝트 내에 분산 DB 환경에서의 API 조합기를 적용하기 위한 과정을 담아보았다. 
{: .notice--info}

# 아키텍처

* 건축물의 구조물

## 소프트웨어 아키텍처

* <strong>소프트웨어의 구조와 구성 요소 간의 상호관계를 설계</strong>하는 것
* <strong>개발 초기 단계</strong>에 이루어지면서 <strong>유지보수와 확장 및 변경에 용이하게 만드는 것</strong>이 목적
* 개발 및 운영 과정에서 <strong>유지보수되는 방식을 결정</strong>하는 중요한 역할

## 등장 배경

* <strong>초기 소프트웨어는 단순</strong>하여 명시적인 **아키텍처 불필요**
* 소프트웨어 **복잡도 증가**
* 아키텍처가 없는 설계에서 문제가 발생
* 개발자들이 연구를 통해 **다양한 아키텍처 패턴이 탄생함**
* 개발자는 <strong>문제해결을 위한 적절한 아키텍처를 선택</strong>하여 사용하게 됨

## 최근 서비스들의 특징

서비스 규모가 점점 커지고 있음.
사용자 수, 트래픽, 처리해야할 데이터의 양 등이 계속해서 커지는 중.

### 대용량 데이터

* 데이터 양이 많아짐
* 대규모 데이터를 저장하고 처리하는 기술 필요

### 다량의 트래픽

* 동시 접속자 수가 많아지고 있음
* 대규모 트래픽을 처리할 수 있는 기술이 필요

### 빠른 응답성 및 고가용성

### 서비스의 다양성 및 유연성

## MSA, DDD

위 서비스 특징을 다루기 위해 <strong>하나의 큰 서비스</strong>를 <strong>작은 단위의 독립적인 서비스</strong>들로 나누며

각각의 서비스에 최적화된 환경을 구축하며 변화에 빠른 대응을 하기 시작.

이러한 소프트웨어 구조를 <strong>MSA</strong>(마이크로서비스 아키텍처)라고 부르게 되었고,

MSA를 진행하는데 있어 핵심 비즈니스 로직을 담당하는 '<strong>도메인</strong>'이라는 개념이 중심을 잡기 시작하면서 <strong>DDD</strong>(도메인 주도 설계)가 중요하게 여겨짐.

이러한 변화 속에서 <strong>헥사고날 아키텍처</strong>는 <strong>도메인 중심의 모듈화 개발과 MSA와의 높은 연계성을 가지는 매우 효과적인 아키텍처로 평가</strong>되고 있음

# 헥사고날 아키텍처

* <strong>클린 아키텍처</strong>의 한 종류
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

* 계층형 아키텍처의 의존성은 항상 다음 계층(아래 방향)을 향함
* 클린 아키텍처의 의존성은 원 안쪽으로 향함
    * <strong>저수준 정책</strong>(기술적인 세부 사항. ex. Web, DB, ...) -> <strong>고수준 정책</strong>(업무 로직)

### 결론 : 내부는 외부가 어떻게 생겼는지 전혀 모름...

<strong>클린 아키텍처는 하나의 사상</strong>이라고 보면 되고 **이를 구현한 구현체가 바로 헥사고날 아키텍처**

## 헥사고날 아키텍처 구조

<strong>비즈니스 로직을 다루는 내부(Port, Application Core)</strong>와 <strong>기술적인 부분을 다루는 외부(Adapter)</strong>로 나뉨
그래서 헥사고날 아키텍처는 <strong>Port & Adapter 아키텍처라고도 불린다</strong>.

### \# 어댑터\(Adapter\)

기술적인 부분을 다루는 외부 계층
언제든지 갈아 끼워질 수 있는 구체적이고 세부적인 기술 계층

* driving adapter (= incoming adapter)
    * 외부 세계의 요청을 통해 <strong>애플리케이션 코어를 호출하는 역할</strong>(외부 세계가 내부 세계를 호출)
    * UI, Controller, EventListener ...
    * <strong>Presentation Layer</strong>(in Layered Architecture)
* driven adapter (= outgoing adapter)
    * <strong>애플리케이션 코어에 의해 호출되는 역할</strong>(내부 세계가 외부 세계 호출)
    * Repository(DB, Redis, ...), EventPublisher, Email, 외부 API ...
    * <strong>Persistence / Infrastructure Layer</strong>(in Layered Architecture)

### \# 포트\(Port\)

<strong>인터페이스로 구성</strong>되어 `어댑터`와 `애플리케이션 코어` 간의 통신을 담당

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
    * <strong>Application Service Layer</strong>(in Layered Architecture)
* Entity(= Domain)
    * 비즈니스의 핵심 로직을 다루는 도메인
    * <strong>Domain Model Layer</strong>(in Layered Architecture)