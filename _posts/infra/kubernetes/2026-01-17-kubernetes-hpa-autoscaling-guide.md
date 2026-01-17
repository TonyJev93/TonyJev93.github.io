---
title: "[Kubernetes] HPA 오토스케일링 실무 가이드 - CPU 기반 스케일링의 이해와 최적화"
last_modified_at: 2026-01-17T16:00:00+09:00
categories:
    - Infra
tags:
    - Kubernetes
    - HPA
    - Autoscaling
    - DevOps
    - Performance
    - EKS
    - Resource Management
toc: true
toc_sticky: true
toc_label: "목차"
---

실무에서 HPA(Horizontal Pod Autoscaler)를 효과적으로 운영하기 위한 CPU target 설정, 파드 수 관리, EKS 환경 최적화 전략을 실전 중심으로 다룬다.
{: .notice--info}

# 들어가며

Kubernetes의 HPA(Horizontal Pod Autoscaler)는 부하에 따라 자동으로 파드 수를 조절하는 강력한 기능이다. 하지만 실무에서는:

- CPU target을 몇 %로 설정해야 할까?
- 파드 수를 많이 늘리면 좋은 건가?
- 왜 스케일링이 예상대로 작동하지 않을까?
- EKS 환경에서 파드가 늘어나면 어떤 문제가 생길까?

이런 질문들에 대한 명확한 답을 찾기 어렵다.

이 글에서는 **CPU 기반 HPA의 본질**을 이해하고, **실무에서 적용 가능한 최적화 전략**을 제시한다. 이론보다는 실전 경험을 바탕으로, HPA를 효과적으로 운영하는 방법을 알아본다.

---

# 1. CPU 기반 HPA의 본질

## HPA는 무엇을 측정하는가?

HPA는 **평균 CPU 사용률**을 기준으로 스케일링을 판단한다.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # ← 핵심 지표
```

## HPA의 판단 기준

HPA는 다음 질문에 답한다:

```
"현재 파드들이 얼마나 바쁜가?"
```

**중요**: HPA는 다음을 **직접 보지 않는다**:
- ❌ 요청 수 (Requests Per Second)
- ❌ 지연 시간 (Latency)
- ❌ 동시 접속자 수
- ❌ 큐 길이

오직 **CPU 사용률의 평균**만 본다.

## 실제 동작 예시

### 시나리오: 3개의 파드

```
Pod 1: CPU 80%
Pod 2: CPU 70%
Pod 3: CPU 60%

평균 CPU: (80 + 70 + 60) / 3 = 70%
```

### HPA의 판단

```
Target: 70%
현재 평균: 70%

판단: 현재 상태 유지 (스케일링 안 함)
```

### 부하 증가 시

```
Pod 1: CPU 90%
Pod 2: CPU 85%
Pod 3: CPU 80%

평균 CPU: (90 + 85 + 80) / 3 = 85%
```

```
Target: 70%
현재 평균: 85%

판단: 스케일 아웃 필요!

계산: 현재 파드 수 × (현재 평균 / 타겟)
     = 3 × (85 / 70)
     = 3 × 1.21
     = 3.64
     → 4 Pods로 증가
