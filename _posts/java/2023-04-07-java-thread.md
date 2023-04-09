---
title: "[Java] Thread"
last_modified_at: 2023-04-07T23:50:00+09:00
categories:
    - Java
tags:
    - Java
    - Thread
toc: true
toc_sticky: true
toc_label: "목차"
---

JAVA : Thread 에 대해 파해쳐 보자.
{: .notice--info}

# Thread

- 자바는 멀티쓰레드를 지원
- 쓰레드라는 클래스를 통해 생성하고 관리
- 하나의 프로세스 내에서 동시에 여러 작업을 처리하기 위한 실행 단위
- 독립적으로 실행 가능한 코드 블록을 의미

## 생성방법

### Thread 클래스 상속

- run() 메서드 오버라이딩 하여 쓰레드가 실행할 코드 작성
- start() 메서드 호출하여 쓰레드 실행

```java
// Thread 클래스 상속
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("MyThread is running.");
    }
}

// Thread 클래스 실행 방법
public class Main {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
    }
}
```

### Runnable 인터페드이스 구현

- run() 메서드를 구현한 객체를 생성하여 Thread 클래스의 생성자에 전달
- start() 메서드 호출하여 쓰레드 실행

```java
// Runnable 인터페이스 구현
public class MyRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println("MyRunnable is running.");
    }
}

// Runnable 구현체 실행 방법
public class Main {
    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();
    }
}
```

## 특징

- 각각의 쓰레드는 동시에 실행되어 서로 다른 쓰레드 간에 동기화 문제 발생 가능
  - 해결 방법
    - synchronized 키워드, Lock 객체 등을 이용한 동기화 방법 제공
- 쓰레드 우선순위 설정하거나, `sleep(), wait()` 메서드 등을 이용하여 쓰레드의 실행을 일시 중지하거나 재개하는 등 기능 제공

<br/><br/>

# Thread Local

- 각 쓰레드마다 독립적인 공간을 제공하여 쓰레드 간 데이터 누수 방지 및 안정성 보장
- Spring 에서 ThreadLocal을 사용해 `트랜젝션, 인증` 등에서 사용되는 데이터를 쓰레드간에 분리
    - **트랜젝션**
      - Spring은 보통 DataSourceTransactionManager(`PlatformTransactionManager`의 구현체)를 통해 트랜잭션을 관리
      - PlatformTransactionManager는 `TransactionSynchronizationManager`를 사용
      - TransactionSynchronizationManager는 `ThreadLocal`을 사용해서 트랜잭션 처리 관련 데이터 저장
    - **인증**
      - `Spring Security` 에서는 현재 인증된 사용자 정보를 `SecurityContextHolder`를 통해 관리
      - SecurityContextHolder는 ThreadLocal를 사용해서 SecurityContext(= Authentication 관리)를 저장 관리

## 기능

- set(T value) : 현재 쓰레드의 ThreadLocal에 값을 설정
- get() : 현재 쓰레드의 ThreadLocal에서 값 반환
- remove() : 현재 쓰레드의 ThreadLocal에서 값 삭제

ThreadLocal에 저장된 객체는 Thread가 종료되지 않는 이상 메모리에서 계속 보존 됨.

특히, Thread Pool을 사용하는 경우 Thread가 작업 완료 후 ThreadPool 에 반환 되어도 Thread 가 종료되지 않기 때문에 ThreadLocal은 여전히 살아있음.

메모리 누수 방지를 위해 작업 완료 후 remove() 메서드를 반드시 호출해서 값을 삭제 해야 함.

## MDC

MDC, Mapped Diagnostic Context

로그 메시지에 특정 데이터를 쉽게 추가할 수 있도록 하여 디버깅 및 모니터링을 용이하게 해줌

ThreadLocal 을 이용하여 구현 됨

- 제공 기능
    - put(String key, String val): 키-값 쌍으로 값을 저장
    - get(String key): 키에 해당하는 값을 반환
    - remove(String key): 키에 해당하는 값을 제거
    - clear(): 모든 값을 제거

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;

public class MyClass {
    private static final Logger logger = LoggerFactory.getLogger(MyClass.class);
    
    public void doSomething() {
        MDC.put("transactionId", "1234");
        logger.info("Doing something");
        MDC.clear(); // Thread Pool 사용 시 Thread Local에 저장된 값이 재사용되는 것과 메모리에 계속 남아있는 것을 막기 위해 필수
    }
}
```


<br/><br/>

# TaskExecutor

- 쓰레드 풀을 이용하여 비동기 작업을 처리할 수 있도록 지원하는 인터페이스
- 쓰레드를 직접 생성하고 관리할 필요 없이 Spring 에서 제공하는 쓰레드 풀을 이용하여 비동기 작업 처리 가능
- 비동기 처리 결과를 Future 객체로 반환하여 작업 결과를 동기적으로 처리할 수 있도록 지원
- @Async 어노테이션을 통해 메서드를 비동기적으로 실행 가능 (단, @EnableAsync 어노테이션을 포함한 Java Config 클래스 필요)

```java
@Component
public class MyService {

  private final ThreadPoolTaskExecutor executor;

  public MyService() {
    this.executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(5);
    executor.setMaxPoolSize(10);
    executor.setQueueCapacity(100);
    executor.setThreadNamePrefix("MyThread-");
    executor.initialize();
  }

  public void doAsyncTask() {
    executor.execute(() -> {
      // 비동기 작업 코드
    });
  }
}
```

## InterruptedException

Java 멀티스레딩 환경에서 사용되는 Exception 중 하나

Thread가 블로킹 되어 있을 때 interrupt() 메서드가 호출되어 해당 Thread를 깨울 때 발생할 수 있다.

Thread가 블로킹 된 상태에서 interrupt() 메서드가 호출되면 Thread는 InterruptedException을 발생시키며,

이는 일반적으로 Thread를 종료시키거나 다른 처리를 진행하도록 한다.

InterruptedException은 sleep(), wait(), join()과 같은 Thread의 블로킹 메서드 호출 시 발생할 수 있으며,

해당 Exception이 발생하면 Thread 상태가 변경될 수 있기 때문에 InterruptedException을 발생시키는 메서드를 호출할 때는 반드시 try-catch 블록으로 예외처리를 해줘야 한다.
