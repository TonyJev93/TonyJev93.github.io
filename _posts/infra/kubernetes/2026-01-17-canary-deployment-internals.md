---
title: "[Kubernetes] Canary 배포 동작 원리 - Flagger의 내부 메커니즘 완벽 분석"
last_modified_at: 2026-01-17T15:00:00+09:00
categories:
    - Infra
tags:
    - Kubernetes
    - Canary
    - Flagger
    - Deployment
    - DevOps
    - HPA
    - Progressive Delivery
toc: true
toc_sticky: true
toc_label: "목차"
---

Flagger를 사용한 Canary 배포의 내부 동작 원리를 깊이 있게 분석하고, Primary/Canary Deployment의 역할, 트래픽 분기 메커니즘, Promotion 과정을 완벽히 이해한다.
{: .notice--info}

# 들어가며

[이전 글](/infra/kubernetes-deployment-guide/)에서 Canary 배포의 개념과 Flagger의 역할을 살펴봤다면, 이번에는 **실제 내부에서 어떻게 동작하는지** 깊이 파고들어 보자.

많은 개발자들이 Canary 배포를 사용하지만, 실제로 내부에서:
- Flagger가 어떤 순서로 리소스를 생성하는지
- Primary와 Canary Deployment가 어떻게 분리되는지
- 트래픽 비율과 Pod 수가 어떤 관계인지
- Promotion이 실제로 무엇을 의미하는지

이런 내부 메커니즘을 제대로 이해하는 경우는 드물다.

이 글에서는 Flagger의 **내부 동작 원리**를 단계별로 분석하여, Canary 배포가 실제로 어떻게 작동하는지 명확히 이해해본다.

---

# 1. Canary 배포의 시작점

## Canary CRD 생성

모든 것은 사용자가 **Canary CRD(Custom Resource Definition)**를 생성하는 것에서 시작된다.

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: my-app
  namespace: default
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app  # 기존 Deployment
  service:
    port: 8080
  analysis:
    interval: 30s
    threshold: 5
    maxWeight: 50
    stepWeight: 10
```

## Flagger의 감시 시작

Flagger는 Kubernetes 컨트롤러로서 **Canary CRD를 watch**하고 있다가, 새로운 Canary 리소스가 생성되거나 targetRef로 지정된 Deployment에 변경이 발생하면 즉시 반응한다.

```
사용자
  │
  ├─ kubectl apply -f canary.yaml
  ↓
Kubernetes API Server
  │
  ├─ Canary CRD 생성
  ↓
Flagger (watch)
  │
  └─ "my-app Deployment를 Canary 방식으로 관리 시작!"
```

---

# 2. Flagger가 수행하는 기본 흐름

Canary CRD가 생성되면 Flagger는 다음 순서로 동작한다.

## Step 1: Primary Deployment 생성

**기존 Deployment를 복제**하여 `<name>-primary` Deployment를 생성한다.

```yaml
# 원본: my-app
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:v1.0

# Flagger가 생성 ↓
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-primary  # ← 복제본
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:v1.0  # 동일한 이미지
```

## Step 2: Canary Deployment 준비

새 버전이 배포되면 `<name>-canary` Deployment를 생성한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-canary  # ← 새 버전
spec:
  replicas: 1  # 초기에는 적은 수
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:v2.0  # 새 버전
```

## Step 3: Service 분리

기존 Service를 Primary/Canary용으로 분리한다.

```yaml
# Primary Service
apiVersion: v1
kind: Service
metadata:
  name: my-app-primary
spec:
  selector:
    app: my-app-primary

---
# Canary Service
apiVersion: v1
kind: Service
metadata:
  name: my-app-canary
spec:
  selector:
    app: my-app-canary
```

## Step 4: IngressRoute 트래픽 제어

Traefik의 경우 **IngressRoute의 weight**를 조정하여 트래픽을 분기한다.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: my-app
spec:
  routes:
  - match: Host(`my-app.example.com`)
    kind: Rule
    services:
    - name: my-app-primary
      port: 8080
      weight: 90        # Primary 90%
    - name: my-app-canary
      port: 8080
      weight: 10        # Canary 10%