```

## HPA의 한계

HPA는 **처리량(Throughput) 대응용 도구**다.

### ✅ HPA가 잘 해결하는 문제

- 갑작스런 트래픽 증가
- 주기적인 부하 패턴 (예: 점심시간 트래픽)
- CPU 집약적 작업

### ❌ HPA가 해결하지 못하는 문제

- **메모리 누수**: 파드를 늘려도 각 파드의 메모리 사용량은 줄지 않음
- **GC 문제**: JVM Old Gen이 꽉 차면 파드 수와 무관
- **데이터베이스 병목**: 파드를 늘려도 DB 커넥션은 한정적
- **싱글 스레드 작업**: 하나의 요청이 오래 걸리는 경우

---

# 2. CPU Target %의 의미와 설정 전략

## CPU Target이란?

**"파드가 유지하려는 평균 부하 수준"**

```yaml
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 70  # ← 이 값의 의미
```

이 설정의 의미:
- HPA는 **평균 CPU를 70%로 유지**하려고 노력한다
- 70%보다 높으면 → 파드 증가
- 70%보다 낮으면 → 파드 감소

## 실무 권장 Target 범위

### 📊 Target 값별 특성

| Target | 성격 | 장점 | 단점 | 사용 케이스 |
|--------|------|------|------|-------------|
| **50-60%** | 매우 보수적 | 여유 많음 | 비용 높음 | 금융, 결제 |
| **65-75%** | 균형형 ✅ | 안정+효율 | - | 일반 서비스 |
| **80-85%** | 공격적 | 비용 절감 | 리스크 높음 | 내부 도구 |
| **90%+** | 매우 공격적 | 최대 절약 | 장애 위험 | 비추천 |

### ✅ 권장: 65-75%

대부분의 프로덕션 환경에서 **70%**가 적절하다.

```yaml
averageUtilization: 70
```

**이유**:
1. **스파이크 대응 여유**: 30%의 여유로 갑작스런 부하 흡수
2. **비용 효율**: 50% 대비 약 30% 리소스 절감
3. **안정성**: 80% 이상 대비 훨씬 안정적

---

# 3. CPU Target이 적절하지 않을 때 문제점

## CPU Target이 너무 낮을 때 (예: 50%)

### 문제 1: 파드 수 급증

```
현재: 5 Pods, 평균 CPU 60%
Target: 50%

HPA 계산: 5 × (60 / 50) = 6 Pods
→ 파드 1개 증가
```

**문제**: 약간의 부하 증가에도 파드가 계속 늘어난다.

### 문제 2: 평균값 희석

```
초기: 5 Pods
  Pod 1: 70%
  Pod 2: 65%
  Pod 3: 60%
  Pod 4: 55%
  Pod 5: 50%
  평균: 60%

HPA가 1개 추가 → 6 Pods
  Pod 1-5: 동일한 부하
  Pod 6: 새로 시작 (CPU 20%)
  새 평균: (60×5 + 20) / 6 = 53%

판단: "아직 Target(50%)보다 높네?" → 계속 증가
```

### 문제 3: 스케일링 반응 둔화

파드가 많아질수록 평균값 변화가 느려진다.

```
10개 파드 중 1개에 부하 증가
→ 평균값 10% 증가

50개 파드 중 1개에 부하 증가
→ 평균값 2% 증가 (감지 어려움)
```

## CPU Target이 너무 높을 때 (예: 85%)

### 문제 1: 스파이크 대응 지연

```
현재: 3 Pods, 평균 CPU 75%
Target: 85%

판단: "아직 여유 있음" → 스케일링 안 함

트래픽 급증
  → 평균 CPU 95%
  → 이제서야 스케일 아웃 시작
  → 새 파드 준비까지 30초~1분 소요
  → 그 동안 요청 지연/실패
```

### 문제 2: Latency 증가

CPU 사용률이 높을수록 응답 시간이 기하급수적으로 증가한다.

```
CPU 50%: 평균 응답 100ms
CPU 70%: 평균 응답 150ms
CPU 85%: 평균 응답 300ms  ← 2배 증가
CPU 95%: 평균 응답 800ms  ← 8배 증가
```

### 문제 3: 장애 전조 감지 실패

```
정상 상황: CPU 85% (Target과 동일)
문제 발생: CPU 88%

차이가 작아서 알람 미발생
→ 문제 인지 늦음
→ 장애로 확대
```

## 실전 사례

### 사례 1: Target 50% 설정의 비극

```
배경: E-commerce 서비스, Target 50%
상황: 정상 트래픽에서도 파드 20개

문제:
  - 불필요한 리소스 낭비
  - 노드 10개 사용 (비용 월 $3000)
  - 로그 모니터링 어려움
  - 배포 시간 5분 소요

개선: Target 70%로 변경
  → 파드 8개로 감소
  → 노드 4개 사용 (비용 월 $1200)
  → 60% 비용 절감
```

### 사례 2: Target 90% 설정의 참사

```
배경: API 서비스, Target 90%
상황: 평상시 CPU 85% 유지

문제:
  - 약간의 부하 증가에도 응답시간 급증
  - 평균 Latency 50ms → 500ms
  - 고객 불만 접수
  - 긴급 Target 70%로 하향

