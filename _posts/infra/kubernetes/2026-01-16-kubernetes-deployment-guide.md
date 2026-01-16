---
title: "[Kubernetes] 배포 전략 완벽 가이드 - Deployment, ArgoCD, Argo Rollouts, Flagger, HPA 한눈에 이해하기"
last_modified_at: 2026-01-16T10:00:00+09:00
categories:
    - Infra
tags:
    - Kubernetes
    - ArgoCD
    - Argo Rollouts
    - Flagger
    - HPA
    - Deployment
    - Canary
    - GitOps
    - DevOps
toc: true
toc_sticky: true
toc_label: "목차"
---

Kubernetes 환경에서 안전하고 효율적인 배포를 위한 핵심 개념들을 쉬운 비유와 함께 정리한다.
{: .notice--info}

# 들어가며

Kubernetes로 애플리케이션을 배포하다 보면 다양한 도구와 개념을 만나게 된다. Deployment, HPA, ArgoCD, Argo Rollouts, Flagger, Canary... 이름만 들어도 복잡해 보이는 이 개념들을 **한눈에 이해할 수 있도록** 정리했다.

각 개념을 **실생활 비유**와 함께 설명하고, **표와 다이어그램**으로 관계를 명확히 하여 쉽게 이해할 수 있도록 구성했다.

---

# 1. Deployment - 기본 배포 단위

## 개념

**Deployment**는 Kubernetes에서 애플리케이션을 배포하는 가장 기본적인 리소스다. 파드(Pod)의 복제본을 몇 개 실행할지, 어떤 컨테이너 이미지를 사용할지 등을 정의한다.

## 비유로 이해하기

> **레스토랑 체인의 지점 관리자**
>
> Deployment는 레스토랑 체인의 지점 관리자와 같다. "서울에 매장 3개를 운영하고, 각 매장은 같은 메뉴(이미지)를 제공한다"라고 정의하면, 관리자가 알아서 3개 매장(파드)을 열고 관리한다.
>
> - 매장 하나가 문을 닫으면(파드 장애) 자동으로 새 매장을 연다
> - 메뉴를 바꾸고 싶으면(이미지 업데이트) 하나씩 순차적으로 교체한다

## 주요 특징

- **선언적 관리**: 원하는 상태를 선언하면 Kubernetes가 알아서 맞춰줌
- **롤링 업데이트**: 무중단으로 새 버전으로 교체
- **롤백**: 문제 발생 시 이전 버전으로 쉽게 복귀
- **복제본 관리**: ReplicaSet을 통해 지정된 수의 파드 유지

## 예시

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3  # 파드 3개 실행
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:v1.0.0
        ports:
        - containerPort: 8080
```

---

# 2. Canary Deployment - 카나리 배포

## 개념

**Canary Deployment**는 새 버전을 전체 사용자가 아닌 **일부 사용자에게만 먼저 배포**하여 안정성을 검증하는 배포 전략이다.

## 비유로 이해하기

> **탄광의 카나리아**
>
> 옛날 광부들은 유독가스를 감지하기 위해 카나리아를 탄광에 먼저 들여보냈다. 카나리아가 괜찮으면 사람도 안전하다고 판단했다.
>
> 마찬가지로 새 버전을 **10%의 사용자에게만 먼저 배포**하여 문제가 없는지 확인한다. 카나리아(10%)가 괜찮으면 점진적으로 50%, 100%로 늘려간다.

## 배포 흐름

```
구버전: ████████████████████ 100%
새버전: (없음)

↓ Canary 시작 (10%)

구버전: ████████████████░░ 90%
새버전: ██ 10%

↓ 문제 없으면 확대 (50%)

구버전: ██████████░░░░░░░░ 50%
새버전: ██████████ 50%

↓ 최종 완료 (100%)

