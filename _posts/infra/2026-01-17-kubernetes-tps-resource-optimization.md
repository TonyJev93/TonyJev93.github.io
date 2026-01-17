---
title: "[Kubernetes] TPS 기준 리소스 최적화 - 실무 사례로 보는 오버스펙 판단과 개선 전략"
last_modified_at: 2026-01-17T14:00:00+09:00
categories:
    - Infra
tags:
    - Kubernetes
    - EKS
    - Performance
    - TPS
    - HPA
    - Resource Optimization
    - Cost
    - API Gateway
    - BFF
toc: true
toc_sticky: true
toc_label: "목차"
---

## 들어가며

Kubernetes 환경에서 애플리케이션을 운영하다 보면 "적정 리소스"를 판단하기가 생각보다 어렵습니다. 특히 처음 설정할 때는 안정성을 우선시하여 과도한 리소스를 할당하는 경우가 많습니다.

이번 글에서는 **실제 운영 중인 API Gateway/BFF 서비스**를 사례로, TPS(Transaction Per Second) 기준으로 리소스를 분석하고 최적화하는 방법을 다룹니다.

**이런 분들에게 추천합니다:**
- Kubernetes에서 리소스 설정에 고민이 있는 분
- HPA 설정의 적정성을 판단하고 싶은 분
- 클라우드 비용 최적화를 고려 중인 분
- TPS 기준 리소스 산정 방법을 알고 싶은 분
{: .notice--info}

---

## 실무 사례: 현재 운영 환경

먼저 분석할 실제 운영 환경의 구성을 살펴보겠습니다.

### 환경 정보

| 항목 | 설정값 |
|------|--------|
| **역할** | API Gateway / BFF (Backend For Frontend) |
| **플랫폼** | AWS EKS (Elastic Kubernetes Service) |
| **평균 TPS** | 400 |
| **최대 TPS** | 600 (피크 시간대) |

### Pod 리소스 설정

```yaml
resources:
  requests:
    cpu: 2000m      # 2 core
    memory: 4Gi
  limits:
    cpu: 2000m
    memory: 4Gi
```

### HPA (Horizontal Pod Autoscaler) 설정

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 20
  maxReplicas: 30
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

### 현재 설정의 구조적 특징

현재 설정을 분석하면 다음과 같은 특징이 보입니다:

✅ **장점:**
- 많은 파드 수로 부하 희석
- CPU 여유율 확보로 안정성 확보
- 급격한 트래픽 증가 대응 가능

❌ **단점:**
- 과도한 리소스 할당
- 높은 인프라 비용
- 운영 복잡도 증가
- EKS 리소스 (IP/ENI) 소모

---

## 파드당 처리량 분석

실제 각 파드가 얼마나 부하를 처리하고 있는지 계산해보겠습니다.

### 평균 TPS 기준

```
평균 TPS: 400
최소 파드 수: 20

파드당 처리량 = 400 TPS ÷ 20 pods = 20 TPS/pod
```

### 최대 TPS 기준

```
최대 TPS: 600
최대 파드 수: 30

파드당 처리량 = 600 TPS ÷ 30 pods = 20 TPS/pod
```

### 분석 결과

```
┌─────────────────────────────────────────┐
│  현재: 20 TPS/pod                        │
│                                         │
│  일반적인 Gateway/BFF 처리 능력:        │
│  40~70 TPS/pod                          │
│                                         │
│  👉 파드당 부하가 매우 낮은 상태         │
└─────────────────────────────────────────┘
```

**핵심 포인트:**
파드당 20 TPS는 Gateway/BFF 애플리케이션의 일반적인 처리 능력(40~70 TPS)의 절반 이하 수준입니다.
{: .notice--warning}

---

## CPU 관점 상세 분석

### 전체 CPU 용량 계산

```
파드당 CPU: 2 core
최소 파드 수: 20

총 CPU 용량 = 2 core × 20 pods = 40 core
```

### HPA 기준 CPU 사용 허용치

```
총 CPU: 40 core
HPA Target: 60%

허용 CPU 사용량 = 40 core × 60% = 24 core
```

### CPU 사용 현황 시각화