교훈: 비용 절감보다 안정성이 우선
```

---

# 4. 파드 수를 많이 잡았을 때의 문제점

## 일반적인 오해

"파드를 많이 띄우면 트래픽을 더 잘 처리할 수 있다?"

### ❌ 잘못된 접근

```yaml
minReplicas: 20  # ← 무조건 많이?
maxReplicas: 100
```

## 실제 발생하는 문제들

### 문제 1: 리소스 낭비

```
서비스: 일일 트래픽 패턴

오전 6시: 최소 부하 (필요 파드 2개)
  └─ 실제 파드 20개 (90% 유휴)

오후 6시: 최대 부하 (필요 파드 15개)
  └─ 실제 파드 20개 (적정)

결론: 대부분의 시간 동안 불필요한 파드 운영
```

**비용 계산**:
```
불필요한 파드: 평균 15개
파드당 비용: $0.05/시간
일일 낭비: 15 × $0.05 × 24 = $18
월 낭비: $540
```

### 문제 2: 스케줄링 오버헤드 증가

Kubernetes 스케줄러는 파드 배치 시 다양한 요소를 고려한다:

```
파드 10개 배치 시간: ~2초
파드 50개 배치 시간: ~15초
파드 100개 배치 시간: ~45초
```

**영향**:
- 배포 시간 증가
- 오토스케일링 반응 지연
- 노드 장애 시 복구 시간 증가

### 문제 3: 네트워크 오버헤드

Service Discovery, Load Balancing 부하 증가:

```
파드 10개:
  - Endpoints 업데이트: 즉시
  - iptables 규칙: 20개

파드 100개:
  - Endpoints 업데이트: 1-2초 소요
  - iptables 규칙: 200개
  - kube-proxy CPU 사용량 증가
```

### 문제 4: HPA 반응성 저하

평균값 계산의 딜레마:

```
파드 5개에서 1개 부하 증가:
  평균 변화: 20%
  HPA 반응: 빠름

파드 50개에서 1개 부하 증가:
  평균 변화: 2%
  HPA 반응: 느림 또는 무반응
```

### 문제 5: 장애 분석 난이도 증가

```
파드 5개:
  - 로그 확인: 5개 파드만 체크
  - 문제 파드 특정: 쉬움
  - 재배포: 빠름 (30초)

파드 50개:
  - 로그 확인: 50개 파드 전부 확인 필요
  - 문제 파드 특정: 어려움
  - 재배포: 느림 (5분)
  - 로그 스토리지 비용 증가
```

---

# 5. EKS 환경에서 파드 과다의 실제 불이익

## AWS EKS의 특수성

EKS는 AWS의 managed Kubernetes지만, 파드 수 증가 시 AWS 특유의 제약이 발생한다.

### 문제 1: 노드 수 증가로 인한 비용 상승

#### 노드당 파드 수 제한

AWS EKS는 **인스턴스 타입별로 최대 파드 수**가 정해져 있다.

```
t3.medium: 최대 17 Pods
t3.large:  최대 35 Pods
t3.xlarge: 최대 58 Pods
m5.large:  최대 29 Pods
m5.xlarge: 최대 58 Pods
```

#### 비용 계산 예시

```
서비스: 파드 100개 필요

옵션 1: t3.medium ($0.0416/시간)
  - 필요 노드: 100 / 17 = 6개
  - 시간당 비용: $0.25
  - 월 비용: $180

옵션 2: t3.xlarge ($0.1664/시간)
  - 필요 노드: 100 / 58 = 2개
  - 시간당 비용: $0.33
  - 월 비용: $238

옵션 3: 파드 수 최적화 (50개로 감소)
  - t3.large 사용 ($0.0832/시간)
  - 필요 노드: 50 / 35 = 2개
  - 시간당 비용: $0.17
  - 월 비용: $122

절감액: $238 - $122 = $116/월 (49% 절감)
```

### 문제 2: VPC CNI IP 소모 가속

#### AWS CNI의 동작 방식

EKS는 **AWS VPC CNI**를 사용한다. 각 파드는 VPC의 **실제 IP 주소**를 받는다.

```
파드 1개 = VPC IP 1개 사용
```

#### IP 고갈 시나리오

```
서브넷: 10.0.1.0/24 (256 IPs)

사용 가능 IP:
  - 전체: 256개
  - AWS 예약: 5개
  - 노드 3개: 3개
  - 실제 사용 가능: 248개

