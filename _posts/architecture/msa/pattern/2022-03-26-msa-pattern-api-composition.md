---
title: "[MSA] Pattern - API Composition"
last_modified_at: 2022-03-27T14:00:00+09:00
categories:
   - Architecture
   - MSA
tags:
   - Architecture
   - MSA
   - API Composition
toc: true
toc_sticky: true
toc_label: "목차"
---

MSA : 분산된 DB 환경에서 join 을 도와주는 API 조합기에 대해 알아보자.
{: .notice--info}

# API 조합기

## 배경

- MSA 를 적용하면 Micro Service 마다의 Database 가 존재하게 된다.
- 이렇게 된 이상 분리된 여러 서비스들에게 join 쿼리를 직접적으로 호출할 수 없게 된다.

## 해결 방법

- `API Composer` 를 통해 쿼리를 구현한다.
- `API Composer` 는 `data` 를 소유하고 있는 서비스들을 호출하고, 결과에 대해 `in-memory join` 을 수행한다.

![img](https://user-images.githubusercontent.com/53864640/160265942-52a41d57-1698-4280-b062-b152a342d9ae.png){: .align-center}
_- [그림 출처](https://microservices.io/patterns/data/api-composition.html)_
{: .text-center} 

- `API Gateway`가 종종 `API Composer` 역할을 수행할 때도 있다.

## 장단점

### 장점

- 간단한 방법으로 MSA 환경에서 join 쿼리 를 수행할 수 있음.

### 단점

- 대용량 쿼리의 경우 `in-memory join` 으로 인한 성능 문제가 있을 수 있다.

### 관련 패턴들

- [분산 DB 설계](/architecture/msa/msa-distributed-db-query-design)
- [CQRS 패턴](/architecture/msa/msa-pattern-cqrs/)

## 참고

- [Pattern: API Composition](https://microservices.io/patterns/data/api-composition.html)