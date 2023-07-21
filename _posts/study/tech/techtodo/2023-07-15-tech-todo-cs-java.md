---
title: "[Study] 기술 부채 - CS(JAVA)"
last_modified_at: 2023-07-15T23:00:00+09:00
categories:
    - interview
tags:
    - Study
    - 기술 부채
    - CAP
toc: true
toc_sticky: true
toc_label: "목차"
---

Study : 내가 부족한 기술(CS - JAVA)에 대해 정리한다.
{: .notice--info}

# JAVA

- 객체지향
- 기본형 제외하면 모든 요소가 객체로 표현됨. 캡슐화, 상속, 다형성이 잘 적용됨
- JVM 사용을 통한 운영체제 독립적으로 실행가능
- GC에 의해 자동으로 메모리 관리가 가능

## JAVA 버전별 특징

LTS 기준으로 정리 해보자.

### Java 8

- Lambda
- Stream
- Optional
- LocalDateTime
- Interface default method
- Parallel GC(default)
- 공식 지원 기간이 제일 김 (~2030년)

### Java 11

- Oracle JDK가 구독형 유료 모델로 전환
- var
- G1GC(Java 9+ default)
- Lambda에 var 사용 가능

### Java 17

- 확장된 Switch 문
- 멀티 스트링
- record
- NPE 에러 개선
- ZGC(Java 15) 등장
- M1 지원
- Sealed Classes
- 패턴 매칭 지원
- 공식 지원 나름 김(~2029년)

# JVM

- 역할
  - 스택 기반으로 동작
  - Java Byte Code를 OS에 맞게 해석해주는 역할
  - GC를 통해 자동으로 메모리 관리

# 컴파일 과정

