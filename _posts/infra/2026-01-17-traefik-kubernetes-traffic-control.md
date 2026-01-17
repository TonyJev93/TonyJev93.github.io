---
title: "[Kubernetes] Traefik과 Kubernetes 트래픽 제어 완벽 이해"
last_modified_at: 2026-01-17T00:00:00+09:00
categories:
    - Infra
tags:
    - Kubernetes
    - Traefik
    - Ingress Controller
    - Service Mesh
    - Canary Deployment
    - Flagger
    - IngressRoute
toc: true
toc_sticky: true
toc_label: "목차"
---

Kubernetes 환경에서 Traefik을 활용한 트래픽 제어의 전체 구조를 이해하고, Ingress Controller, CRD, Canary 배포의 관계를 명확히 정리한다.
{: .notice--info}

# Traefik이란?

`Traefik`은 Kubernetes에서 동작하는 **Ingress Controller**이자 **L7 리버스 프록시**다. 단순히 트래픽을 전달하는 것을 넘어, 트래픽 분기, 가중치 기반 라우팅, 미들웨어 적용 등 정교한 트래픽 제어를 담당한다.

## 주요 역할

1. **외부/내부 트래픽 라우팅**
   - 클라이언트 요청을 적절한 서비스로 전달
   - 도메인, 경로, 헤더 등 다양한 조건으로 라우팅

2. **트래픽 분기 및 가중치 제어**
   - 특정 비율로 트래픽을 여러 서비스에 분산
   - Canary 배포, A/B 테스트 지원

3. **미들웨어 적용**
   - 인증, 압축, 속도 제한 등
   - 요청/응답 변환

## 왜 Traefik인가?

- **Kubernetes 네이티브**: CRD를 통한 자연스러운 통합
- **동적 설정**: 설정 변경 시 재시작 불필요
- **강력한 라우팅**: 복잡한 트래픽 시나리오 지원
- **현대적 아키텍처**: Cloud Native 환경에 최적화

# Ingress Controller 이해하기

## Ingress Controller란?

`Ingress Controller`는 **Kubernetes 리소스를 실제 네트워크 동작으로 변환하는 컴포넌트**다.

```
Ingress 리소스 (선언)  →  Ingress Controller (실행)  →  실제 트래픽 라우팅
```

### Ingress vs Ingress Controller

**Ingress (리소스)**
- 단순한 YAML 선언
- "이렇게 라우팅해야 한다"는 규칙만 정의
- 실제 트래픽을 처리하지 않음

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**Ingress Controller (실행자)**
- 실제 동작하는 애플리케이션
- Ingress 리소스를 감시(Watch)
- 규칙을 읽어 실제 네트워크 라우팅 수행
- Traefik, Nginx, HAProxy 등이 대표적

## Ingress Controller의 동작 원리

```
1. Kubernetes API 감시
   ↓
2. Ingress/IngressRoute 리소스 변경 감지
   ↓
3. 내부 라우팅 테이블 업데이트
   ↓
4. 실제 트래픽을 새로운 규칙대로 라우팅
```

이 과정이 **자동으로, 실시간으로** 일어난다.

# Traefik과 CRD의 관계

## Traefik은 CRD가 아니다

많은 사람들이 혼동하는 부분이다. 명확히 정리하자면:

**Traefik의 정체**
- Kubernetes **컨트롤러 애플리케이션**
- Pod로 배포되어 실행되는 프로그램
- CRD를 **해석하고 실행하는 주체**

**CRD (Custom Resource Definition)**
- Kubernetes API 확장 메커니즘
- 새로운 리소스 타입을 정의
- 데이터 구조일 뿐, 실행 로직 없음

## Traefik이 정의하는 CRD들

Traefik은 자체적으로 여러 CRD를 제공한다:

### 1. IngressRoute
```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: my-ingressroute
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`example.com`)
      kind: Rule
      services:
        - name: primary-service
          port: 80
          weight: 90
        - name: canary-service
          port: 80
          weight: 10
```

**역할**: 트래픽 라우팅 규칙을 세밀하게 정의

