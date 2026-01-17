---
title: "[Kubernetes] Canary, A/B 테스트, Progressive Delivery 완벽 이해 - Flagger와 함께하는 안전한 배포 전략"
last_modified_at: 2026-01-17T12:00:00+09:00
categories:
    - Infra
tags:
    - Kubernetes
    - Canary
    - A/B Testing
    - Progressive Delivery
    - Flagger
    - Deployment
    - DevOps
toc: true
toc_sticky: true
toc_label: "목차"
---

Kubernetes 환경에서 안전한 배포를 위한 Canary, A/B 테스트, Progressive Delivery 개념을 명확히 정리하고, Flagger의 역할과 동작 원리를 이해한다.
{: .notice--info}

# 들어가며

새로운 버전을 배포할 때 가장 두려운 것은 무엇일까? 바로 **예상치 못한 장애**다. 전체 사용자에게 문제가 있는 버전이 배포되면 서비스 전체가 마비될 수 있다.

이런 위험을 최소화하기 위해 등장한 것이 **Canary 배포**, **A/B 테스트**, **Progressive Delivery**다. 하지만 이 용어들이 비슷해 보여서 헷갈리기 쉽다.

이 글에서는 각 개념의 **정확한 의미와 차이점**을 정리하고, Kubernetes 환경에서 이를 자동화하는 **Flagger의 역할**을 명확히 이해해본다.

---

# 1. Canary 배포란 무엇인가

## 개념

**Canary 배포**는 새 버전을 전체 사용자에게 바로 배포하지 않고, **일부 트래픽만 새 버전으로 보내 검증한 뒤** 문제가 없으면 점진적으로 확장하는 배포 방식이다.

## 비유로 이해하기

> **탄광의 카나리아**
>
> 옛날 광부들은 유독가스를 감지하기 위해 카나리아 새를 탄광에 먼저 들여보냈다. 카나리아가 괜찮으면 사람도 안전하다고 판단했다.
>
> 마찬가지로 새 버전을 **10%의 사용자에게만 먼저 배포**하여 문제가 없는지 확인한다. 카나리아(10%)가 괜찮으면 점진적으로 25% → 50% → 100%로 늘려간다.

## 핵심 목적

### 1. 장애 반경 최소화

- 문제가 있어도 전체 사용자가 아닌 일부만 영향받음
- 빠른 발견과 롤백 가능

### 2. 실제 트래픽 기반 검증

- 테스트 환경이 아닌 **실제 프로덕션 환경**에서 검증
- 실사용자의 피드백을 통해 안정성 확인

## 배포 흐름

```
초기 상태 (구버전 100%)
────────────────────────────
구버전: ████████████████████ 100%
새버전: (없음)

↓ 1단계: Canary 시작 (10%)

구버전: ████████████████░░ 90%
새버전: ██ 10%

↓ 2단계: 검증 후 확대 (25%)

구버전: ███████████████░░░ 75%
새버전: █████ 25%

↓ 3단계: 계속 확대 (50%)

구버전: ██████████░░░░░░░░ 50%
새버전: ██████████ 50%

↓ 4단계: 최종 완료 (100%)

구버전: (종료)
새버전: ████████████████████ 100%
```

---

# 2. Canary vs A/B 테스트 차이

많은 사람들이 Canary 배포와 A/B 테스트를 혼동한다. 하지만 **목적과 사용 방식이 완전히 다르다**.

## 비교표

| 구분 | Canary 배포 | A/B 테스트 |
|------|-----------|-----------|
| **목적** | 안정적인 배포 | 사용자 반응 비교 |
| **대상** | 새 버전 검증 | 기능/UX 비교 실험 |
| **트래픽 분배** | 점진적 증가 (10% → 25% → 50% → 100%) | 고정 비율 (50% vs 50%) |
| **종료 조건** | 성공 시 전면 적용, 실패 시 롤백 | 실험 종료 후 더 나은 버전 선택 |
| **실행 기간** | 짧음 (수분~수시간) | 김 (수일~수주) |
| **관점** | 인프라/운영 | 프로덕트/비즈니스 |
| **평가 기준** | 에러율, 응답시간, 성공률 | 전환율, 클릭률, 매출 |