```

---

# 3. Deployment 구성의 의미

## Primary Deployment의 역할

**현재 안정 버전을 제공하는 실제 운영 Deployment**다.

- 실제 프로덕션 트래픽의 기준
- 항상 안정적인 버전 유지
- Canary 검증이 끝나면 새 버전으로 업데이트

## Canary Deployment의 역할

**새 버전을 검증하는 테스트 Deployment**다.

- 새 버전의 안정성을 확인하는 대상
- 검증이 끝나면 삭제되거나 대기 상태로 유지
- 실패 시 즉시 중단 가능

## 원본 Deployment의 운명

기존 "원본 Deployment"는:

```
초기 상태
─────────────────────
my-app (원본)
  ├─ 트래픽 받음
  └─ 실제 서비스 제공

Flagger 초기화 후
─────────────────────
my-app (원본)
  ├─ 트래픽 받지 않음 ✗
  ├─ Flagger가 관리용으로 사용
  └─ 실제로는 사용되지 않음

my-app-primary (복제본)
  ├─ 실제 트래픽 받음 ✓
  └─ 운영 서비스 제공
```

**중요**: 원본 Deployment는 Flagger가 Primary를 생성한 이후 더 이상 트래픽을 받지 않는다. Flagger가 새 버전을 감지하는 템플릿으로만 사용된다.

---

# 4. 트래픽 분기 방식

## Traefik IngressRoute의 Weight 기반 분기

Flagger는 **IngressRoute의 weight 값**을 단계별로 조정하여 트래픽을 분배한다.

### 단계별 트래픽 분배 예시

```yaml
# 1단계: 초기 (90% / 10%)
services:
- name: my-app-primary
  weight: 90
- name: my-app-canary
  weight: 10

# 2단계: 검증 통과 (75% / 25%)
services:
- name: my-app-primary
  weight: 75
- name: my-app-canary
  weight: 25

# 3단계: 계속 진행 (50% / 50%)
services:
- name: my-app-primary
  weight: 50
- name: my-app-canary
  weight: 50

# 4단계: 최종 (0% / 100%)
services:
- name: my-app-primary
  weight: 0
- name: my-app-canary
  weight: 100
```

## 실제 동작 흐름

```
사용자 요청
    ↓
Traefik Ingress Controller
    │
    ├─ weight 계산 (90:10)
    ├─ 90%는 → my-app-primary Service
    └─ 10%는 → my-app-canary Service
        ↓
각 Service의 Endpoints로 전달
    │
    ├─ my-app-primary Pod 1, 2, 3
    └─ my-app-canary Pod 1
```

## Flagger의 역할

Flagger는 트래픽을 **직접 제어하지 않는다**. 대신:

1. 메트릭을 분석
2. 다음 단계로 진행 가능한지 판단
3. IngressRoute의 weight 값을 업데이트
4. 실제 트래픽 분배는 Traefik이 수행

```
Flagger (판단)
    ↓
"메트릭 OK → weight를 20%로 증가"
    ↓
IngressRoute 수정 (kubectl patch)
    ↓
Traefik (실행)
    ↓
"weight 20% 적용 완료"
```

---

# 5. Canary Pod 수와 트래픽 100%의 관계

## 오해하기 쉬운 점

많은 사람들이 **"트래픽 100%를 받으려면 Canary Pod 수도 100%여야 한다"**고 생각한다.

### ❌ 잘못된 이해

```
Primary: 10 Pods (0% 트래픽)
Canary: 10 Pods (100% 트래픽)
```

### ✅ 올바른 이해

```
Primary: 10 Pods (0% 트래픽)
Canary: 3 Pods (100% 트래픽) ← 적은 수로 시작
    ↓
트래픽 증가 감지
    ↓
HPA가 Canary를 자동 확장
    ↓
Canary: 10 Pods (100% 트래픽)
```

## 동작 원리

### 초기 상태

```yaml
# Canary Deployment
spec:
  replicas: 1  # 처음엔 적게 시작
