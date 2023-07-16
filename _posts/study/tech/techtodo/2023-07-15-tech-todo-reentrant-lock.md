---
title: "[Study] 기술 부채 - ReentrantLock"
last_modified_at: 2023-07-15T23:00:00+09:00
categories:
    - interview
tags:
    - Study
    - 기술 부채
    - ReentrantLock
toc: true
toc_sticky: true
toc_label: "목차"
---

Study : 내가 부족한 기술(ReentrantLock)에 대해 정리한다.
{: .notice--info}

# ReentrantLock

- lock을 소유한 쓰레드만이 unlock 가능

## vs synchronized

- 공정 잠금, 비공정 잠금 기능 제공
- 잠금 획득, 해제 시기 명시적 제어 가능
- tryLock을 통해 특정 시간동안 대기하며 락 시도 가능

## 원리

- AQS(Abstract Queued Synchronizer) 라는 동기화 기법 사용
  - 스레드의 대기 및 신호 메커니즘 제공
  - FIFO 기반의 동기화 큐를 사용하여 대기중인 스레드 관리

1. lock 획득 시도
2. lock 획득 완료 -> AQS에게 획득 알림
3. AQS : lock 카운트 +1
4. unlock 호출 -> AQS에게 해제 알림
5. AQS : lock 카운트 -1. 카운트 == 0 이면 다른 스레드에게 잠금 양도

ReentrantLock 생성시 AQS를 확장한 Sync를 생성한다. Sync는 ReentrantLock 생성자 호출 시 시 파라미터 설정에 따라 FairSync, NonFairSync로 결정된다.

Lock을 얻기 위해 tryLock()을 호출하면 Sync내 initialTrySync()를 수행하며 AQS의 속성인 state를 통해 0(미점유중), 1(점유중)의 값을 volatile 로 조회한 후 0인 경우 UnSafe를 통해 CAS 알고리즘을 사용하여 0을 1로 상태 변경 후 해당 쓰레드가 lock을 점유하였음을 기록한다.

만약 tryLock에 waitTime을 설정 하였다면 `tryLock()`이 아닌 `tryLockNanos()`가 수행되며 `initialTrySync`에 더불어 `tryAcquireNanos()`가 추가적으로 수행되어 `tryAcquire()`를 최초에 한번 시도하고 fasle를 반환받으면 Queue내에서 첫 번째 쓰레드가 되면 `tryAcquire()`를 시도하는 `acquire(...)`를 수행하여 `성공` 또는 `Thread interruptedException` 또는 `wait time이 도래`할 때 까지 무한히 lock 획득을 시도한다.

`FairSync`의 경우 `tryAcquire()`를 오버라이딩 하여 `!hasQueuedPredecessors()`를 만족할 경우 Lock을 점유하도록 구현되어 있다. `hasQueuedPredecessors`는 Queue 내에 본인 보다 앞선 Thread 가 존재하는지 확인하는 메서드로 비어있거나 본인이 제일 앞에 있을 때 `false`를 반환한다. 즉, 앞에 아무도 없을 때 Lock을 점유하겠다는 의미로 Queue가 쌓인 순서에 맞게 Lock을 점유할 수 있게 된다.