## 비유로 이해하기

> **Canary = 새 다리의 안전성 검증**
>
> 새로 지은 다리가 안전한지 확인하기 위해 먼저 소수의 사람만 건너게 한다. 문제없으면 모든 사람이 사용한다.

> **A/B 테스트 = 두 메뉴 중 더 인기 있는 것 찾기**
>
> 레스토랑에서 새 메뉴 A와 B 중 어느 것이 더 인기 있는지 알아보기 위해 고객의 50%에게는 A를, 50%에게는 B를 제공하고 판매량을 비교한다.

## 정리

- **Canary = 배포 전략** (기술적 안정성)
- **A/B = 실험 전략** (비즈니스 효과)

---

# 3. Progressive Delivery란?

## 개념

**Progressive Delivery**는 "한 번에 배포"가 아니라 **점진적으로 위험을 관리하는 배포 철학**을 의미하는 상위 개념이다.

## 비유로 이해하기

> **단계별 상품 출시 전략**
>
> 새 제품을 출시할 때 한 번에 전국에 풀지 않고 다양한 전략을 사용한다:
>
> - **Canary**: 서울 일부 매장에서만 먼저 판매
> - **Blue/Green**: 새 매장을 완전히 준비한 후 한 번에 전환
> - **A/B**: 두 가지 패키지 디자인 중 어느 것이 더 잘 팔리는지 테스트
>
> 이 모든 전략을 통칭하여 **Progressive Delivery**라고 한다.

## Progressive Delivery에 포함되는 전략

```
Progressive Delivery
├─ Canary 배포
├─ Blue/Green 배포
├─ A/B 테스트
├─ Feature Flag
└─ 단계적 롤아웃
```

## 핵심 철학

1. **점진적 노출**: 한 번에 모든 사용자에게 배포하지 않음
2. **관찰 가능성**: 메트릭을 통해 지속적으로 모니터링
3. **빠른 피드백**: 문제를 조기에 발견
4. **자동 제어**: 메트릭 기반으로 자동 진행 또는 롤백

## 관계 정리

```
Progressive Delivery (철학/개념)
    ↓
Canary는 그 구현 방식 중 하나
```

**Canary는 Progressive Delivery의 대표적인 구현 방식**이다.

---

# 4. Flagger의 역할

## 개념

**Flagger**는 Kubernetes 환경에서 **Canary 배포를 자동화하는 컨트롤러**다.

## 비유로 이해하기

> **자동차의 크루즈 컨트롤**
>
> 운전자가 속도를 일일이 조절하지 않아도 크루즈 컨트롤이 자동으로 속도를 유지하고, 앞차와의 거리를 조절하며, 위험하면 자동으로 감속한다.
>
> Flagger도 마찬가지로:
> - **자동 가속**: 메트릭이 좋으면 트래픽 비율 자동 증가
> - **자동 감속**: 메트릭이 나빠지면 롤백
> - **목적지 도착**: 문제없으면 100%까지 자동 배포

## Flagger가 하는 일

### 1. Canary CRD 감시

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary  # ← Flagger가 감시하는 커스텀 리소스
metadata:
  name: my-app
spec:
  targetRef:
    kind: Deployment
    name: my-app
```

Flagger는 이 **Canary CRD**를 watch하며, 변경사항을 감지한다.

### 2. 자동 배포 진행

```
1. 새 버전 감지
    ↓
2. Canary Deployment 생성
    ↓
3. 트래픽 10%로 시작
    ↓
4. 메트릭 분석 (30초)
    ↓
5. 성공 → 20%로 증가
   실패 → 롤백
    ↓
