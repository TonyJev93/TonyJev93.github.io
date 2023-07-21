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

## JDBC

- 자바 프로그래밍 언어와 다양한 데이터베이스 SQL 또는 테이블 형태의 데이터 사이에 독립적인 연결을 지원하는 표준(DB 작업을 위한 표준)
- DBMS 회사들이 JDBC 인터페이스를 구현하여 제공(= JDBC 드라이버)


## ORM

- Object Relational Mapping
- 객체와 데이터베이스의 관계를 매핑해주는 도구

## 하이버네이트

- Java 언어를 위한 ORM 프레임워크
- JPA의 구현체

## JPA 란?
 
- 관계형 데이터베이스와 객체의 패러다임 불일치 문제를 해결
- 자바 애플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 **인터페이스**
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

### [@Transactional](https://kafcamus.tistory.com/30)

> readOnly

- defalut = false
- readOnly = true 설정 : 스프링은 해당 트랜잭션의 FlushMode를 NEVER로 설정
  - 이 경우 flush가 일어나지 않으므로 비용이 절감되며, 또한 생성/수정/삭제가 일어나지 않으므로 별도의 스냅샷을 만들 필요가 없어 성능상 이점이 생긴다.

### [@Transactional & Lazy Loading](https://kafcamus.tistory.com/31)

### [@Transactional isolation & propagation](https://kafcamus.tistory.com/33)

### [JPA Lock](https://velog.io/@recordsbeat/JPA%EC%97%90%EC%84%9C-Write-Skew-%EB%B0%A9%EC%A7%80%ED%95%98%EA%B8%B0-locking-%EC%A0%84%EB%9E%B5)

## 실무 적용 및 운영 주의사항

### N+1

- 즉시로딩
  - JPA 기본 반찬 사용 시 문제는 없음. 쿼리 수행 시 내부적으로 join을 통해서 한번에 가져옴.
  - JPQL 사용 시 문제가 발생. 부모를 조회 했는데 eager 에 의해 자식도 다 조회해오는 사태 발생.
- 지연로딩
  - 최초에 가져올 때만 문제없는듯 보이지만 결국 자식을 호출하는 순간 캐싱된 프록시 객체에 의해 즉시로딩과 동일해짐
- fetch join (with. 지연로딩)
  - 지연 로딩이 걸려있는 관계에 대해 한번에 즉시로딩 해주는 구문
  - fetch join 없이 jpql로 join 했을 경우 > 지연로딩과 동일
  - 즉시로딩과 함께 사용 불가
    - 둘은 동일한 쿼리 실행 전략을 가지고 있어 JPA가 둘 중 어떤 전략을 선택할지 혼란을 가짐
    - 즉시로딩 : 영속성 컨텍스트가 엔티티를 즉시로딩 하도록 하는 설정. 1을 조회한 후 N개에 대해 영속성 컨텍스트가 쿼리를 수행
    - fetch join : 명시적으로 특정 연관 관계 로딩 시 사용. 연관된 엔티티 즉시 로딩

```java
@Query("select distinct u from User u left join fetch u.articles")
List<User> findAllJPQLFetch();
```

- @EntityGraph
  - fetch join 시 하드코딩하는 단점 존재

```java
@EntityGraph(attributePaths = {"articles"}, type = EntityGraphType.FETCH)
@Query("select distinct u from User u left join u.articles")
List<User> findAllEntityGraph();
```

- fetch join 주의할 점
  - Pagination 사용 시
    - `WARN --- [    Test worker] o.h.h.internal.ast.QueryTranslatorImpl   : HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!`
    - 쿼리 수행 전 인메모리를 이용하여 조인을 하였다는 경고 문구가 발생 -> 모든 데이터 조회 후 메모리에서 페이징 처리, OOM 원인
    - 이유 : fetch join 시 distinct를 사용하기 때문에 쿼리문에 Limit, Offset 이 적용 안됨.(그치만 응답은 적용이 된 결과를 받음)
    - 해결책1 : ToOne 관계에서 페이징 사용

```java
@EntityGraph(attributePaths = {"user"}, type = EntityGraphType.FETCH)
@Query("select a from Article a left join a.user")
Page<Article> findAllPage(Pageable pageable);
```

이렇게 하면 ManyToOne 관계로 limit을 걸어서 가져옴

- Pagination 해결책 2 : Batch Size 사용

```java
Page<User> findAll(Pageable pageable);
``` 

```java
// User.java
@BatchSize(size = 100)
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
private Set<Article> articles = emptySet();
```

이제 findAll() 쿼리 수행 시 limit 포함됨. 근데 article에 대한 select 쿼리가 추가적으로 수행 됨.

이유는 지연로딩으로 Batch성 로딩을 하게 된 것. 그때 마다 쿼리를 날리는 N+1이 아니라 조회할 대상을 배치로 묶어서 한번에 쿼리를 수행

단점으로는 최적화된 Batch Size를 구하기 어렵고 where에서 in 구문을 이용하기 때문에 1000개 이상의 데이터 조회가 어렵

- 해결책 3 : @Fetch(FetchMode.SUBSELECT)

```java
// User.java
@Fetch(FetchMode.SUBSELECT)
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
private Set<Article> articles = emptySet();
```

where 문에 in을 통해 user를 전체 조회하도록 쿼리가 작성 됨. 이는 모든 유저를 가져온다는 뜻으로 성능상 좋지 않음.

- 문제 2 : 2개 이상의 Collection Fetch join(~ToMany) 불가능

> [결론](https://velog.io/@jinyoungchoi95/JPA-%EB%AA%A8%EB%93%A0-N1-%EB%B0%9C%EC%83%9D-%EC%BC%80%EC%9D%B4%EC%8A%A4%EA%B3%BC-%ED%95%B4%EA%B2%B0%EC%B1%85#%EA%B2%B0%EB%A1%A0) - 이글 보면 됩니다.

