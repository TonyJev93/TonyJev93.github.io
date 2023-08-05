---
title: "[Spring] Spring Security - CORS 해결하기"
last_modified_at: 2022-07-27T23:50:00+09:00
categories:
    - Back End
    - Spring
    - Spring Security
tags:
    - Back End
    - Spring
    - Spring Security
toc: true
toc_sticky: true
toc_label: "목차"
---

Spring : Spring Security 사용 시 CORS 문제를 해결하기 위한 방법을 정리하자. 
{: .notice--info}
Spring Data Envers

# Spring Data Envers

## Envers 란?

* Hibernate 에서 만든 데이터 변경 감지를 통한 이력 관리 라이브러리이다.
* insert, update, delete 가 발생한 경우 자동으로 이력테이블에 데이터를 쌓아주는 편리한 기능을 제공한다.
* 기본적으로 JPA 로 구현되어 있으며, Spring에서도 Spring Data Envers 프로젝트를 추가하였다.

## 장점

* **가성비 좋은 라이브러리** - 사용 방법에 비해 얻는 것이 많다!
* **이력 관리 누락 위험이 없음** - 엔티티에 어노테이션(@Audited) 만 달아주면 해당 엔티티의 모든 변경을 자동으로 추적한다.
* **Spring Data 프로젝트로 제공 될 정도** - 그 만큼 유용하고 편리한 기능을 제공한다.

## 단점

* **잘 모르고 사용하면 장애발생 가능성 다분하다.** - 기본으로 제공해주는 이력ID의 속성값이 INT로 20억건 이상 이력쌓이면 장애 발생~ (INT -> LONG 필히 변경하자.)
* **테이블 생성을 직접 해야하는 상황에서 불편 및 위험!** - ddl-auto create 사용하면 더할 나위 없이 편리.. 근데 운영환경에서 사용하게 되면 직접 테이블명, 테이블 필드, 속성 등등 설정해줘야한다. (오타라도 발생하면 장애..?)

## 의존성 추가

* build.gradle

```
compile('org.springframework.data:spring-data-envers')
```


## 사용 방법

### <span style="color:#006b75;">* @Audited</span>

* 변경 감지할 대상 Entity 에@Audited 어노테이션을 추가하기만 하면 된다 ! (매우 간단)
* 모든 컬럼에 대해 이력을 관리할 때는 Entity 클래스 위에 어노테이션을 달면 되지만, 특정 컬럼만 이력을 남기고 싶다면 컬럼 위에 달도록 한다.