6. 반복하여 100%까지
    ↓
7. Promotion (승격)
```

### 3. 트래픽 제어

**중요**: Flagger는 트래픽을 **직접 제어하지 않는다**.

```
Flagger의 역할:
  ├─ Canary CRD 감시
  ├─ 메트릭 분석
  ├─ 배포 단계 제어
  └─ Ingress Controller 조정 ← 트래픽 제어는 여기서!

실제 트래픽 제어:
  └─ Traefik, Istio, NGINX 등의 Ingress Controller
```

Flagger는 **Ingress Controller의 설정을 조정**하여 간접적으로 트래픽을 제어한다.

## 동작 흐름 예시

```
┌─────────────────┐
│  Flagger        │
│  (컨트롤러)      │
└────┬────────────┘
     │ 1. Canary CRD 감지
     │ 2. "트래픽 10%로 설정해줘"
     ↓
┌─────────────────┐
│  Traefik        │  ← 실제 트래픽 분배 수행
│  (Ingress)      │
└────┬────────────┘
     │ 3. 10% 트래픽을 Canary로 전달
     ↓
┌─────────────────┐
│  Canary Pod     │
│  (새 버전)       │
└─────────────────┘
```

---

# 5. Canary CRD란?

## 개념

**Canary CRD(Custom Resource Definition)**는 Flagger가 감시하는 **배포 선언 객체**다.

"이 서비스는 Canary 방식으로 배포하겠다"는 명세를 담고 있다.

## Canary CRD의 구조

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: my-app
  namespace: default
spec:
  # 1. 배포 대상
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app

  # 2. 서비스 설정
  service:
    port: 8080

  # 3. 트래픽 증가 전략
  analysis:
    interval: 30s        # 30초마다 메트릭 분석
    threshold: 5         # 5회 연속 실패 시 롤백
    maxWeight: 50        # 최대 50%까지만
    stepWeight: 10       # 10%씩 증가

    # 4. 성공/실패 판단 메트릭
    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99          # 성공률 99% 이상
      interval: 1m

    - name: request-duration
      thresholdRange:
        max: 500         # 응답시간 500ms 이하
      interval: 1m
```

## CRD에 포함되는 정보

| 항목 | 설명 | 예시 |
|------|------|------|
| **targetRef** | 배포 대상 지정 | Deployment, DaemonSet 등 |
| **service** | 서비스 포트 설정 | 8080, 9090 등 |
| **stepWeight** | 트래픽 증가 비율 | 10% → 20% → 30% |
| **maxWeight** | 최대 트래픽 비율 | 50% (나머지는 수동 승인) |
| **interval** | 메트릭 분석 주기 | 30초, 1분 등 |
| **threshold** | 실패 허용 횟수 | 5회 연속 실패 시 롤백 |
| **metrics** | 성공/실패 판단 기준 | 성공률, 응답시간, 에러율 |

## 비유로 이해하기

> **자동차 자율주행 설정**
>
> Canary CRD는 자율주행 자동차의 설정 파일과 같다:
>
> - **목적지**: my-app Deployment
> - **속도 제한**: 최대 50%까지만 (maxWeight)
> - **가속도**: 10%씩 증가 (stepWeight)
> - **안전 기준**: 성공률 99% 이상, 응답시간 500ms 이하
> - **제동 조건**: 5회 연속 기준 미달 시 정지 (threshold)

---

# 6. Promotion(승격)의 의미

## 개념

**Promotion**은 Canary 검증이 끝난 뒤, **Canary 이미지/스펙을 Primary Deployment에 반영하는 행위**다.

## 중요한 오해 바로잡기

많은 사람들이 "Canary Deployment가 Primary가 된다"고 생각하지만, **이것은 틀렸다**.

### ❌ 잘못된 이해

```
Canary Deployment → Primary가 됨
Primary Deployment → 삭제됨
```

### ✅ 올바른 이해

