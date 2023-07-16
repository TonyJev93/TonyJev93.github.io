---
title: "[Study] 기술 부채 - DB Isolation & Lock"
last_modified_at: 2023-07-15T23:00:00+09:00
categories:
    - interview
tags:
    - Study
    - 기술 부채
    - DB
toc: true
toc_sticky: true
toc_label: "목차"
---

Study : 내가 부족한 기술(DB Isolation & Lock)에 대해 정리한다.
{: .notice--info}


# 격리 수준(Isolation)

- [참고](https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation)
- [참고](https://onduway.tistory.com/103?category=912549)

## Read Uncommitted
- Dirty Read 발생

## Read Committed
- Undo 영역 조회
- Unrepeatable Read 발생
- 일반적 DBMS가 기본 모드로 채택중
- 구현 방식
  - Lock : 읽기락 사용하여 트랜젝션 커밋 전까지 Lock 사용.(??? 읽기락이 read commit이랑 무슨 상관이지?? 다른곳에서 쓰는게 안되지 읽는건 여전히 가능한거 아닌가.) (ex. DB2, SQL Server, Sybase)
  - Undo : 쿼리 시작 시점의 Undo 데이터를 제공하는 방식 (ex. Oracle)

## Repeatable Read
- MySQL InnoDB 기본 설정
  - MVCC(Multi Version Concurrency Control) 방식 사용
    - 트랜젝션 발생 시점의 상태를 버저닝 하여 해당 트랜젝션은 그 버전의 데이터를 사용하도록 함.
- Oracle이 명시적으로 지원하지 않는 레벨. (for update로 구현은 가능)
- Phantom Read 발생
- 
## Serializable

- 후행 트랜젝션이 갱신 및 삭제 못하도록 막음. 심지어 새로운 레코드 추가 조차 못함
- 완벽히 다른 트랜젝션은 읽기 모드만 가능하도록 함
- 인덱스에 공유락

# Lock

[참고 - [DB] Lock이란?](https://chrisjune-13837.medium.com/db-lock-%EB%9D%BD%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-d908296d0279)

## Lock Level

1. Row level, 변경하려는 row에만 lock을 설정하는 것을 의미합니다.
2. Page level, 변경하려는 row가 담긴 데이터 page (또는 인덱스 페이지)에 lock을 설정합니다. 같은 페이지에 속한 row들은 변경작업과 무관하게 모두 lock에 의해 잠깁니다.
3. Tabel level, 테이블과 인덱스에 모두 잠금을 설정합니다. Select table, Alter table, Vacuum, Refresh, Index, Drop, Truncate 등의 작업에서 해당레벨의 락이 설정됩니다.
4. Database level, 데이터베이스를 복구하거나 스키마를 변경할 때 발생합니다.

## Escalation

Lock리소스가 임계치를 넘으면 Lock 레벨이 확장되는 것을 의미합니다. 

Lock 레벨이 낮을 수록 동시성이 좋아지지만, 관리해야할 Lock이 많아지기 때문에 메모리 효율성은 떨어집니다. 

반대로 Lock레벨이 높을 수록 관리리소스는 낮지만, 동시성은 떨어집니다