구버전: (종료)
새버전: ████████████████████ 100%
```

## 장점

- **위험 최소화**: 소수 사용자에게만 영향
- **빠른 롤백**: 문제 발견 시 즉시 구버전으로 복귀
- **실사용 테스트**: 실제 프로덕션 환경에서 검증

## 단점

- **복잡도 증가**: 두 버전을 동시에 관리
- **모니터링 필수**: 메트릭을 지속적으로 확인해야 함

---

# 3. HPA - 자동 확장

## 개념

**HPA(Horizontal Pod Autoscaler)**는 CPU, 메모리 사용량 등의 메트릭을 기반으로 **파드 개수를 자동으로 조절**하는 Kubernetes 리소스다.

## 비유로 이해하기

> **식당의 자동 직원 호출 시스템**
>
> 점심시간에 손님이 몰리면(부하 증가) 자동으로 홀 직원을 더 부른다. 손님이 줄어들면(부하 감소) 직원을 퇴근시킨다.
>
> - 최소 인원: 2명 (항상 유지)
> - 최대 인원: 10명 (예산 한계)
> - 기준: 1인당 테이블 3개 담당 (CPU 70%)

## 동작 원리

```
부하 낮음      부하 증가       부하 높음
────────────────────────────────────
파드 2개  →   파드 5개  →   파드 10개
 ██           █████         ██████████
(최소)        (자동 증가)     (최대)

↓ 부하 감소

파드 3개
 ███
(자동 감소)
```

## 예시

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2    # 최소 2개
  maxReplicas: 10   # 최대 10개
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # CPU 70% 목표
```

## 주요 특징

- **자동 스케일링**: 부하에 따라 자동으로 파드 증감
- **비용 최적화**: 필요할 때만 리소스 사용
- **다양한 메트릭**: CPU, 메모리, 커스텀 메트릭 지원
- **쿨다운**: 급격한 변화 방지 (기본 5분)

---

# 4. ArgoCD - GitOps 배포 도구

## 개념

**ArgoCD**는 **Git 저장소를 단일 진실 공급원(Single Source of Truth)**으로 사용하여 Kubernetes 리소스를 자동으로 배포하고 동기화하는 GitOps 도구다.

## 비유로 이해하기

> **건축 설계도를 보고 짓는 자동 건설 시스템**
>
> Git 저장소는 건축 설계도, Kubernetes 클러스터는 실제 건물이다. ArgoCD는 설계도를 보고 자동으로 건물을 짓는 로봇이다.
>
> - 설계도(Git)를 수정하면 자동으로 건물(클러스터)도 변경된다
> - 누군가 건물을 임의로 수정하면(직접 kubectl) 설계도와 다르다고 경고하고 원상복구한다
> - 모든 변경 이력이 Git에 남아 감사(Audit) 가능

## GitOps 워크플로우

```
┌─────────────┐
│   개발자     │
└──────┬──────┘
       │ 1. PR 생성
       ↓
┌─────────────┐
│  Git Repo   │  ← 단일 진실 공급원
└──────┬──────┘
       │ 2. ArgoCD가 감지
       ↓
┌─────────────┐
│   ArgoCD    │  ← 지속적 모니터링
└──────┬──────┘
       │ 3. 자동 배포
       ↓
┌─────────────┐
│ Kubernetes  │
│  Cluster    │
└─────────────┘
```

## 주요 특징

- **선언적 배포**: Git에 정의된 대로 클러스터 상태 유지
- **자동 동기화**: Git 변경 시 자동으로 배포
- **웹 UI**: 시각적으로 배포 상태 확인
- **멀티 클러스터**: 여러 클러스터 통합 관리
- **롤백**: Git 히스토리로 쉽게 롤백

## 장점

| 항목 | 설명 |
|------|------|
| **감사 가능성** | Git 커밋 히스토리로 모든 변경 추적 |
| **재현성** | Git만 있으면 언제든 동일한 환경 구성 |
| **협업** | PR 리뷰를 통한 배포 검토 |
| **보안** | 클러스터 직접 접근 불필요 |

---

# 5. Argo Rollouts - 고급 배포 전략

## 개념

**Argo Rollouts**는 Kubernetes의 기본 Deployment를 확장하여 **Blue/Green, Canary와 같은 고급 배포 전략**을 제공하는 컨트롤러다.

## 비유로 이해하기

> **신제품 출시 전문가**
>
> 일반 매장 관리자(Deployment)는 단순히 제품을 교체하지만, 신제품 출시 전문가(Argo Rollouts)는 다양한 출시 전략을 구사한다.
>
> - **Canary**: 일부 매장에만 먼저 출시
> - **Blue/Green**: 새 매장을 완전히 준비한 후 한 번에 전환
> - **단계적 출시**: 10% → 30% → 50% → 100%로 점진적 확대
> - **자동 롤백**: 판매량(메트릭)이 떨어지면 자동으로 구제품으로 복귀