```
┌─────────────────────────────────────────────────────────┐
│ 총 CPU 용량: 40 core                                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│ [========================================] 60% (24 core) │
│                                                          │
│ 👆 HPA 스케일 아웃 기준점                                 │
│                                                          │
│ 실제 사용률 추정: ~30% (12 core)                         │
│ [===================                     ]               │
│                                                          │
│ 유휴 리소스: ~28 core (70%)                              │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 분석 결과

400 TPS를 처리하는 데 24 core를 항상 예약해두는 것은 과도한 리소스 할당입니다.

---

## 오버스펙 판단 근거

### 1. 애플리케이션 특성 관점

#### API Gateway / BFF의 일반적인 특성

```java
// Gateway/BFF는 주로 이런 작업을 수행
public class ApiGatewayController {

    @GetMapping("/api/users/{id}")
    public UserResponse getUser(@PathVariable Long id) {
        // 1. 요청 검증 (경량)
        validateRequest(id);

        // 2. 라우팅 (경량)
        String targetService = routingService.resolve("/users");

        // 3. 외부 API 호출 (I/O 대기)
        UserDto user = restTemplate.getForObject(
            targetService + "/users/" + id,
            UserDto.class
        );

        // 4. 응답 변환 (경량)
        return mapper.toResponse(user);
    }
}
```

**특징:**
- CPU 집약적 작업이 적음
- I/O 대기 시간이 주를 이룸
- 메모리 안정성이 더 중요
- **1 pod당 40~70 TPS 처리 충분히 가능**

### 2. 리소스 활용률 관점

| 구분 | 현재 설정 | 실제 필요 | 과할당률 |
|------|----------|----------|---------|
| **파드 수 (평균)** | 20 | 6~8 | **2.5~3.3배** |
| **총 CPU** | 40 core | 12~16 core | **2.5~3.3배** |
| **파드당 TPS** | 20 | 50~67 | **50% 미만 활용** |

### 3. GC 문제를 파드 확장으로 해결한 형태

많은 경우, 이런 오버스펙 설정은 **메모리 관리 문제를 파드 수로 우회**한 결과입니다.

```
┌──────────────────────────────────────────────────────────┐
│ 문제 상황                                                 │
├──────────────────────────────────────────────────────────┤
│ "파드가 GC로 인해 응답이 느려져요"                        │
│ "메모리 사용률이 80%를 넘어요"                            │
│ "Old GC가 자주 발생해요"                                  │
└──────────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────────┐
│ 잘못된 해결 방법                                          │
├──────────────────────────────────────────────────────────┤
│ "파드를 20개로 늘려서 파드당 부하를 줄이자"               │
│                                                          │
│ ❌ 근본 원인(GC 튜닝, 메모리 누수) 미해결                 │
│ ❌ 비용만 증가                                            │
└──────────────────────────────────────────────────────────┘
                        ↓
┌──────────────────────────────────────────────────────────┐
│ 올바른 해결 방법                                          │
├──────────────────────────────────────────────────────────┤
│ 1. JVM 힙 메모리 튜닝                                     │
│ 2. GC 알고리즘 최적화 (G1GC, ZGC)                         │
│ 3. 메모리 누수 분석 및 해결                               │
│ 4. 적절한 파드 수로 운영                                  │
└──────────────────────────────────────────────────────────┘
```

**참고:** GC 최적화에 대한 자세한 내용은 [[Java] GC(Garbage Collection) 핵심 개념]({% post_url /java/2026-01-17-java-gc-core-concepts %}) 글을 참고하세요.
{: .notice--info}

---

## EKS 관점에서의 문제점

과도한 파드 수는 EKS 환경에서 여러 문제를 야기합니다.

### 1. 노드 수 및 비용 증가

```
┌─────────────────────────────────────────────────────────┐
│ 현재 구성 (20 pods, 2 core each)                        │
├─────────────────────────────────────────────────────────┤
│ 노드 타입: m5.2xlarge (8 vCPU)                          │
│                                                         │
│ 필요 노드 수: 최소 6개                                   │
│ (40 core ÷ 8 core/node = 5, 여유분 포함)                │
│                                                         │
│ 월 비용: 약 $1,152                                      │
│ (노드당 $0.384/시간 × 6 × 730시간)                      │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│ 최적화 구성 (8 pods, 2 core each)                       │
├─────────────────────────────────────────────────────────┤
│ 노드 타입: m5.2xlarge (8 vCPU)                          │
│                                                         │
│ 필요 노드 수: 2~3개                                      │
│ (16 core ÷ 8 core/node = 2, 여유분 포함)                │
│                                                         │
│ 월 비용: 약 $561                                        │
│ (노드당 $0.384/시간 × 3 × 730시간)                      │
│                                                         │
│ 💰 절감액: 약 $591/월 (51% 절감)                         │
└─────────────────────────────────────────────────────────┘
```

### 2. IP/ENI 리소스 소모

```
AWS VPC CNI 제약사항:
┌─────────────────────────────────────────────┐
│ m5.2xlarge 인스턴스                         │
│ - ENI 최대: 4개                             │
│ - ENI당 IP: 15개                            │
│ - 총 사용 가능 IP: 58개                     │
│   (4 × 15 - 노드용 IP)                     │
└─────────────────────────────────────────────┘