```

### 트래픽 증가

```
1단계: Canary 10% 트래픽
  ├─ Pod 1개로도 충분히 처리 가능
  └─ HPA 동작 안 함

2단계: Canary 25% 트래픽
  ├─ 트래픽 증가 → CPU/메모리 사용량 증가
  └─ HPA가 감지 → Pod 2개로 증가

3단계: Canary 50% 트래픽
  ├─ 더 많은 요청 처리 필요
  └─ HPA가 Pod 5개로 증가

4단계: Canary 100% 트래픽
  ├─ 모든 트래픽을 Canary가 처리
  └─ HPA가 필요한 만큼 확장 (예: 10개)
```

## 핵심 정리

| 구분 | 설명 |
|------|------|
| **트래픽 비율** | IngressRoute의 weight로 제어 |
| **Pod 수** | HPA가 실제 부하에 따라 자동 조정 |
| **관계** | 트래픽 비율 ≠ Pod 비율 |
| **수용 능력** | HPA가 자동으로 보정 |

```
트래픽 제어 (Flagger)
    ↓
트래픽 증가
    ↓
부하 증가 (CPU/메모리)
    ↓
HPA 감지
    ↓
Pod 자동 확장
```

**결론**: Canary Pod는 처음에 적은 수로 시작하고, 트래픽이 증가하면 HPA가 자동으로 확장한다. 트래픽 비율과 Pod 비율은 독립적이다.

---

# 6. 메트릭 기반 검증

Flagger는 각 단계에서 **메트릭을 분석**하여 다음 단계로 진행할지 결정한다.

## 검증 메트릭

```yaml
analysis:
  metrics:
  # 1. 오류율
  - name: request-success-rate
    thresholdRange:
      min: 99  # 성공률 99% 이상

  # 2. 지연 시간
  - name: request-duration
    thresholdRange:
      max: 500  # 응답시간 500ms 이하

  # 3. 에러율
  - name: error-rate
    thresholdRange:
      max: 1  # 에러율 1% 이하
```

## 검증 프로세스

```
1. Canary 10% 배포
    ↓
2. interval (30초) 동안 대기
    ↓
3. Prometheus에서 메트릭 수집
    ├─ request-success-rate: 99.5% ✓
    ├─ request-duration: 450ms ✓
    └─ error-rate: 0.3% ✓
    ↓
4. 모든 메트릭이 기준 통과
    ↓
5. 다음 단계로 진행 (20%)
```

## 기준 초과 시 동작

```
1. Canary 20% 배포
    ↓
2. 메트릭 수집
    ├─ request-success-rate: 97% ✗ (기준: 99%)
    ├─ request-duration: 650ms ✗ (기준: 500ms)
    └─ error-rate: 3% ✗ (기준: 1%)
    ↓
3. 실패 카운트 증가 (1/5)
    ↓
4. 다시 메트릭 수집
    ├─ 여전히 기준 미달
    └─ 실패 카운트 증가 (2/5, 3/5, ...)
    ↓
5. threshold (5) 도달
    ↓
6. 자동 롤백 시작
    ├─ Canary 중단
    ├─ 트래픽 100% Primary로 복귀
    └─ Canary Deployment 스케일 다운
```

## 메트릭 수집 흐름

```
┌──────────────┐
│  Canary Pod  │
│   (v2.0)     │
└──────┬───────┘
       │ 메트릭 노출 (:9090/metrics)
       ↓
┌──────────────┐
│  Prometheus  │  ← Scrape 메트릭
└──────┬───────┘
       │ 저장된 메트릭
       ↓
┌──────────────┐
│   Flagger    │  ← PromQL 쿼리
└──────┬───────┘
       │ 분석 결과
       ↓
  진행 / 롤백 결정
