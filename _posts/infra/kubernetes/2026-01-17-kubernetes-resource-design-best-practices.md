---
title: "[Kubernetes] 실무 리소스 설계 완벽 가이드 - CPU/Memory/Pod 최적화 전략"
last_modified_at: 2026-01-17T16:00:00+09:00
categories:
    - Infra
tags:
    - Kubernetes
    - Resource
    - Memory
    - CPU
    - Pod
    - GC
    - HPA
    - Performance
    - Optimization
toc: true
toc_sticky: true
toc_label: "목차"
---

Kubernetes 환경에서 애플리케이션의 CPU, Memory, Pod 수를 어떻게 설정해야 하는지 실무 관점에서 명확한 기준과 전략을 제시한다.
{: .notice--info}

# 들어가며

Kubernetes에서 애플리케이션을 운영하다 보면 가장 자주 마주치는 질문들이 있다:

- "CPU를 늘려야 할까, 메모리를 늘려야 할까?"
- "파드 수를 늘리면 GC 문제가 해결될까?"
- "API Gateway는 어떤 리소스 전략을 써야 할까?"

이런 질문들에 대한 답은 단순히 "더 많이 주면 된다"가 아니다. **왜 그런 문제가 발생했는지**, **어떤 리소스가 실제 병목인지**를 이해해야 올바른 결정을 내릴 수 있다.

이 글에서는 실무에서 자주 겪는 리소스 설계 상황들을 바탕으로, **CPU, Memory, Pod 수를 언제, 어떻게 조정해야 하는지** 명확한 기준을 제시한다.

---

# 1. 메모리 vs 파드 수 결정 기준

## CPU 사용률 패턴 분석

리소스 증설 결정의 첫 단계는 **CPU 사용률 패턴을 분석**하는 것이다.

### Case 1: CPU 사용률이 높고 GC 빈도/시간이 증가

**증상**
```
CPU 사용률: 70~90% (지속)
GC 빈도: 초당 2~3회 (정상: 0.5회 이하)
GC 시간: 200~500ms (정상: 50ms 이하)
응답 시간: 점진적 증가
```

**원인 분석**

힙 메모리가 부족하여 Young GC가 빈번하게 발생하고, Old 영역이 빠르게 채워지면서 CPU를 과도하게 사용하는 상황이다.

```
메모리 부족
    ↓
Young Generation 빠른 포화
    ↓
Young GC 빈번 발생
    ↓
Old Generation으로 승격 증가
    ↓
Old 영역 포화
    ↓
Full GC 발생
    ↓
CPU 급증
```

**해결 방법**

✅ **메모리 증설 우선**

```yaml
# Before
resources:
  requests:
    memory: 512Mi
  limits:
    memory: 512Mi

# After
resources:
  requests:
    memory: 1Gi  # 2배 증설
  limits:
    memory: 1Gi
```

**효과**
- Young Generation 크기 증가 → GC 빈도 감소
- Old Generation 여유 확보 → Full GC 지연
- CPU 사용률 정상화 (30~50%)

---

### Case 2: CPU 사용률이 트래픽에 비례해 선형 증가

**증상**
```
트래픽 증가: 1000 → 2000 req/s
CPU 사용률: 50% → 100%
GC 빈도: 정상 유지
응답 시간: 증가 (큐잉 발생)
```

**원인 분석**

메모리나 GC 문제가 아닌, **순수 처리량 한계**에 도달한 상황이다.

```
트래픽 증가
    ↓
요청 처리 스레드 포화
    ↓
CPU 100% 도달
    ↓
추가 요청 대기 (큐잉)
    ↓
응답 지연 증가
```

**해결 방법**

✅ **파드 수 증설 우선**

```yaml
# HPA 설정
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # CPU 70% 초과 시 스케일 아웃
```

**효과**
- 부하 분산으로 CPU 사용률 정상화
- 처리량 증가
- 응답 시간 개선

---

### Case 3: 트래픽이 줄어도 CPU가 내려가지 않음

**증상**
```
트래픽 감소: 2000 → 500 req/s
CPU 사용률: 80% 유지 (변동 없음)
메모리 사용: 꾸준히 증가
```

**원인 분석**

Old GC 또는 백그라운드 작업이 CPU를 지속적으로 점유하는 상황이다.

**가능한 원인들**

1. **메모리 누수**
```java
// 잘못된 코드 예시
private static final Map<String, Object> cache = new HashMap<>();

public void cacheData(String key, Object value) {
    cache.put(key, value);  // 계속 누적, 삭제 안 됨
}
```

2. **Old GC 빈발**
```
Old Generation 지속 포화 (>90%)
    ↓
Full GC 반복 발생 (수 초마다)
    ↓
CPU 지속 사용
```

3. **백그라운드 작업**
```java
// 문제가 되는 패턴
@Scheduled(fixedDelay = 100)  // 100ms마다 실행
public void heavyTask() {
    // 무거운 작업
    processLargeDataSet();
}
```

**해결 방법**

✅ **메모리 증설 또는 코드 개선**

**1단계: 메모리 프로파일링**
```bash
# Heap dump 생성
kubectl exec -it <pod-name> -- jmap -dump:format=b,file=/tmp/heap.hprof 1

# 로컬로 복사
kubectl cp <pod-name>:/tmp/heap.hprof ./heap.hprof

# VisualVM, Eclipse MAT 등으로 분석
```