현재: 20 pods → 노드당 IP 소모가 빠름
최적: 8 pods → IP 리소스 효율적 사용
```

### 3. HPA 반응성 저하

```yaml
# 현재 설정
minReplicas: 20  # 항상 20개 파드 운영
maxReplicas: 30  # 50% 증가 여유분

# 문제점
# 1. 평소에도 20개 파드 유지 (비용 ↑)
# 2. 스케일 아웃 여지가 적음 (20→30, 1.5배)
# 3. 급격한 트래픽 증가 시 대응력 ↓
```

```yaml
# 최적화 설정
minReplicas: 6   # 평소 6개 파드
maxReplicas: 15  # 2.5배 증가 가능

# 장점
# 1. 평소 비용 절감
# 2. 스케일 아웃 여유 큼 (6→15, 2.5배)
# 3. 유연한 대응 가능
```

### 4. 장애 분석 난이도 증가

```
파드가 많을수록 발생하는 문제:

1. 로그 분석
   - 20개 파드 × N개 로그 = 로그 양 폭증
   - 어느 파드에서 문제가 발생했는지 추적 어려움

2. 모니터링
   - 그래프가 복잡해짐
   - 개별 파드 상태 추적 어려움

3. 디버깅
   - 특정 파드 선택 어려움
   - 재현 어려움 증가

4. 배포
   - 롤링 업데이트 시간 증가
   - 배포 실패 시 롤백 시간 증가
```

---

## 더 효율적인 대안 구조

현재 TPS(400~600)를 기준으로 최적화된 구성을 제안합니다.

### 권장 설정

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  replicas: 8  # HPA가 관리, 여기서는 초기값
  template:
    spec:
      containers:
      - name: api-gateway
        resources:
          requests:
            cpu: 1500m       # 1.5 core (현재 2 core에서 조정)
            memory: 3Gi      # 3Gi (현재 4Gi에서 조정)
          limits:
            cpu: 2000m       # 버스트 허용
            memory: 4Gi      # OOM 방지
        env:
        - name: JAVA_OPTS
          value: >-
            -Xms2g -Xmx2g
            -XX:+UseG1GC
            -XX:MaxGCPauseMillis=200
            -XX:+ParallelRefProcEnabled
---
# HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 6              # 평소: 6개 (현재: 20개)
  maxReplicas: 15             # 최대: 15개 (현재: 30개)
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # 70% (현재: 60%)
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 75    # 메모리 기준 추가
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # 5분 안정화
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60   # 1분 안정화
      policies:
      - type: Percent
        value: 100                     # 빠른 스케일 업
        periodSeconds: 60
```

### 설정 변경 근거

| 항목 | 기존 | 제안 | 근거 |
|------|------|------|------|
| **Min Replicas** | 20 | 6 | 파드당 67 TPS 처리 가능 (400÷6) |
| **Max Replicas** | 30 | 15 | 파드당 40 TPS 처리 (600÷15) |
| **CPU Request** | 2 core | 1.5 core | Gateway 특성상 CPU 집약도 낮음 |
| **CPU Limit** | 2 core | 2 core | 버스트 대응 여유 유지 |
| **Memory Request** | 4Gi | 3Gi | 실제 사용률 기반 조정 |
| **HPA CPU Target** | 60% | 70% | 더 효율적인 리소스 사용 |
| **HPA Metric** | CPU만 | CPU + Memory | 메모리 기반 스케일링 추가 |