### 2. Middleware
```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: rate-limit
spec:
  rateLimit:
    average: 100
    burst: 50
```

**역할**: 요청/응답 처리 미들웨어 정의

### 3. TraefikService
```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: TraefikService
metadata:
  name: weighted-service
spec:
  weighted:
    services:
      - name: v1-service
        port: 80
        weight: 3
      - name: v2-service
        port: 80
        weight: 1
```

**역할**: 복잡한 서비스 로드밸런싱 정의

## 관계 정리

```
CRD 정의 (IngressRoute, Middleware 등)
    ↓ (정의)
Traefik (Controller)
    ↓ (읽고 해석)
실제 트래픽 라우팅 동작
```

따라서:
- Traefik ≠ CRD
- Traefik = **CRD를 읽고 실행하는 Ingress Controller**

# IngressRoute의 역할

## 표준 Ingress의 한계

Kubernetes 기본 Ingress는 단순한 라우팅만 지원한다:
- 호스트 기반 라우팅
- 경로 기반 라우팅
- 기본 TLS 설정

**부족한 기능들:**
- 가중치 기반 트래픽 분배 ✗
- 복잡한 라우팅 조건 ✗
- 미들웨어 체인 ✗

## IngressRoute의 강점

Traefik의 IngressRoute는 이러한 한계를 극복한다.

### 1. 가중치 기반 트래픽 분기

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: canary-route
spec:
  routes:
    - match: Host(`app.example.com`)
      kind: Rule
      services:
        - name: primary
          port: 80
          weight: 90  # 90% 트래픽
        - name: canary
          port: 80
          weight: 10  # 10% 트래픽
```

**활용 시나리오:**
- Canary 배포: 신규 버전에 10% 트래픽만 전달
- A/B 테스트: 50:50 분배
- 점진적 마이그레이션: 비율을 점진적으로 조정

### 2. 복잡한 라우팅 규칙

```yaml
routes:
  - match: Host(`api.example.com`) && PathPrefix(`/v2`)
    kind: Rule
    services:
      - name: api-v2
        port: 8080
```

### 3. 미들웨어 체인

```yaml
routes:
  - match: Host(`secure.example.com`)
    kind: Rule
    middlewares:
      - name: auth
      - name: rate-limit
      - name: compress
    services:
      - name: secure-app
        port: 443
```

## 실제 Canary 배포 예시

**초기 상태 (100% 안정 버전)**
```yaml
services:
  - name: app-v1
    port: 80
    weight: 100
```

**Canary 시작 (10% 신규 버전)**
```yaml
services:
  - name: app-v1
    port: 80
    weight: 90
  - name: app-v2
    port: 80
    weight: 10
```

**점진적 증가 (50% 신규 버전)**
```yaml
services:
  - name: app-v1
    port: 80
    weight: 50
  - name: app-v2
    port: 80
    weight: 50
```

**완전 전환 (100% 신규 버전)**
```yaml
services:
  - name: app-v2
    port: 80
    weight: 100
```

# Flagger와 Traefik의 관계

## Flagger란?

`Flagger`는 **Progressive Delivery를 자동화하는 Kubernetes Operator**다.

### 주요 역할

1. **Canary 리소스 감시**
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
       port: 80
     analysis:
       interval: 1m
       threshold: 5
       maxWeight: 50
       stepWeight: 10
   ```

2. **자동 트래픽 조정**
   - 메트릭 분석 (에러율, 레이턴시 등)
   - 단계별 트래픽 증가
   - 문제 발생 시 자동 롤백

3. **IngressRoute 수정**
   - Flagger가 IngressRoute의 weight를 동적으로 변경
   - Traefik은 변경을 감지하고 즉시 반영

## Flagger ↔ Traefik 협업 구조

```
1. 개발자가 새 버전 배포
   ↓
2. Flagger가 Canary 리소스 감지
   ↓
3. Canary Deployment 생성
   ↓
4. Flagger가 IngressRoute의 weight를 10%로 설정
   ↓
5. Traefik이 IngressRoute 변경 감지
   ↓
6. Traefik이 실제 트래픽의 10%를 Canary로 라우팅
   ↓
7. Flagger가 메트릭 분석
   ↓
8. (성공 시) Flagger가 weight를 20%, 30%... 순차 증가
   ↓
9. (실패 시) Flagger가 weight를 0%로 되돌림 (롤백)
```