![image](https://github.com/TonyJev93/Study/assets/53864640/4eb5f804-0e43-41e0-9214-4842b00ac4dc)

- JIT 컴파일러 : 
  - Just In Time 컴파일러 
  - 소스 코드를 실행 시점에 바로 기계어로 변환하여 실행하는 컴파일러
  - 보통의 컴파일러는 실행 전 기계어로 미리 변환하는데, JIT는 실행 중 실시간으로 필요 부분만 컴파일 하여 실행
  - 중간 코드 또는 바이트 코드로 변환 후 기계어로 변환하여 사용.(플랫폼 독립성, 빠른 실행 가능)
  - 역할은 Java 프로그램을 실행하는 동안 중간 단계의 Java 바이트 코드를 기계어로 변환하여 최적화된 원시 기계어 코드를 생성하는 것

```
JVM(Java Virtual Machine)에서 JIT(Just-In-Time) 컴파일러의 역할은 Java 프로그램을 실행하는 동안 중간 단계의 Java 바이트 코드를 기계어로 변환하여 최적화된 원시 기계어 코드를 생성하는 것입니다. 이렇게 생성된 원시 기계어 코드는 프로그램의 실행 속도를 향상시키는 데 도움이 됩니다.

JVM은 Java 언어로 작성된 소스 코드를 컴파일하여 Java 바이트 코드라는 중간 형태로 변환합니다. 이 Java 바이트 코드는 JVM이 이해할 수 있는 중간 코드이며, JVM은 Java 바이트 코드를 해석하고 실행하는 인터프리터로써 동작합니다. 인터프리터는 바이트 코드를 한 줄씩 해석하고 실행하는데, 이는 시작 속도가 빠르지만 전체적인 실행 성능은 느릴 수 있습니다.

이제 JIT 컴파일러가 등장합니다. JIT 컴파일러는 JVM의 일부로 포함되어 있으며, 인터프리터와 협력하여 프로그램의 실행 성능을 향상시킵니다. JIT 컴파일러는 인터프리터가 실행하는 중간 코드를 모니터링하고, 빈번하게 실행되는 코드 블록을 식별합니다. 그런 다음 이러한 빈번하게 실행되는 코드 블록을 원시 기계어로 컴파일하여, 최적화된 형태로 메모리에 캐싱합니다.

이렇게 최적화된 원시 기계어 코드는 이후에 인터프리터가 다시 해당 코드 블록을 만나면, 인터프리터의 해석을 거치지 않고 바로 기계어 코드를 실행하므로, 더 빠른 실행 성능을 제공합니다. 이러한 기술은 "동적 번역(Dynamic Translation)" 또는 "실행 시간 최적화(Runtime Optimization)"라고도 합니다.

JIT 컴파일러를 사용하면 Java 프로그램은 인터프리터만 사용하는 것보다 더 높은 실행 성능을 보일 수 있습니다. 특히, 프로그램이 오랜 시간 동안 실행되는 경우에는 JIT 컴파일러의 최적화 효과가 더욱 두드러집니다.
```


# 객체지향 프로그래밍이란?

- 현실 세계의 문제들을 객체로 정의하고 각 객체마다의 상태와 책임을 부여하므로써 각 객체간의 상호작용을 통해 문제를 해결해나가는 프로그래밍 방식
- 문제 해결의 중심이 기능이 아닌 객체의 역할로 봄
- 캡슐화, 상속, 다형성, 추상화

## 심화) is a, has a

- is a : A는 B이다. 할 때의 '~이다.'
- has a : A가 B를 가지고 있다.(composite type)

`is a`는 상속 하는 관계로 두 클래스간의 밀접도가 높아진다. 기저 클래스가 변경 시 상속 클래스의 손상이 발생함. 그치만 클레스 계층 구조에서 볼 때 안정적임.

`has a`는 한 클래스가 다른 클래스의 구성 요소로 들어가는 것을 의미함. 두 클래스간에 느슨한 결합을 이루고 구성 요소를 쉽게 변경할 수 있음. 유연성 제공.

> is a vs has a

은탄환은 없음. 상황에 맞게 사용

- is a : is 관계가 확실하면 사용.(ex. 사람은 인간, 고양이는 동물, 사각형은 도형, ...)
  - 강한 결합을 통해 설계가 더 단단해짐
- has a : 한 객체가 다른 객체의 부분을 갖는 경우(ex. 자동차는 베터리를 가지고 있다, 사람은 심장을 가지고 있다, ...)

# 불변 객체

- 객체 생성 후 상태 값이 변하지 않는 객체
- 원시 타입인 경우 final 사용해서 만들 수 있음.
- 참조 타입인 경우 추가적인 작업 필요
  - 객체, List, 배열 참조할 수 있음
    - 객체인 경우 사용중인 필드값 전부 final이어야 함
    - 배열의 경우 배열을 받아 copy 후 getter를 clone으로 반환하도록 하면 됨.
    - List도 마찬가지로 새 List 만든 후 값 복사하도록 함
      - 배열, 리스트는 내부를 복사해서 넘겨주는 방어적 복사(defensive copy) 사용
- 불변 객체 사용 이유
  - Thread-Safe 해서 병렬 프로그래밍에 유용, 동기화 고려하지 않아도 됨
  - 실패 원자적 메서드 사용가능. 예외 발생 전과 후가 값이 같으므로 다음 로직 처리 가능
  - 파라미터로 넘길 때 값이 변하지 않음을 보장
  - 가비지 컬렉션 성능 향상 
    - 가비지 컬렉터가 스캔하는 객체의 수가 줄어듦

# 추상 클래스 vs 인터페이스

- 추상 클래스는 클래스 내 추상 메서드가 하나 이상 존재 혹은 abstract로 정의된 경우
- 인터페이스 : 모든 메서드가 추상클래스로 이루어진 경우
- 공통점
  - new 연산자로 인스턴스 생성 불가
  - 사용을 위해 하위 클래스에서 상속해야 함
- 차이점
  - 인터페이스는 모든 메서드 구현을 강제함
  - 추상클래스는 공통적인 로직을 추상화 시키고, 기능 확장을 위해 사용
  - 추상클래스는 다중 상속 불가능, 인터페이스는 가능

# 싱글톤 패턴

- 단 하나의 인스턴스를 생성해 사용하는 디자인 패턴
- 인스턴스가 1개만 존재하는 것을 보장하고 싶을 때 사용
- 동일 인스턴스를 자주 생성해야 할 때 사용
- 대표 예 : Spring Bean 
  - 요청 때 마다 새로운 객체 생성해서 반환하는 기능도 제공 > 프로토 타입 빈, @Scope("prototype") -> IoC 컨테이너의 관리를 받지 못함.(요청 쪽에서 관리)

# [가비지 컬렉션](https://coding-factory.tistory.com/829)

- JVM 메모리 관리 기법
- 힙 영역에서 동적으로 할당됐던 메모리 영역 중 필요없어진 메모리 영역을 회수하여 메모리를 관리해주는 기법

## GC 과정

- 사용하지 않는 메모리 제거(Mark and Sweep 과정)
  - Mark and Sweep 알고리즘
    - GC 동작 원리
    - 루트에서부터 해당 객체에 접근 가능한지 여부를 메모리 해제의 기준으로 봄
      
![image](https://github.com/TonyJev93/Study/assets/53864640/d7d9f4c5-b70e-4b07-8916-8058cc511ce2)
- Makr : Root 로부터 그래프 순환하여 참조 객체 마킹
- Seep : Unreachable 객체들을 Heap 에서 제거
- Compact : 빈공간을 채워 재정렬 하여 압축(GC 종류에 따라 안할 때도 있음)

![image](https://github.com/TonyJev93/Study/assets/53864640/33658402-b6b0-4aa9-b5df-83b60c96de91)
- Eden이 꽉 찰 때마다 마이너 GC 발생하고 살아남은 객체에 대해 age-bit ++ 됨
- Eden이 꽉 찰 때마다 Survivor 0, 1을 왔다갔다 이동하면서 지정해둔 숫자 이상으로 age-bit가 넘어갈 경우 Old Generation으로 이동(이를 promotion이라고 함.) 
  - Survivor 두 개인 이유 = 외부 단편화 발생 방지(할당, 해제 반복하며 메모리 공간은 남지만 파편화되어 메모리 할당할 수 없는 문제 발생.)
- Old 영역이 허용치를 넘어서면 Major GC가 발생하며 이 때 모든 JVM이 멈추는 Stop the World가 발생.(성능에 악영향)
- 작업 재개

young 영역에 대한 minor GC, old 영역에 대한 major GC 로 구분

## GC 종류

- Serial
  - 쓰레드 한개 일 때 사용
  - 저사양일때 사용
  - 실무에서 사용 안함.
  - stop the world으로 인한 지연 심함
- Parallel
  - Java 8 디폴트
  - Serial과 알고리즘은 동일
  - Young 영역의 Minor GC를 멀티 스레드로 수행, Major GC는 여전히 싱글 스레드
  - Serial 보단 stop the world 감소 
  - GC 스레드는 기본적으로 cpu 개수만큼 할당
- Parallel Old GC (Parallel Compacting Collector)
  - Parallel 개선
  - Old 도 멀티 스레드
  - 새로운 방식의 Mark Summary Compact 사용
- CMS GC(Concurrent Mark Seep)
  - 어플리케이션 스레드와 GC 스레드를 동시에 수행하여 Stop the world 개선을 위해 고안
  - GC 과정이 복잡해짐(GC대비 CPU 사용량 높음)
  - Java9 버젼부터 deprecated, Java14 사용 중지
- G1GC(Garbage First)
  - CMS GC를 대체하기 위해 jdk 7 버전에서 최초로 release된 GC
  - Java 9+ 버전의 디폴트 GC로 지정
  - 4GB 이상의 힙 메모리, Stop the World 시간이 0.5초 정도 필요한 상황에 사용 (Heap이 너무작을경우 미사용 권장)
  - 물리적인 Young / Old 영역 나누지 않고 Region 이라는 개념 도입
  - 전체 영역을 체스같이 분할하여 상황에 따라 Eden, Survivor, Old 역할을 고정이 아닌 동적으로 부여
  - Garbage로 가득찬 부분을 청소하여 빈공간을 만듦으로서 GC 빈도를 낮추는 효과 활용
- Shenandoah GC
  - Java 12에 release
  - 기존 CMS가 가진 단편화, G1이 가진 pause의 이슈를 해결
  - 강력한 Concurrency와 가벼운 GC 로직으로 heap 사이즈에 영향을 받지 않고 일정한 pause 시간이 소요가 특징
- ZGC (Z Garbage Collector)
  - Java 15에 release
  - G1의 Region 처럼,  ZGC는 ZPage라는 영역 사용
  - G1의 Region은 크기가 고정인데 비해, ZPage는 2mb 배수로 동적으로 운영됨. (큰 객체가 들어오면 2^ 로 영역을 구성해서 처리)
  - ZGC가 내세우는 최대 장점 중 하나는 힙 크기가 증가하더도 'stop-the-world'의 시간이 절대 10ms를 넘지 않는다는 것

> G1GC 원리

- 기존처럼 메모리를 돌며 객체를 제거하지 않음
- 메모리가 많이 차있는 지역을 인식하는 기능을 이용해 해당 지역을 우선적으로 GC
- Heap 전체가 아닌 Region을 나눠 탐색하고 Region별로 GC 발생
- `Eden → Survivor0 → Survivor1` 순차적 이동 없이 더 효율적이라 생각하는 위치로 Reallocate(재할당)

> 기타
- [튜닝](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98-GC-%ED%8A%9C%EB%8B%9D-%EB%A7%9B%EB%B3%B4%EA%B8%B0?category=976278)
- 모니터링
  - `jstat -gcutil -t {process_id} {milli_seccond} {count}`
  - jstat = JDK 1.6 이후 기본으로 제공 되는 모니터링 툴
  - 'jconsole', 'jvisualvm', 'Visual GC', 'gceasy.io' 등 다양한 java  GUI GC 모니터링 분석 툴이 존재


# 자바 메모리 구조

![image](https://github.com/TonyJev93/Study/assets/53864640/a8f4934b-7f82-416f-8662-3979ab7cf01b)

- Method 영역
  - 전역 변수, static 변수, 데이터 타입, 접근 제어자 정보와 같은 필드 정보들(final class, Constant pool, static 변수)
- Heap 영역
  - new 키워드로 생성된 객체, 배열, List, ...
  - GC에 의해 주기적을 제거
- Stack 영역
  - 지역 변수, 파라미터, 리턴 값, 연산에 사용되는 임시 값
- PC 레지스터
  - Thread 생성 될 때마다 생성되는 영역, 프로그램 카운트 저장(현재 쓰레드가 실행되는 부분의 주소와 명령을 저장)
- 네이티브 메서드 스택
  - 자바 외의 언어(C, C++, 어셈블리 등)로 작성된 코드 실행할 때 사용되는 메모리 영역 (일반적으로 C 스택 사용 - 모름...)
- Permanent Heap
  - JVM에 의해 크기가 강제되던 영역
  - Java 8 부터 사라지고 Metaspace 영역(Native 메모리 영역, OS가 자동으로 크기 조절, 기존 보다 더 큰 영역 사용 가능. OOM 발생 확률 감소)으로 전환 됨.

# 각 요소들의 저장 위치

- 클래스 : 메서드
- 객체(인스턴스) : 힙
- 객체 참조 값 : 스택
- String 상수 풀(큰 따옴표) : 힙 영역
- new String : 힙 영역(substring() 같은 연산으로 새로 생긴 String도 힙)

# String vs StringBuffer vs StringBuilder

- String
  - 불변 객체
  - 스레드 안정성 제공
  - 문자열 상수 풀 사용하여 메모리 절약
  - 문자열 연산 많은 경우 성능 영향 줌
- StringBuffer
  - 가변 문자열
  - 문자열 변경 시 새로운 객체 생성 없이 기존 객체 변경
  - 쓰레드 안전(내부 메서드에서 synchronized 사용)
  - 단일 쓰레드에서는 StringBuilder에 비해 성능 떨어질 수 있음
- StringBuilder
  - 가변 문자열
  - 쓰레드 안전하지 않음
  - 단일 스레드일때 빠름

# 예외 처리

- @SuppressWarnings
  -  컴파일러에게 특정 경고를 무시하도록 지시하는 역할

# 제네릭

- Java 5 등장
- 컴파일 시 강한 타입 체크
  - 런타임 시 발견될 수 있는 잘못된 타입의 사용 문제를 컴파일 시점에 발견하여 런타임 에러를 방지하기 위함

```java
// List Integer 타입으로 제네릭 타입 적용
List<Integer> list = new ArrayList();
list.add(1);
list.add("5"); // ERROR java: incompatible types: java.lang.String cannot be converted to java.lang.Integer
```

- 불필요한 타입 변환 제거

```java
List list = new ArrayList();
list.add(1);

// 강제 형 변환 필요 
Integer number = (Integer) list.get(0);
```

# [Stream](https://zangzangs.tistory.com/171)

![image](https://github.com/TonyJev93/Study/assets/53864640/71de2100-f7c7-4cc7-8775-514a2470af74)

컬렉션, 배열 등에 저장된 요소들을 하나씩 참조하면서 코드를 실행할 수 있는 기능

- 특징
  - Stream은 데이터를 담는 저장소는 아니다. 
  - Stream은 데이터를 변경하지 않는다.
  - Stream은 재사용할 수 없다.
  - Stream은 각 요소가 1번씩 처리된다.
  - Stream은 무제한일 수도 있다. (실시간으로 계속 들어올 수 있음)
- ParallelStream
  - 데이터 병렬 처리
  - 대용량 데이터의 경우 성능이 뛰어남
  - Thread Pool을 사용하는 것이 아닌 Thread Pool 1개를 모든 ParallelStream에서 공유하는 구조
  - (주의) db, http 요청과 같은 코드를 실행하면 병목 발생

멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다.

- 명령형 vs **함수형**
  - 명령형 : 무엇을 **어떻게** 해
  - 함수형 : **무엇을** 해
- 파이프 라이닝 : 부분 스트림 연산은 스트림 연산끼리 연결해서 커다란 파이프 라인을 구성할 수 있도록 스트림 자신을 반환
  - layziness(게으름) : 최종연산이 실행되기 전까지 **중간연산(filter, sorted, map, ..)이 실행되지 않는 것**
  - short-circuiting(쇼트서킷) : 여러 개의 조건이 중첩된 상황에서 **값이 결정 나면 더 이상 불필요한 실행을 하지 않도록** 하여 실행 속도를 올리는 기법
- 내부 반복 : 리스트의 반복자 사용과 달리 자체적으로 내부 반복 지원

> vs 컬렉션

- 컬렉션
  - DVD 처럼 모든 데이터를 불러온 상태에서 연산을 수행
  - 외부 반복(for each)
- 스트림
  - 스트리밍 서비스, 필요 부분만 불러와서 수행
  - 내부 반복

> 내부 반복자

개발자는 요소 처리 코드에만 집중

멀티코어 CPU를 최대한 활용하기 위해 요소들을 분배시켜 병렬 처리 작업을 할 수 있다.

한가지 작업을 서브 작업으로 나누고, 서브 작업들을 분리된 스레드에서 병렬적으로 처리한 후, 서브 작업들의 결과들을 최종 결합하는 방법(자바는 ForkJoinPool 프레임워크를 이용해서 병렬 처리)

- [ForkJoinPool 프레임 워크](https://ict-nroo.tistory.com/43) 
  - 포크 : 데이터를 서브 데이터로 반복적으로 나누고 멀티 코어에서 서브 데이터를 병렬로 처리
  - 조인 : 서브 결과 결합하여 최종 결과 생성

![image](https://github.com/TonyJev93/Study/assets/53864640/6b3c1ab6-dcab-4c91-b2c2-ed3d24db8994)

- 포크조인풀
  - 각 코어 별 서브 요소를 처리하는 것은 개별 스레드가 해야하기 때문에 스레드 관리 필요
  - ForkJoinPool 프레임 워크는 ExecutorService의 구현 객체인 ForkJoinPool을 사용해서 작업 스레드를 관리

> 병렬 처리 성능

항상 병렬 처리가 빠른 것은 아님

- 병렬 처리 미치는 3가자ㅣ 요인
  - 요소의 수와 요소당 처리 시간
    - 스레드 풀 생성, 스래드 생성이라는 추가 비용으로 인해 적은 양에 대한 처리는 순차가 빠름
  - 스트림 소스의 종류
    - ArrayList, 배열은 랜덤 엑세스 지원(인덱스 접근) 해서 포크단계 빠름
    - HashSet, TreeSet은 요소를 분리하기가 쉽지 않고, LinkedList는 랜덤 액세스를 지원하지 않아 링크를 따라가야 하므로 역시 요소를 분리하기가 쉽지않음
  - 코어(Core)의 수
    - 싱글코어는 순차처리가 더 빠름 -> 병렬 시 스레드수만 증가하고 번갈아 수행됨
    - 코어수가 많을수록 병렬 처리 빠름

> 스트림 연산

중간연산, 최종연산으로 나뉨

- 중간연산
  - 요소들의 매핑, 필터링, 정렬
  - 다른 스트림을 반환 -> 중간연산을 연결하여 질의를 만들 수 있음 = 루프 퓨전
  - Lazy : 단말 연산 실행 전까지 아무 연산도 수행하지 않음 
- 최종연산
  - 반복, 카운트, 평균, 총합
  - 스트림 파이프라인에서 최종 결과를 도출
  - List, Integer, void 등 스트림 이외의 결과 반환

```java
scores.stream()
    .filter(g -> {
        System.out.println("filter 연산 : " + g.toString());
        return g.getGrade().equals(GRADE.GRADE4);
    })
    .map(m -> {
        System.out.println("map 연산  : " + m.toString());
        return m.getMath();
    })
    .limit(2)
    .forEach(System.out::println);
```

 중간 연산인 filter, map이 한 과정으로 병합(루프 퓨전)하여 Lazy 특성을 이용해 단말 연산 limit을 만나는 순간 수행.

forEach는 void 반환하는 최종 연산.

> [스트림 종류](https://ict-nroo.tistory.com/43)

자바 8 부터 java.util.stream 패키지에서 인터페이스 타입으로 제공

- BaseStream : 모든 스트림에서 사용할 수 있는 공통 메소드들이 정의
  - Stream 
  - IntStream
  - LongStream 
  - DoubleStream



# 참고

- [신입 개발자 기술면접 질문 정리 - 자바](https://dev-coco.tistory.com/153)