## Deployment vs Rollout

| 구분 | Deployment | Argo Rollouts |
|------|-----------|---------------|
| **배포 전략** | 롤링 업데이트만 | Canary, Blue/Green, 롤링 |
| **트래픽 제어** | 불가능 | 세밀한 트래픽 분배 |
| **자동 롤백** | 수동 | 메트릭 기반 자동 |
| **분석** | 없음 | Analysis Template 지원 |
| **일시 정지** | 제한적 | 단계별 수동 승인 가능 |

## Canary 배포 예시

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    canary:
      steps:
      - setWeight: 10   # 1단계: 10% 트래픽
      - pause:          # 5분 대기
          duration: 5m
      - setWeight: 30   # 2단계: 30% 트래픽
      - pause:
          duration: 5m
      - setWeight: 50   # 3단계: 50% 트래픽
      - pause:
          duration: 5m
      - setWeight: 100  # 4단계: 100% 트래픽
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:v2.0.0
```

## 주요 기능

### 1. 트래픽 관리

```
기존: 모든 요청이 균등하게 분배
──────────────────────────────
파드1  파드2  파드3  파드4
 25%   25%   25%   25%

Argo Rollouts: 세밀한 트래픽 제어
──────────────────────────────
구버전  구버전  구버전  새버전
  30%    30%    30%    10%
```

### 2. Analysis (분석)

- 메트릭 기반 자동 판단
- Prometheus, Datadog, New Relic 연동
- 성공률, 응답시간, 에러율 등 분석

### 3. Blue/Green 배포

```
Blue (구버전)          Green (신버전)
  ████████              준비 중...
    ↓                      ↓
실제 트래픽           테스트만 수행

↓ 전환

Blue (구버전)          Green (신버전)
 대기 중                 ████████
                       실제 트래픽
```

---

# 6. Flagger - 자동화된 카나리 배포

## 개념

**Flagger**는 메트릭을 분석하여 **카나리 배포를 자동으로 진행하거나 롤백**하는 Progressive Delivery 도구다.

## 비유로 이해하기

> **자동차의 자율주행 시스템**
>
> Argo Rollouts가 반자동(수동 승인 필요)이라면, Flagger는 완전 자율주행이다.
>
> - **센서(메트릭)**로 주변 상황 파악: 에러율, 응답시간, 성공률
> - **자동 가속**: 메트릭이 좋으면 카나리 비율을 자동으로 증가
> - **긴급 제동**: 메트릭이 나빠지면 즉시 롤백
> - **목적지까지 완주**: 문제없으면 100%까지 자동 배포

## 동작 흐름

```
시작: 새 버전 배포
  ↓
┌─────────────────────────┐
│ Canary 10% 배포        │
└───────┬─────────────────┘
        │
        ↓
┌─────────────────────────┐
│ 메트릭 분석 (30초)      │
│ - 에러율 < 1%?          │
│ - 응답시간 < 500ms?     │
└───┬─────────────────┬───┘
    │ OK              │ FAIL
    ↓                 ↓
  증가             자동 롤백
    │
    ↓
┌─────────────────────────┐
│ Canary 30% 배포        │
└───────┬─────────────────┘
        │
      (반복)
        │
        ↓
    100% 완료
```

## 주요 특징

### 1. 자동 진행

- 사람 개입 없이 자동으로 배포 진행
- 설정된 임계값 기반 판단

### 2. 자동 롤백

- 메트릭 악화 시 즉시 이전 버전으로 복귀
- 알림 발송 (Slack, Teams 등)

### 3. A/B 테스팅

- 특정 헤더, 쿠키 기반 트래픽 라우팅
- 실험 그룹별 성능 비교

## 예시

```yaml
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
    interval: 30s        # 30초마다 분석
    threshold: 5         # 5회 연속 실패 시 롤백
    maxWeight: 50        # 최대 50%까지만
    stepWeight: 10       # 10%씩 증가
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

---

# 7. 전체 비교표

## 개념별 요약

