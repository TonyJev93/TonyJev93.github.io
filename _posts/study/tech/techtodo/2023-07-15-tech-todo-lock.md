---
title: "[Study] 기술 부채 - Lock"
last_modified_at: 2023-07-15T23:00:00+09:00
categories:
    - interview
tags:
    - Study
    - 기술 부채
    - Lock
toc: true
toc_sticky: true
toc_label: "목차"
---

Study : 내가 부족한 기술(Lock)에 대해 정리한다.
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