```
Canary Deployment → 삭제됨 (또는 대기 상태)
Primary Deployment → 이미지만 Canary 버전으로 교체
```

## Promotion 과정

### 1단계: Canary 검증 성공

```
Primary (구버전 v1.0)     Canary (새버전 v2.0)
  ████████ 90%              ██ 10%
     ↓                         ↓
메트릭 안정적               메트릭 안정적
```

### 2단계: Promotion 결정

```
Flagger: "Canary 검증 완료! Primary에 v2.0 적용"
```

### 3단계: Primary 이미지 교체

```yaml
# Primary Deployment의 이미지가 변경됨
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-primary
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:v1.0  # Before
        image: my-app:v2.0  # After (Canary 버전으로 변경)
```

### 4단계: Kubernetes Rolling Update

```
Primary Deployment의 일반적인 Rolling Update 진행
─────────────────────────────────────────────
v1.0 Pod 1개 → v2.0 Pod 1개
v1.0 Pod 1개 → v2.0 Pod 1개
v1.0 Pod 1개 → v2.0 Pod 1개
...
```

### 5단계: Canary Deployment 정리

```
Canary Deployment → 스케일 다운 (0개) 또는 삭제
Primary Deployment → 모두 v2.0으로 실행 중
```

## 비유로 이해하기

> **신메뉴 정식 채택**
>
> - **Canary**: 한 매장에서 신메뉴 테스트 판매
> - **검증 성공**: 고객 반응이 좋음
> - **Promotion**: 전체 매장의 메뉴판을 신메뉴로 교체
> - **테스트 매장**: 정리하거나 다음 테스트 대기

## 핵심 정리

| 항목 | 설명 |
|------|------|
| **Promotion 대상** | Primary Deployment의 이미지 |
| **Promotion 방법** | Kubernetes의 일반적인 Rolling Update |
| **Canary 역할** | 검증 도구 (검증 후 제거) |
| **Primary 역할** | 실제 서비스 제공 (계속 유지) |

---

# 7. 전체 흐름 정리

## Canary 배포 전체 프로세스

```
1. 개발자가 새 버전 배포
   │
   ↓
2. Flagger가 Deployment 변경 감지
   │
   ├─ Primary Deployment: v1.0 유지
   └─ Canary Deployment: v2.0 생성
   │
   ↓
3. 트래픽 분배 시작 (Primary 90%, Canary 10%)
   │
   ├─ Flagger가 Ingress Controller에 지시
   └─ Ingress가 실제 트래픽 분배
   │
   ↓
4. 메트릭 분석 (30초 간격)
   │
   ├─ Prometheus에서 메트릭 수집
   ├─ 성공률, 응답시간, 에러율 확인
   │
   ↓
5. 분석 결과에 따라 진행
   │
   ├─ 성공 → 트래픽 20%로 증가 (3단계로 이동)
   └─ 실패 → 자동 롤백 (Canary 삭제)
   │
   ↓
6. 반복하여 트래픽 100%까지 증가
   │
   ↓
7. Promotion (승격)
   │
   ├─ Primary Deployment 이미지를 v2.0으로 변경
   ├─ Kubernetes Rolling Update 실행
   └─ Canary Deployment 스케일 다운
   │
   ↓
8. 배포 완료
   │
   └─ Primary: v2.0 (100%)
      Canary: 0 (대기)
```

## 각 컴포넌트의 역할

| 컴포넌트 | 역할 | 비유 |
|----------|------|------|
| **Flagger** | Canary 배포 오케스트레이션 | 지휘자 |
| **Canary CRD** | 배포 전략 정의 | 악보 |
| **Primary Deployment** | 실제 서비스 제공 | 주 공연장 |
| **Canary Deployment** | 새 버전 검증 | 리허설 공연장 |
| **Ingress Controller** | 트래픽 분배 | 관객 안내원 |
| **Prometheus** | 메트릭 수집 | 평론가 |

---