| 도구/개념 | 역할 | 비유 | 핵심 기능 |
|-----------|------|------|-----------|
| **Deployment** | 기본 배포 리소스 | 레스토랑 체인 관리자 | 롤링 업데이트, 복제본 관리 |
| **Canary** | 배포 전략 | 탄광의 카나리아 | 단계적 배포, 위험 최소화 |
| **HPA** | 자동 스케일링 | 자동 직원 호출 시스템 | CPU/메모리 기반 파드 조절 |
| **ArgoCD** | GitOps 도구 | 자동 건설 시스템 | Git 기반 선언적 배포 |
| **Argo Rollouts** | 고급 배포 컨트롤러 | 신제품 출시 전문가 | Blue/Green, Canary, 분석 |
| **Flagger** | 자동 Progressive Delivery | 자율주행 시스템 | 메트릭 기반 자동 배포/롤백 |

## 배포 전략 비교

| 전략 | 속도 | 위험도 | 복잡도 | 적합한 상황 |
|------|------|--------|--------|-------------|
| **롤링 업데이트** | 중간 | 중간 | 낮음 | 일반적인 배포 |
| **Blue/Green** | 빠름 | 낮음 | 중간 | 빠른 롤백 필요 |
| **Canary** | 느림 | 매우 낮음 | 높음 | 중요 서비스, 대규모 변경 |
| **A/B 테스트** | 느림 | 낮음 | 높음 | 기능 실험, 비교 분석 |

## 도구별 조합

| 시나리오 | 조합 | 설명 |
|----------|------|------|
| **기본 구성** | Deployment + HPA | 간단한 배포 및 자동 스케일링 |
| **GitOps** | ArgoCD + Deployment + HPA | Git 기반 배포 자동화 |
| **고급 배포** | ArgoCD + Argo Rollouts + HPA | Canary/Blue-Green 배포 |
| **완전 자동화** | ArgoCD + Flagger + HPA | 메트릭 기반 자동 배포/롤백 |

---

# 8. 실전 아키텍처 예시

## 시나리오: 대규모 서비스 배포

```
┌──────────────────────────────────────────────┐
│               개발자                         │
└────────────┬─────────────────────────────────┘
             │
             │ 1. Git Push
             ↓
┌──────────────────────────────────────────────┐
│          GitHub Repository                   │
│  ├─ deployment.yaml                          │
│  ├─ service.yaml                             │
│  └─ rollout.yaml                             │
└────────────┬─────────────────────────────────┘
             │
             │ 2. ArgoCD 감지 및 동기화
             ↓
┌──────────────────────────────────────────────┐
│              ArgoCD                          │
│  "Git과 클러스터 상태 동기화"                  │
└────────────┬─────────────────────────────────┘
             │
             │ 3. Rollout 생성
             ↓
┌──────────────────────────────────────────────┐
│           Argo Rollouts                      │
│  "Canary 배포 시작"                           │
│  ├─ 10% 트래픽                                │
│  ├─ 30% 트래픽                                │
│  ├─ 50% 트래픽                                │
│  └─ 100% 트래픽                               │
└────────────┬─────────────────────────────────┘
             │
             │ 4. 메트릭 수집 및 분석
             ↓
┌──────────────────────────────────────────────┐
│           Prometheus                         │
│  "에러율, 응답시간, CPU 등 모니터링"            │
└────────────┬─────────────────────────────────┘
             │
             │ 5. 부하 감지
             ↓
┌──────────────────────────────────────────────┐
│               HPA                            │
│  "CPU 80% 넘음 → 파드 증가"                   │
│  파드 3개 → 파드 8개                          │
└──────────────────────────────────────────────┘
```

---

# 9. 언제 무엇을 사용할까?

## 상황별 선택 가이드

### 소규모 프로젝트 / 스타트업

```yaml
추천 조합: Deployment + HPA

이유:
- 간단하고 빠른 구성
- Kubernetes 기본 기능만 사용
- 러닝 커브 낮음
```

### 중규모 프로젝트

```yaml
추천 조합: ArgoCD + Deployment + HPA

이유:
- GitOps로 배포 이력 관리
- 협업 시 코드 리뷰 가능
- 롤백 용이
```

### 대규모 프로젝트 / 엔터프라이즈