**2단계: 문제 패턴 수정**
```java
// 개선된 코드
private final Map<String, Object> cache = new ConcurrentHashMap<>();
private static final int MAX_CACHE_SIZE = 1000;

public void cacheData(String key, Object value) {
    if (cache.size() >= MAX_CACHE_SIZE) {
        // LRU 정책으로 오래된 항목 제거
        cache.remove(cache.keySet().iterator().next());
    }
    cache.put(key, value);
}
```

**3단계: 백그라운드 작업 최적화**
```java
// 개선: 실행 빈도 조정
@Scheduled(fixedDelay = 60000)  // 1분마다로 변경
public void heavyTask() {
    // 배치 크기 제한
    processDataSetInBatches(1000);
}
```

---

## 의사결정 플로우차트

```
CPU 사용률이 높은가?
    ├─ YES
    │   ├─ GC 빈도/시간이 비정상인가?
    │   │   ├─ YES → 메모리 증설
    │   │   └─ NO → 트래픽에 비례하는가?
    │   │       ├─ YES → 파드 수 증설
    │   │       └─ NO → 트래픽과 무관한가?
    │   │           └─ YES → 메모리 또는 코드 개선
    │   └─ 결정 완료
    └─ NO → 현상 유지 또는 다른 병목 확인
```

---

# 2. API Gateway / BFF의 리소스 성향

API Gateway와 BFF(Backend For Frontend)는 일반적인 백엔드 서비스와 다른 리소스 특성을 가진다.

## 리소스 사용 패턴 분석

### CPU 사용 특성

**주요 CPU 소비 작업**

1. **데이터 변환 (Transformation)**
```java
// API Gateway의 전형적인 패턴
public ResponseDTO transform(InternalResponse internal) {
    return ResponseDTO.builder()
        .userId(internal.getUserId())
        .userName(internal.getUserName())
        .orders(internal.getOrders().stream()
            .map(this::convertOrder)  // 변환 작업
            .collect(Collectors.toList()))
        .build();
}
```

2. **직렬화/역직렬화 (Serialization)**
```java
// JSON 변환이 빈번
String json = objectMapper.writeValueAsString(response);  // CPU 소비
ResponseDTO dto = objectMapper.readValue(json, ResponseDTO.class);
```

3. **라우팅 및 필터링**
```java
// Spring Cloud Gateway
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    // 헤더 검증, 인증, 로깅 등
    return chain.filter(exchange)
        .then(Mono.fromRunnable(() -> {
            // 응답 후처리
        }));
}
```

**권장 CPU 설정**

```yaml
resources:
  requests:
    cpu: 500m      # 중간 이상 필요
  limits:
    cpu: 1000m     # burst 허용
```

---

### 메모리 사용 특성

**메모리 압박 요인**

1. **다수의 동시 연결**
```
1000 req/s × 평균 응답시간 100ms
= 100개의 동시 처리 요청
= 각 요청당 메모리 할당
```

2. **응답 버퍼링**
```java
// Reactive 스트림에서의 버퍼
Flux<DataBuffer> body = response.getBody();
// 메모리에 버퍼링될 수 있음
```

3. **빈번한 객체 생성**
```java
// 요청마다 새로운 객체 생성
for (Request req : requests) {
    ResponseDTO dto = new ResponseDTO();  // Young Gen 압박
    // ... 처리
}
```

**GC 안정성을 위한 메모리 설정**

```yaml
resources:
  requests:
    memory: 1Gi    # 충분한 Young Generation 확보
  limits:
    memory: 1Gi

# JVM 옵션
env:
- name: JAVA_OPTS
  value: |
    -Xms1g -Xmx1g
    -XX:+UseG1GC
    -XX:MaxGCPauseMillis=200
    -XX:InitiatingHeapOccupancyPercent=45
```

**왜 메모리가 중요한가?**

```
요청 증가
    ↓
Young Generation 빠른 포화
    ↓
Young GC 빈번 발생
    ↓
GC 중 "Stop-The-World" 발생
    ↓
응답 지연 발생 (tail latency 증가)
    ↓
사용자 체감 성능 저하
```

---

### Pod 수 전략

**과도한 파드 수가 불필요한 이유**

1. **Gateway는 Stateless**
   - 세션 저장 불필요
   - 단순 요청 전달 및 변환

2. **수평 확장보다 수직 확장이 효율적**
   - 파드 수 증가 = 네트워크 홉 증가
   - 파드 간 부하 분산 오버헤드

3. **비용 효율**
   - 적은 수의 파드에 충분한 리소스 할당이 더 경제적

**권장 설정**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
spec:
  replicas: 3  # 최소한의 HA 보장
  template:
    spec:
      containers:
      - name: gateway
        resources:
          requests:
            cpu: 500m
            memory: 1Gi  # 메모리 여유 확보
          limits:
            cpu: 2000m   # burst 허용
            memory: 1Gi

---
# HPA 설정
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-gateway
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-gateway
  minReplicas: 3
  maxReplicas: 10  # 너무 많이 확장하지 않음
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## 권장 리소스 전략 요약

| 리소스 | 우선순위 | 이유 |
|--------|----------|------|
| **메모리** | 최우선 | GC 안정성 확보, tail latency 최소화 |
| **CPU** | 중간 | 데이터 변환, 직렬화 처리, burst 허용 |
| **Pod 수** | 최소화 | HA 보장 수준만 유지, 비용 효율 |