## 역할 분리

이 구조의 핵심은 **관심사의 분리**다:

| 컴포넌트 | 역할 | 책임 범위 |
|---------|-----|----------|
| **Flagger** | 전략 결정자 | 언제, 얼마나 트래픽을 나눌지 결정 |
| **Traefik** | 실행자 | 어떻게 트래픽을 나눌지 실행 |
| **IngressRoute** | 계약서 | 두 컴포넌트 간의 인터페이스 |

**Flagger의 결정**
- "메트릭이 좋으니 20%로 증가시키자"
- "에러율이 높으니 롤백하자"
- "모든 테스트 통과, 100% 전환하자"

**Traefik의 실행**
- "IngressRoute에 weight: 20이라고 적혀있네"
- "그럼 20%만 Canary로 보내자"
- "설정이 weight: 0으로 바뀌었네, Canary로 안 보내야겠다"

## 왜 직접 제어하지 않는가?

Flagger가 Traefik을 직접 제어하지 않고 CRD를 통해 간접 제어하는 이유:

1. **표준화**: Kubernetes 네이티브 방식
2. **확장성**: Traefik뿐 아니라 다른 Ingress Controller도 지원 가능
3. **선언적 관리**: GitOps와 잘 맞음
4. **가시성**: kubectl로 현재 상태 확인 가능
5. **역할 분리**: 각자의 책임에만 집중

# Service to Service Canary

## 내부 트래픽 제어의 필요성

외부에서 들어오는 트래픽만 Canary를 적용하면 부족한 경우가 많다.

### 마이크로서비스 환경의 트래픽 흐름

```
외부 클라이언트
    ↓
API Gateway (Canary 적용)
    ↓
User Service (여기도 Canary 필요!)
    ↓
Order Service (여기도!)
    ↓
Payment Service (여기도!)
```

**문제 상황:**
- API Gateway에서만 Canary를 적용
- User Service는 새 버전으로 100% 전환
- Order Service 호출 시 예상치 못한 버그 발생
- 전체 시스템 장애

**해결책:**
- **모든 서비스 간 호출에도 Canary 적용**
- User Service → Order Service 호출도 점진적으로

## 구현 방법

### 1. Traefik으로 내부 트래픽 제어

```yaml
# Order Service에 대한 IngressRoute (내부용)
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: order-service-internal
spec:
  routes:
    - match: Host(`order-service.default.svc.cluster.local`)
      kind: Rule
      services:
        - name: order-service-primary
          port: 80
          weight: 90
        - name: order-service-canary
          port: 80
          weight: 10
```

**클러스터 내부 호출:**
```
User Service → order-service.default.svc.cluster.local
                  ↓
              Traefik (90% primary, 10% canary)
                  ↓
              실제 Order Service Pods
```

### 2. Service Mesh 활용

Service Mesh(Istio, Linkerd 등)를 사용하면 더 강력한 제어 가능:

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
    - order-service
  http:
    - match:
        - headers:
            user-type:
              exact: "beta"
      route:
        - destination:
            host: order-service
            subset: canary
          weight: 100
    - route:
        - destination:
            host: order-service
            subset: primary
          weight: 90
        - destination:
            host: order-service
            subset: canary
          weight: 10
```

**고급 기능:**
- 헤더 기반 라우팅 (특정 사용자만 Canary)
- 지연/장애 주입 (카오스 엔지니어링)
- 서킷 브레이커
- 재시도 정책

## 전체 시스템 Canary 전략

```
[외부 트래픽]
    ↓ (Ingress/Traefik - 10% canary)
[API Gateway]
    ↓ (Service Mesh - 10% canary)
[User Service]
    ↓ (Service Mesh - 10% canary)
[Order Service]
    ↓ (Service Mesh - 10% canary)
