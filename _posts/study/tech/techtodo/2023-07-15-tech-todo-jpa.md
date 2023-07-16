---
title: "[Study] 기술 부채 - JPA"
last_modified_at: 2023-07-15T23:00:00+09:00
categories:
    - interview
tags:
    - Study
    - 기술 부채
    - JPA
toc: true
toc_sticky: true
toc_label: "목차"
---

Study : 내가 부족한 기술(JPA)에 대해 정리한다.
{: .notice--info}

# JPA 파헤치기

## JPA 란?
 
- 데이터 접근 레이어를 편하게 구현할 수 있도록 지원하는 프레임워크 
- RDB 활용 시 널리 사용
- 하이버네이트의 기능을 스프링 프레임워크에 맞게 확장한 모듈(= 기본적 구조와 동작은 하이버네이트 구현을 기반으로 함)

## JPA 원리(개념)

- 영속성 컨텍스트
  - 1차 캐시
  - DB와 연동되는 엔티티를 관리하고 캐싱 함.
  - 엔티티 메니저 또는 세션과 1:1 관계를 갖음
    - 세션(SessionImpl)은 엔티티 매니저의 구현체이며 DB와 연결을 맺으며 생성

하이버 네이트는 이벤트 기반으로 동작하며 각 이벤트마다 Event Listener Group 이 존재. 기본으로 등록된 이벤트 리스너가 존재.

persist 역시 이벤트로 발행되며 DefaultPersistEventListener가 등록되어 있음.

persist 호출 시 PersistEvent(= AbstractEvent의 구현체)를 넘겨 DefaultPersistEventListener의 onPersist가 호출 됨.

DefaultPersistEventListener에서 엔티티 영속화(PersistContext내 엔티티 등록) 작업이 동작 함.

영속된 엔티티는 리스트 자료구조로 EntityEntryContext에서 관리됨. 

DB에 Insert 명령어 실행 전 ActionQueue에 EntityInsertAction 을 Enque 함.(쓰기 지연)

Session의 flush가 호출 됨. FlushEvent가 발행되며 DefaultFlushEventListener 가 ActionQueue의 executeActions 를 실행 하여 Action 일괄 실행

- [참고](https://brunch.co.kr/@anonymdevoo/47)

### [Repository 생성 과정](https://brunch.co.kr/@anonymdevoo/40)

> **JpaRepository**

spring-data-jpa 사용 시 JpaRepository를 인터페이스로 선언하여 사용한다.

인터페이스 선언만으로 어떻게 save(), saveAll(), findAll() 같은 메서드들을 사용할 수 있게 된 것 일까?

정답은 `SimpleJpaRepository`를 통해 가능하게 된다.

인터페이스로 선언된 JpaRepository의 save()를 호출하면 내부적으로 SimpleJpaRepository의 save()를 호출한다.

> **RepositoryFactorySupport**

`JpaRepositoryFactory` 클래스가 JpaRepository 를 생성. 개발자가 정의한 Repository 인터페이스를 참고하여 JpaRepository 를 대신 구현하는 역할을 한다.

JpaRepositoryFactory는 spring-data-commons 모듈의 `RepositoryFactorySupport`를 확장하고 있음

`RepositoryFactorySupport`는 전달받은 Repository 인터페이스로 프록시 인스턴스를 생성하는 추상 클래스임.

RepositoryFactorySupport를 확장하여 다양한 `spring-data-XXX`에서 Repository 를 생성

RepositoryFactorySupport에는 getRepository라는 Repository 인스턴스를 생성하는 메서드가 기본으로 존재하며 이게 Repository 인터페이스를 구현한 Proxy 를 생성.

JpaRepositoryFactory가 부모클래스의 getRepository를 상속 받음.

> **JpaRepositoryFactory**

spring-data-jpa는 JpaRepositoryFactory가 추상 메서드인 getRepsotiroyBaseClass를 SimpleJpaRepository.class를 반환하도록 오버라이드 되어있음.

> **RepsoitoryBaseClass**

`RepsoitoryBaseClass`는 실제 Repository 인스턴스를 지원하는 클래스이다.

즉, SimpleJpaRepository는 실제 생성된 JpaRepository 인스턴스를 지원하는(Backing) 인스턴스의 클래스가 된다.

RepositoryBaseClass는 getRepositoryInformation에서 RepositoryInformation 인스턴스를 조립할 때 전달 된다.

> **RepositoryInformation**

RepositoryInformation는 Repository의 정보(엔티티 타입, 아이디 타입, Base Class는 무엇인지 등)를 담고 있다.

getTargetRepository가 반환하는 인스턴스의 클래스가 BaseClass와 같아야 함.

RepositoryInformation는 getTargetRepository의 인자로 넘어가서 BaseClass 정보를 전달하여 TargetRepository 인스턴스의 클래스 타입을 결정한다.

> **TargetRepository**

QueryProxy 를 지원하는 Repository 인스턴스라고 한다.

getTargetRepository는 JpaRepositoryImplementation 을 반환하고 이를 ProxyFactory의 target 속성으로 세팅한다.

그 후 ProxyFactory로 부터 프록시 객체를 생성하여 최종 repository 로 반환한다.

> 정리

1. Bean 으로 생성되는 CustomRepository는 Proxy 인스턴스이다. (ProxyRepository)
2. JpaRepository 는 Proxy 인스턴스의 Target을 SimpleJpaRepository.cass의 인스턴스로 주입한다.
3. Proxy Repository 인스턴스 내부에서 SimpleJpaRepository가 실 구현체를 실행한다.

결론 : ProxyFactory 에 의해 생성된 JpaRepository 의 인스턴스는 JpaRepositoryFactory가 반환한 SimpleJpaRepository가 된다.

### [save() 동작 원리](https://brunch.co.kr/@anonymdevoo/37)

- JpaRepository는 인터페이스로 실제로는 구현체의 메서드인 SimpleDataJpaRepository.save() 호출
- SimpleDataJpaRepository 내 EntityInformation 타입의 필드를 통해 isNew() 메서드 호출
  - EntityInformation(= I/F) : Entity의 메타 정보(이름, 필드, 타입 등) 가지고 잇음
- EntityInformation의 구현체(Entity가 Persistale 구현체인 경우 제외)인 JpaMetamodelEntityInformation 의 isNew() 호출 됨
- JpaMetamodelEntityInformation > isNew() > super.isNew() == AbstractEntityInformation
  - AbstractEntityInformation.isNew() = `id is null` or `Number type == 0` 일 경우 `true`
- But, Version 필드가 존재하면 `Version 필드 is null` 일 때 isNew = true 이다. (id 를 isNew() 판단에 사용 X)

## 코드 레벨에서의 지식


## 실무 적용 및 운영 주의사항