**실전 예시**

```yaml
# ❌ 잘못된 설정
spec:
  replicas: 20  # 과도한 파드 수
  resources:
    requests:
      memory: 256Mi  # 메모리 부족 → GC 빈발
      cpu: 100m

# ✅ 올바른 설정
spec:
  replicas: 3  # 최소한의 HA
  resources:
    requests:
      memory: 1Gi    # 충분한 메모리
      cpu: 500m
    limits:
      cpu: 2000m     # burst 허용
      memory: 1Gi
```

---

# 3. 메모리와 GC의 관계

Java 애플리케이션에서 **메모리 설정은 GC 동작에 직접적인 영향**을 미친다.

## 메모리 크기별 GC 동작 비교

### Case 1: 메모리가 작을 때 (512Mi)

**힙 구조**
```
Total Heap: 512Mi
├─ Young Generation: ~170Mi (1/3)
│   ├─ Eden: ~136Mi
│   └─ Survivor: ~34Mi
└─ Old Generation: ~342Mi (2/3)
```

**GC 동작 패턴**
```
[00:00] 애플리케이션 시작
    ├─ Eden 영역 빠르게 채워짐 (수십 MB/초)
    └─ 10초 후 Eden 포화

[00:10] Young GC 발생 (1회차)
    ├─ Stop-The-World: 50ms
    ├─ 살아남은 객체 → Survivor로 이동
    └─ 일부 → Old로 승격

[00:20] Young GC 발생 (2회차)
    ├─ Stop-The-World: 60ms
    ├─ Old 영역 계속 증가 (50% → 60%)
    └─ CPU 사용률 증가 시작

[00:40] Young GC 발생 (4회차)
    ├─ Stop-The-World: 80ms
    ├─ Old 영역 포화 (90%)
    └─ CPU 70% 도달

[01:00] Full GC 발생
    ├─ Stop-The-World: 2000ms (2초!)
    ├─ 애플리케이션 완전 정지
    └─ 사용자 요청 타임아웃 발생
```

**문제점**
- Young GC 빈도: **초당 0.1~0.5회** (매우 빈번)
- Full GC 빈도: **수 분마다**
- CPU 사용률: **지속 상승** (60~90%)
- 응답 시간: **불안정** (tail latency 높음)

---

### Case 2: 메모리가 클 때 (2Gi)

**힙 구조**
```
Total Heap: 2Gi
├─ Young Generation: ~680Mi (1/3)
│   ├─ Eden: ~544Mi
│   └─ Survivor: ~136Mi
└─ Old Generation: ~1368Mi (2/3)
```

**GC 동작 패턴**
```
[00:00] 애플리케이션 시작
    ├─ Eden 영역 여유 있음
    └─ 객체 할당 여유롭게 진행

[01:00] Young GC 발생 (1회차)
    ├─ Stop-The-World: 100ms
    ├─ Eden이 커서 GC 시간 증가
    └─ Old로 승격 최소화 (충분한 Young 영역)

[05:00] Young GC 발생 (2회차)
    ├─ Stop-The-World: 120ms
    ├─ Old 영역 천천히 증가 (20% → 25%)
    └─ CPU 안정적 유지 (40%)

[30:00] Young GC 발생 (6회차)
    ├─ Old 영역 50% 수준
    └─ Full GC 아직 발생 안 함

[60:00] 운영 1시간 후
    ├─ Young GC 총 12회
    ├─ Full GC 0회
    └─ CPU 안정적 (30~50%)
```

**장점**
- Young GC 빈도: **수십 초~수 분마다** (안정적)
- Full GC 빈도: **거의 없음** 또는 드물게
- CPU 사용률: **안정적** (30~50%)
- 응답 시간: **일정** (tail latency 낮음)

**단점**
- Young GC 한 번의 시간 증가 (50ms → 100~150ms)
- 메모리 비용 증가

---

## GC 빈도와 비용의 균형

### 목표: Sweet Spot 찾기

```
메모리 크기와 GC의 관계
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

메모리 너무 작음 (256Mi~512Mi)
    ├─ GC 빈도: 매우 높음 (초당 여러 번)
    ├─ GC 시간: 짧음 (10~50ms)
    ├─ Full GC: 빈번 (수 분마다)
    └─ CPU: 높음 (70~90%)

메모리 적정 (1Gi~2Gi)
    ├─ GC 빈도: 적정 (수십 초마다)
    ├─ GC 시간: 중간 (50~150ms)
    ├─ Full GC: 드물게 (수 시간~하루)
    └─ CPU: 안정 (30~50%)

메모리 너무 큼 (4Gi~8Gi)
    ├─ GC 빈도: 매우 낮음 (수 분마다)
    ├─ GC 시간: 김 (200~500ms)
    ├─ Full GC: 거의 없음
    └─ CPU: 안정하지만 GC 발생 시 스파이크
```

### 실전 튜닝 가이드

**1단계: 현재 GC 패턴 분석**

```bash
# GC 로그 활성화
JAVA_OPTS: |
  -Xlog:gc*:file=/var/log/gc.log:time,uptime,level,tags
  -XX:+UseG1GC

# GC 로그 분석
kubectl logs <pod-name> | grep "GC"
```