```

---

# 7. Promotion (승격) 단계

Canary가 모든 검증을 통과하면 **Promotion(승격)**이 수행된다.

## Promotion의 정확한 의미

많은 사람들이 오해하는 부분이다.

### ❌ 잘못된 이해

```
Canary Deployment가 Primary Deployment가 된다
```

### ✅ 올바른 이해

```
Primary Deployment의 이미지를
Canary 이미지로 교체한다
```

## Promotion 실제 동작

### 1단계: Canary 검증 완료

```
Primary Deployment (my-app-primary)
  ├─ image: my-app:v1.0
  └─ replicas: 10

Canary Deployment (my-app-canary)
  ├─ image: my-app:v2.0
  └─ replicas: 3
  └─ 모든 메트릭 통과 ✓
```

### 2단계: Flagger가 Primary 업데이트

```yaml
# Before
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-primary
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:v1.0  # ← 구버전

# After (Flagger가 수정)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-primary
spec:
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:v2.0  # ← 새버전으로 교체
```

### 3단계: Kubernetes Rolling Update

Primary Deployment에서 **일반적인 Rolling Update**가 발생한다.

```
Primary Deployment의 Rolling Update
──────────────────────────────────

1. 새 Pod (v2.0) 생성
   ├─ v1.0 Pod: 10개
   └─ v2.0 Pod: 1개 (생성 중)

2. 새 Pod 준비 완료
   ├─ v2.0 Pod: Ready
   └─ 구 Pod 1개 종료 시작

3. 순차적 교체
   ├─ v1.0 Pod: 9개
   └─ v2.0 Pod: 2개

4. 반복...
   ├─ v1.0 Pod: 5개
   └─ v2.0 Pod: 6개

5. 최종 완료
   ├─ v1.0 Pod: 0개 (종료)
   └─ v2.0 Pod: 10개 (실행 중)
```

### 4단계: Canary Deployment 정리

```
Canary Deployment (my-app-canary)
  ├─ 더 이상 사용되지 않음
  ├─ Scale down to 0
  └─ 또는 삭제
```

## Promotion 후 최종 상태

```
Primary Deployment (my-app-primary)
  ├─ image: my-app:v2.0  ← 새 버전
  ├─ replicas: 10
  └─ 트래픽 100%

Canary Deployment (my-app-canary)
  ├─ image: my-app:v2.0
  ├─ replicas: 0  ← 대기 상태
  └─ 다음 배포를 위해 대기

원본 Deployment (my-app)
  ├─ Flagger가 계속 watch
  └─ 다음 이미지 변경 감지 대기
```

---

# 8. 실패 시 동작 (Rollback)

메트릭이 기준을 충족하지 못하면 **자동 롤백**이 발생한다.

## 롤백 트리거

```yaml
analysis:
  threshold: 5  # 5회 연속 실패 시 롤백
```

## 롤백 프로세스

### 1단계: 메트릭 실패 감지

```
Canary 25% 진행 중
    ↓
메트릭 수집
  ├─ success-rate: 97% ✗ (기준: 99%)
  └─ duration: 700ms ✗ (기준: 500ms)
    ↓
실패 카운트: 1/5
```

### 2단계: 재시도

```
30초 대기 (interval)
    ↓
다시 메트릭 수집
  ├─ 여전히 기준 미달
  └─ 실패 카운트: 2/5
    ↓
계속 재시도...
  └─ 실패 카운트: 3/5, 4/5, 5/5
```

### 3단계: 자동 롤백 시작

```
threshold (5) 도달
    ↓
Flagger: "롤백 시작!"
    ↓
1. IngressRoute weight 즉시 원복
   ├─ Canary: 0%
   └─ Primary: 100%

2. Canary Deployment 중단
   └─ Scale down to 0

3. 알림 발송
   └─ Slack: "Canary 배포 실패 - 롤백 완료"
```

### 4단계: 롤백 완료

```
Primary Deployment
  ├─ image: my-app:v1.0  ← 구버전 유지
  ├─ replicas: 10
  └─ 트래픽 100% (영향 없음)

Canary Deployment
  ├─ image: my-app:v2.0  ← 실패한 버전
  ├─ replicas: 0
  └─ 중단됨