파드 100개 배포:
  - 남은 IP: 148개
  - 여유: 충분

파드 200개로 증가:
  - 필요 IP: 200개
  - 남은 IP: 48개
  - 여유: 부족

파드 250개로 증가:
  - 필요 IP: 250개
  - 남은 IP: 0개
  - 상태: IP 고갈! 새 파드 생성 불가
```

#### 실제 장애 사례

```
시나리오: 점진적 IP 고갈

1. 정상 운영: 파드 150개
   └─ IP 사용률 60%

2. 트래픽 증가: HPA가 파드 200개로 증가 시도
   └─ IP 사용률 80%

3. 갑작스런 장애: 파드 대량 재시작 필요
   └─ 새 파드 생성 시도
   └─ "insufficient IP addresses" 에러
   └─ 서비스 다운!

근본 원인: 평소 파드를 너무 많이 운영
해결: minReplicas 감소 + 서브넷 확장
```

### 문제 3: ENI 리소스 압박

#### ENI (Elastic Network Interface) 제한

각 노드는 **제한된 수의 ENI**를 가질 수 있다.

```
t3.medium: 최대 3 ENI, ENI당 6 IP = 18 IPs
t3.large:  최대 3 ENI, ENI당 12 IP = 36 IPs
m5.xlarge: 최대 4 ENI, ENI당 15 IP = 60 IPs
```

#### ENI 고갈 문제

```
노드: t3.large (3 ENI, 36 IPs)
파드: 35개 실행 중

상황: 1개 파드 추가 필요
문제: IP는 있지만 ENI 제한으로 할당 불가
결과: 파드 Pending 상태

경고 메시지:
"failed to allocate a private IP address: InsufficientFreeAddressesInSubnet"
```

### 문제 4: conntrack 테이블 고갈

#### conntrack이란?

Linux 커널의 **연결 추적 테이블**. 모든 네트워크 연결을 기록한다.

```
파드 1개당 평균 연결:
  - Upstream 서비스: 10개
  - 외부 API: 5개
  - 데이터베이스: 2개
  총: ~17개 연결

파드 100개:
  100 × 17 = 1,700 연결

파드 500개:
  500 × 17 = 8,500 연결
```

#### conntrack 제한

```
기본 conntrack 최대값: 65,536

파드 많을 때:
  - 연결 수 증가
  - conntrack 테이블 가득 참
  - 새 연결 거부
  - "nf_conntrack: table full, dropping packet" 에러
```

#### 실제 장애

```
증상:
  - 일부 요청이 랜덤하게 실패
  - "connection timeout" 에러
  - 파드는 정상, 네트워크만 문제

원인:
  - 파드 300개 운영
  - conntrack 테이블 포화
  - 새 연결 생성 실패

해결:
  1. 단기: sysctl net.netfilter.nf_conntrack_max 증가
  2. 장기: 파드 수 최적화 (100개로 감소)
```

### 문제 5: 스케줄러 및 컨트롤 플레인 부하

#### kube-scheduler 병목

```
파드 10개 스케줄링: ~1초
파드 100개 스케줄링: ~10초
파드 500개 스케줄링: ~60초+
```

#### API Server 부하

```
파드 많을 때:
  - Endpoints 업데이트 빈번
  - API 요청 급증
  - etcd 부하 증가
  - 전체 클러스터 느려짐
```

#### EKS Control Plane 제한

AWS EKS는 컨트롤 플레인을 관리하지만, **과도한 부하 시 throttling** 발생:

```
경고:
"API rate limit exceeded"
"etcd apply duration exceeded"

영향:
  - kubectl 명령 지연
  - HPA 반응 느림
  - 배포 실패
```

---

# 6. 권장 HPA 운영 전략

## 기본 설정 가이드

### ✅ 권장 설정

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app

  # 파드 수 범위
  minReplicas: 2        # ← 최소 안정 단위
  maxReplicas: 10       # ← 감당 가능한 상한

  # 스케일링 속도 제어
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60    # 1분간 관찰
      policies:
      - type: Percent
        value: 50       # 최대 50%씩 증가
        periodSeconds: 60
      - type: Pods
        value: 2        # 또는 2개씩 증가
        periodSeconds: 60
      selectPolicy: Min # 더 보수적인 정책 선택

    scaleDown:
      stabilizationWindowSeconds: 300   # 5분간 관찰 (신중하게)
      policies:
      - type: Percent
        value: 25       # 최대 25%씩 감소
        periodSeconds: 60

  # 메트릭
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # ← 균형점
```