**2단계: 메트릭 확인**

```bash
# Prometheus 쿼리
# Young GC 빈도
rate(jvm_gc_pause_seconds_count{gc="G1 Young Generation"}[5m])

# Full GC 빈도
rate(jvm_gc_pause_seconds_count{gc="G1 Old Generation"}[5m])

# GC 소요 시간
jvm_gc_pause_seconds_sum / jvm_gc_pause_seconds_count
```

**3단계: 메모리 조정**

```yaml
# GC가 빈번한 경우 (초당 0.1회 이상)
resources:
  requests:
    memory: 1Gi  # 현재의 2배
  limits:
    memory: 1Gi

# GC 한 번이 너무 긴 경우 (500ms 이상)
# → 메모리를 줄이거나 GC 알고리즘 변경
env:
- name: JAVA_OPTS
  value: |
    -XX:+UseG1GC
    -XX:MaxGCPauseMillis=200  # 목표 GC 시간
```

---

## 핵심 원칙

| 증상 | 원인 | 해결 |
|------|------|------|
| GC 빈도 높음 | Young Gen 작음 | 메모리 증설 |
| Full GC 빈번 | Old Gen 빠른 포화 | 메모리 증설 또는 메모리 누수 확인 |
| GC 시간 김 | 힙 크기 과도 | 메모리 감소 또는 GC 알고리즘 변경 |
| CPU 지속 높음 | GC 반복 | 메모리 증설 |

**결론**: 메모리는 GC 빈도를 조절하는 가장 중요한 요소다. **충분한 메모리**는 GC 빈도를 낮추고 CPU 사용을 안정화한다.

---

# 4. CPU 코어 수와 GC의 관계

GC는 CPU를 사용하는 작업이므로, **CPU 코어 수는 GC 성능에 직접 영향**을 미친다.

## CPU 리소스와 GC 경쟁

### Case 1: CPU가 부족한 경우 (500m = 0.5 코어)

**동작 시나리오**
```
[트래픽 처리 중]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CPU 0.5코어
├─ 애플리케이션 스레드: 0.4코어 사용 (80%)
└─ 여유: 0.1코어 (20%)

        ↓ Young GC 발생

[GC 실행]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GC 스레드가 CPU 요구
├─ GC 필요 CPU: 0.3코어
└─ 가용 CPU: 0.5코어

애플리케이션 스레드와 경쟁
├─ 애플리케이션: 일시 중단 또는 느려짐
└─ GC: CPU 부족으로 느리게 실행

결과
├─ GC 시간: 200ms (정상: 50ms)
├─ 응답 지연: 증가
└─ 처리량: 감소
```

**문제점**
- GC와 애플리케이션이 CPU를 놓고 경쟁
- GC 시간 증가 (Stop-The-World 길어짐)
- 응답 시간 불안정

---

### Case 2: CPU가 충분한 경우 (2000m = 2 코어)

**동작 시나리오**
```
[트래픽 처리 중]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CPU 2코어
├─ 애플리케이션 스레드: 1.2코어 사용 (60%)
└─ 여유: 0.8코어 (40%)

        ↓ Young GC 발생

[GC 실행]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GC 스레드가 CPU 사용
├─ GC 사용 CPU: 0.6코어
└─ 가용 CPU: 2코어

병렬 GC 효율적 실행
├─ 애플리케이션: 계속 실행 가능 (Concurrent GC)
├─ GC: 빠르게 완료
└─ 상호 간섭 최소

결과
├─ GC 시간: 50ms (정상)
├─ 응답 지연: 최소
└─ 처리량: 유지
```

**장점**
- GC 병렬 처리 효율 증가
- CPU 스파이크 완화
- 안정적인 응답 시간

---

## G1 GC와 CPU 코어

G1 GC는 **병렬 GC**를 수행하므로 CPU 코어 수의 영향을 크게 받는다.

### G1 GC 스레드 수 계산

```bash
# G1 GC의 기본 병렬 스레드 수
-XX:ParallelGCThreads=<N>

# 기본값 계산
N = 8 + (CPU코어 - 8) * 5/8  (CPU > 8인 경우)
N = CPU코어                  (CPU ≤ 8인 경우)
```

**예시**

| CPU 코어 | ParallelGCThreads | 설명 |
|----------|-------------------|------|
| 0.5 (500m) | 1 | 단일 스레드 GC (느림) |
| 1 (1000m) | 1 | 단일 스레드 GC |
| 2 (2000m) | 2 | 병렬 GC 가능 |
| 4 (4000m) | 4 | 효율적인 병렬 GC |

### CPU 코어별 GC 성능

**실험 결과** (동일한 워크로드, 2Gi 메모리)

```
CPU: 500m (0.5코어)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Young GC 평균 시간: 180ms
Full GC 시간: 5000ms
CPU 사용률: 90% (GC 포함)
응답 시간 p99: 800ms

CPU: 1000m (1코어)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Young GC 평균 시간: 100ms
Full GC 시간: 2500ms
CPU 사용률: 70%
응답 시간 p99: 400ms

CPU: 2000m (2코어)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Young GC 평균 시간: 50ms
Full GC 시간: 1200ms
CPU 사용률: 50%
응답 시간 p99: 200ms
```

---

## 실무 권장 설정

### 일반 백엔드 서비스