```

## 롤백의 특징

| 항목 | 설명 |
|------|------|
| **속도** | 즉시 (수 초 내) |
| **영향 범위** | Canary 트래픽만 (Primary 무영향) |
| **데이터 손실** | 없음 |
| **사용자 영향** | 최소화 (일부 트래픽만 영향) |

---

# 9. 전체 흐름 다이어그램

## 성공 시나리오

```
[초기 상태]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
원본 Deployment (my-app)
  └─ image: v1.0

        ↓ Canary CRD 생성

[Flagger 초기화]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary Deployment (my-app-primary)
  └─ image: v1.0
  └─ 트래픽: 100%

Canary Deployment (my-app-canary)
  └─ image: v1.0
  └─ 대기 중

        ↓ 새 버전 배포 (v2.0)

[Canary 시작]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary: v1.0 (90%)
Canary: v2.0 (10%)

        ↓ 메트릭 검증 (30초)

메트릭 분석
  ├─ success-rate: 99.5% ✓
  ├─ duration: 450ms ✓
  └─ error-rate: 0.3% ✓

        ↓ 통과

[Canary 진행]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary: v1.0 (75%)
Canary: v2.0 (25%)

        ↓ 계속 검증

Primary: v1.0 (50%)
Canary: v2.0 (50%)

        ↓ 최종 단계

Primary: v1.0 (0%)
Canary: v2.0 (100%)

        ↓ 모든 검증 통과

[Promotion]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary 이미지 교체: v1.0 → v2.0
Rolling Update 시작...

        ↓ Rolling Update 완료

[완료 상태]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary Deployment
  ├─ image: v2.0
  └─ 트래픽: 100%

Canary Deployment
  ├─ image: v2.0
  └─ 대기 (replicas: 0)
```

## 실패 시나리오

```
[Canary 진행 중]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary: v1.0 (80%)
Canary: v2.0 (20%)

        ↓ 메트릭 검증

메트릭 분석
  ├─ success-rate: 97% ✗
  ├─ duration: 700ms ✗
  └─ 실패 카운트: 1/5

        ↓ 재시도

메트릭 분석
  ├─ 여전히 기준 미달
  └─ 실패 카운트: 2/5

        ↓ 계속 재시도

실패 카운트: 5/5 (threshold 도달)

        ↓ 자동 롤백

[롤백 수행]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
트래픽 즉시 원복
  ├─ Primary: 100%
  └─ Canary: 0%

Canary Deployment 중단
  └─ Scale down to 0

알림 발송
  └─ Slack: "배포 실패 - 롤백 완료"

        ↓

[롤백 완료]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary Deployment
  ├─ image: v1.0 (유지)
  └─ 트래픽: 100%

Canary Deployment
  ├─ image: v2.0 (실패)
  └─ 중단됨
```

---

# 10. 실전 예시

## 시나리오: 결제 서비스 새 버전 배포

### 초기 환경

```yaml
# payment-service Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-service
spec:
  replicas: 5
  template:
    spec:
      containers:
      - name: payment
        image: payment:v1.0.0
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
```

### HPA 설정

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: payment-service
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Canary CRD 생성

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: payment-service
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment-service

  service:
    port: 8080

  analysis:
    interval: 1m
    threshold: 3
    maxWeight: 50
    stepWeight: 10

    metrics:
    - name: request-success-rate
      thresholdRange:
        min: 99.9  # 결제 서비스는 높은 기준

    - name: request-duration
      thresholdRange:
        max: 200  # 빠른 응답 필요

    webhooks:
    - name: load-test
      url: http://flagger-loadtester/
      metadata:
        cmd: "hey -z 1m -q 50 -c 5 http://payment-service:8080/health"
```

### 새 버전 배포

```bash
# 이미지 업데이트
kubectl set image deployment/payment-service payment=payment:v2.0.0
```

### Flagger 동작 로그