### minReplicas 설정 전략

**"최소한 안정적으로 서비스할 수 있는 파드 수"**

```
고려 사항:
1. 가용성: 최소 2개 (1개 장애 시 대응)
2. 배포 전략: Rolling Update 시 여유
3. 리소스: 최소 부하에서 필요한 수
4. 비용: 항상 실행되는 파드의 비용

권장:
  - 중요 서비스: 3-5개
  - 일반 서비스: 2-3개
  - 내부 도구: 1-2개
```

#### 예시

```yaml
# 결제 서비스 (고가용성)
minReplicas: 5
maxReplicas: 20

# 일반 API 서비스
minReplicas: 2
maxReplicas: 10

# 내부 관리자 도구
minReplicas: 1
maxReplicas: 3
```

### maxReplicas 설정 전략

**"장애 상황에서 감당 가능한 최대 파드 수"**

```
고려 사항:
1. 노드 용량: 클러스터가 수용 가능한 수
2. DB 커넥션: 파드 × 커넥션 풀 크기
3. 외부 API 제한: Rate limit 고려
4. 비용: 최대 부하 시 예산

계산 방법:
  maxReplicas = 평균 파드 수 × 2~3배
```

#### 위험한 설정

```yaml
# ❌ 나쁜 예
minReplicas: 1
maxReplicas: 100   # 너무 큰 gap

# ✅ 좋은 예
minReplicas: 3
maxReplicas: 15    # 5배 이내
```

## CPU Target 선택 가이드

### 서비스 유형별 권장값

```yaml
# 1. 금융/결제 서비스 (초고가용성)
averageUtilization: 60
# 이유: 여유 충분, 스파이크 대응 우수

# 2. 일반 프로덕션 서비스
averageUtilization: 70
# 이유: 안정성과 비용 효율 균형

# 3. 내부 도구/배치
averageUtilization: 80
# 이유: 비용 우선, 약간의 지연 허용

# 4. 개발/테스트 환경
averageUtilization: 75
# 이유: 프로덕션보다 여유롭게
```

## 메모리 문제는 HPA로 해결하지 않음

### ❌ 잘못된 접근

```
문제: 파드 메모리 사용률 90%
시도: HPA로 파드 증가

결과:
  Old Pod: 메모리 90% (문제 그대로)
  New Pod: 메모리 5% (방금 시작)

판단: "평균 메모리 47.5% → 괜찮네?"

실제: Old Pod는 여전히 OOMKilled 위험
```

### ✅ 올바른 접근

```
메모리 문제 해결 순서:

1. 메모리 누수 확인
   └─ Heap dump 분석
   └─ 프로파일링

2. 리소스 limits 조정
   └─ memory request/limit 증가

3. 애플리케이션 최적화
   └─ 캐시 크기 조정
   └─ GC 튜닝

4. 버티컬 스케일링 고려
   └─ VPA (Vertical Pod Autoscaler)
```

---

# 7. 실전 시나리오별 HPA 설정

## 시나리오 1: E-commerce API (트래픽 패턴 명확)

### 요구사항

```
트래픽 패턴:
  - 오전 9시~11시: 피크 (평상시 대비 3배)
  - 오후 2시~5시: 중간 (평상시 대비 1.5배)
  - 야간: 최소 (평상시 대비 0.3배)

기존 설정: 항상 15개 파드 유지
문제: 야간 리소스 낭비
```

### 최적 HPA 설정

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ecommerce-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ecommerce-api

  minReplicas: 3    # 야간 최소 (기존 15 → 3)
  maxReplicas: 20   # 피크 대응

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 30  # 빠른 대응
      policies:
      - type: Percent
        value: 100    # 2배까지 빠르게 증가
        periodSeconds: 30

    scaleDown:
      stabilizationWindowSeconds: 600 # 10분 관찰
      policies:
      - type: Pods
        value: 1      # 천천히 감소
        periodSeconds: 60
```

### 효과

```
비용 절감 계산:

기존:
  - 파드 15개 × 24시간 = 360 파드-시간
  - 월 비용: 360 × 30 × $0.05 = $540

HPA 적용 후:
  - 피크: 20개 × 6시간 = 120
  - 중간: 10개 × 6시간 = 60
  - 야간: 3개 × 12시간 = 36
  - 총: 216 파드-시간
  - 월 비용: 216 × 30 × $0.05 = $324

절감: $216/월 (40%)
```

## 시나리오 2: 배치 처리 워커 (CPU 집약적)

### 요구사항

```
특성:
  - CPU 집약적 작업 (이미지 처리, 데이터 분석)
  - 큐 기반 (RabbitMQ, SQS)
  - 처리 속도보다 안정성 우선
```

### 최적 HPA 설정

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: batch-worker
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: batch-worker

  minReplicas: 2
  maxReplicas: 30   # CPU 작업은 확장 효과 좋음

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80  # CPU 집약적이므로 높게 설정

  # 추가: 큐 길이 기반 스케일링 (KEDA 사용 시)
  # - type: External
  #   external:
  #     metric:
  #       name: rabbitmq_queue_messages_ready
  #     target:
  #       type: AverageValue
  #       averageValue: "30"  # 큐에 30개 이상이면 확장

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0   # 즉시 확장
      policies:
      - type: Percent
        value: 100    # 빠르게 증가
        periodSeconds: 15

    scaleDown:
      stabilizationWindowSeconds: 300 # 작업 완료 확인
      policies:
      - type: Pods
        value: 2      # 천천히 감소
        periodSeconds: 60
```

### 주의사항

```
배치 워커 HPA 주의점:

1. 작업 중단 방지:
   - terminationGracePeriodSeconds: 600
   - 진행 중인 작업 완료 대기

2. 메모리 관리:
   - 대용량 데이터 처리 시 메모리 증가
   - memory limits 충분히 설정

3. DB 연결 제한:
   - 파드 수 × 커넥션 풀 고려
   - maxReplicas 제한 필수
```

## 시나리오 3: WebSocket 서버 (Stateful)

### 요구사항

```
특성:
  - 실시간 통신 (채팅, 알림)
  - Stateful (연결 유지)
  - 파드 재시작 시 연결 끊김
```

### 주의 깊은 HPA 설정

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: websocket-server
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: websocket-server

  minReplicas: 5    # 최소 연결 분산
  maxReplicas: 20

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 65  # 보수적

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 120 # 2분 관찰
      policies:
      - type: Pods
        value: 1      # 천천히 증가 (연결 재분배)
        periodSeconds: 60

    scaleDown:
      stabilizationWindowSeconds: 600 # 10분 관찰
      policies:
      - type: Pods
        value: 1      # 매우 천천히 감소
        periodSeconds: 120
```

### 연결 드레이닝 설정

```yaml
# Deployment에 추가
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: websocket-server
        lifecycle:
          preStop:
            exec:
              command:
              - /bin/sh
              - -c
              - |
                # 새 연결 거부
                touch /tmp/shutdown
                # 기존 연결 종료 대기
                sleep 50
```

---

# 8. 모니터링 및 알람 설정

## 핵심 메트릭

### 1. HPA 상태

```promql
# HPA desired vs current
kube_horizontalpodautoscaler_status_desired_replicas
kube_horizontalpodautoscaler_status_current_replicas

# HPA 메트릭 값
kube_horizontalpodautoscaler_status_current_metrics_average_utilization
```

### 2. 스케일링 이벤트

```promql
# 스케일 아웃/인 빈도
rate(kube_horizontalpodautoscaler_status_desired_replicas[5m])

# 파드 수 변화
delta(kube_deployment_status_replicas[10m])
```

### 3. 리소스 사용률

```promql
# CPU 사용률 (HPA target과 비교)
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)
  / sum(container_spec_cpu_quota/container_spec_cpu_period) by (pod)
  * 100

# 메모리 사용률
sum(container_memory_working_set_bytes) by (pod)
  / sum(container_spec_memory_limit_bytes) by (pod)
  * 100