```yaml
resources:
  requests:
    cpu: 1000m     # 최소 1코어
    memory: 1Gi
  limits:
    cpu: 2000m     # burst 허용 (2코어)
    memory: 1Gi

env:
- name: JAVA_OPTS
  value: |
    -Xms1g -Xmx1g
    -XX:+UseG1GC
    -XX:MaxGCPauseMillis=200
    -XX:ParallelGCThreads=2  # CPU limits에 맞춤
```

### API Gateway / BFF

```yaml
resources:
  requests:
    cpu: 500m      # 최소 0.5코어
    memory: 1Gi
  limits:
    cpu: 2000m     # burst 허용 (2코어, 트래픽 급증 대비)
    memory: 1Gi

env:
- name: JAVA_OPTS
  value: |
    -Xms1g -Xmx1g
    -XX:+UseG1GC
    -XX:MaxGCPauseMillis=100  # 더 낮은 목표 (latency 중요)
```

### 고처리량 서비스

```yaml
resources:
  requests:
    cpu: 2000m     # 2코어
    memory: 2Gi
  limits:
    cpu: 4000m     # burst 허용 (4코어)
    memory: 2Gi

env:
- name: JAVA_OPTS
  value: |
    -Xms2g -Xmx2g
    -XX:+UseG1GC
    -XX:MaxGCPauseMillis=200
    -XX:ParallelGCThreads=4
    -XX:ConcGCThreads=1
```

---

## 핵심 정리

| CPU 설정 | GC 영향 | 권장 사항 |
|----------|---------|-----------|
| **requests** | 보장된 CPU, GC 최소 성능 | 최소 1코어 (1000m) |
| **limits** | burst 허용, GC 스파이크 대응 | 2~4코어 (트래픽 패턴에 따라) |
| **ParallelGCThreads** | 병렬 GC 성능 | limits 값에 맞춤 |

**결론**: CPU는 **burst를 허용**하는 것이 중요하다. requests는 안정적인 운영을 위한 최소값, limits는 트래픽 급증 시 GC 성능 유지를 위한 여유분이다.

---

# 5. 실무 리소스 설계 원칙

## 역할별 우선순위

### 메모리는 "품질 안정성"

메모리는 **애플리케이션 안정성의 기반**이다.

**메모리 부족 시 발생하는 문제들**
```
메모리 부족
    ↓
GC 빈번 발생
    ↓
CPU 사용률 증가
    ↓
응답 시간 증가 (tail latency)
    ↓
최악의 경우: OutOfMemoryError
    ↓
Pod Crash → Restart
    ↓
서비스 불안정
```

**메모리 충분 시 효과**
```
충분한 메모리
    ↓
GC 빈도 감소
    ↓
CPU 안정
    ↓
응답 시간 일정
    ↓
사용자 경험 향상
```

**권장 사항**
- 메모리는 **여유 있게** 설정
- 비용보다 **안정성 우선**
- GC 메트릭을 지속 모니터링

---

### 파드 수는 "처리량 대응"

파드 수는 **트래픽 처리량을 확장**하는 수단이다.

**파드 수 증가가 효과적인 경우**
```
트래픽 증가 → CPU 100% 도달
    ↓
처리량 한계
    ↓
파드 수 증가 (HPA)
    ↓
부하 분산
    ↓
처리량 증가
```

**파드 수 증가가 비효과적인 경우**
```
GC 문제로 인한 CPU 사용
    ↓
파드 수 증가
    ↓
각 파드는 여전히 GC 문제 유지
    ↓
전체 파드 수만 증가
    ↓
비용 증가, 문제 미해결
```

**권장 사항**
- 파드 수는 **필요 시에만** 증가
- HPA로 **자동 조절**
- 최소 replicas는 **HA 보장 수준**(예: 3)

---

### CPU는 burst 허용이 중요

CPU는 **순간적인 부하 증가**에 대응해야 한다.

**CPU requests vs limits**

```yaml
# ❌ 잘못된 설정
resources:
  requests:
    cpu: 1000m
  limits:
    cpu: 1000m  # burst 불가능
```

**문제점**
```
정상 트래픽: CPU 70% 사용
    ↓
트래픽 급증 (1.5배)
    ↓
CPU 100% 도달 (limits 제한)
    ↓
요청 큐잉 발생
    ↓
응답 지연 증가
```

```yaml
# ✅ 올바른 설정
resources:
  requests:
    cpu: 1000m     # 보장
  limits:
    cpu: 2000m     # burst 허용
```

**효과**
```
정상 트래픽: CPU 70% 사용
    ↓
트래픽 급증 (1.5배)
    ↓
CPU 120% 사용 가능 (limits 여유)
    ↓
부하 흡수
    ↓
응답 시간 유지
```

---

## 잘못된 접근 사례

### 안티패턴 1: 파드 수로 GC 문제 해결 시도

**상황**
```
문제: CPU 사용률 80%, GC 빈번
잘못된 접근: "파드를 3개에서 10개로 늘리자"
```

**결과**
```
Before (3 Pods)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
각 Pod
├─ CPU: 80% (GC 때문)
├─ Memory: 512Mi
└─ GC: 초당 0.3회

After (10 Pods)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
각 Pod
├─ CPU: 여전히 80% (GC는 해결 안 됨)
├─ Memory: 여전히 512Mi
└─ GC: 여전히 초당 0.3회

총 비용: 3.3배 증가
문제 해결: ✗
```

