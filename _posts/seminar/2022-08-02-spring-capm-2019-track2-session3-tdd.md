---
title: "[Spring Capm 2019] Track 2 Session 3 - 무엇을 테스트할 것인가? 어떻게 테스트할 것인가?(by. 권용근)"
last_modified_at: 2022-08-01T21:00:00+09:00
categories:
    - 세미나
    - Spring
    - Test
tags:
    - 세미나
    - Spring
    - Test
    - Spring Camp
toc: true
toc_sticky: true
toc_label: "목차"
---

세미나 : Spring Capm 2019 권용근 개발자님께서 발표하신 "무엇을 테스트할 것인가? 어떻게 테스트할 것인가?" 내용을 듣고 정리해보도록 하자.
{: .notice--info}

# 테스트로부터 얻을 수 있는 것

- 목적: **안정감**과 **자신감**
- 대상: 현재와 미래의 나, 동료

<br/>

# 무엇을 테스트할 것인가

## 1. 테스트 대상

구현을 테스트하는 것이 아닌

설계를 테스트해야 한다.

## 2. 테스트 가능한 것. 불가능한 것

![image](https://user-images.githubusercontent.com/53864640/182278179-70e0dd90-b5f5-4db2-80e9-a06b7b6472ec.png)
_[[그림 출처] - 권용근개발자님 발표영상 화면 캡쳐](https://youtu.be/YdtknE_yPk4)_
{: .text-center}

`Method Call Tree` 입장에서 테스트 불가능한 Tree 하나가 테스트 가능한 전체 Tree를 오염시킨다.

- 제어할 수 없는 영역
  - Random, Shuffle, LocalDate.now()
  - 외부 세계
    - HTTP
    - 외부 저장소
  - 테스트하기 어려운 이유: 제어할 수 없는 영역은 멱등한 결과를 보장할 수 없기 때문

항상 성공할 수 있는 것. 항상 동일한 결과가 나올 수 있는 것을 테스트 해야 한다.

<br/>

# 어떻게 테스트할 것인가

![image](https://user-images.githubusercontent.com/53864640/182278326-5946a584-fbe5-4e3d-9dbc-15fab81c4449.png)
_[[그림 출처] - 권용근개발자님 발표영상 화면 캡쳐](https://youtu.be/YdtknE_yPk4)_
{: .text-center}

테스트 불가능한 영역을 `Method Call Tree` 로 부터 격리시켜 **Boundary Layer** 로 끌어올려 테스트 불가능 영역을 최소화 한다.

적절한 **Boundary Layer**는 무엇일까?

![image](https://user-images.githubusercontent.com/53864640/182279372-ae96baa2-98de-4231-8576-9669109e2f56.png)
_[[그림 출처] - 권용근개발자님 발표영상 화면 캡쳐](https://youtu.be/YdtknE_yPk4)_
{: .text-center}

![image](https://user-images.githubusercontent.com/53864640/182279292-7143810f-5459-47da-9c2d-5f1a05f879a8.png)
_[[그림 출처] - 권용근개발자님 발표영상 화면 캡쳐](https://youtu.be/YdtknE_yPk4)_
{: .text-center}

권용근 개발자님께서는 "한 모듈로서의 의미를 지니는 가장 바깥 쪽" 이라고 표현 하셨다. (이는 명확한 정의라기 보다는 적절한 정의라고 말한다.)

<br/>

## Java. Spring Framework 

### @SpringBootTest를 꼭 해야할까?

@SpringBootTest는 Spring Context를 자동으로 올려주는 역할을 한다.

그치만 Spring Context는 느리다. 즉, 빠른 피드백을 받을 수 없다.

```java
// @Autowired의 required 속성의 기본값은 true. 즉, 해당 필드는 필수값임을 뜻 함.(필수 의존성)
@Autowired
private ShuffleStrategy shuffleStrategy;

// @Autowired 가 없다면 Java 입장에서 해당 필드는 Nullable 하다.
private ShuffleStrategy shuffleStrategy;
```

위 예제와 같이, Java에서는 아무 의미가 없지만 Spring에서는 의미를 가지게 되는 `@Autowired` 같은 어노테이션 때문에<br>
Spring Context의 오용은 **언어의 본질**을 망각하게 될 수 있다.

## 결론

Context, Framework 종속적이지 않은 테스트를 우선시 해야 한다.

<br/>

# 무엇을 Test Double로 처리해야 할까?

## Test Double

- 테스트 할 수 없는 영역에 대한 외부 요인을 부여할 수 있도록 도와주는 도구(ex. Mockito)

![image](https://user-images.githubusercontent.com/53864640/182281327-a08b3dda-dca2-416f-a07c-a85488d729b7.png)
_[[그림 출처] - 권용근개발자님 발표영상 화면 캡쳐](https://youtu.be/YdtknE_yPk4)_
{: .text-center}

- 4번 테스트를 위해 1, 2, 3번의 구현을 알아야만 한다...
- Test Double 의 남용은 **구현 테스트**로 유도할 수도 있다.
- Test Double의 대상
  - Boundary Context 까지 끌어올려진 Non-Testable
  - 순수 자바 어플리케이션으로는 테스트할 수 없는 것
    - 저장소 입출력 검증
    - SPEC 검증(내부 Controller, 외부 API)

## Embedded

![image](https://user-images.githubusercontent.com/53864640/182282117-76002f91-b614-4e24-8923-ac5cedcd73b3.png)
_[[그림 출처] - 권용근개발자님 발표영상 화면 캡쳐](https://youtu.be/YdtknE_yPk4)_
{: .text-center}

- 제어할 수 없는 영역(Spring을 통해 추상화된 부분)을 Embedded 를 활용하여 가능하도록 하자.
- 테스트와 Embedded 시스템은 동일한 라이프사이클을 갖도록 구성
  - 테스트 시작 -> Embedded 시스템 시작
  - 테스트 종료 -> Embedded 시스템 종료

### Local vs Embedded

- 테스트 정확도: Local > Embedded
- 테스트 피드백 속도: Local < Embedded
- 테스트 안정성: Local < Embedded

## Endpoint Test

- Spring Framework Support
  - MockMvc
  - REST Assured
  - WebTestClient

![image](https://user-images.githubusercontent.com/53864640/182282871-081c29f3-551f-43d8-9063-e1ac0d44dcfa.png)
_[[그림 출처] - 권용근개발자님 발표영상 화면 캡쳐](https://youtu.be/YdtknE_yPk4)_
{: .text-center}

- REST Docs: Endpoint Test + 문서화

## Spring Cloud Contract([참고](https://velog.io/@csh0034/Spring-Cloud-Contract))

- CDC(Consumer Driven Contract) 테스트를 수행할 수 있도록 지원하는 Tool
- API 검증 테스트 가능
- 어려우니 개인 보다는 팀 단위로 고려해보자

<br/>

# Tip & Rule

## 1. 테스트 코드는 상호 독립적으로 작성

모든 테스트의 순서와 관계를 생각하며 테스트를 작성하기 어렵다.

공유되는 자원은 초기화하여 다른 테스트에 영향을 받지 않도록 해야 한다.

단계가 필요하다면 JUnit5의 DynamicTest 추천

DynamicTest
: 한 라이프사이클에서 테스트 수행 가능

## 2. 테스트 안에 의도와 목적이 드러나도록 작성

```java
@Test
{
    // 어떠한 조건에서
    
    // 무엇을 수행했을 때
    
    // 어떤 결과가
}
```

```java
// 조건을 외부에 선언...
Import.sql
@Before
TestConfiguration
@Test
{
    // 어떠한 조건에서
    
    // 무엇을 수행했을 때
}

// 실제 테스트 코드 -> 의도파악하기 어려움...
@Test
{
    
    
    // 어떤 결과가
}
```

테스트 코드 역시 **가독성**이 중요하다.

## 3. 테스트 코드도 리펙토링 대상이다.

코드양에 대한 이야기가 아닌 가독, 안정성, 요구사항 정리 등 비즈니스 코드와 동일한 수준의 리펙토링이 함께 이루어져야 한다.

<br/>

# 정리

![image](https://user-images.githubusercontent.com/53864640/182284587-b856daa1-8157-44d7-a145-9cfe0a059d49.png)
_[[그림 출처] - 권용근개발자님 발표영상 화면 캡쳐](https://youtu.be/YdtknE_yPk4)_
{: .text-center}

가장 중요한 것은 **안정감과 자신감**

기준을 최대한 지키되 기준을 지키기 위해 안정감과 자신감을 포기할 필요는 없다.

<br/>

# 참고

- [[Spring Capm 2019] Track 2 Session 3 발표 영상 - Youtube](https://www.youtube.com/watch?v=YdtknE_yPk4)