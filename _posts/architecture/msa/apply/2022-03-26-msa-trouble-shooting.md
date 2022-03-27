---
title: "[MSA - Trouble Shooting] MSA 분산 DB 조회 - API Composition 적용"
last_modified_at: 2022-03-27T14:00:00+09:00
categories:
   - Architecture
   - MSA
   - Trouble Shooting
tags:
   - Architecture
   - MSA
   - 분산 DB
   - API Composition
toc: true
toc_sticky: true
toc_label: "목차"
---

MSA : 회사 프로젝트 내에 분산 DB 환경에서의 API 조합기를 적용하기 위한 과정을 담아보았다. 
{: .notice--info}

# 목차

1. [현제 상황](#1-현제-상황)
    - 설계 의도
    - 현제 프로젝트 구조
2. [직면한 문제](#2-직면한-문제)
3. [해결 방법](#3-해결-방법)
4. [해결 과정](#4-해결-과정)

# 1. 현제 상황

## - 설계 의도

- DDD를 통한 도메인 추출
- MSA를 고려한 설계 - 추 후 도메인 별 Micro Service 분리 가능성 고려
- 각 도메인 간의 낮은 의존성, 높은 응집도 유지

## - 현제 프로젝트 구조

- [Onion Architecture](https://tonyjev93.github.io/architecture/onion%20architecture/onion-architecture/) 참고
- 크게 Core Module, Service Modules 로 구성
    - Core Module
        - 모든 Application 의 도메인을 관리 (Domain Layer)
        - 도메인과 DB 를 연결지어주는 Repository Layer 관리
        - 도메인 Service 관리 (Domain Service Layer)
            - 해당 Service 는 Interface 를 통해 외부에 노출되어 오직 DTO 로만 커뮤니케이션 한다.
    - Service Modules
        - 특정 목적을 지닌 Application 을 모듈별로 구현 (결제 서비스, 어드민 등) = Micro Service
        - 각 서비스는 Core Module 을 import 하여 도메인에 접근한다.
        - Application Service 관리 (Application Service Layer)
        - 서비스 호출 역할 수행(UI Layer) - Web Client, Web API ...
- 특징
    - Core Module 내에서 도메인 별 패키지가 분리되어 있음
        - 이는 언제든지 도메인 별 Module 분리 가능성을 고려한 것
        - 즉, 폴리글랏(= 다중 DB) 구조 고려
    - 지금 당장은 하나의 DB 에 모든 도메인 엔티티가 관리되는 중

<br>

# 2. 직면한 문제

## - 어드민 개발

- 어드민의 경우 하나의 요청에 여러가지 도메인이 연관되어 동작하는 기능들이 다수 존재함.
- 그 중 해당 프로젝트에서 구현해야하는 대표적인 도메인간의 협력이 필요한 기능은 아래와 같다.

## - 결제내역 검색 기능
- 검색조건 : 결제 ID, 학생 이름, 상품명 ...
- 검색 결과 : [ 결제정보 + 학생정보 + 상품정보 ] Page 목록
- 위와 같이 하나의 검색요청에 `결제, 학생, 상품 ...` 등등 여러 도메인이 연관되어 있음

## - 문제점
1. 도메인간의 **의존성 분리**를 깨지 않고 어떻게 처리할 수 있을까?
2. 검색조건에 맞는 데이터를 각 **도메인 서비스를 호출하여 조합**을 통해 의존성 분리하면 되나?
3. 그렇다면 조합된 결과의 **페이징 처리는 정상적으로 될까?**
4. 다른 기능에서도 **이와 같은 고민을 반복**해야하는 것인가?

<br>

# 3. 해결 방법

## 참고

- [분산 데이터 쿼리하기](https://tonyjev93.github.io/architecture/msa/msa-distributed-data-query/)
    - [CQRS](https://tonyjev93.github.io/architecture/msa/msa-pattern-cqrs/)
- [분산 DB 조회 설계](https://tonyjev93.github.io/architecture/msa/msa-distributed-db-query-design/)

## 현실적인 적용 방법

- **방법 1 : API 조합 패턴** `In-memory Join`
    - [분산 데이터 쿼리하기](https://tonyjev93.github.io/architecture/msa/msa-distributed-data-query/) > `API 조합 패턴`
    - 이유
        - 대용량 데이터가 아닌 경우 `In-memory Join` 을 해도 성능상의 Issue 는 없을 것
        - 분산 DB 에서 실시간 데이터 조회가 가능하기 때문에 데이터 `무결성 보장` 가능 (실시간성이 보장되면 좋은 경우 사용)
    - 단점
        - 서비스 복잡도에 따라 조합 과정이 번거로울 수 있음
        - 별도의 조합기 구현 필요
- **방법 2 : CQRS 패턴**
    - [분산 데이터 쿼리하기](https://tonyjev93.github.io/architecture/msa/msa-distributed-data-query/) > `CQRS 패턴`
    - 배치를 통한 DB 덮어쓰기
    - 이유
        - `Join Table` 구성을 통해 간단한 방식으로 데이터 조회 가능
    - 단점
        - 마이크로서비스 별 Event Handler 구현 필요 - 데이터 일관성 보장 필요(Saga pattern)
            - ex) 마이크로서비스 DB 내에 CUD 성공. But, CQRS 조회 DB 에 저장하는 Event 가 살패하는 경우 처리필요
        - 실시간성 issue 존재 (해당 프로젝트에서는 중요하지 않음)

## 최종 선택

- `방법 1 : API 조합 패턴` 선택
    - 선택 이유
        - 현제 진행중인 프로젝트의 규모는 트래픽이 적고, 데이터양이 많지 않음.
        - CQRS 적용을 위해 요구되는 infra 환경 구축(메시지큐 적용, 이벤트 기반 통신)은 오버엔지니어링 이라고 판단 됨. (CQRS 는 성능 향상에 포커스가 있다고 생각함.)
        - 데이터의 양이 많지 않다는 점에서 API Composition 의 활용의 단점인 `In-memory join`의 성능 저하는 커버가 됨
        - `Service Layer` 대신 `Infrastructure Layer(외부 API)`를 호출 한다는 점에서 기존 구조에서 변화가 크지 않음.
        - CQRS 에 비해 구조적으로 복잡도가 단순함.
        - 개발 기한을 고려하였을 때 짧은 시간안에 기존 설계(DDD)를 헤치지 않을 수 있는 방법이라고 생각 됨.
    - 고려사항
        - API Composition 통신 방식 (동기 vs 비동기)
        - 인증인가 방식
        - `API Gateway`와의 상대적 위치

---

- `API 조합 패턴`과 유사한 `Facade Pattern`이 있어서 정리해보았다.

### VS Facade Pattern

![img_5](https://user-images.githubusercontent.com/53864640/160266807-ac6796de-5816-4fdb-9f96-c03f60d3a57f.png){: .align-center}

- [Facade Pattern 란 ?](https://tonyjev93.github.io/design%20pattern/design-pattern-facade/)
    - **하나의 요청**을 수행하기 위해 **여러 서비스들이 호출** 되는 경우 클라이언트 단에서 각각의 서비스들의 정체를 알 필요 없이 하나의 요청에 대한 정보만 제공해주는 것.
    - 여러 서비스들의 수행을 하나의 요청으로 묶는 패턴
- API Composer 와의 차이점
    - API Composer : **서로 다른 도메인**을 하나로 묶어주는 역할 수행 (Component)
    - Facade Pattern : **서로 다른 서비스**를 하나로 묶어주는 역할 수행 (Design Pattern)
- 실무 적용
    - 예시
        - 결제 내역 조회
            - 응답 결과 : 결제, 학생, 지원서 .. 도메인 필요
            - 처리 방법 :
                - 여러 도메인의 조합이 필요하기 때문에 `API Composer` 내에 구현
                - 구현은 `Facade Pattern`을 통해 Facade Layer 에서 결제 조회, 학생 조회, 지원서 조회 서비스 호출
        - 결제 처리
            - 처리 과정 : 결제 데이터 생성 -> 지원서 상태 변경
            - 처리 방법 : `Facade Pattern` 을 통해 Facade Layer 에서 여러 서비스를 호출하여 처리
    - 즉, `API Composer` 는 서로 다른 도메인에 대한 조합이 필요할 때 필요되고,
    - `Facade Pattern` 은 도메인이 서로 다르던 같던지 간에 여러 서비스들의 조합이 필요할 때 사용한다.

<br>

# 4. 해결 과정

## 1 단계 : Module 로써의 API Composer

![img_2](https://user-images.githubusercontent.com/53864640/160266804-dbb4bc59-8033-4539-aae9-554ea13d46c2.png){: .align-center}

- API Composer 를 `Core Module` 내에 구현
    - 현제 모든 도메인에 대한 데이터가 동일 DB 내에 존재하기 때문에 `Core Module` 내에 API Composer 을 구현하여 외부 통신을 최소화 하여 개발 편의성을 높힌다.

![img_1](https://user-images.githubusercontent.com/53864640/160266803-c32b5097-ee29-46ee-8752-03eeb0a87ae4.png){: .align-center}

- 각각의 Micro Service 는 동일한 `Core Module`을 import 하고 있기 때문에 모두 동일한 `API Composer`를 사용할 수 있다.
- `API Composer Client` 인터페이스를 통신하므로서 `클라이언트`로부터 `API Composer` 구현체의 의존성을 분리할 수 있다.
- 이를 통해 `API Composer`의 구현 자유도가 높아진다.

## 현제 구도의 한계

- 단일 DB 사용

## 2 단계 : MicroService 로써의 API Composer

![img_4](https://user-images.githubusercontent.com/53864640/160266806-afc5515e-9a94-4617-a146-182eebaa24ff.png){: .align-center}

- `Core Module`로 부터 도메인 별 마이크로 서비스 분리
- `API Composer` 별도 서비스 구동
- 각 도메인 별 DB 분리

![img](https://user-images.githubusercontent.com/53864640/160266802-9267380c-3057-4d81-85d8-78a893dd5be0.png){: .align-center}

- `API Composer Client`를 통한 `API Composer` 서비스 호출

<br>

## 실무 적용

- 실무에 적용하기 위해 API Composer를 구성하려 했으나, 실제 프로젝트 내에 있는 Facade Layer 와 역할이 똑같다는 의견이 있어 별도의 Module을 두기 보다는 Facade Layer 를 활용하기로 결정하였다.
- 추 후, 프로젝트 규모가 커지게 된다면 그때 대응해도 늦지 않을 것이라는 판단이 들어 우선적으로 단일 DB내에 queryDsl 을 활용해 table join 을 허용하는 것으로 결정하였다.
- 만약 DB 분리가 필요하다 하면 API Composer 보다는 CQRS를 통해 실용성과 확장성을 높히지 않을까 싶다. 그러기 위해 데이터 동기화, 장애 관리, 동시성 이슈 등에 대한 학습이 더욱 필요할 것 같다.

<br>

# 결론

- 아직 API Composition을 적용시키진 못했다. 실제로 `in-memory join`을 구현해보았으나 여간 귀찮은 작업이 아닐 수 없다... 유지보수 차원에서도 많이 복잡하고 단순한 구조가 아니라는 생각이 들었다.
- `오버엔지니어링`이라는 생각이 많이 들었고, 설계원칙이 깨지는 한이 있더라도 당장은 QueryDsl을 이용하여 Join하는 것이 바람직해 보인다. 추 후, 여건이 된다면 CQRS를 도입하는것은 좋을 것 같다.