**올바른 접근**
```
문제 분석: GC가 원인
해결: 메모리 512Mi → 1Gi 증설

결과 (3 Pods 유지)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
각 Pod
├─ CPU: 40% (GC 감소로 정상화)
├─ Memory: 1Gi
└─ GC: 초당 0.05회

총 비용: 2배 증가 (메모리만)
문제 해결: ✓
```

---

### 안티패턴 2: 메모리 과도한 절약

**상황**
```
목표: 비용 절감
잘못된 접근: "메모리를 최소화하자 (256Mi)"
```

**결과**
```
비용 절감 효과
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
메모리 비용: 50% 감소 ✓

성능 저하
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GC 빈도: 10배 증가
CPU 사용: 2배 증가 (GC 때문)
응답 시간: 3배 증가
장애 발생: OutOfMemoryError 빈발

추가 비용
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
파드 수 증가 필요: 2배
온콜 대응 비용: 증가
사용자 이탈: 비즈니스 손실

총 비용: 오히려 증가
```

**올바른 접근**
```
메모리는 충분히 확보 (1Gi)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
비용: 약간 증가

성능 향상
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GC 안정: 초당 0.05회
CPU 안정: 40%
응답 시간: 안정적
장애: 없음

장기 비용: 최적화됨 (파드 수 최소화)
```

---

## 실전 의사결정 가이드

### 문제 발생 시 체크리스트

**1단계: 증상 확인**
```bash
# CPU 사용률
kubectl top pods

# GC 메트릭
# Prometheus에서 확인
jvm_gc_pause_seconds_count
jvm_gc_pause_seconds_sum

# 메모리 사용
kubectl exec -it <pod> -- jmap -heap 1
```

**2단계: 원인 분석**
```
CPU 높음 + GC 빈번
    → 메모리 부족
    → 해결: 메모리 증설

CPU 높음 + GC 정상
    → 처리량 한계
    → 해결: 파드 수 증가 (HPA)

CPU 높음 + 트래픽 무관
    → 메모리 누수 또는 백그라운드 작업
    → 해결: 코드 개선
```

**3단계: 조치**
```yaml
# 메모리 증설이 필요한 경우
resources:
  requests:
    memory: 2Gi  # 현재의 2배
  limits:
    memory: 2Gi

# 파드 수 증가가 필요한 경우
# HPA 설정 또는 replicas 증가
spec:
  replicas: 5  # 3 → 5

# CPU burst 허용이 필요한 경우
resources:
  limits:
    cpu: 2000m  # requests의 2배
```

---

# 6. 핵심 요약

## 의사결정 매트릭스

| 증상 | CPU 사용 | GC 상태 | 트래픽 관계 | 해결책 |
|------|----------|---------|-------------|--------|
| 높은 CPU | 70~90% | 빈번 | - | 메모리 증설 |
| 높은 CPU | 70~90% | 정상 | 비례 | 파드 수 증가 |
| 높은 CPU | 70~90% | 정상 | 무관 | 코드 개선 |
| 응답 지연 | 정상 | 빈번 | - | 메모리 증설 |
| 처리량 부족 | 100% | 정상 | 비례 | 파드 수 증가 |

---

## 리소스별 핵심 원칙

### 메모리 (Memory)
```
목적: 품질 안정성
전략: 여유 있게 확보
근거: GC 빈도 감소 → CPU 안정 → 응답 시간 안정
권장: 최소 1Gi (일반 서비스)
```

### CPU
```
목적: 순간 부하 대응
전략: burst 허용 (requests < limits)
근거: 트래픽 급증 시 처리 능력 유지
권장: requests 1코어, limits 2코어
```

### Pod 수
```
목적: 처리량 확장
전략: 필요 시에만 증가 (HPA)
근거: 비용 효율, HA 보장
권장: 최소 3개, HPA로 자동 조절
```

---

## Gateway/BFF 특화 전략

```yaml
# API Gateway / BFF 권장 설정
spec:
  replicas: 3  # 최소한의 HA

  resources:
    requests:
      memory: 1Gi    # 품질 안정성 최우선
      cpu: 500m
    limits:
      cpu: 2000m     # burst 허용
      memory: 1Gi

  # HPA
  hpa:
    minReplicas: 3
    maxReplicas: 10  # 과도한 확장 방지
    targetCPU: 70%

  # JVM 옵션
  jvm:
    heap: 1g
    gc: G1GC
    maxGCPauseMillis: 200
```

**전략 요약**
- 파드 수: **최소화**
- 메모리: **여유 확보** (GC 안정성)
- CPU: **burst 허용** (트래픽 급증 대비)

---

## 안티패턴 회피

### ❌ 하지 말아야 할 것

1. **파드 수로 GC 문제 해결 시도**
   - GC는 각 파드의 메모리 문제
   - 파드 수 증가는 비용만 증가

2. **메모리 과도한 절약**
   - 단기 비용 절감 → 장기 비용 증가
   - 성능 저하 → 사용자 경험 악화

3. **CPU limits를 requests와 동일하게 설정**
   - burst 불가능
   - 순간 부하 대응 불가

4. **메트릭 없이 추측으로 결정**
   - 문제 원인 오판
   - 잘못된 해결책 적용

### ✅ 해야 할 것