# 8. 실전 예시

## 시나리오: 쇼핑몰 API 새 버전 배포

### 1. 기존 환경

```yaml
# my-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 10
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:v1.0.0  # 현재 버전
```

### 2. Canary CRD 생성

```yaml
# canary.yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: my-app
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app

  service:
    port: 8080

  analysis:
    interval: 1m          # 1분마다 분석
    threshold: 3          # 3회 실패 시 롤백
    maxWeight: 50         # 최대 50%까지만 자동
    stepWeight: 10        # 10%씩 증가

    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99.5         # 성공률 99.5% 이상

    - name: request-duration
      thresholdRange:
        max: 300          # 응답시간 300ms 이하

    webhooks:
    - name: load-test
      url: http://flagger-loadtester/
      timeout: 5s
      metadata:
        cmd: "hey -z 1m -q 10 -c 2 http://my-app:8080/"
```

### 3. 새 버전 배포

```bash
# 이미지 업데이트
kubectl set image deployment/my-app my-app=my-app:v2.0.0
```

### 4. Flagger 자동 진행

```
[00:00] Canary 감지
────────────────────────────────
Deployment/my-app 이미지 변경 감지
Canary Deployment 생성 중...

[00:01] Canary 초기화 완료
────────────────────────────────
Primary: v1.0.0 (100%)
Canary: v2.0.0 (0%)

[00:02] Canary 진행 시작
────────────────────────────────
트래픽 10%로 설정
Primary: 90%, Canary: 10%

[00:03] 메트릭 분석 중 (1분)
────────────────────────────────
성공률: 99.7% ✓
응답시간: 250ms ✓
상태: 정상

[01:04] 트래픽 증가
────────────────────────────────
Primary: 80%, Canary: 20%

[02:05] 트래픽 증가
────────────────────────────────
Primary: 70%, Canary: 30%

[03:06] 트래픽 증가
────────────────────────────────
Primary: 60%, Canary: 40%

[04:07] 트래픽 증가
────────────────────────────────
Primary: 50%, Canary: 50%
(maxWeight 도달, 수동 승인 대기 또는 자동 진행)

[05:08] Promotion 시작
────────────────────────────────
Primary Deployment 이미지를 v2.0.0으로 변경
Rolling Update 시작...

[06:09] 배포 완료
────────────────────────────────
Primary: v2.0.0 (100%)
Canary: v2.0.0 (0, 대기 상태)
```

### 5. 실패 시나리오 (롤백)

```
[00:00] Canary 진행 시작
────────────────────────────────
Primary: 90%, Canary: 10%

[00:01] 메트릭 분석 중
────────────────────────────────
성공률: 99.7% ✓
응답시간: 280ms ✓

[01:02] 트래픽 증가
────────────────────────────────
Primary: 80%, Canary: 20%

[01:03] 메트릭 분석 중
────────────────────────────────
성공률: 98.5% ✗ (기준: 99.5%)
응답시간: 450ms ✗ (기준: 300ms)
실패 카운트: 1/3

[02:04] 메트릭 분석 중
────────────────────────────────
성공률: 98.2% ✗
응답시간: 520ms ✗
실패 카운트: 2/3

[03:05] 메트릭 분석 중
────────────────────────────────
성공률: 97.8% ✗
응답시간: 600ms ✗
실패 카운트: 3/3

[03:06] 자동 롤백 시작
────────────────────────────────
Canary 배포 중단
트래픽을 Primary로 100% 복귀
Canary Deployment 스케일 다운

[03:07] 알림 발송
────────────────────────────────
Slack: "my-app Canary 배포 실패 - 롤백 완료"

[03:08] 롤백 완료
────────────────────────────────
Primary: v1.0.0 (100%)
Canary: v2.0.0 (0, 실패)
```

---

# 9. 핵심 요약

## 개념 정리

### Canary 배포