```
[00:00:00] Canary 감지
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Deployment/payment-service 이미지 변경 감지
  From: payment:v1.0.0
  To:   payment:v2.0.0

[00:00:05] 초기화 시작
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ payment-service-primary Deployment 생성
  ├─ image: payment:v1.0.0
  └─ replicas: 5

✓ payment-service-canary Deployment 생성
  ├─ image: payment:v2.0.0
  └─ replicas: 1  ← 적은 수로 시작

✓ Service 분리 완료
  ├─ payment-service-primary
  └─ payment-service-canary

✓ IngressRoute 설정 완료
  ├─ Primary: 100%
  └─ Canary: 0%

[00:01:00] Canary 진행 시작
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IngressRoute 업데이트
  ├─ Primary: 90%
  └─ Canary: 10%

Load test 시작...
  └─ hey -z 1m -q 50 -c 5 http://payment-service:8080/health

[00:02:00] 메트릭 분석 (1차)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Prometheus 쿼리 실행...

✓ request-success-rate: 99.95% (기준: 99.9%)
✓ request-duration: 150ms (기준: 200ms)

결과: 통과 ✓

HPA 상태 확인
  ├─ Primary: 5 Pods (CPU: 45%)
  └─ Canary: 1 Pod (CPU: 60%) ← 아직 확장 안 함

[00:03:00] 트래픽 증가
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IngressRoute 업데이트
  ├─ Primary: 80%
  └─ Canary: 20%

[00:04:00] 메트릭 분석 (2차)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ request-success-rate: 99.92%
✓ request-duration: 165ms

결과: 통과 ✓

HPA 상태 확인
  ├─ Primary: 5 Pods (CPU: 35%)
  └─ Canary: 2 Pods (CPU: 75%) ← HPA가 확장 시작

[00:05:00] 트래픽 증가
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IngressRoute 업데이트
  ├─ Primary: 70%
  └─ Canary: 30%

[00:06:00] 메트릭 분석 (3차)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ request-success-rate: 99.93%
✓ request-duration: 170ms

결과: 통과 ✓

HPA 상태 확인
  ├─ Primary: 4 Pods (CPU: 40%)
  └─ Canary: 3 Pods (CPU: 70%)

[00:07:00] 트래픽 증가
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IngressRoute 업데이트
  ├─ Primary: 60%
  └─ Canary: 40%

[00:08:00] 메트릭 분석 (4차)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ request-success-rate: 99.91%
✓ request-duration: 175ms

결과: 통과 ✓

HPA 상태 확인
  ├─ Primary: 3 Pods (CPU: 35%)
  └─ Canary: 4 Pods (CPU: 72%)

[00:09:00] 트래픽 증가 (maxWeight 도달)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IngressRoute 업데이트
  ├─ Primary: 50%
  └─ Canary: 50%

maxWeight (50%) 도달
  └─ 자동 진행 일시 중지

[00:10:00] 메트릭 분석 (5차)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ request-success-rate: 99.94%
✓ request-duration: 172ms

결과: 통과 ✓

HPA 상태 확인
  ├─ Primary: 3 Pods (CPU: 40%)
  └─ Canary: 5 Pods (CPU: 68%)

모든 검증 통과 → Promotion 시작

[00:11:00] Promotion 수행
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary Deployment 이미지 교체
  From: payment:v1.0.0
  To:   payment:v2.0.0

Kubernetes Rolling Update 시작...
  ├─ v1.0.0: 3 Pods
  └─ v2.0.0: 1 Pod (생성 중)

[00:11:30] Rolling Update 진행 중
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ├─ v1.0.0: 2 Pods
  └─ v2.0.0: 2 Pods

[00:12:00] Rolling Update 진행 중
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  ├─ v1.0.0: 1 Pod
  └─ v2.0.0: 3 Pods

[00:12:30] Rolling Update 완료
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Primary Deployment
  ├─ image: payment:v2.0.0
  ├─ replicas: 5 (모두 v2.0.0)
  └─ 트래픽: 100%

Canary Deployment
  ├─ Scale down to 0
  └─ 대기 상태

[00:13:00] 배포 완료
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ 배포 성공
✓ 총 소요 시간: 13분
✓ 롤백 없음
✓ 사용자 영향 최소화

알림 발송
  └─ Slack: "payment-service v2.0.0 배포 완료 ✓"
```

---

# 11. 핵심 요약