1. **GC 메트릭 모니터링**
   - Prometheus + Grafana
   - GC 빈도, 시간, 원인 파악

2. **메모리 우선 확보**
   - 안정성의 기초
   - GC 최적화

3. **HPA 활용**
   - 자동 스케일링
   - 비용 효율

4. **부하 테스트**
   - 프로덕션 배포 전 검증
   - 적절한 리소스 설정 파악

---

# 7. 실전 적용 가이드

## 신규 서비스 초기 설정

### 1단계: 기본 설정으로 시작

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: new-service
spec:
  replicas: 3  # HA 기본값

  template:
    spec:
      containers:
      - name: app
        image: new-service:1.0.0

        resources:
          requests:
            memory: 1Gi   # 기본값
            cpu: 500m
          limits:
            cpu: 2000m    # burst 허용
            memory: 1Gi

        env:
        - name: JAVA_OPTS
          value: |
            -Xms1g -Xmx1g
            -XX:+UseG1GC
            -XX:MaxGCPauseMillis=200

---
# HPA 설정
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: new-service
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: new-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

### 2단계: 부하 테스트

```bash
# k6를 사용한 부하 테스트
k6 run --vus 100 --duration 10m load-test.js

# 또는 hey
hey -z 10m -c 100 -q 10 http://new-service/api/endpoint
```

**관찰할 메트릭**
```
1. CPU 사용률
   ├─ 평균: 50% 미만 (양호)
   ├─ 최대: 80% 미만 (양호)
   └─ 90% 이상 (조정 필요)

2. 메모리 사용
   ├─ 안정적 유지 (양호)
   └─ 계속 증가 (메모리 누수 의심)

3. GC 메트릭
   ├─ Young GC: 10초~1분마다 (양호)
   ├─ Full GC: 거의 없음 (양호)
   └─ 빈번한 GC (메모리 증설 필요)

4. 응답 시간
   ├─ p50: <100ms
   ├─ p95: <500ms
   └─ p99: <1000ms
```

---

### 3단계: 리소스 최적화

**CPU 조정**
```yaml
# CPU 사용률이 지속적으로 80% 이상
resources:
  requests:
    cpu: 1000m  # 500m → 1000m
  limits:
    cpu: 2000m  # 유지
```

**메모리 조정**
```yaml
# GC가 빈번한 경우 (초당 0.1회 이상)
resources:
  requests:
    memory: 2Gi  # 1Gi → 2Gi
  limits:
    memory: 2Gi
```

**파드 수 조정**
```yaml
# 트래픽 패턴에 맞춘 HPA 조정
spec:
  minReplicas: 5   # 3 → 5 (평시 트래픽 기준)
  maxReplicas: 30  # 20 → 30 (피크 타임 대비)
```

---

## 기존 서비스 최적화

### 문제 진단 프로세스

**1단계: 현재 상태 파악**
```bash
# Pod 리소스 사용 확인
kubectl top pods -n production

# Pod 수 확인
kubectl get deployment -n production

# HPA 상태 확인
kubectl get hpa -n production
```

**2단계: 메트릭 분석**
```bash
# Prometheus에서 쿼리
# CPU 사용률 추이 (최근 24시간)
rate(container_cpu_usage_seconds_total{namespace="production"}[5m])

# 메모리 사용 추이
container_memory_usage_bytes{namespace="production"}

# GC 메트릭
rate(jvm_gc_pause_seconds_count[5m])
jvm_gc_pause_seconds_sum / jvm_gc_pause_seconds_count
```

**3단계: 로그 분석**
```bash
# GC 로그 확인
kubectl logs <pod-name> | grep "GC"

# OOM 확인
kubectl logs <pod-name> | grep -i "OutOfMemory"

# 에러 로그
kubectl logs <pod-name> | grep -i "error"
```

---

### 최적화 시나리오별 가이드

#### 시나리오 1: 높은 CPU, 빈번한 GC

**진단**
```
CPU: 80% (지속)
GC: 초당 0.3회
메모리: 512Mi
```

**해결**
```yaml
# 메모리 2배 증설
resources:
  requests:
    memory: 1Gi  # 512Mi → 1Gi
  limits:
    memory: 1Gi

# JVM 힙 조정
env:
- name: JAVA_OPTS
  value: "-Xms1g -Xmx1g -XX:+UseG1GC"
```

**예상 결과**
```
CPU: 40% (감소)
GC: 초당 0.05회 (개선)
응답 시간: 50% 감소
```

---

#### 시나리오 2: 트래픽 증가 시 응답 지연

**진단**
```
평시: CPU 50%, 응답 200ms
피크: CPU 100%, 응답 2000ms
HPA: minReplicas 3, maxReplicas 10
```

**해결**
```yaml
# HPA 조정
spec:
  minReplicas: 5   # 평시 트래픽도 여유 있게
  maxReplicas: 30  # 피크 대비
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60  # 70% → 60% (빠른 스케일 아웃)

# CPU burst 여유 확보
resources:
  limits:
    cpu: 3000m  # 2000m → 3000m
```

**예상 결과**
```
평시: 5 Pods, CPU 30~40%
피크: 15~20 Pods, CPU 60%
응답 시간: 안정적 (p99 < 500ms)
```

---

#### 시나리오 3: 메모리 누수 의심