- **목적**: 안전한 배포
- **방식**: 일부 트래픽으로 검증 후 점진적 확대
- **관점**: 인프라/운영

### A/B 테스트

- **목적**: 기능 비교 실험
- **방식**: 고정 비율로 장기간 비교
- **관점**: 프로덕트/비즈니스

### Progressive Delivery

- **목적**: 점진적 배포 철학
- **방식**: Canary, Blue/Green 등 포함하는 상위 개념
- **관점**: 전략

## Flagger의 역할

1. **Canary CRD 감시**: Kubernetes 리소스 변경 감지
2. **배포 오케스트레이션**: 트래픽 비율 조정 지시
3. **메트릭 분석**: Prometheus 등에서 수집한 데이터 분석
4. **자동 진행/롤백**: 기준에 따라 자동 판단

## Canary CRD

- Flagger가 감시하는 **배포 선언 객체**
- 트래픽 증가 비율, 메트릭 기준, 롤백 조건 등을 정의

## Promotion

- Canary 검증 성공 후 **Primary에 새 버전 반영**
- Canary가 Primary가 되는 것이 아님
- Primary의 이미지만 교체 (일반 Rolling Update)

---

# 10. 베스트 프랙티스

## Canary 배포 설정

### ✅ DO

- **적절한 stepWeight 설정**: 너무 크면 위험, 너무 작으면 느림 (권장: 10%)
- **충분한 분석 간격**: 최소 30초~1분 (메트릭 수집 시간 고려)
- **명확한 성공 기준**: 성공률, 응답시간, 에러율 등 구체적 지표
- **알림 설정**: Slack, Email 등으로 배포 상태 공유

### ❌ DON'T

- **너무 짧은 interval**: 메트릭이 안정화되기 전에 판단
- **너무 관대한 threshold**: 문제를 놓칠 수 있음
- **maxWeight 100%**: 자동으로 전체 배포는 위험

## 메트릭 선정

### 기본 메트릭

```yaml
metrics:
# 1. 성공률 (필수)
- name: request-success-rate
  thresholdRange:
    min: 99.5

# 2. 응답시간 (필수)
- name: request-duration
  thresholdRange:
    max: 500

# 3. 에러율 (권장)
- name: error-rate
  thresholdRange:
    max: 1
```

### 비즈니스 메트릭 (선택)

```yaml
# 4. 주문 성공률 (이커머스)
- name: order-success-rate
  thresholdRange:
    min: 98

# 5. API 호출 성공률 (서비스)
- name: api-call-success-rate
  thresholdRange:
    min: 99
```

## 배포 전략

### 단계별 가이드

```yaml
# 1. 보수적 (중요 서비스)
analysis:
  interval: 2m        # 2분마다 분석
  threshold: 3        # 3회 실패 허용
  maxWeight: 30       # 최대 30%까지만 자동
  stepWeight: 5       # 5%씩 증가

# 2. 균형적 (일반 서비스)
analysis:
  interval: 1m        # 1분마다 분석
  threshold: 5        # 5회 실패 허용
  maxWeight: 50       # 최대 50%까지만 자동
  stepWeight: 10      # 10%씩 증가

# 3. 공격적 (테스트 환경)
analysis:
  interval: 30s       # 30초마다 분석
  threshold: 3        # 3회 실패 허용
  maxWeight: 100      # 전체 자동 배포
  stepWeight: 20      # 20%씩 증가
```

---

# 11. 트러블슈팅

## 문제 1: Canary가 진행되지 않음

### 증상

```bash
$ kubectl describe canary my-app
...
Status:
  Phase: Progressing
  Failed Checks: 0
  Canary Weight: 0
```

### 원인

1. **메트릭 수집 실패**: Prometheus 연동 문제
2. **Ingress 설정 오류**: 트래픽 분배 불가
3. **이미지 변경 없음**: Deployment 업데이트 안 됨

### 해결