### 예상 효과

```
┌────────────────────────────────────────────────────────────┐
│ 리소스 비교                                                 │
├────────────────────────────────────────────────────────────┤
│                    현재        최적화       절감률          │
│ 평균 파드 수       20개        6개          70% ↓          │
│ 총 CPU            40 core    9 core        77.5% ↓        │
│ 총 Memory         80Gi       18Gi          77.5% ↓        │
│ 월 비용           $1,152     $280          75.7% ↓        │
│                                                            │
│ 성능 지표                                                   │
│ 파드당 TPS        20         67            235% ↑         │
│ 전체 처리 능력    600        ~400          유지            │
│ 스케일 여유       1.5배      2.5배         66% ↑          │
└────────────────────────────────────────────────────────────┘
```

---

## 안정성과 비용의 균형

### 현재 설정 = "안정성 과잉 투자"

```
안정성 스펙트럼:

과소 투자 ←──────────────[현재]──→ 적정 투자
                         ↑
                    여기가 적절

비용:   낮음 ←───────────────────→ 높음
안정성: 낮음 ←───────────────────→ 높음
```

### 안정성 투자가 정당화되는 경우

다음 조건에 해당한다면 현재의 높은 파드 수 설정이 합리적일 수 있습니다:

✅ **정당화 가능한 케이스:**

1. **트래픽 급등이 빈번한 경우**
   ```
   예: 쇼핑몰의 특가 이벤트
   - 평소 400 TPS → 순간 5,000 TPS
   - 빠른 스케일 아웃 필수
   ```

2. **SLA가 매우 엄격한 경우**
   ```
   예: 금융/결제 시스템
   - 99.99% 가용성 요구
   - 장애 시 법적/재무적 리스크
   ```

3. **장애 민감도가 극도로 높은 경우**
   ```
   예: 실시간 의료 시스템
   - 서비스 중단이 생명과 직결
   ```

❌ **정당화 어려운 케이스:**

1. **트래픽 패턴이 일정한 일반적인 API**
2. **내부 관리 도구나 백오피스 시스템**
3. **개발/스테이징 환경**

### 의사결정 프레임워크

```
┌─────────────────────────────────────────────────────────┐
│ 질문 1: 트래픽이 평소 대비 2배 이상 급증하나?           │
│         YES → 다음 질문                                 │
│         NO  → 최적화 권장                               │
├─────────────────────────────────────────────────────────┤
│ 질문 2: SLA가 99.9% 이상인가?                           │
│         YES → 다음 질문                                 │
│         NO  → 최적화 권장                               │
├─────────────────────────────────────────────────────────┤
│ 질문 3: 장애 시 비즈니스 손실이 월 $1,000 이상인가?     │
│         YES → 현재 설정 유지 가능                       │
│         NO  → 최적화 권장                               │
└─────────────────────────────────────────────────────────┘
```

---

## 최적화 실행 계획

### 1단계: 현황 파악 (1주)

```bash
# 1. 실제 리소스 사용률 수집
kubectl top pods -n production --sort-by=cpu
kubectl top pods -n production --sort-by=memory

# 2. HPA 이벤트 확인
kubectl describe hpa api-gateway-hpa -n production

# 3. 메트릭 수집 (Prometheus 쿼리 예시)
# CPU 사용률
rate(container_cpu_usage_seconds_total{pod=~"api-gateway.*"}[5m])

# Memory 사용률
container_memory_working_set_bytes{pod=~"api-gateway.*"}

# 파드당 처리량
rate(http_requests_total{pod=~"api-gateway.*"}[1m])
```

### 2단계: 비프로덕션 환경 테스트 (1주)

```yaml
# dev 환경에 최적화 설정 적용
kubectl apply -f optimized-hpa.yaml -n dev

# 부하 테스트
# k6, Gatling, JMeter 등 활용
```

```javascript
// k6 부하 테스트 예시
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 400 },  // 평균 부하
    { duration: '5m', target: 400 },  // 유지
    { duration: '2m', target: 600 },  // 피크 부하
    { duration: '5m', target: 600 },  // 유지
    { duration: '2m', target: 0 },    // 종료
  ],
};

export default function () {
  let res = http.get('http://api-gateway.dev/api/health');
  check(res, {
    'status is 200': (r) => r.status === 200,
    'response time < 200ms': (r) => r.timings.duration < 200,
  });
}
```