**진단**
```
메모리 사용: 지속 증가 (1시간마다 +100Mi)
Full GC: 빈번 (5분마다)
CPU: 지속 상승 (60% → 90%)
```

**해결**
```bash
# 1단계: Heap dump 생성
kubectl exec -it <pod-name> -- jmap -dump:format=b,file=/tmp/heap.hprof 1

# 2단계: 분석
# Eclipse MAT, VisualVM 등으로 메모리 누수 위치 파악

# 3단계: 코드 수정
# 누수 원인 제거 (캐시 미정리, 리스너 미해제 등)

# 4단계: 임시 조치 (코드 수정 전)
# Pod 자동 재시작 설정
livenessProbe:
  exec:
    command:
    - sh
    - -c
    - |
      # 메모리 사용률 80% 초과 시 재시작
      USED=$(cat /sys/fs/cgroup/memory/memory.usage_in_bytes)
      LIMIT=$(cat /sys/fs/cgroup/memory/memory.limit_in_bytes)
      if [ $((USED * 100 / LIMIT)) -gt 80 ]; then exit 1; fi
  periodSeconds: 60
```

---

## 모니터링 설정

### Prometheus + Grafana 대시보드

**필수 메트릭**
```yaml
# GC 메트릭
- jvm_gc_pause_seconds_count
- jvm_gc_pause_seconds_sum
- jvm_gc_memory_allocated_bytes_total
- jvm_gc_memory_promoted_bytes_total

# 메모리 메트릭
- jvm_memory_used_bytes
- jvm_memory_max_bytes
- container_memory_usage_bytes

# CPU 메트릭
- container_cpu_usage_seconds_total
- container_cpu_cfs_throttled_seconds_total

# 애플리케이션 메트릭
- http_server_requests_seconds_count
- http_server_requests_seconds_sum
```

**알람 설정**
```yaml
# Prometheus Alert Rules
groups:
- name: resource-alerts
  rules:

  # GC가 빈번한 경우
  - alert: HighGCRate
    expr: rate(jvm_gc_pause_seconds_count[5m]) > 0.1
    for: 10m
    annotations:
      summary: "GC가 빈번합니다 (초당 {{ $value }}회)"
      description: "메모리 증설을 고려하세요"

  # Full GC 발생
  - alert: FullGCDetected
    expr: rate(jvm_gc_pause_seconds_count{gc="G1 Old Generation"}[5m]) > 0
    for: 5m
    annotations:
      summary: "Full GC가 발생했습니다"
      description: "메모리 부족 또는 누수를 확인하세요"

  # CPU 지속 높음
  - alert: HighCPU
    expr: rate(container_cpu_usage_seconds_total[5m]) > 0.8
    for: 15m
    annotations:
      summary: "CPU 사용률이 높습니다 ({{ $value | humanizePercentage }})"
      description: "GC 또는 처리량 문제를 확인하세요"

  # 메모리 부족
  - alert: HighMemory
    expr: container_memory_usage_bytes / container_spec_memory_limit_bytes > 0.9
    for: 10m
    annotations:
      summary: "메모리 사용률이 높습니다 ({{ $value | humanizePercentage }})"
      description: "메모리 증설 또는 누수 확인이 필요합니다"
```

---

# 결론

Kubernetes 환경에서의 리소스 설계는 **단순히 숫자를 늘리는 것이 아니라, 문제의 본질을 이해하고 올바른 해결책을 적용하는 것**이다.

## 핵심 원칙 다시 보기

### 1. 메모리는 품질 안정성
- GC 빈도 감소
- CPU 안정화
- 응답 시간 개선
- **여유 있게 확보**

### 2. 파드 수는 처리량 대응
- 부하 분산
- HPA로 자동 조절
- **필요 시에만 증가**

### 3. CPU는 burst 허용
- 순간 부하 흡수
- GC 성능 유지
- **requests < limits**

## 실전 적용 체크리스트

**배포 전**
- [ ] 메모리 최소 1Gi 확보
- [ ] CPU limits가 requests의 2배 이상
- [ ] HPA 설정 (minReplicas ≥ 3)
- [ ] GC 옵션 설정 (G1GC, MaxGCPauseMillis)
- [ ] 부하 테스트 수행

**운영 중**
- [ ] GC 메트릭 모니터링
- [ ] CPU/메모리 사용률 추적
- [ ] 응답 시간 (p95, p99) 확인
- [ ] 알람 설정 및 대응 프로세스

**문제 발생 시**
- [ ] CPU + GC 패턴 분석
- [ ] 메모리 vs 파드 수 결정
- [ ] 코드 개선 필요 여부 판단
- [ ] 점진적 조정 및 검증

## 마지막 조언

> "메모리는 아껴서 좋을 게 없다. GC 문제는 파드 수로 해결되지 않는다. CPU는 여유를 두어야 한다."

리소스 설계의 목표는 **비용 최소화가 아니라 안정성과 비용의 균형**이다. 초기에 충분한 리소스를 확보하고, 모니터링을 통해 점진적으로 최적화하는 것이 가장 현명한 접근 방식이다.

---

# 참고 자료

- [Kubernetes Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [G1 GC Tuning Guide](https://www.oracle.com/technical-resources/articles/java/g1gc.html)
- [HPA Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- [JVM Memory Management](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-collector-implementation.html)
- [Prometheus JVM Metrics](https://github.com/prometheus/client_java)
