---
title: "[인프라 공방 5기] 3주차 - 안정적 인프라 만들기 : 배경지식"
last_modified_at: 2022-05-06T23:50:00+09:00
categories:
    - Infra
    - NEXTSTEP
    - 인프라 공방 5기 
tags:
    - Infra
    - NEXTSTEP
    - 인프라공방 5기
toc: true
toc_sticky: true
toc_label: "목차"
---

인프라 공방 5기 : NEXTSTEP 에서 진행하는 '인프라 공방 5기' 3주차 '안정적 인프라 만들기' 실습에 앞서 배경지식에 대한 내용을 정리한다. 해당 내용들은 인프라공방 강의자료에서 발췌하였다.
{: .notice--info}

---

<div style="
  display: flex;
  justify-content: space-between;
">
  <a href="/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/">⬅️ 인프라 공방 5기 목차보기</a>
</div>

---

# 학습 목표

- HTTP 개선에 따른 차이를 이해하고 Reverse Proxy 성능 개선을 해본다.
- HTTP Cache 전략을 이해하여 적절한 정책을 설정해본다.
- 쿼리를 최적화하여 조회 성능을 개선해본다.
- 인덱스를 설정하여 조회 성능을 개선해본다.

<br>

# 리버스 프록시 개선하기

웹 성능 진단을 하고 보면 개선할 부분은 아래와 같이 분류 된다.

- 불필요한 다운로드 제거
- 불필요한 작업을 지연로딩
- 다양한 압축 기술을 통해 각 리소스의 전송 인코딩을 최적화
- 스크립트 병합하여 요청수 최소화
- 스크립트 크기를 최소화하여 패킷 크기 자체를 줄임
- 웹 프로토콜 최적화
- 캐싱을 활용하여 요청 수 최소화
- 애플리케이션 로직 개선
- 데이터베이스 SQL 최적화로 디스크 I/O 개선

## nginx web server 특징

- worker 프로세스 / 싱글 스레드 채택을 통한 Context Switch overhead 발생 X
- 비동기 처리로 인해 적은 메모리 사용량으로 동시성 보장
- 프록시 외에도 로드밸런싱과 캐시 기능 제공

싱글 프로세스 스레드로 이벤트 구동에 의한 Non-Blocking 처리로 처리속도가 매우 빠르다. 

그러나 실제 데이터를 읽고 쓰는것은 OS(커널) 내에 시스템 호출 프로그램과 하드웨어 사이에서 실행되므로, 해당 처리가 너무 길어지면 시스템 호출 큐에 요청이 쌓이게 되어 성능이 저하될 수 있다.

따라서 CPU 자원에 대한 사용량보다는 네트워크 자원에 대한 의존도가 높아, **클라이언트와의 커넥션을 어떻게 효율적으로 관리할 것인지가 성능 튜닝의 포인트**다.

## HTTP1.1 성능 개선

keep alive
: 한번 맺은 세션을 요청이 끝나더라도 유지하는 기능

nginx는 기본적으로 keepalive 설정이 되어 있음.

1. keepalive timeout 시간이 지나면 서버에서 Keepalive 확인 패킷 전송
2. 해당 패킷에 대한 응답을 받으면 타이머는 원래 값으로 돌아가서 다시 카운트 진행
3. 응답 받지 못했을 경우, tcp_keepalive_intvl에 정의된 시간만큼 경과한 후 요청을 다시 보냄. (이 때, tcp_keepalive_probes에 정의된 횟수만큼 보냄)
4. 이 후에도 응답이 없을 경우, 클라이언트는 연결이 끊어졌다고 인지하여 서버에 RST(Reset) 패킷을 보낸 다음 자신의 소켓을 닫는 것으로 연결을 종료

이로 인해 Keepalive를 사용할 경우 연결이 끊어졌음에도 FIN 패킷을 받지 못해 정리되지 않고 남아있던 좀비 커넥션을 없애는 효과도 발생한다.

Nginx 최대 요청 수(keepalive_requests)는 기본 100개로 설정되어 있고, timeout 시간은 75초로 설정되어 있다.

추가적으로, 클라이언트 입장에서는 커널 파라미터를 사용하여 TIME_WAIT 소켓을 재사용할 수 있다.