## Canary 배포 내부 메커니즘

### 1. 시작점

- 사용자가 Canary CRD 생성
- Flagger가 CRD를 watch
- Deployment 변경 감지 시 자동 시작

### 2. Deployment 구성

| Deployment | 역할 | 이미지 | 트래픽 |
|-----------|------|--------|--------|
| **원본** | 템플릿 (사용 안 됨) | 최신 | 0% |
| **Primary** | 실제 운영 서비스 | 안정 버전 | 90% → 0% |
| **Canary** | 새 버전 검증 | 새 버전 | 10% → 100% |

### 3. 트래픽 분기

- **제어**: IngressRoute의 weight 기반
- **실행**: Traefik/Istio 등 Ingress Controller
- **조정**: Flagger가 단계별로 weight 변경

### 4. Pod 수와 트래픽

```
트래픽 비율 (IngressRoute weight)
    ↓
부하 증가
    ↓
HPA 감지
    ↓
Pod 자동 확장
```

**핵심**: 트래픽 비율 ≠ Pod 비율

### 5. 메트릭 검증

- 각 단계마다 메트릭 분석
- 기준 초과 시 자동 롤백
- threshold 도달 시 중단

### 6. Promotion

```
Canary 검증 완료
    ↓
Primary 이미지 교체
    ↓
Kubernetes Rolling Update
    ↓
Canary 정리
```

**핵심**: Canary가 Primary가 되는 것이 아니라, Primary 이미지만 교체

### 7. 실패 시 동작

- threshold 도달 시 자동 롤백
- 트래픽 즉시 Primary로 복귀
- Canary 중단
- Primary는 영향 없음

---

# 결론

Canary 배포는 단순히 "일부 트래픽으로 테스트"하는 것을 넘어, **정교한 내부 메커니즘**을 통해 안전한 배포를 보장한다.

## 핵심 이해

1. **Flagger의 역할**: 오케스트레이터 (실제 트래픽 제어는 Ingress Controller)
2. **Deployment 분리**: Primary (운영) / Canary (검증)
3. **트래픽과 Pod**: 독립적 (HPA가 자동 조정)
4. **Promotion**: 이미지 교체 + Rolling Update
5. **안전장치**: 메트릭 기반 자동 롤백

## 실전 적용 팁

### ✅ DO

- **충분한 interval**: 최소 30초~1분 (메트릭 안정화)
- **적절한 stepWeight**: 10% 권장 (너무 크면 위험)
- **명확한 메트릭**: 성공률, 응답시간, 에러율
- **HPA 설정**: Canary Pod 자동 확장 허용
- **알림 설정**: Slack, Email로 상태 공유

### ❌ DON'T

- **너무 짧은 interval**: 메트릭 수집 전에 판단
- **너무 관대한 threshold**: 문제를 놓칠 수 있음
- **maxWeight 100%**: 전체 자동 배포는 위험
- **HPA 미설정**: Canary가 트래픽 처리 못 함

## 다음 단계

1. **실습 환경 구축**
   - Kubernetes 클러스터 준비
   - Flagger 설치
   - Prometheus 설정
   - Traefik 또는 Istio 설정

2. **테스트 배포**
   - 간단한 애플리케이션으로 시작
   - 메트릭 기준 조정
   - 롤백 시나리오 테스트

3. **프로덕션 적용**
   - 중요도 낮은 서비스부터
   - 메트릭 모니터링 강화
   - 알림 체계 구축
   - 점진적 확대

내부 메커니즘을 정확히 이해하면, Canary 배포를 더 효과적으로 활용하고 문제 상황에서도 빠르게 대응할 수 있다!

---

# 참고 자료

- [Flagger 공식 문서](https://docs.flagger.app/)
- [Kubernetes HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Traefik IngressRoute](https://doc.traefik.io/traefik/routing/providers/kubernetes-crd/)
- [Prometheus 메트릭 수집](https://prometheus.io/docs/introduction/overview/)
- [Progressive Delivery](https://www.weave.works/blog/what-is-progressive-delivery-all-about)
