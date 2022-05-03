---
title: "[인프라] 웹 성능 예산"
last_modified_at: 2022-05-03T23:00:00+09:00
categories:
    - Infra
    - 웹 성능 예산
tags:
    - Infra
    - 웹 성능 예산
toc: true
toc_sticky: true
toc_label: "목차"
---

웹 성능 예산 : 인프라 성능 진단을 위해 사용되는 웹 성능 예산에 대한 내용을 정리한다.
{: .notice--info}

# 웹 성능 예산

- 진단 사이트 : [PageSpeed](https://pagespeed.web.dev)

![image](https://user-images.githubusercontent.com/53864640/166457424-d9366018-6833-4390-894c-e8676ba441c1.png)

![image](https://user-images.githubusercontent.com/53864640/166460521-7f35d8ff-d962-4a67-9461-0e5a0c3a4f3e.png)

진단 결과로 다양한 성능 예산에 대한 지표들이 나오고 추천하는 성능 개선 방법에 대해서도 제안해준다.

그럼 진단하는 사이트는 알았으니 이제 각 성능 예산 지표들에 대해 알아보자.

<br>

# FCP

## FCP 란?

- FCP : First Contentful Paint

FCP 는 페이지가 로드되기 시작한 시점부터 페이지 콘텐츠의 일부가 화면에 렌더링되는데 까지 걸리는 시간이다.

![image](https://user-images.githubusercontent.com/53864640/166457993-e0e5a7c9-ab36-4f4e-8ba1-1480e694e29e.png)

해당 메트릭에서의 콘텐츠는 텍스트, 이미지, `<svg>` 요소 등 흰색이 아닌 `<canvas>` 요소를 뜻한다. 즉, 전체가 아닌 일부 콘텐츠만 렌더링 되는 시간을 FCP 라고 한다.

## 높은 점수의 FCP

- 좋은 사용자 경험을 제공하기 위해 **FCP**는 **1.8초 이하**여야 한다. 

## 개선 방법

- 렌더링 차단 리소스 제거
- CSS 축소
- 사용하지 않는 CSS 제거
- 필요한 원본에 사전 연결
- 서버 응답 시간 단축(TTFB)
- 여러 페이지 리디렉션 방지
- 핵심 요청 사전 로드
- 막대한 네트워크 페이로드 방지
- 효율적인 캐시 정책으로 정적 자산 제공
- 과도한 DOM 크기 방지
- 크리티컬 요청 깊이 최소화
- 웹폰트 로드 중에 텍스트가 계속 표시되는지 확인
- 요청 수를 낮게 유지하고 전송 크기를 작게 유지

<br>

# TTI

## TTI 란?

- TTI : Time To Interactive

TTI는 페이지가 완전히 상호작용하는데 까지 얼마만큼의 시간이 걸리는지에 대한 측정값이다.

기술적으로, 완전히 웹 페이지와 상호작용한다는 것은 다음을 뜻한다.

- FCP(browser에 이미지, 텍스트, 흰색이 아닌 `<canvas>`가 랜더링 되는 시간) 발생
- 지난 5초 동안 **긴 Javascript 작업**(실행 시 브라우저를 50밀리초 이상 잠그는 스크립트)이 발생하지 않는 경우
- 동시에 두 개 이상의 진행 중인 GET 요청이 발생하지 않는 경우 

## 점수 측정

- Lighthouse 기준
  - 0 ~ 3.8 초 — Green (빠름)
  - 3.9 ~ 7.3 초 — Orange (보통)
  - 7.3 초 이상 — Red (느림)

## 낮은 점수 원인

- 네트워크 페이로드가 너무 큰 경우
- 페이지 로드 시 자바스크립트가 너무 많은 경우
- 네트워크 요청이 동시에 너무 많은 경우

## 개선 방법

- 메인 스레드 작업 최소화
- 자바스크립트 실행 시간 단축
- 최적화된 이미지 지연 로드

<br>

# Speed Index

## Speed Index 란?

Speed Index는 페이지 로드 중에 콘텐츠가 시각적으로 표시되는 속도를 나타내는 측정값이다.

Speed Index를 측정하기 위해 브라우저에서 페이지가 로드되는 비디오를 캡처하고 프레임 간의 시각적인 진행사항들을 비교한다.

그런 다음 측정기는 특정 모듈을 사용하여 Speed Index 점수를 생성한다.

## 점수 측정

- Lighthouse 기준
    - 0 ~ 3.4 초 — Green (빠름)
    - 3.4 ~ 5.8 초 — Orange (보통)
    - 5.8 초 이상 — Red (느림)

## 개선 방법

- 메인 스레드 작업 최소화
- 자바스크립트 실행 시간 단축
- 웹폰트 로드 중에 텍스트가 계속 표시되는지 확인

<br>

# LCP

## LCP 란

- LCP : Largest Contentful Paint

LCP 는 viewport 에서 가장 큰 컨텐츠 요소가 화면에 렌더링되는 시간을 측정한 값이다.

주요 컨텐츠가 사용자에게 표시되는 시간이라고 볼 수 있다.

## 점수 측정

- Lighthouse 기준
    - 0 ~ 2.5 초 — Green (빠름)
    - 2.5 ~ 4 초 — Orange (보통)
    - 4 초 이상 — Red (느림)

## 주요 컨텐츠

LCP에 영향을 많이 주는 `element`의 유형은 다음과 같다.

- `<img>` 요소
- `<svg>` 요소 내부의 `<image>`
- `<video>` 요소(포스터 이미지 사용)
- url() 함수를 통해 로드된 배경 이미지가 있는 요소(CSS 그라데이션과는 대조적임)
- 텍스트 노드 또는 기타 인라인 수준 텍스트 하위 요소를 포함하는 블록 수준 요소

## 낮은 점수 원인

- 느린 서버 응답 시간
- 렌더링 차단 JavaScript 및 CSS
- 리소스 로드 시간
- 클라이언트 측 렌더링

## 개선 방법

- PRPL 패턴으로 즉각적 로딩 적용
- 중요 렌더링 경로 최적화
- CSS 최적화
- 이미지 최적화
- 웹 글꼴 최적화
- JavaScript 최적화(클라이언트 렌더링 사이트용)

<br>

# TBT

## TBT 란?

- TBT : Total Blocking Time

TBT 는 메인 스레드가 입력 응답을 막을 만큼 오래 차단되었을 때 First Contentful Paint(FCP)와 Time to Interactive(TTI) 사이 총 시간을 측정한 값이다.

메인 스레드에서 긴 작업(50ms 이상 실행되는 작업)이 있을 때마다 메인 스레드는 "차단"된 것으로 간주된다.

메인 스레드가 "차단"되었다고 하는 이유는 브라우저가 진행 중인 작업을 중단할 수 없기 때문이다.

따라서 사용자가 긴 작업 중에 페이지와 상호작용을 하기 위해서는 우선적으로 해당 작업이 끝나야만 한다.

작업이 길어지는 경우(50ms 이상) 사용자는 지연을 알아차리고 페이지가 느리거나 버벅거리는 것으로 인지하게 된다.

## 측정 방법

TBT 는 FCP와 TTI 사이에서 발생하는 각각의 긴 작업에 대한 차단 시간을 합한 것이다.

![image](https://user-images.githubusercontent.com/53864640/166471398-5297e1b6-5ba6-44ca-8bda-e754bb86f28d.png){: .align-center}

위의 타임라인에는 5개의 작업이 있으며 그 중 3개는 지속 시간이 50ms를 초과하기 때문에 긴 작업으로 간주된다.

다음 다이어그램은 각각의 긴 작업에 대한 차단 시간을 보여줍니다.

![image](https://user-images.githubusercontent.com/53864640/166471429-e1a7dac8-97ad-4f8c-9863-973901986d50.png){: .align-center}
_- [그림출처](https://web.dev/tbt/)_
{: .text-center}

따라서 메인 스레드에서 작업을 실행하는 데 소요된 총 시간은 560ms 이지만 총 차단 시간(TBT)은 345ms 이다.

## TTI와의 관련성

- TBT는 페이지가 안정적인 상호 작용 환경이 되기 전, 상호 작용이 불가능했을 때의 심각성을 수량화하는 데 도움이 되기 때문에 TTI와 함께 보기에 가장 좋은 지표이다.
- TBT는 TTI 동안 얼마나 "차단"이 많이 발생했는지 확인 할 수 있게 해준다. (TTI는 같으나 TBT가 다를 수 있음.)

## 점수 측정

- Lighthouse 기준
  - 0 ~ 200 ms — Green (빠름)
  - 200 ~ 600 ms — Orange (보통)
  - 600 ms 이상 — Red (느림)

## 개선 방법

- 타사 코드의 영향 줄이기
- JavaScript 실행 시간 단축
- 메인 스레드 작업 최소화
- 요청 수를 낮게 유지하고 전송 크기를 작게 유지

<br>

# CLS

## CLS 란?

- CLS : Cumulative Layout Shift(누적 레이아웃 이동)

CLS 는 사용자가 예상치 못한 레이아웃 이동을 경험하는 빈도를 수량화한 **시각적 안정성을 측정할 때 사용**되는 값이다. (낮을 수록 좋음)

**예상치 못한 레이아웃 이동**은 페이지가, 텍스트, 링크 또는 버튼 등이 갑자기 생기거나 움직이는 경우 등을 말한다.

누적 레이아웃 이동(CLS)은 실제 사용자에게 이러한 일이 발생하는 빈도를 측정하여 이 문제를 해결하는 데 도움을 준다.

**레이아웃 이동**은 시각적 요소가 렌더링된 프레임에서 다음 프레임으로 넘어갈 때 위치가 변경된 경우 발생한다.

## 점수 측정

- Lighthouse 기준
    - 0 ~ 0.1 — Green (빠름)
    - 0.1 ~ 0.25 — Orange (보통)
    - 0.25 이상 — Red (느림)

## 개선 방법

- 이미지 및 비디오 요소에 항상 크기 속성을 포함하거나 CSS 가로 세로 비율 상자와 같은 방식으로 필요한 공간을 미리 확보한다.
  - 이러한 접근 방식을 사용하면 이미지가 로드되는 동안 브라우저가 문서에 올바른 양의 공간을 할당할 수 있음. 
  - unsized-media 기능 정책을 사용하여 기능 정책을 지원하는 브라우저에서 이 동작을 강제할 수도 있음.
- 사용자 상호 작용에 대한 응답을 제외하고는 기존 콘텐츠 위에 콘텐츠를 삽입하지 않는다. 
  - 이렇게 하면 레이아웃 이동이 발생하기 때문이다.
- 레이아웃 변경을 트리거하는 속성의 애니메이션보다 전환 애니메이션을 사용하라.
  - 상태에서 상태로 컨텍스트와 연속성을 제공하는 방식으로 애니메이션 전환을 수행하는 것이 좋다.

<br>

# 참고

- [First Contentful Paint(최초 콘텐츠풀 페인트, FCP)](https://web.dev/i18n/ko/fcp/)
- [TTI (Time To Interactive): What It Is and What It Tells About Your Website](https://uploadcare.com/blog/time-to-interactive/)
- [Speed Index](https://web.dev/speed-index/)
- [Largest Contentful Paint(최대 콘텐츠풀 페인트, LCP)](https://web.dev/lcp/#how-to-improve-largest-contentful-paint-on-your-site)
- [Total Blocking Time(총 차단 시간, TBT)](https://web.dev/tbt/)
- [Cumulative Layout Shift(누적 레이아웃 이동, CLS)](https://web.dev/cls/)