### 3단계: 프로덕션 단계적 적용 (2주)

```bash
# 1. 카나리 배포 (10% 트래픽)
kubectl apply -f optimized-hpa-canary.yaml -n production

# 2. 모니터링 (3일)
# - CPU/Memory 사용률
# - 응답 시간
# - 에러율
# - HPA 스케일링 빈도

# 3. 문제 없으면 50% 확대
kubectl patch hpa api-gateway-hpa -n production \
  --patch '{"spec":{"minReplicas":12}}'

# 4. 최종 100% 적용
kubectl apply -f optimized-hpa.yaml -n production
```

### 4단계: 모니터링 및 튜닝 (지속)

```yaml
# Prometheus Alert 설정
groups:
- name: api-gateway-optimization
  rules:
  - alert: HighCPUUsage
    expr: |
      avg(rate(container_cpu_usage_seconds_total{pod=~"api-gateway.*"}[5m])) > 0.8
    for: 5m
    annotations:
      summary: "API Gateway CPU 사용률 80% 초과"

  - alert: HighMemoryUsage
    expr: |
      avg(container_memory_working_set_bytes{pod=~"api-gateway.*"} /
          container_spec_memory_limit_bytes{pod=~"api-gateway.*"}) > 0.85
    for: 5m
    annotations:
      summary: "API Gateway Memory 사용률 85% 초과"

  - alert: HPAMaxedOut
    expr: |
      kube_horizontalpodautoscaler_status_current_replicas{hpa="api-gateway-hpa"} >=
      kube_horizontalpodautoscaler_spec_max_replicas{hpa="api-gateway-hpa"}
    for: 5m
    annotations:
      summary: "HPA 최대 파드 수 도달"
```

---

## 핵심 요약

### 1. TPS 대비 과한 파드 수는 비효율

```
현재: 20 TPS/pod (처리 능력의 30% 미만 활용)
권장: 50~67 TPS/pod (적정 활용률)
```

### 2. Gateway/BFF는 메모리 안정성이 우선

```
중요도: Memory 관리 > CPU 최적화

✅ 해야 할 것:
- JVM 힙 메모리 적절히 설정
- GC 알고리즘 최적화
- 메모리 누수 모니터링

❌ 하지 말아야 할 것:
- 메모리 문제를 파드 수로 해결
```

### 3. 파드 수는 처리량 한계 도달 시 확장

```
스케일링 기준:

1. CPU 사용률 70% 이상 지속
2. 응답 시간 증가 추세
3. 처리 대기 큐 증가

NOT: "일단 많이 띄워두자"
```

### 4. EKS에서 파드 수 = 비용 + 운영 복잡도

```
파드 수 증가 시 고려사항:

- 노드 수 증가 → 비용 ↑
- IP 리소스 소모 → VPC 설계 영향
- 로그/모니터링 복잡도 ↑
- 배포 시간 ↑
```

### 실무 적용 체크리스트

- [ ] 현재 파드당 TPS 계산
- [ ] 애플리케이션 유형별 적정 TPS 확인 (Gateway: 40~70)
- [ ] CPU/Memory 실제 사용률 측정
- [ ] HPA 스케일링 이력 분석
- [ ] 트래픽 패턴 및 급증 빈도 파악
- [ ] SLA 요구사항 확인
- [ ] 비프로덕션 환경 테스트
- [ ] 단계적 프로덕션 적용
- [ ] 지속적 모니터링 체계 구축

---

## 참고 자료

- [Kubernetes HPA 공식 문서](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [AWS EKS Best Practices - Cluster Autoscaling](https://aws.github.io/aws-eks-best-practices/cluster-autoscaling/)
- [AWS VPC CNI - IP Address Management](https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html)
- [[Java] GC(Garbage Collection) 핵심 개념]({% post_url /java/2026-01-17-java-gc-core-concepts %})
- [[Infra] CPU, RPS/TPS, GC의 관계 완벽 이해]({% post_url /infra/2026-01-17-cpu-rps-tps-gc-relationship %})

---

**다음 글 예고:** "Kubernetes 리소스 최적화 Part 2 - Vertical Pod Autoscaler (VPA) 활용 전략"
{: .notice--success}