불필요한 TCP 3way handshake가 일어날 수 있으므로 요청할 때마다 소켓을 새로 연결하는 방식(Connectionless)가 아닌, 미리 소켓을 열어놓고 처리하는 방식(Connection Pool)으로 해결한다.

```shell
$ netstat -napo
```

![img](https://user-images.githubusercontent.com/53864640/167155986-76ac5e38-23d9-4874-abfe-637493e83a47.png)

```shell
# I option : 응답에 Content 만 출력하지 않고 서버의 Reponse 도 포함해서 출력한다. (디버깅에 유용)
$ curl -I edu.nextstep.camp
```

<img width="353" alt="image" src="https://user-images.githubusercontent.com/53864640/167156244-3e9d4c4d-3038-49ec-85e2-47ddca1e9561.png">

### TCP Keep-alive vs nginx HTTP Keep-alive

- TCP keepalive
  - 서버간에 ACK 패킷을 보내 세션 테이블이 지워지지 않고 계속 세션 정보를 유지
  - mq, kafka 등 TCP 기반의 서비스들을 대상으로 지속적 연결을 유지해야 하는 경우 사용
- HTTP Keep-alive
  - 일정 시간이 지나면 nginx 등의 서버가 능동적으로 연결을 끊음
  - Apache, Nginx 등 웹 애플리케이션에서 설정된 기간까지 최대한 연결을 유지하기 위해 사용

## HTTP 2.0

### HTTP 2.0에 대한 요구사항

![img](https://techcourse-storage.s3.ap-northeast-2.amazonaws.com/80d9210dd96c44d7be0bd575c41e3759)

- 기존 스펙 문서를 활용하여 HTTP 메서드, 상태 코드, URI, 헤더 필드를 비롯한 HTTP 1.1의 기본 틀은 유지 해야함
- TCP를 사용하며, 대부분의 경우 HTTP 1.1 보다 대폭적으로 사용자단의 레이턴시를 개선해야 함
- 병렬화를 위해 서버에 다수의 커넥션을 요구하지 않고 혼잡 제어에 있어서 TCP 사용 효율을 높여야 한다.
- HTTP1.1의 문제점인 HOL(Head of Line Blocking) 블로킹을 해결해야 한다. 

<br>

## Contents-encoding

- 텍스트 기반 자산의 인코딩 및 전송 크기 최적화
- 텍스트 파일(js, css, html)의 압축률은 70~80%

### GZIP 을 사용한 텍스트 압축

- GZIP은 텍스트 기반 자산인 CSS, 자바스크립트, HTML에서 최상의 성능을 보임
- 모든 최신 브라우저는 GZIP 압축을 지원하고 이를 자동으로 요청할 수 있음
- 서버는 GZIP 압축을 활성화하도록 구성해야 함
- 일부 CDN의 경우 특별히 주의하여 GZIP이 활성화되었는지 확인해야 함

<br>

# 캐싱 활용하기

## HTTP Cache

- 참고 : [HTTP Cache로 불필요한 네트워크 요청 방지](https://web.dev/http-cache/)

<br>

# MySQL 최적화 대상

대부분의 웹 애플리케이션의 주 작업은 DB 데이터 조회와 저장이다. 통상 서버 처리시간의 70% 이상은 SQL을 처리하는데 사용되곤 한다.

따라서 안정적인 서비스를 운영하기 위해 DB를 최족화할 필요가 있다.

## 조인문

```sql
SELECT * FROM Products 
JOIN OrderDetails ON Products.ProductID = OrderDetails.ProductID 
WHERE Products.ProductID IN (1,30)
```

![img](https://techcourse-storage.s3.ap-northeast-2.amazonaws.com/06cb1909d5964b1796aa111c1c70941a)
_출처 : [인프라 공방 5기](https://edu.nextstep.camp/c/VI4PhjPA/) - 강의자료_
{: .text-center}

- ProductID 1과 30을 검색하기 위해 Product 테이블을 먼저 찾는다.
- 테이블에 동시 접근이 불가능하여 위 그림 처럼 먼저 접근하는 테이블을 **드라이빙 테이블**, 뒤늦게 검색되는 테이블을 **드리븐 테이블**이라고 한다.
- 가능하면 적은 결과가 반환될 것 같은 테이블을 드라이빙 테이블로 선정하면 좋다. 드라이빙 테이블의 추출 건수는 곧 드리븐 테이블의 엑세스 반복 횟수이기 때문이다.

## DB 최적화 대상


<img width="574" alt="image" src="https://user-images.githubusercontent.com/53864640/167648418-6b0c1c40-5309-431f-8f17-c56bd4095674.png">{: .align-center}

### Client

- 호출 횟수를 줄인다.
  - 복수 건의 레코드를 한번의 호출로 집합 처리
  - 두 개 이상의 쿼리를 한 쿼리로 통합 처리
- JDBC Statement는 쿼리 문장 분석, 컴파일, 실행의 단계를 캐싱한다. PreparedStatement는 처음 한 번만 세 단계를 거친 후 캐시에 담아서 재사용한다.
- DB Connection Pool을 사용하여 객체를 생성하는 부분에서 발생하는 대기시간을 줄이고 네트워크 부담을 줄일 수 있다. 
- Fetchsize 조정하거나 Paging을 활용한다.

### Database Engine

- 파일시스템에 저장된 데이터가 조회되면 해당 데이터를 메모리에 저장해 이후 동일 데이터 조회 시 파일시스템의 물리적인 입출력이 발생하지 않도록 한다.
- 서버 파라미터를 튜닝한다.

### Filesystem

- SSD 사용
- SQL을 최적화하여 필요 이상의 데이터 블록을 읽는 것을 방지(즉, SQL 튜닝이란 읽는 블록 수를 줄여주는 것을 의미)

Spring Data Access, Mysql5.7 이상, Public Cloud 를 활용한다면 상당 부분 최적화되어 있을 것이다.<br>
우리는 SQL 최적화와 몇가지 DB 서버 튜닝에 집중하면 된다.(참고 > [쿼리 최적화 첫걸음 - 7가지 체크리스트](https://medium.com/watcha/%EC%BF%BC%EB%A6%AC-%EC%B5%9C%EC%A0%81%ED%99%94-%EC%B2%AB%EA%B1%B8%EC%9D%8C-%EB%B3%B4%EB%8B%A4-%EB%B9%A0%EB%A5%B8-%EC%BF%BC%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%9C-7%EA%B0%80%EC%A7%80-%EC%B2%B4%ED%81%AC-%EB%A6%AC%EC%8A%A4%ED%8A%B8-bafec9d2c073))

## SQL 최적화 대상

실행계획을 확인할 경우 어떻게 수행되는지 알아본다.<br>
(참고 > [MySQL 내부 구조](https://brunch.co.kr/@jehovah/21))

### 쿼리 동작 방식

![img](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/09610baf606b4854b273a8ffc452b009)
_출처 : [인프라 공방 5기](https://edu.nextstep.camp/c/VI4PhjPA/) - 강의자료_
{: .text-center}

- **Query Caching** : (Key, Value) = (SQL문, 쿼리의 실행결과) 인 Map
  - [캐시 확인 절차]
    - 요청 쿼리가 **Query cache**에 존재하는가?
    - 해당 사용자가 그 결과를 볼 수 있는 **권한**이 있는가?
    - **트랜잭션 내**에서 실행된 쿼리인 경우 **가시 범위 내**에 있는 결과인가?
    - 호출 시점에 따라 **결과가 달라지는 요소**(RAND, CURRENT_DATE 등)가 있는가?
    - 캐시가 만들어지고 난 이후 해당 데이터가 **다른 사용자에 의해 변경**되지 않았는가?
    - 쿼리 결과가 **캐시하기에 너무 크지 않은가**?
    - 다만, 데이터가 변경되면 모두 삭제해야 하는데 이는 **동시 처리 성능 저하를 유발**하고 **많은 버그의 원인**이 되어 MySQL 8.0으로 올라오면서 제거되었음.
- **Parsing** : 사용자로부터 요청된 SQL을 잘게 쪼개서 서버가 이해할 수 있는 수준으로 분리
- **Preprocessor** : 해당 쿼리가 문법적으로 틀리지 않은지 확인하여 부정확하다면 여기서 처리를 중단 (일괄처리(batch) 내에 있다면 일괄처리 전체를 중단)
- **Optimization** : 실행계획(Exception Plan)
  - 쿼리 분석: Where 절의 검색 조건인지 Join 조건인지 판단
  - 인덱스 선택: 각 테이블에 사용된 조건과 인덱스 통계 정보를 이용해 사용할 인덱스를 결정
  - 조인 처리: 여러 테이블의 조인이 있는 경우 어떤 순서로 테이블을 읽을지 결정
- **Handler** (Storage Engine)
  - MySQL 실행엔진의 요청에 따라 데이터를 디스크로 저장하고 디스크로부터 읽어오는 역할을 담당
  - MySQL 엔진에서는 Storage Engine 으로부터 받은 레코드를 조인하거나 정렬하는 작업을 수행

### Index Range Scan / Table Full Scan

- **Sequential access**
  - 물리적으로 인접한 페이지를 차례대로 읽는 순차 접근 방식
  - 인접한 페이지를 여러 개 읽는 다중 페이지 읽기 방식으로 수행
- **Random access**
  - 물리적으로 떨어진 페이지들에 임의로 접근하는 임의 접근 방식
  - 정해진 순서없이 이동하는 만큼 디스크의 물리적인 움직임이 필요하고 다중 페이지 읽기가 불가능해 데이터의 접근 수행 시간이 오래 걸림
  
- DB 테이블에서 데이터 찾는 방법
  1) 테이블 전체 스캔(Table Full Scan)
  2) 인덱스 이용
  
**Table Full Scan**은 **Sequential access**와 Multi block I/O 방식으로 디스크를 읽어 한 블록에 속한 모든 레코드를 한번에 읽어들이는데 반해,<br>
**Index Range Scan**은 **Random access**와 Single Block I/O로 레코드 하나를 읽기 위해 매번 I/O가 발생한다.<br>
따라서 읽을 데이터가 일정량을 넘으면 인덱스보다 Table Full Scan이 유리하다. 즉, **인덱스**는 **큰 테이블에서 소량 데이터를 검색**할 때 사용합니다.

**OLTP 시스템에서는 소량 데이터를 주로 검색**하므로 인덱스를 효과적으로 활용하는 것이 중요하다. (기본적으로 사용하는 NL(Nested Loops) Join도 인덱스를 이용한 조인이다.)<br> 
**대량 데이터를 빠르게 처리**하려면, 인덱스와 NL 조인보다 **Table Full Scan과 해시 조인이 유리**합니다. Table Full scan 비용은 파티션 활용 전략과 병렬처리로 줄일 수 있습니다.

결론, Index 사용 시 Random I/O 횟수 줄이는 것이 목표!

### 수직 / 수평 탐색

인덱스 탐색과저은 스캔 시작시점을 찾는 **수직적 탐색**과 데이터를 찾는 **수평적 탐색**으로 나뉜다.<br>
이에 대해 MySQL InnoDB 기준 기본 인덱스 구조인 B-Tree 기준으로 설명하도록 한다.

<img width="588" alt="image" src="https://user-images.githubusercontent.com/53864640/167659315-2b63b992-cf5c-493a-ab32-5ebf3dee3eb2.png">{: .align-center}
_출처 : [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/create-index.html)_
{: .text-center}

- **수직적 탐색**
  - 인덱스 수직적 탐색은 **루트 노드에서부터 시작**한다.
  - **루트 노드와 브랜치 노드**는 인덱스 키와 자식 노드 정보로 구성된 **페이지**(단위)이다.
  - 수직적 탐색 과정에 찾고자 하는 값보다 **크거나 같은 값을 만나면**, 바로 **직전 레코드가 가리키는 하위 노드로 이동**한다.
  - InnoDB의 경우 Secondary Index를 통해 알아낸 Primary Key로 한번 더 수직적 탐색이 이루어진다.
- **수평적 탐색**
  - 수직적 탐색을 통해 스캔 시작점을 찾았으면 수평적 탐색을 통해 데이터를 찾는다.
  - 인덱스 리프 노드끼리는 양방향 Linked List이므로 서로 앞뒤 블록에 대한 주소값을 갖는다.
  - 필요한 컬럼을 인덱스가 모두 갖고 있어 인덱스만 스캔하고 끝나는 경우도 있지만, 그렇지 않을 경우 테이블도 액세스해야 한다.
  - 이 때, ROWID가 필요하며, ROWID는 `데이터블럭 주소 + 로우 번호(블록내 순번)`로 구성된다.
  - InnoDB의 경우, Primary Key 가 ROWID 역할을 합니다.
  - Primary Key는 Clustered Index, 즉 순차적으로 저장되어 있어 Data record의 물리적인 위치를 알 수 있기 때문이다.
  - ROWID가 가리키는 데이터 페이지를 버퍼풀에서 먼저 찾아보고 못찾을 때만 디스크에서 블록을 읽는다. (읽은 후에는 버퍼풀에 적재.)

일단, 강의노트에 있는 내용을 적어보았지만 현재 나의 지식으로는 이해하기 어렵다...(나중에 공부를 더 해보는 걸로)

### 인덱스 튜닝

인덱스 튜닝에는 크게 **인덱스 스캔 효율화**와 **랜덤 액세스 최소화** 두가지가 있다. 

후자는 인덱스 스캔 후 테이블 레코드를 액세스할 때, 랜덤 I/O 횟수를 줄이는 것을 의미한다. 학생 명부를 뒤지는 과정에서의 비효율보다, 학생 명부에 없는 정보를 위해 직접 교실에 가는 부담이 더 크듯, 랜덤 액세스 최소화 튜닝이 더 중요하다.

### 인덱스 손익분기점

Table Full Scan의 성능은 **시퀀셜 액세스 방식**이기 때문에 데이터 수(1건 이나 1000만 건 이나)에 상관없이 일정하게 유지된다.<br>
반면, 인덱스(ROWID)를 이용한 테이블 액세스는 **랜덤 액세스 방식**이기 때문에  점점 느려진다.

테이블 데이터가 10~100만건 이내의 경우 조회 건수가 5~20% 가량에서 손익분기점이 형성된다.
조회건수가 늘수록 데이터를 버퍼캐시에서 찾을 가능성이 낮아지기 때문에, 1000만건 중 100만건 이상 액세스한다면 캐시 히트율은 극히 낮을 수밖에 없다.<br>
게다가 1000만건 정도 테이블이면 클러스터링 팩터도 낮을 가능성이 높다.

## 성능 개선 대상 식별하기

```sql
## 프로세스 목록
SHOW PROCESSLIST;

## 슬로우 쿼리 확인
SELECT query, exec_count, sys.format_time(avg_latency) AS "avg latency", rows_sent_avg, rows_examined_avg, last_seen
FROM sys.x$statement_analysis
ORDER BY avg_latency DESC;

## 성능 개선 대상 식별
SELECT DIGEST_TEXT                                                      AS query,
       IF(SUM_NO_GOOD_INDEX_USED > 0 OR SUM_NO_INDEX_USED > 0, '*', '') AS full_scan,
       COUNT_STAR                                                       AS exec_count,
       SUM_ERRORS                                                       AS err_count,
       SUM_WARNINGS                                                     AS warn_count,
       SEC_TO_TIME(SUM_TIMER_WAIT / 1000000000000)                      AS exec_time_total,
       SEC_TO_TIME(MAX_TIMER_WAIT / 1000000000000)                      AS exec_time_max,
       SEC_TO_TIME(AVG_TIMER_WAIT / 1000000000000)                      AS exec_time_avg_ms,
       SUM_ROWS_SENT                                                    AS rows_sent,
       ROUND(SUM_ROWS_SENT / COUNT_STAR)                                AS rows_sent_avg,
       SUM_ROWS_EXAMINED                                                AS rows_scanned,
       DIGEST                                                           AS digest
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC;

##  I/O 요청이 많은 테이블 목록
SELECT *
FROM sys.io_global_by_file_by_bytes
WHERE file LIKE '%ibd';

## 테이블별 작업량 통계 
SELECT table_schema,
       table_name,
       rows_fetched,
       rows_inserted,
       rows_updated,
       rows_deleted,
       io_read,
       io_write
FROM sys.schema_table_statistics
WHERE table_schema NOT IN ('mysql', 'performance_schema', 'sys');


## 총 메모리 사용량 확인
SELECT *
FROM sys.memory_global_total;

## 스레드별 메모리 사용량 확인
SELECT thread_id, user, current_allocated
FROM sys.memory_by_thread_by_current_bytes
LIMIT 10;

## 최근 실행된 쿼리 이력 기능 활성화
UPDATE performance_schema.setup_consumers
SET ENABLED = 'yes'
WHERE NAME = 'events_statements_history'

UPDATE performance_schema.setup_consumers
SET ENABLED = 'yes'
WHERE NAME = 'events_statements_history_long';

## 최근 실행된 쿼리 이력 확인
SELECT *
FROM performance_schema.events_statements_history;
```

## DB 서버 튜닝

### 메모리 튜닝

- Thread
  - MySQL은 커넥션마다 하나의 Thread를 생성하여 요청을 처리한다.
  - Thread를 메모리에 할당하고 해제하는데 비용이 크기 때문에 이를 줄여야 한다.

```sql
## 현재 쓰레드(연결) 개수 확인
SELECT * FROM performance_schema.threads;

SHOW STATUS LIKE '%THREAD%';
```

thread_cache_size는 지나치게 높여둘 필요는 없으며 일반적으로 threads_connected의 피크 치보다 약간 낮은 수치 정도를 설정하는 것이 좋다. 

이를 통해 쓰레드가 생성되고 소멸되면서 겪게 되는 메모리, 각종 자원, 시간 등의 낭비를 줄일 수 있습니다.

- Caching
  - 버퍼는 MySQL 내부적으로 하나만 확보되는 Global Buffer와 Thread(Connection)별로 확보되는 Thread Buffer가 있다.
  - Thread Buffer에 많은 메모리를 할당하면 성능이 올라가지만, 설정값 * Connection 수만큼 확보하므로 Connection이 갑자기 늘어나면 메모리가 부족해져 swap이 발생할 수 있다.
  - **innodb_buffer_pool_size**
    - InnoDB의 데이터나 인덱스를 캐시하기 위한 메모리상의 영역
    - 글로벌 버퍼이므로 크게 할당할 것을 권한다.
    - 보통 시스템 전체 메모리의 80% 수준으로 설정한다.(최대 512MB)
  - **key_buffer_size**
    - 인덱스를 메모리에 저장하는 버퍼 크기를 의미
    - 보통 총 메모리 크기의 25% 정도를 설정
    - Key Buffer 사용률 = 1 - (Key_reads/Key_read_requests) * 100 (90% 이상일 경우 key_buffer_size 가 효율적으로 설정되어 있다고 판단)

```sql
SHOW STATUS LIKE '%key%';
```

### 커넥션 튜닝

```shell
mysql> SHOW VARIABLES LIKE '%max_connection%';
mysql> SHOW STATUS LIKE '%CONNECT%';
mysql> SHOW STATUS LIKE '%CLIENT%';
```

- connect_timeout
  - MySQL이 클라이언트로부터 접속 요청을 받은 경우 몇 초까지 기다릴지 설정하는 변수
  - 기본 값은 5초이며 일반적으로 수정할 필요는 없다.
- Interactive_timeout
  - 'mysql>'과 같은 콘솔이나 터미널 상에서의 클라이언트 접속을 의미
  - 기본 값으로 8시간이 잡혀 있으나 1시간 정도로 낮추는 것이 좋다.
- wait_timeout
  - 접속 후 쿼리가 들어올 때 까지 기다리는 시간
  - 접속이 많은 DBMS에서는 이 값을 낮추어 sleep 상태의 Connection 들을 정리하여 전체 성능을 향상시킬 수 있다.
  - 하지만 값을 너무 낮추게 되면 잦은 지나치게 커넥션이 발생할 수 있어 보통 15~20 사이의 값을 설정한다.
  - Aborted client는 2% 아래인 것이 바람직한 상태이다.
- max_connections
  - 서버가 허용하는 최대한의 커넥션 수
  - 서버의 사양에 따라 달라질 수 있으며 일반적으로 120~250개 정도로 설정
  - 하지만 접속이 많고 고용량 서버의 경우 1000개 정도의 높은 값을 설정하는 것도 가능하다.
  - Too many connection 에러가 발생하지 않도록 적절한 값을 설정하는 것이 중요
- back_log
  - max_connection 설정값 이상의 접속이 발생할 때 얼마만큼의 커넥션을 큐에 보관할지에 대한 설정 값
  - 기본 값은 50이며 접속이 많은 서버의 경우 이 값을 늘릴 필요가 있다.

<br>

# 참고

- [인프라 공방 5기](https://edu.nextstep.camp/c/VI4PhjPA/) - 강의자료

<br>

---

<div style="
  display: flex;
  justify-content: space-between;
">
  <a href="/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/">⬅️ 인프라 공방 5기 목차보기</a>
</div>

---