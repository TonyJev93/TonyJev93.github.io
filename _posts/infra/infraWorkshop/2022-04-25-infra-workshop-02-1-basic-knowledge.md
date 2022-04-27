---
title: "[인프라 공방 5기] 2주차 - 성능 진단하기(1) : 배경지식"
last_modified_at: 2022-04-19T23:00:00+09:00
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

인프라 공방 5기 : NEXTSTEP 에서 진행하는 '인프라 공방 5기' 2주차 '성능 진단하기' 실습에 앞서 이에 필요한 배경지식을 정리한다.
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

- USE 방법론과 쓰레드 덤프 활용을 통한 서버 진단
- webpagetest, pagespeed 활용을 통한 웹 성능 예산 고민
- 성능 목표치 설정 및 부하테스트 수행 

<br>

# 1. 웹 성능 진단하기

## 1) 웹 성능 측정하기

### WebPageTest

- 접속 URL : [https://www.webpagetest.org](https://www.webpagetest.org)

**특징**
- 연결 시도 위치 및 환경 설정
- Repeat View(캐싱 등 테스트)
- 동일 조건으로 여러 차례 테스트
- 테스트 결과를 영상으로 기록

**사용법**

![image](https://user-images.githubusercontent.com/53864640/165339060-adbea074-801b-4403-b938-6a663be75dd6.png)

- 테스트 할 Site 입력
- 테스트 지역 선택(서울 선택)
- Start Test 클릭

![image](https://user-images.githubusercontent.com/53864640/165338779-31ac593a-e1d2-43ab-8470-be20a9569b6b.png)

- 테스트 결과 확인

## 2) 웹 성능 예산

### PageSpeed

- 접속 URL : [https://pagespeed.web.dev](https://pagespeed.web.dev)
- PageSpeed를 통해 구체적으로 어떤 전략을 취하면 좋을지 가이드 받을 수 있음.

![image](https://user-images.githubusercontent.com/53864640/165340655-86984b81-3f25-4cbd-943b-97ffd6a09c9b.png)

![image](https://user-images.githubusercontent.com/53864640/165340721-3cb7a2b5-26b3-4856-b706-505c5cd0c84e.png)


[3초의 법칙](https://www.thinkwithgoogle.com/intl/en-ca/marketing-strategies/app-and-mobile/mobile-page-speed-new-industry-benchmarks/)
: 3초 안에 로딩되지 않으면 53%의 사용자가 떠나고 로딩 시간이 길어질수록 사용자 이탈률이 늘어남

[웹 성능 예산](https://addyosmani.com/blog/performance-budgets/)은 정량 / 시간 / 규칙 기반으로 산정 가능하다.<br>
성능 목표에는 정답이 있지는 않지만 **3초의 법칙**이 존재하듯 서비스 성능 목표를 두는 것이 필요

- 메인 페이지의 모든 오브젝트 파일 크기는 10MB 미만으로 제한한다.
- 모든 웹 페이지의 각 페이지 내 포함된 자바스크립트 크기는 1MB를 넘지 않아야 한다.
- 검색 페이지에는 2MB 미만의 이미지가 포함되어야합니다.
- LTE 환경에서의 모바일 기기의 Time to Interactive는 5초 미만이어야 한다.
- Dom Content Loaded는 10초, First Meaningful Paint는 15초 미만이어야 한다.
- Lighthouse 성능 감사에서 80 점 이상이어야한다.

## 3) 예산 설정

### 예비 분석

- 사용자 트래픽이 많은 페이지 혹은 제품 방문 페이지 등 **가장 중요한 페이지가 무엇인지 파악**
- PageSpeed를 활용하여 Desktop, Mobile 사이트 등에서 측정된 FCP, TTI 등의 지표를 확인

### 경쟁사 분석

- 경쟁 사이트 또는 유사한 사이트의 성능을 조사
- 어떤 사이트를 봐야 할지 탐색을 위해 [Alexa](https://www.alexa.com), [similarweb](https://www.similarweb.com/), Google 검색 시 `related:`키워드 등 활용
- 연구에 따르면 사용자는 응답시간이 20% 이상 차이 날 때 인식 한다고 함. (경쟁사 대비 20% 이상 성능 차이가 나는지 확인)

### 성능 기준 설정

- 정량 / 시간 / 규칙 등을 기반으로 성능 기준을 세움
  - 정량 지표
    - 이미지 최대 크기
    - 웹 글꼴 최대 크기
    - 글꼴 최대 갯수
    - 스크립트 최대 크기
    - 스크립트 최대 갯수
    - HTML, CSS 최대 크기
    - 동영상 최대 크기
  - 시간 지표
    - FCP, LCP, TTI, TBT, CLS 등 pagespeed에서 제공하는 데이터
  - 규칙 지표
    - WebPageTest, pagespeed 등 웹 사이트에서 측정한 점수를 지표로 사용
- 모든 페이지에 대해 세우기 어려운 경우 서비스의 **중요 페이지부터 작성**
- 보편적인 성능 예산 가이드
  - 페이지로드 3초 미만
  - TTI 5초 미만
  - 압축된 리소스 최대 크기 200KB 미만

### 우선순위

- 여러 지표 중 우선순위가 무엇인지 생각하기
- ex)
  - 사용자에게 컨텐츠가 빠르게 노출되고 렌더링하는 것이 중요할 경우 -> FCP 낮게 유지
  - 사용자가 관련 링크를 빠르게 클릭해야 할 경우 -> TTI 우선순위 높아짐

<br>

# 2. 부하 테스트

현재 시스템이 어느 정도의 부하를 견딜 수 있는지, 한계치에서 병목이 발생하는 지점은 언제인지 파악하여 장애 조치와 복구를 사전에 계획해둘 필요가 있다.

## 가용성
: 시스템이 서비스를 정상적으로 제공할 수 있는 상태

장애 혹은 점검기간, 높은 부하로 인한 타임아웃 등으로 서비스를 이용할 수 없는 경우에 가용성이 낮아졌다고 함

가용성을 높이기 위해 단일 장애점(SPOF)를 없애고, 확장성 있는 서비스를 만들어야 함

- 단일 장애점(SPOF)
  - 서버 한대로 운영할 경우
    - 서버 장비 장애, 애플리케이션 서버 장애, DB 서버 장애 등의 상황에서 모두 서비스가 중단 됨
  - 이중화할 경우
    - DB 분산에 의해 동기화가 되지 않을 경우 어느 서버에 요청을 보내느냐에 따라 다른 결과 응답
  - DNS 이용한 트래픽 분산할 경우
    - DNS는 애플리케이션 서버의 상태를 확인하지 않음(애플리케이션 서버가 장애나도 해당 서버로 요청을 보냄)
    - DNS는 일반적으로 캐싱되므로 사용자가 직접 캐시를 날리지 않으면 장애가 유지 됨
  - 애플리케이션 서버만 이중화한 경우
    - DB 서버가 단일장애점(SPOF)이 됨
    - 데이터가 백업되지 않을 경우 서비스 가치 및 신뢰에 큰 손해
  - **결론**
    - 모든 요소를 다중화해야 함
- 다중화
  - 예비 운용장비로 시스템의 기능을 계속 유지 (장애내성)
  - SPOF를 없애고 무중단/고가용성 서비스를 위해 다중화 필요
  - 유휴장비 역시 비용이므로 ROI를 잘 따져야 함
  - 다중화 대상 : Server, Load Balancer, Network Device 등
  - Failover(active-passive), Replication(master-slave)
    - active : 평상시 Request를 받아 처리할 수 있도록 running 중인 시스템
    - passive(= standby, backup) : active 시스템의 장애시 Request 를 처리할 수 있는 시스템
    - master : Write 를 처리할 수 있는 Active 시스템
    - slave : Read 를 처리할 수 있는 Active 시스템

## 성능

성능상 문제 발생 시 [수익에 직접적 영향](https://web.dev/why-speed-matters/)을 준다. 즉, 느린 화면에 대한 분석이 필요하다.

부하
: 처리를 실행할 수 없어 대기중인 프로세스 수

성능 개선에는 한계가 있기 때문에 **부하**의 원인을 파악하여 제거해야 한다. 성능을 바라보는 관점은 상황에 따라 다르다.

- **사용자**(Users) : 얼마나 많은 사람들이 동시에 사용 가능한지
  - 시스템 관리자 관점 : 등록된 사용자 / 등록되지 않은 사용자
  - 서버 관점 : 로그이난 사용자 / 로그인하지 않은 사용자
  - 성능 테스터 관점 : Concurrent User / Active User
    - Concurrent User : 언제든지 부하를 줄 수 있는 사용자
    - Active User : 요청 결과를 대기하는 실제 서버에 부하를 주고 있는 사용자(성능 테스트에서의 VUser와 유사)

![img](https://techcourse-storage.s3.ap-northeast-2.amazonaws.com/510a4f766c344f588a2e8b414645c510)
_- 성능 관점의 사용자 (그림 출처 : 인프라공방 강의 자료)_
{: .text-center}

- **처리량**(TPS) : 동일 시간 내 얼마나 많은 처리가 가능한지
  - 처리량 계산 공식
    - 서비스 처리 건수 / 측정 시간
    - 요청 사용자 수 / 평균 응답시간
    - 동시 사용자 수 / 서비스 요청 간격
  - User 증가 시 TPS 는 어느 정도 증가하다가 더 이상 증가하지 않게 됨
  - Time 은 일정하게 유지되다가 점차적으로 증가
  - 부하(TPS)가 증가할 경우 지연시간이 급격히 변하는 변곡점 발생 (이는 시스템 리소스의 누수를 의심해봐야 함)
  - Time 과 달리 TPS 는 Scale out/up 을 통해 증가시킬 수 있음.
    - 성능 문제일 경우 : 단일 사용자에 대한 응답 속도가 느려짐
    - 확장성 문제일 경우 : 부하가 많아질수록 응답 속도가 느려짐

![img](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/7adc1abb8e1f429ea41cc6ad3351df72)
_- 성능 비교 (그림 출처 : 인프라공방 강의 자료)_
{: .text-center}

- **시간**(Time) : 얼마나 빠른지
  - 성능 테스트 시 지연시간이 발생하는 구간 파악 필요
  - 브라우저 - 웹 서버 구간 : 정적 파일 크기, Connection 관리, 네트워크 환경 등
  - Server 구간 : DB - 애플리케이션 연결 문제, 프로그램 로직 문제, 서버 리소스 부족 등
  - 네트워크 이슈 : 테스트 환경에 따라 다름. 출시 전 테스트를 통해 최대 응답시간 파악 및 튜닝 대상 선정 필요.

![img](https://techcourse-storage.s3.ap-northeast-2.amazonaws.com/ec32b1b96da94003af520619183bf58d)
_- 요청~응답 구간 (그림 출처 : 인프라공방 강의 자료)_
{: .text-center}

## 테스트 준비

### 목적

- 각 시스템의 응답 성능 및 한계치 확인
- 부하 발생 시 증상 확인 및 성능 개선
- 서비스 확장성 확인

### 테스트 종류

- Smoke Test
  - 최소한의 부하로 구성된 테스트 -> 테스트 시나리오 정상 동작여부 확인 
  - VUser 1~2로 구성하여 테스트
- Load Test
  - 서비스의 평소 트래픽과 최대 트래픽 상황에서 성능이 어떤지 확인. 이 때 정상 동작여부도 확인
  - 애플리케이션 배포 및 인프라 변경(scale out, DB failover 등)시 성능 변화 확인
  - 외부 요인(결제 등)에 따른 예외 상황 확인
- Stress Test
  - 서비스가 극한의 상황에서 어떻게 동작하는지 확인
  - 장기간 부하 발생 시 한계치 확인. (기능 정상동작 여부 확인)
  - 최대 사용자, 최대 처리량 확인
  - 스트레스 테스트 후 수동작업 없이 자동복구 가능 여부 확인

### 테스트 도구

- Apache JMeter, nGrinder, Gatling, Locust, K6 등 존재
- 조건
  - 시나리오 기반 테스트 가능
  - 동시 접속자 수, 요청 간격, 최대 처리량(Throughput) 등 부하를 조정할 수 있어야 함
  - 부하 테스트 서버 스케일 아웃을 지원하는 등 충분한 부하를 줄 수 있어야 함

### 주의사항

- 실제 사용자가 접속하는 환경에서 진행
  - 내부 네트워크에서 부하 발생 시 응답시간 차이 발생
- 부하 테스트는 클라이언트 내부 처리시간이 배제되어 있음을 염두
- 테스트 DB 데이터 양 = 실제 운영 DB 데이터 양
  - 통상 전체 성능의 70% 이상이 DB에 좌우 됨.
- 서버에 일정하게 발생하는 부하(별도 수행되는 배치, 후속작업 등)가 있다면 성능 테스트 시나리오에도 포함
- 외부 요인(결제 등)의 경우 시스템과 분리된 **별도의 서버**로 구성 
  - Mocking 하는 경우 Http Connection Pool, Connection Thread 등을 미사용하게 되어 IO가 발생하지 않아 테스트가 부정확 함
  - 동일 애플리케이션에 Dummy Controller를 구성한 경우 테스트 시스템의 자원과 리소스를 같이 사용하여 테스트 신뢰도가 떨어짐

### 테스트 계획하기

- **전제 조건 정리**
  - 테스트 Target 시스템 범위 정의
  - 부하 테스트에 지정될 데이터 건수, 크기 정의(서비스 이용자 수, 사용자 행동 패턴, 사용 기간 등 고려하여 계산)
  - 목푯값에 대한 성능 유지기간 정의
  - 동일 서버 내 동작하고 있는 다른 시스템, 제약 사항 파악
- **목푯값 설정**
  - 예상 1일 사용자 수(DAU) 정의
  - 피크 시간대 집중률 예상(최대 트래픽 / 평소 트래픽)
  - 1명당 1일 평균 접속 혹은 요청수 예상
  - 이를 바탕으로 Throughput 계산
    - Throughput = 1일 평균 rps ~ 1일 최대 rps
      - 1일 사용자 수(DAU) * 1명당 1일 평균 접속수 = 1일 총 접속 수
      - 1일 총 접속 수 / 86,400 (초/일) = 1일 평균 rps
      - 1일 평균 rps * (최대 트래픽 / 평소 트래픽) = 1일 최대 rps
    - Latency : 일반적으로 50~100ms 이하로 잡는 것이 좋음
    - 사용자가 검색하는 데이터 양, 갱신하는 데이터 양 등 파악

### 테스트 시나리오 대상

- **접속 빈도수가 높은 기능**
  - 홈페이지(main 페이지)
- **서버 리소스 소비량이 높은 기능**
  - CPU
    - 이미지, 동영상 변환
    - 인증
    - 파일 압축/해제
  - Network
    - 응답 컨텐츠 크기가 큰 페이지
    - 이미지, 동영상 업로드/다운로드
  - Disk
    - 로그가 많은 페이지
- **DB를 사용하는 기능**
  - 많은 리소스를 조합하여 결과를 보여주는 페이지
  - 여러 사용자가 같은 리소스를 갱신하는 페이지
- **외부 시스템과 통신하는 기능**
  - 결제 기능
  - 알림 기능
  - 인증/인가

[//]: # (<br>)

[//]: # ()
[//]: # (# 3. 서버 진단하기)

[//]: # ()
[//]: # (<br>)

[//]: # ()
[//]: # (# 4. 애플리케이션 진단하기)

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