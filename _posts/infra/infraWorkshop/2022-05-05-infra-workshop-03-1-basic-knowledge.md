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