```

## 권장 알람 설정

### Prometheus Alert Rules

```yaml
groups:
- name: hpa_alerts
  interval: 30s
  rules:

  # 1. HPA가 max에 도달
  - alert: HPAMaxedOut
    expr: |
      kube_horizontalpodautoscaler_status_current_replicas
        >= kube_horizontalpodautoscaler_spec_max_replicas
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "HPA {{ $labels.horizontalpodautoscaler }} reached max replicas"
      description: "파드 수가 최대치에 도달했습니다. maxReplicas 증가를 고려하세요."

  # 2. HPA가 자주 스케일링
  - alert: HPAFlapping
    expr: |
      changes(kube_horizontalpodautoscaler_status_desired_replicas[30m]) > 10
    labels:
      severity: warning
    annotations:
      summary: "HPA {{ $labels.horizontalpodautoscaler }} is flapping"
      description: "30분간 10회 이상 스케일링이 발생했습니다. CPU target 조정이 필요합니다."

  # 3. CPU 사용률이 target을 지속적으로 초과
  - alert: HighCPUUtilization
    expr: |
      avg(rate(container_cpu_usage_seconds_total[5m])) by (deployment)
        / avg(container_spec_cpu_quota/container_spec_cpu_period) by (deployment)
        > 0.85
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: "High CPU utilization for {{ $labels.deployment }}"
      description: "CPU 사용률이 85%를 10분 이상 유지하고 있습니다."

  # 4. 파드가 Pending 상태
  - alert: PodsPending
    expr: |
      sum(kube_pod_status_phase{phase="Pending"}) by (namespace, pod) > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Pods pending in {{ $labels.namespace }}"
      description: "파드가 5분 이상 Pending 상태입니다. 리소스 부족이나 스케줄링 문제를 확인하세요."
```

## Grafana 대시보드

### 권장 패널

```
1. HPA Overview
   - Current vs Desired Replicas
   - CPU Utilization vs Target
   - Scale Events Timeline

2. Resource Usage
   - CPU Usage per Pod
   - Memory Usage per Pod
   - Network I/O

3. Scaling Metrics
   - Scale Up/Down Count
   - Stabilization Windows
   - Scaling Duration

4. Cost Tracking
   - Total Pod Hours
   - Estimated Cost
   - Resource Efficiency
```

---

# 9. 핵심 요약

## HPA 운영 핵심 원칙

### 1. CPU HPA의 본질

```
✅ HPA는 "현재 파드들이 얼마나 바쁜가"를 본다
✅ 평균 CPU 사용률로 판단
❌ 요청 수, 지연 시간은 직접 보지 않음
❌ 메모리 문제는 해결하지 못함
```

### 2. CPU Target 설정

```
권장: 65-75%
  - 대부분의 서비스: 70%
  - 고가용성 서비스: 60-65%
  - 내부 도구: 75-80%

피해야 할 설정:
  - 50% 이하: 리소스 낭비
  - 85% 이상: 리스크 과다
```

### 3. 파드 수 관리

```
minReplicas:
  - 최소 안정 단위
  - 일반적으로 2-3개
  - 중요 서비스는 3-5개

maxReplicas:
  - 장애 시 감당 가능한 상한
  - minReplicas의 3-5배
  - DB 커넥션 등 외부 제약 고려
```

### 4. EKS 환경 주의사항

```
파드 수 증가의 실제 비용:
  ✗ 노드 수 증가 → 비용 증가
  ✗ VPC IP 소모 → IP 고갈 위험
  ✗ ENI 제한 → 파드 생성 실패
  ✗ conntrack 테이블 → 연결 실패
  ✗ 컨트롤 플레인 부하 → 전체 느려짐

→ 파드 수를 최소화하는 것이 핵심
```

### 5. 메모리 문제 대응

```
HPA로 해결 안 됨:
  - 메모리 누수
  - GC 문제
  - Old Gen Full

올바른 해결책:
  - 애플리케이션 최적화
  - 리소스 limits 조정
  - VPA 사용 고려
