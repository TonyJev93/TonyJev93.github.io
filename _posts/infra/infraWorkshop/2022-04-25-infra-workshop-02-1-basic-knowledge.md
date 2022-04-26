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

[//]: # (# 2. 부하 테스트)

[//]: # ()
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