[Payment Service]
```

**장점:**
1. 엔드투엔드 점진적 배포
2. 서비스 간 호환성 문제 조기 발견
3. 전체 시스템 안정성 향상

**복잡도:**
- 설정 관리 어려움 → GitOps로 해결
- 트러블슈팅 복잡 → Observability 강화 필요

# 핵심 요약

## Traefik의 정체

- **Ingress Controller**: Kubernetes 트래픽 제어의 실행자
- **L7 Reverse Proxy**: 애플리케이션 레벨 라우팅
- **CRD가 아니라 CRD를 해석하는 컨트롤러**

## Ingress Controller의 역할

- Kubernetes 리소스(선언) → 실제 네트워크 동작(실행)
- 변경 사항을 실시간으로 감지하고 반영
- "트래픽 규칙의 실행자"

## CRD와의 관계

- Traefik은 IngressRoute, Middleware 등 여러 CRD를 정의
- 이 CRD들을 읽어서 실제 라우팅 수행
- **CRD = 설정 데이터, Traefik = 실행 엔진**

## Flagger와의 협업

- Flagger: 전략 결정 (언제, 얼마나)
- Traefik: 실행 (어떻게)
- IngressRoute: 두 컴포넌트를 연결하는 계약서
- **직접 제어가 아닌 CRD를 통한 간접 제어**

## Canary 트래픽 분기

- IngressRoute의 weight 속성으로 구현
- 외부 트래픽뿐 아니라 **내부 서비스 간 호출도 제어 가능**
- Service Mesh와 결합하면 더욱 강력

# 실전 활용 가이드

## 1. Traefik 설치

```bash
# Helm으로 설치
helm repo add traefik https://traefik.github.io/charts
helm repo update
helm install traefik traefik/traefik
```

## 2. 기본 IngressRoute 작성

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: my-app
  namespace: default
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`myapp.example.com`)
      kind: Rule
      services:
        - name: my-app-service
          port: 80
```

## 3. Flagger 설정 (Canary 자동화)

```yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: my-app
  namespace: default
spec:
  provider: traefik
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  service:
    port: 80
  analysis:
    interval: 1m
    threshold: 5
    maxWeight: 50
    stepWeight: 10
    metrics:
      - name: request-success-rate
        thresholdRange:
          min: 99
      - name: request-duration
        thresholdRange:
          max: 500
```

## 4. 모니터링

```bash
# IngressRoute 확인
kubectl get ingressroute

# Canary 상태 확인
kubectl get canary

# Traefik 로그
kubectl logs -f deployment/traefik -n traefik
```

# 마무리

Kubernetes에서 Traefik을 활용한 트래픽 제어는 단순해 보이지만, 그 내부에는 여러 계층의 추상화와 협업이 존재한다.

**핵심 개념 정리:**
1. Ingress Controller는 선언을 실행으로 바꾸는 실행자
2. Traefik은 CRD가 아니라 CRD를 해석하는 컨트롤러
3. Flagger는 Traefik을 직접 제어하지 않고 CRD를 통해 간접 제어
4. Canary 배포는 IngressRoute의 weight로 구현
5. 내부 서비스 간 호출도 제어 가능 (Service Mesh 결합)

이러한 구조를 이해하면:
- 트러블슈팅이 쉬워진다
- 적절한 도구를 선택할 수 있다
- Progressive Delivery를 안전하게 구현할 수 있다

**다음 단계:**
- 실제 환경에 Traefik 설치해보기
- 간단한 Canary 배포 실습
- Prometheus + Grafana로 메트릭 수집
- Flagger로 자동화 구성

트래픽 제어는 현대 클라우드 네이티브 애플리케이션의 핵심 요소다. 제대로 이해하고 활용하면, 안전하고 빠른 배포를 실현할 수 있다.

# 참고 자료

- [Traefik 공식 문서](https://doc.traefik.io/traefik/)
- [Flagger 공식 문서](https://docs.flagger.app/)
- [Kubernetes Ingress 문서](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [CNCF Landscape - Service Mesh](https://landscape.cncf.io/card-mode?category=service-mesh)