```

## 체크리스트

### HPA 설정 전 확인사항

- [ ] 서비스의 트래픽 패턴 파악 완료
- [ ] CPU target 70% 기준으로 설정
- [ ] minReplicas는 최소 안정 단위
- [ ] maxReplicas는 실제 감당 가능한 수
- [ ] behavior 설정으로 급격한 변화 방지
- [ ] DB 커넥션 풀 크기 고려
- [ ] EKS의 경우 IP 가용성 확인
- [ ] 메모리 문제는 별도 해결 계획 수립

### 운영 중 확인사항

- [ ] HPA가 max에 자주 도달하는가?
- [ ] 파드 수가 자주 변경되는가? (flapping)
- [ ] CPU 사용률이 target 근처에서 안정적인가?
- [ ] 스케일 아웃 시간이 적절한가?
- [ ] 비용이 예상 범위 내인가?
- [ ] IP 사용률이 80% 미만인가? (EKS)
- [ ] conntrack 테이블 사용률 확인

---

# 결론

HPA는 단순한 자동 스케일링 도구가 아니라, **비즈니스 요구사항과 기술적 제약을 균형 있게 조절하는 핵심 메커니즘**이다.

## 핵심 교훈

### 1. 만능이 아니다

HPA는 **CPU 기반 처리량 대응**에 강력하지만:
- 메모리 문제는 해결 못 함
- GC 문제는 해결 못 함
- 데이터베이스 병목은 악화시킬 수 있음

### 2. 파드 수는 많다고 좋은 게 아니다

```
적은 파드로 효율적으로 운영 > 많은 파드로 비효율적 운영
```

특히 **EKS에서는 파드 수가 곧 비용과 복잡도**다.

### 3. 70%의 마법

대부분의 경우 **CPU target 70%**가 최적의 균형점이다:
- 안정성 확보
- 비용 효율
- 스파이크 대응
- 운영 편의성

### 4. 측정하고 개선하라

```
1. 초기 설정 (권장값 적용)
2. 모니터링 (메트릭 수집)
3. 분석 (패턴 파악)
4. 조정 (최적화)
5. 반복
```

## 실전 적용 로드맵

### Phase 1: 기본 설정 (1주)

```
1. HPA 리소스 생성
   - CPU target: 70%
   - minReplicas: 2-3
   - maxReplicas: 10

2. 기본 모니터링 설정
   - Prometheus metrics
   - Grafana dashboard

3. 알람 설정
   - HPAMaxedOut
   - HighCPUUtilization
```

### Phase 2: 관찰 및 분석 (2-4주)

```
1. 트래픽 패턴 파악
   - 시간대별 부하
   - 요일별 차이
   - 이벤트 영향

2. 스케일링 이벤트 분석
   - 얼마나 자주 발생하는가?
   - 적절한 타이밍인가?
   - 불필요한 스케일링은?

3. 비용 분석
   - 파드-시간 계산
   - 최적화 여지 파악
```

### Phase 3: 최적화 (2-4주)

```
1. CPU target 미세 조정
   - 안정성 vs 비용 균형
   - 서비스별 차별화

2. behavior 튜닝
   - scaleUp 속도 조정
   - scaleDown 안정화

3. min/max 조정
   - 실제 패턴 반영
   - 여유분 확보
```

### Phase 4: 고도화 (진행 중)

```
1. 다중 메트릭 활용
   - CPU + Custom Metrics
   - KEDA 도입 고려

2. 예측 기반 스케일링
   - CronHPA
   - ML 기반 예측

3. 비용 최적화
   - Spot Instances
   - Cluster Autoscaler 연동
```

## 마지막 조언

```
"파드를 늘리는 것은 쉽다.
 파드를 줄이는 것이 진정한 최적화다."
```

HPA를 통한 오토스케일링은 **"필요한 만큼만, 적절한 시점에"** 리소스를 사용하는 것이 목표다. 무작정 파드를 늘리는 것이 아니라, **최소한의 리소스로 최대의 효율**을 내는 것이 진정한 HPA 마스터의 길이다.

실무에서 만나는 대부분의 문제는 **"너무 많은 파드"**에서 시작된다. CPU target 70%, 적절한 min/max, 그리고 지속적인 모니터링으로 안정적이고 효율적인 HPA 운영을 실현하자!

---

# 참고 자료

- [Kubernetes HPA 공식 문서](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [HPA Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- [AWS EKS Best Practices - Autoscaling](https://aws.github.io/aws-eks-best-practices/karpenter/)
- [AWS VPC CNI](https://github.com/aws/amazon-vpc-cni-k8s)
- [KEDA - Kubernetes Event Driven Autoscaling](https://keda.sh/)
- [VPA - Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