```yaml
추천 조합: ArgoCD + Argo Rollouts + HPA

이유:
- 고급 배포 전략 필요
- 무중단 배포 필수
- 단계적 검증 필요
```

### Mission Critical 서비스

```yaml
추천 조합: ArgoCD + Flagger + HPA

이유:
- 자동화된 안전장치
- 메트릭 기반 자동 롤백
- 장애 최소화
```

## 의사결정 플로우

```
Q1: GitOps 필요?
    NO → Deployment
    YES ↓

Q2: 고급 배포 전략 필요?
    NO → ArgoCD + Deployment
    YES ↓

Q3: 자동 롤백 필요?
    NO → ArgoCD + Argo Rollouts
    YES ↓

ArgoCD + Flagger
```

---

# 10. 실습 가이드

## 1단계: 기본 Deployment + HPA

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        image: nginx:1.21
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi

---
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: demo-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: demo-app
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

```bash
# 배포
kubectl apply -f deployment.yaml
kubectl apply -f hpa.yaml

# 확인
kubectl get pods
kubectl get hpa
```

## 2단계: ArgoCD 설치 및 연동

```bash
# ArgoCD 설치
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 비밀번호 확인
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# 포트 포워딩
kubectl port-forward svc/argocd-server -n argocd 8080:443

# 웹 UI 접속
# https://localhost:8080
# ID: admin
# PW: 위에서 확인한 비밀번호
```

```yaml
# argocd-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-repo/k8s-manifests
    targetRevision: HEAD
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## 3단계: Argo Rollouts 적용

```bash
# Argo Rollouts 설치
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# CLI 설치
curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
chmod +x kubectl-argo-rollouts-linux-amd64
sudo mv kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
```

```yaml
# rollout.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: demo-app
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 1m}
      - setWeight: 40
      - pause: {duration: 1m}
      - setWeight: 60
      - pause: {duration: 1m}
      - setWeight: 80
      - pause: {duration: 1m}
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```bash
# 배포
kubectl apply -f rollout.yaml

# 대시보드 실행
kubectl argo rollouts dashboard

# 이미지 업데이트 (새 배포 시작)
kubectl argo rollouts set image demo-app demo-app=nginx:1.22

# 상태 확인
kubectl argo rollouts get rollout demo-app --watch

# 수동 승인
kubectl argo rollouts promote demo-app

# 롤백
kubectl argo rollouts undo demo-app
```

---

# 11. 모니터링 및 트러블슈팅

## 주요 확인 사항

### Deployment

```bash
# 상태 확인
kubectl get deployments
kubectl describe deployment <deployment-name>

# 롤아웃 히스토리
kubectl rollout history deployment/<deployment-name>

# 롤백
kubectl rollout undo deployment/<deployment-name>
```

### HPA

```bash
# HPA 상태
kubectl get hpa
kubectl describe hpa <hpa-name>

# 메트릭 확인
kubectl top pods
kubectl top nodes
```

### Argo Rollouts

```bash
# Rollout 상태
kubectl argo rollouts get rollout <rollout-name>

# 라이브 로그
kubectl argo rollouts get rollout <rollout-name> --watch

# 일시 정지
kubectl argo rollouts pause <rollout-name>

# 재개
kubectl argo rollouts resume <rollout-name>

# 중단
kubectl argo rollouts abort <rollout-name>
```

## 일반적인 문제 해결

### 문제 1: HPA가 스케일링하지 않음

```
원인: Metrics Server 미설치 또는 리소스 requests 미설정

해결:
1. Metrics Server 설치 확인
   kubectl get deployment metrics-server -n kube-system

2. 파드의 resources.requests 설정 확인
   kubectl describe pod <pod-name>
```

### 문제 2: ArgoCD 동기화 실패

```
원인: Git 권한 없음, 잘못된 매니페스트

해결:
1. ArgoCD 로그 확인
   kubectl logs -n argocd deployment/argocd-application-controller

2. Git 연결 확인
   argocd repo list

3. 매니페스트 검증
   kubectl apply --dry-run=client -f <manifest>
```

### 문제 3: Rollout이 진행되지 않음

```
원인: 이전 단계 미완료, 메트릭 분석 실패

해결:
1. Rollout 상태 확인
   kubectl argo rollouts get rollout <name>

2. 수동 승인
   kubectl argo rollouts promote <name>

3. 롤백
   kubectl argo rollouts undo <name>
```