```bash
# 1. Flagger 로그 확인
kubectl logs -n flagger-system deploy/flagger -f

# 2. Prometheus 메트릭 확인
curl http://prometheus:9090/api/v1/query?query=request_success_rate

# 3. Ingress 설정 확인
kubectl get ingress my-app -o yaml

# 4. Deployment 이미지 강제 업데이트
kubectl set image deployment/my-app my-app=my-app:v2.0.0
```

## 문제 2: 자동 롤백 발생

### 증상

```bash
$ kubectl describe canary my-app
...
Events:
  Type     Reason  Message
  ----     ------  -------
  Warning  Synced  Halt advancement: request-duration check failed
```

### 원인

- 메트릭이 threshold를 충족하지 못함

### 해결

```bash
# 1. 실제 메트릭 확인
kubectl logs -n flagger-system deploy/flagger | grep my-app

# 2. Canary Pod 로그 확인
kubectl logs -l app=my-app-canary

# 3. 메트릭 임계값 조정 (테스트 환경)
# canary.yaml에서 threshold 완화
```

## 문제 3: Promotion이 완료되지 않음

### 증상

```bash
$ kubectl get canary my-app
NAME     STATUS      WEIGHT   LASTTRANSITIONTIME
my-app   Succeeded   0        5m
```

Canary는 성공했지만 Primary가 업데이트되지 않음

### 원인

- maxWeight에 도달 후 수동 승인 대기

### 해결

```bash
# 1. 수동 Promotion (Flagger 1.18 이상)
kubectl annotate canary my-app flagger.app/promote="true"

# 2. 또는 maxWeight를 100으로 설정하여 자동 진행
```

---

# 결론

Canary 배포, A/B 테스트, Progressive Delivery는 각기 다른 목적과 용도를 가진 개념이다.

## 핵심 정리

| 개념 | 한 줄 요약 |
|------|-----------|
| **Canary 배포** | 일부 트래픽으로 안전성 검증 후 점진 확대 (배포 전략) |
| **A/B 테스트** | 고정 비율로 기능 비교 실험 (비즈니스 전략) |
| **Progressive Delivery** | 점진적 배포 철학 (상위 개념) |
| **Flagger** | Canary 배포 자동화 컨트롤러 (도구) |
| **Canary CRD** | Flagger가 감시하는 배포 선언 객체 (설정) |
| **Promotion** | Canary 성공 후 Primary에 새 버전 반영 (절차) |

## 언제 사용할까?

```
상황별 선택 가이드
─────────────────────────────────

Q1: 배포 안정성이 최우선?
    → YES: Canary 배포

Q2: 기능 효과를 비교하고 싶다?
    → YES: A/B 테스트

Q3: 자동화된 안전장치가 필요?
    → YES: Flagger 사용

Q4: 수동 제어가 필요?
    → YES: Argo Rollouts 사용
```

## 다음 단계

1. **학습 순서**
   - Kubernetes Deployment 이해
   - Prometheus 메트릭 수집 구축
   - Flagger 설치 및 기본 Canary 실습
   - 실제 서비스에 점진적 적용

2. **실전 적용**
   - 테스트 환경에서 충분히 실험
   - 메트릭 기준을 서비스에 맞게 조정
   - 알림 채널 구성
   - 롤백 시나리오 테스트

3. **더 알아보기**
   - Blue/Green 배포
   - Feature Flag
   - Service Mesh (Istio, Linkerd)

안전하고 효율적인 배포 전략으로 서비스의 안정성을 높여보자!

---

# 참고 자료

- [Flagger 공식 문서](https://flagger.app/)
- [Progressive Delivery 소개](https://www.weave.works/blog/what-is-progressive-delivery-all-about)
- [Canary vs A/B 비교](https://blog.getambassador.io/canary-vs-blue-green-vs-rolling-deployment-4a4c5b5e8ee4)
- [Kubernetes Deployment 전략](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Prometheus Metrics](https://prometheus.io/docs/introduction/overview/)