![image](https://github.com/TonyJev93/Study/assets/53864640/2490e175-44b2-4f68-873f-92bceb4e5fcd)


<br>

### <span style="color:#006b75;">* spring.jpa.hibernate.ddl-auto : create</span>

Entity에 @Audited를 붙이면 이력관리를 위해 {Entity}_aud 라는 이력 테이블과 자동으로 매칭이 된다.
최초에는 {Entity}_aud 테이블이 형성되어 있지 않기 때문에 <b><span style="color:#0052cc;">테스트 환경</span>에서는 ddl-auto</b> 설정을 켜두는 것이 좋다.<span style="color:#e11d21;"></span>**<span style="color:#e11d21;">(운영 환경에서는 직접 테이블을 만들도록 하자 !)</span>**

* application.properties

`spring.jpa.hibernate.ddl-auto : create`
  #**{Entity}_aud 테이블**

![image](https://github.com/TonyJev93/Study/assets/53864640/fad7270e-4440-491b-91e3-7a52cb973d72)

이제 서버를 구동하면 ddl-auto에 의해 자동으로 <b><span style="color:#0052cc;">{Entity}_aud 테이블과 revinfo 테이블이 생성</span></b>된 것을 확인 할 수 있다. **<span style="color:#e11d21;">( 실제 운영환경에 적용시 오브젝트 생성 요청 필수 ! )</span>**
<strong>_aud</strong>가 붙은 테이블과 기존 테이블과 차이점은 **rev, revtype** 컬럼이 추가된다는 점이다.

* rev : 이력관리 ID
* revtype : 이력 생성 유형 ( 0 : insert / 1 : update / 2 : delete )


**# revinfo테이블**

![image](https://github.com/TonyJev93/Study/assets/53864640/3cd6158f-c81e-49a3-8dcb-7cdd9fc122e8)

**revinfo** 테이블은 아래와 같이 rev와 revstmp 컬럼으로 이루어져 있다. (중요하지 않은 테이블 같아 설명은 생략..)

<br>
실제 서버를 통해 데이터에 변경이 일어났을 때 아래와 같이 변경 이전 데이터가 쌓이고, rev 가 채번되며, revtype이 세팅되는 것을 볼 수 있다.

![image](https://github.com/TonyJev93/Study/assets/53864640/7592ba52-382a-44bf-8f8e-78d95899e376)

<br>

### <span style="color:#006b75;">* Spring Data Envers 사용해서 이력 조회하기</span>

#### Configuration 설정 (필수!)

- Spring Data Envers 를 이용하기 위해서는 아래와 같은 설정을 @SpringBootApplication 또는 @Configuration 이 존재하는 클래스에 선언해주어야 한다.

```
@EnableJpaRepositories(repositoryFactoryBeanClass = EnversRevisionRepositoryFactoryBean.class)
@SpringBootApplication
public class XXXxxxApplication { .. }

/* or */

@EnableJpaRepositories(repositoryFactoryBeanClass = EnversRevisionRepositoryFactoryBean.class)
@Configuration
public class YYYyyyConfig { .. }
```


이 후 Repository 클래스 내에<strong>RevisionRepository를 상속</strong>(extends)받아 사용한다.

```
@Repository
public interface CaregiverRepository extends ..., RevisionRepository<SampleEntity, Long, Integer> { ... }
```


<strong>RevisionRepository</strong>내 에는 아래와 같이 이력을 조회할 수 있는 Method 들이 존재한다.

```
// Entity ID를 통해, 마지막 수정데이터 조회
Optional<Revision<N, T>> findLastChangeRevision(ID id); 

// Entity ID를 통해, Entity의 이력 목록을 조회
Revisions<N, T> findRevisions(ID id); 

// Entity ID를 통해, Entity의 이력 목록(페이징)을 조회
Page<Revision<N, T>> findRevisions(ID id, Pageable pageable); 

// Entity ID와 이력 ID를 통해, 특정 Entity 이력 조회
Optional<Revision<N, T>> findRevision(ID id, N revisionNumber);
```


이제 아래와 같이 서비스 단에서 호출하여 사용하면 끝!

![image](https://github.com/TonyJev93/Study/assets/53864640/b4a946c4-b058-4141-9431-382c35d608be)

<span style="color:#e11d21;"># 주의 !</span>
- 정렬 조회 하는 경우 아래와 같이 PageRequest 클래스를 직접 생성하여 전달해야 한다.
- 또한 rev 컬럼에 대해서만 정렬이 가능하다..ㅠ

```
caregiverRepository.findRevisions(caregiverId, PageRequest.of(pageable.getPageNumber(), pageable.getPageSize(), RevisionSort.desc()))
```


## 추가 기능들

### * rev 필드 타입 INT (default) <span style="color:#e11d21;"><- 중요!</span>

* {Entity}_aud 테이블의 rev 필드는 기본적으로 INT 이기 때문에 데이터가 20억개 이상 넘어가면 오류가 발생한다
* 이를 방지하기 위해 int -> long 으로 타입변경을 해주어야 한다.

![image](https://github.com/TonyJev93/Study/assets/53864640/998a7978-ece8-4a04-9981-dfc01bc47ffa)

=> <strong>@RevisionEntity</strong>를 달고있는 Entity를 만들어, 자동으로 생성되는 <strong>rev, revtstmp 필드의 속성을 강제로 변경</strong>해준다.
=> **주의!** revtstmp 필드는 Long or Date 속성만 사용 가능하다.

### * 테이블 연관관계 처리

* Entity가 다른 Entity와 연관관계가 있는 경우

```
@Audited(targetAuditMode = NOT_AUDITED) // 상속된 엔티티의 이력 테이블은 생성하되, 변경 이력을 쌓고싶지 않은 경우
private SampleEntity sampleEntity;
@NotAudited // 상속된 엔티티의 이력 테이블 조차 생성하고 싶지 않은 경우
private SampleEntity sampleEntity;

/* ================ 양방향 관계 ================ */
@AuditMappedBy(mappedBy = “name”) 
private SampleEntity sampleEntity;

...
// (상대 클래스)
@AuditJoinTable
@OneToMany
@JoinColumn(name="sampleId")
private List<SampleEntity>sampleEntityList;
/* ========================================= */
```


* 상속받은 Entity의 이력도 같은 테이블로 관리하고 싶은 경우 <b><span style="color:#e11d21;">( 중요 ! )</span></b>

```
// 상속관계에 있는 테이블도 함께 이력관리 하고싶은 경우 클래스 상단에 추가 
// Audit 클래스도 audited 해주어야 이력테이블에 수정/생성자, 수정/생성일자 관리가 가능하다. (상속받은 class 내에 또 상속받은 class 가 있으면 그것도 같이 AuditOverride 추가해야 함.)
@Audited
@AuditOverride(forClass = AbstractAuditableEntity.class)  // modified 관련
@AuditOverride(forClass = AbstractCreatedEntity.class)  // create 관련
public class SampleClass() extends AbstractAuditableEntity{
... 
}
```


### * 변경된 필드 탐지 => @Audited(withModifiedFlag = true)

```
@Audited(withModifiedFlag = true)
public class SampleEntity { ... }
```

위와 같이withModifiedFlag = true 속성을 주게 되면

![image](https://github.com/TonyJev93/Study/assets/53864640/c8059406-c7af-45fc-8f05-ff0207caab57)

이 처럼 생성된 {Entity}_aud 테이블에서 자동으로 '필드명_mod' 라는 컬럼이 추가된다!
만약 '필드명'에 해당하는 데이터에 변경이 일어난 경우 자동으로 <strong>1</strong>(true)이 세팅된다.
이를 통해, 변경된 데이터를 손 쉽게 찾아낼 수 있다. (활용도가 높은 기능 같다.)

### * Property config 설정들

- Spring boot property 를 통해 default로 생성되는 테이블 명, 컬럼 명 그 외 기타 기능들을 설정할 수 있다.

```
spring:
    jpa:
      org:
        hibernate:
          envers:
            audit_table_suffix: _history <- 기존 이력테이블 뒤에 _aud 가 붙는 것을 사용자 지정한다.
            revision_field_name: rev_id <- 기존 이력 ID(rev)를 사용자 지정한다.
            store_data_at_delete: true  <- 데이터 삭제시(delete) 원래는 pk만 쌓고 그 외 Null이 저장되지만, 해당 설정을 통해 삭제 직전 데이터를 쌓을 수 있음.
            modified_flag_suffix: _changed  <- 기존 withModifiedFlag = true 설정 시 필드명 뒤에 _mod 붙는 것을 사용자 지정한다.
```


<br>