---

# 12. 베스트 프랙티스

## Deployment

✅ **DO**
- 항상 리소스 requests/limits 설정
- readinessProbe, livenessProbe 사용
- 적절한 롤링 업데이트 전략 설정 (maxSurge, maxUnavailable)
- 버전 태그 명시 (latest 태그 지양)

❌ **DON'T**
- 프로덕션에서 latest 태그 사용
- 리소스 제한 없이 배포
- Health Check 생략

## HPA

✅ **DO**
- minReplicas를 트래픽 기준선보다 높게 설정
- CPU와 메모리 메트릭 함께 사용
- 스케일 다운 쿨다운 충분히 설정
- 비용을 고려한 maxReplicas 설정

❌ **DON'T**
- minReplicas를 1로 설정 (단일 장애 지점)
- 너무 공격적인 스케일링 (리소스 낭비)
- 메트릭 없이 HPA만 설정

## ArgoCD

✅ **DO**
- Git을 단일 진실 공급원으로 관리
- 환경별 디렉토리 분리 (dev, staging, prod)
- PR 리뷰 프로세스 적용
- 자동 동기화 활성화 (syncPolicy.automated)

❌ **DON'T**
- kubectl로 직접 리소스 수정
- Git과 클러스터 상태 불일치 방치
- 프로덕션에서 selfHeal 없이 운영

## Argo Rollouts

✅ **DO**
- 단계별 pause duration 충분히 설정
- Analysis Template으로 자동 검증
- 중요 서비스는 수동 승인 단계 추가
- 알림 설정 (Slack, Email)

❌ **DON'T**
- 메트릭 분석 없이 자동 진행
- 너무 빠른 단계 전환
- 롤백 계획 없이 배포

## Flagger

✅ **DO**
- 적절한 메트릭 임계값 설정
- 충분한 분석 간격 (interval)
- 점진적인 stepWeight 설정
- 알림 채널 구성

❌ **DON'T**
- 너무 짧은 분석 간격 (부정확한 판단)
- 임계값을 너무 엄격하게 설정
- maxWeight 100% 설정 (위험)

---

# 결론

Kubernetes 배포는 단순히 애플리케이션을 실행하는 것을 넘어, **안전하고 효율적인 배포 전략**이 필요하다.

## 핵심 요약

| 개념 | 한 줄 요약 |
|------|-----------|
| **Deployment** | Kubernetes 기본 배포 단위, 롤링 업데이트 제공 |
| **Canary** | 일부에게만 먼저 배포하여 위험 최소화 |
| **HPA** | 부하에 따라 자동으로 파드 개수 조절 |
| **ArgoCD** | Git을 기준으로 클러스터 상태를 자동 동기화 |
| **Argo Rollouts** | Blue/Green, Canary 등 고급 배포 전략 제공 |
| **Flagger** | 메트릭 기반으로 배포를 자동 진행하거나 롤백 |

## 시작 로드맵

```
1주차: Deployment + HPA
  └─ 기본 배포와 자동 스케일링 익히기

2주차: ArgoCD
  └─ GitOps 워크플로우 구축

3주차: Argo Rollouts
  └─ Canary 배포 전략 적용

4주차: Flagger (선택)
  └─ 완전 자동화 배포 파이프라인
```

## 마지막 조언

- **작게 시작하기**: Deployment부터 시작하여 점진적으로 복잡도 높이기
- **메트릭이 핵심**: HPA든 Flagger든 정확한 메트릭이 있어야 제대로 동작
- **GitOps 권장**: ArgoCD를 사용하면 배포 이력 관리와 협업이 훨씬 쉬워짐
- **자동화는 신중히**: 완전 자동화 전에 충분한 테스트와 모니터링 구축

안전하고 효율적인 Kubernetes 배포로 서비스의 안정성과 개발 생산성을 모두 높여보자!

---

# 참고 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/)
- [ArgoCD 공식 문서](https://argo-cd.readthedocs.io/)
- [Argo Rollouts 공식 문서](https://argoproj.github.io/argo-rollouts/)
- [Flagger 공식 문서](https://flagger.app/)
- [HPA 공식 문서](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [GitOps 소개](https://www.gitops.tech/)
