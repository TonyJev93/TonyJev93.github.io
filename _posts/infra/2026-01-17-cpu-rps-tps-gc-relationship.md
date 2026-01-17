---
title: "[Performance] CPU, RPS/TPS, GC의 관계 완벽 이해"
last_modified_at: 2026-01-17T00:00:00+09:00
categories:
    - Infra
tags:
    - Performance
    - CPU
    - RPS
    - TPS
    - GC
    - Garbage Collection
    - BFF
    - API Gateway
    - Optimization
toc: true
toc_sticky: true
toc_label: "목차"
---

서비스 성능 최적화를 위해 CPU 사용량, RPS/TPS, 그리고 GC의 복잡한 관계를 이해하고, 왜 단순히 트래픽을 줄인다고 CPU가 감소하지 않는지 명확히 파악한다.
{: .notice--info}

# CPU, RPS/TPS, GC의 관계

애플리케이션 성능 튜닝을 하다 보면 흔히 이런 의문이 든다.

> "트래픽(RPS)을 줄였는데 왜 CPU는 그대로지?"

이는 단순히 요청 수만으로 CPU 사용량을 판단할 수 없기 때문이다. CPU 사용량에는 **코드 경로(Code Path)**, **GC(Garbage Collection)**, **스레드 관리** 등 여러 요소가 복합적으로 작용한다.

이 글에서는 CPU 사용량을 결정하는 핵심 요소들과 그 관계를 명확히 정리한다.

# RPS와 TPS의 차이

성능 지표를 논할 때 RPS와 TPS를 혼용하는 경우가 많다. 하지만 두 개념은 명확히 다르다.

## RPS (Requests Per Second)

**클라이언트 요청 기준**의 처리량 지표다.

```
클라이언트 → 서버
    ↓
1개의 HTTP 요청 = 1 RPS
```

### 특징
- 사용자 관점의 지표
- 외부에서 들어오는 요청 횟수
- 하나의 요청이 내부적으로 여러 작업을 포함할 수 있음

### 예시
```
사용자가 상품 조회 API를 호출
  ↓
1 RPS 발생
  ↓ (내부 처리)
- 캐시 조회
- DB 조회 (3번)
- 외부 API 호출 (2번)
```

## TPS (Transactions Per Second)

**실제 처리 단위 기준**의 처리량 지표다.

```
1개의 요청 안에서:
- DB 쿼리 3회 실행
- 외부 API 호출 2회
  ↓
총 5 TPS 발생 (1 RPS에서)
```

### 특징
- 시스템 관점의 지표
- 실제로 처리되는 트랜잭션 수
- 하나의 요청 안에서 여러 트랜잭션 발생 가능

## API Gateway / BFF 환경에서의 차이

현대의 마이크로서비스 아키텍처, 특히 BFF(Backend For Frontend) 패턴이나 API Gateway 환경에서는 이 차이가 더욱 두드러진다.

```
[클라이언트]
    ↓ (1 RPS)
[API Gateway / BFF]
    ↓
    ├─→ User Service (1 TPS)
    ├─→ Product Service (2 TPS)
    ├─→ Order Service (1 TPS)
    └─→ Payment Service (1 TPS)

총: 1 RPS, 5 TPS
```

### 실제 시나리오

**사용자가 주문 페이지를 조회하는 경우:**

1. **RPS 관점** (1 RPS)
   - 클라이언트가 `/api/order-details` 호출

2. **TPS 관점** (10+ TPS)
   - 사용자 정보 조회
   - 주문 이력 조회 (3건)
   - 상품 정보 조회 (5건)
   - 배송 상태 조회 (2건)
   - 포인트 정보 조회

**결과:**
- 1 RPS 안에 10+ TPS가 포함
- **TPS가 CPU 부담을 더 정확히 반영**

## 왜 중요한가?

### RPS만 보면 놓치는 것들

```
시나리오 A: 단순 조회
- 1000 RPS
- 각 요청당 1 TPS
- 총 1000 TPS

시나리오 B: 복잡한 집계
- 100 RPS
- 각 요청당 50 TPS
- 총 5000 TPS
```

**시나리오 B가 RPS는 1/10이지만, TPS는 5배 높다.**

따라서:
- **RPS가 낮아도 CPU가 높을 수 있음**
- **TPS가 CPU 사용량을 더 정확히 예측**

# CPU에 영향을 주는 3가지 핵심 요소

CPU 사용량은 단순히 "얼마나 많은 요청을 처리하느냐"로만 결정되지 않는다. 다음 세 가지 요소가 복합적으로 작용한다.

## 1. 실제 요청 처리량

가장 직관적인 요소다. 처리하는 요청 수가 많을수록 CPU 사용량이 증가한다.

### 기본 관계

```
요청 수 ↑ → CPU 사용량 ↑
```

### 하지만...

단순 연산 위주의 작업이라면 CPU 영향은 제한적이다.

**예시:**

```java
// 가벼운 작업 - CPU 영향 낮음
@GetMapping("/health")
public String health() {
    return "OK";  // 문자열 반환만
}

// 무거운 작업 - CPU 영향 높음
@GetMapping("/report")
public Report generateReport() {
    // 대량 데이터 조회
    // 복잡한 연산
    // 집계 및 변환
    return report;
}
```

**같은 1000 RPS라도:**
- `/health` 엔드포인트: CPU 10%
- `/report` 엔드포인트: CPU 80%

따라서 **요청 수보다 요청의 '무게'가 중요하다.**

## 2. 코드 경로 (Code Path)

같은 요청이라도 **어떤 로직을 타느냐**에 따라 CPU 사용량이 크게 달라진다.

### 코드 경로란?

요청이 처리되는 동안 거치는 실행 경로를 의미한다.

```
같은 상품 조회 API
    ↓
경로 1: 캐시 HIT → 즉시 반환 (CPU 낮음)
경로 2: 캐시 MISS → DB 조회 → 변환 → 캐시 저장 (CPU 높음)
```

### 실제 예시

```java
@GetMapping("/product/{id}")
public ProductDto getProduct(@PathVariable Long id) {
    // 경로 1: 캐시에서 조회 (CPU 사용량: 낮음)
    ProductDto cached = cache.get(id);
    if (cached != null) {
        return cached;  // 빠른 반환
    }

    // 경로 2: DB 조회 + 변환 (CPU 사용량: 높음)
    Product product = productRepository.findById(id);
    ProductDto dto = convertToDto(product);  // 객체 변환

    // 경로 3: 연관 데이터 조회 (CPU 사용량: 매우 높음)
    List<Review> reviews = reviewRepository.findByProductId(id);
    dto.setReviews(convertReviews(reviews));  // 추가 변환

    cache.put(id, dto);
    return dto;
}
```

### 코드 경로가 중요한 이유

**캐시 히트율 변화 시나리오:**

```
초기 상태:
- 1000 RPS
- 캐시 히트율 90%
- CPU 사용량 30%

캐시 서버 장애:
- 1000 RPS (동일)
- 캐시 히트율 0%
- CPU 사용량 85%
```

**RPS는 그대로인데 CPU가 급증!**

### CPU 스파이크의 주요 원인

1. **캐시 미스 증가**
   - 캐시 서버 장애
   - 캐시 키 변경
   - 대량의 신규 데이터 요청

2. **데이터 조합/변환 로직**
   - 여러 소스에서 데이터 수집
   - 복잡한 DTO 변환
   - JSON 직렬화/역직렬화

3. **조건부 로직**
   - 특정 조건에서만 실행되는 무거운 로직
   - A/B 테스트 분기

## 3. GC / Thread 사용

Java, Kotlin 등의 JVM 언어를 사용하는 환경에서는 **GC(Garbage Collection)**가 CPU 사용량에 큰 영향을 미친다.

### GC와 CPU의 관계

```
객체 생성량 ↑
    ↓
Heap 메모리 사용량 ↑
    ↓
GC 빈도 ↑
    ↓
GC가 CPU 연산 사용
    ↓
CPU 사용량 ↑
```

### Young GC vs Old GC

**Young GC (Minor GC)**
- 빈번하게 발생
- 짧은 시간 소요 (수 ms ~ 수십 ms)
- CPU 영향: 누적되면 상당함

**Old GC (Major GC / Full GC)**
- 드물게 발생
- 긴 시간 소요 (수백 ms ~ 수 초)
- CPU 영향: 발생 시 급격한 스파이크

### 실제 모니터링 예시

```
[정상 상태]
- RPS: 1000
- Young GC: 10회/분 (각 20ms)
- CPU: 40%

[객체 생성 증가]
- RPS: 1000 (동일)
- Young GC: 60회/분 (각 30ms)
- Old GC: 2회/시간 → 5회/시간
- CPU: 65%
```

### Thread Context Switching

스레드 수가 많으면 **Context Switching** 비용이 증가한다.

```
Thread Pool 크기: 200
실제 CPU 코어: 8

200개의 스레드가 8개의 코어를 놓고 경쟁
    ↓
빈번한 Context Switching
    ↓
CPU 사용량 증가 (실제 작업보다 스위칭 비용)
```

### 최적 스레드 수

```
CPU Bound 작업: 코어 수 + 1
I/O Bound 작업: 코어 수 * (1 + Wait Time / Service Time)
```

**잘못된 설정:**

```yaml
# CPU 8코어 서버에서
server:
  tomcat:
    threads:
      max: 500  # 너무 많음!
```

**적절한 설정:**

```yaml
server:
  tomcat:
    threads:
      max: 100  # I/O Bound 환경
```

# RPS 감소 ≠ CPU 감소인 이유

많은 개발자들이 오해하는 부분이다.

> "트래픽을 줄였는데 왜 CPU는 안 줄어들지?"

## 요청 수가 줄어도 CPU가 그대로인 이유

### 1. 무거운 코드 경로가 유지됨

```
[이전]
- 1000 RPS
- 캐시 히트율 80%
- 실제 DB 조회: 200 RPS
- CPU: 50%

[트래픽 50% 감소]
- 500 RPS
- 캐시 서버 장애로 히트율 0%
- 실제 DB 조회: 500 RPS (오히려 증가!)
- CPU: 60% (증가!)
```

**RPS는 줄었지만, 실제 처리 복잡도는 증가했다.**

### 2. GC는 백그라운드에서 계속 동작

```
[트래픽 감소 직후]
- RPS: 1000 → 500
- 하지만 Heap에는 이전 트래픽의 객체가 남아있음
    ↓
- Old Generation GC가 여전히 발생
- GC가 CPU를 계속 사용
```

**GC는 즉각적으로 멈추지 않는다.**

### 3. Old 영역 GC는 트래픽과 무관하게 발생

```
[시나리오]
10:00 - 대량 트래픽 (5000 RPS)
10:05 - 많은 객체가 Old Generation으로 승격
10:10 - 트래픽 감소 (1000 RPS)
10:15 - Old GC 발생! (10:05의 객체들 정리)
        → CPU 스파이크
```

**현재 트래픽이 낮아도, 과거 트래픽의 영향이 남아있다.**

## 특히 주의할 서비스 유형

### 객체를 많이 생성하는 서비스

```java
// 요청당 수십~수백 개의 임시 객체 생성
@GetMapping("/api/products")
public List<ProductDto> getProducts() {
    List<Product> products = productRepository.findAll();  // 객체 1

    return products.stream()
        .map(p -> {
            // 각 Product마다 여러 객체 생성
            ProductDto dto = new ProductDto();       // 객체 2
            dto.setImages(getImages(p.getId()));     // 객체 3...N
            dto.setReviews(getReviews(p.getId()));   // 객체 N+1...M
            return dto;
        })
        .collect(Collectors.toList());  // 새로운 List 객체
}
```

**이런 서비스의 특징:**
- 트래픽이 줄어도 CPU가 쉽게 내려가지 않음
- Young GC가 매우 빈번
- Old GC도 주기적으로 발생

### 예시: BFF / API Gateway

```
1 요청당:
- JSON 파싱: 10개 객체
- DTO 변환: 20개 객체
- 응답 직렬화: 15개 객체
  ↓
총 45개 임시 객체

1000 RPS = 45,000개 객체/초
```

**결과:**
- 요청 수가 절반으로 줄어도
- 여전히 22,500개 객체/초 생성
- GC 부담이 크게 줄지 않음

# API Gateway / BFF가 GC-heavy한 이유

BFF(Backend For Frontend)나 API Gateway는 구조적으로 **객체 생성이 많은** 서비스다.

## 주요 객체 생성 지점

### 1. JSON 파싱/직렬화

```java
// 외부 API 응답 받기
String jsonResponse = externalApi.call();  // JSON 문자열
ObjectMapper mapper = new ObjectMapper();
ApiResponse response = mapper.readValue(jsonResponse, ApiResponse.class);
// ↑ 수많은 임시 객체 생성

// 클라이언트에게 응답 보내기
String output = mapper.writeValueAsString(result);
// ↑ 또다시 임시 객체 생성
```

**과정:**
```
JSON 문자열
  ↓ (파싱)
JsonNode 객체들 (임시)
  ↓ (변환)
DTO 객체
  ↓ (직렬화)
JSON 문자열 (임시 버퍼, StringBuilder 등)
```

### 2. DTO 변환

```java
public class ProductConverter {
    public ProductDto convert(Product product) {
        // 각 변환마다 새 객체 생성
        ProductDto dto = new ProductDto();          // 객체 1
        dto.setId(product.getId());
        dto.setName(product.getName());
        dto.setPrice(new Money(product.getPrice())); // 객체 2
        dto.setImages(
            product.getImages().stream()
                .map(ImageDto::new)                  // 객체 3...N
                .collect(Collectors.toList())        // 객체 N+1
        );
        return dto;
    }
}
```

**1개 상품 변환 = 10개 이상의 객체 생성**

### 3. 외부 API 응답 가공

```java
@GetMapping("/api/user-dashboard")
public DashboardDto getDashboard(@AuthUser User user) {
    // 3개 외부 API 호출
    UserInfo userInfo = userServiceClient.getInfo(user.getId());      // 객체 생성
    List<Order> orders = orderServiceClient.getOrders(user.getId());  // 객체 생성
    List<Point> points = pointServiceClient.getPoints(user.getId());  // 객체 생성

    // 데이터 결합 및 변환
    DashboardDto dashboard = new DashboardDto();  // 객체 생성
    dashboard.setUser(convertUser(userInfo));     // 객체 생성
    dashboard.setOrders(
        orders.stream()
            .map(this::convertOrder)               // 각각 객체 생성
            .collect(Collectors.toList())
    );
    dashboard.setTotalPoints(
        points.stream()
            .mapToInt(Point::getAmount)
            .sum()  // 박싱/언박싱으로 Integer 객체 생성 가능
    );

    return dashboard;
}
```

**1 요청당 수십~수백 개의 임시 객체**

### 4. 임시 객체 대량 생성

```java
// Stream API 사용 시
list.stream()
    .filter(...)      // 임시 객체
    .map(...)         // 임시 객체
    .collect(...)     // 새로운 컬렉션 객체

// String 연결
String result = "";
for (String s : list) {
    result += s;  // 매번 새로운 String 객체 생성!
}

// 올바른 방식
StringBuilder sb = new StringBuilder();
for (String s : list) {
    sb.append(s);  // 객체 재사용
}
```

## 실제 측정 예시

**간단한 API Gateway 엔드포인트:**

```java
@GetMapping("/api/products")
public List<ProductDto> getProducts() {
    // 외부 API 호출
    String json = productService.getProductsJson();
    List<Product> products = parseJson(json);

    // 변환
    return products.stream()
        .map(this::convertToDto)
        .collect(Collectors.toList());
}
```

**객체 생성 측정:**
- JSON 파싱: ~50개 객체
- Product 변환: 20개 * 상품 수
- DTO 생성: 30개 * 상품 수
- Stream/Collection: ~10개 객체

**100개 상품 조회 시:**
- 총 ~5,060개 객체 생성
- 1000 RPS = 5,060,000개 객체/초

## 결과

**요청 수보다 객체 수가 CPU 사용량을 결정한다.**

```
서비스 A: 단순 조회
- 1000 RPS
- 요청당 10개 객체
- 총 10,000개/초
- CPU: 30%

서비스 B: API Gateway
- 500 RPS (절반!)
- 요청당 5000개 객체
- 총 2,500,000개/초
- CPU: 70% (2배 이상!)
```

# GC가 많은 서비스의 의미

GC가 빈번한 서비스는 다음과 같은 특징을 가진다.

## 짧은 생명의 객체를 대량 생성

```java
public void processRequest() {
    // 임시 객체들 - 요청 처리 후 바로 버려짐
    Request req = parseRequest();      // 수명: ~100ms
    ValidatedRequest validated = validate(req);  // 수명: ~50ms
    ProcessedData data = process(validated);     // 수명: ~30ms
    Response resp = createResponse(data);        // 수명: ~20ms
    // 모든 객체가 메서드 종료 후 GC 대상
}
```

**특징:**
- 대부분의 객체가 요청-응답 사이클 동안만 존재
- Young Generation에서 바로 정리됨
- **하지만 생성 속도가 빠르면 Young GC 빈번**

## Young GC가 매우 빈번

### 정상적인 서비스

```
Young GC: 5~10회/분
각 GC 시간: 10~20ms
영향: 거의 없음
```

### GC-heavy 서비스

```
Young GC: 60~100회/분
각 GC 시간: 30~50ms
영향: 상당함 (분당 3~5초 GC에 소비)
```

### 실제 모니터링

```
[GC 로그 예시]
2026-01-17T10:00:00.123: [GC pause (G1 Evacuation Pause) (young) 512M->128M(2G), 0.0234567 secs]
2026-01-17T10:00:05.456: [GC pause (G1 Evacuation Pause) (young) 512M->145M(2G), 0.0298765 secs]
2026-01-17T10:00:10.789: [GC pause (G1 Evacuation Pause) (young) 512M->132M(2G), 0.0267890 secs]
...
(5초마다 GC 발생)
```

**계산:**
- 5초마다 약 25ms GC
- 1분에 12회 GC
- 총 300ms/분 = GC에 0.5% 시간 소비

**만약 1초마다 GC:**
- 1분에 60회 GC
- 총 1,500ms/분 = GC에 2.5% 시간 소비

## 객체 일부가 Old 영역으로 승격

모든 객체가 Young Generation에서 정리되면 이상적이지만, 실제로는 일부가 Old Generation으로 승격된다.

### 승격(Promotion) 발생 이유

1. **Young GC 사이클을 여러 번 생존**
   ```java
   // 요청 처리 시간이 긴 경우
   public Response slowProcess() {
       Data data = loadData();  // 생성
       // Young GC 발생 (1차)
       Thread.sleep(100);       // 오래 걸리는 작업
       // Young GC 발생 (2차)
       process(data);           // 아직 사용 중
       // Young GC 발생 (3차)
       // → data 객체가 Old로 승격!
       return createResponse(data);
   }
   ```

2. **객체가 너무 큼**
   ```java
   // 큰 객체는 직접 Old Generation으로
   byte[] largeArray = new byte[10 * 1024 * 1024];  // 10MB
   ```

3. **Young Generation이 작음**
   ```
   JVM 설정:
   -Xms2G -Xmx2G -XX:NewRatio=2

   Total Heap: 2GB
   Young: 약 670MB
   Old: 약 1330MB

   객체 생성 속도가 빠르면 Young이 빨리 차고
   → GC 전에 Old로 승격되는 객체 증가
   ```

### Old GC의 영향

```
[Young GC]
- 빈도: 높음 (수십 회/분)
- 시간: 짧음 (수십 ms)
- 영향: 누적되면 상당

[Old GC]
- 빈도: 낮음 (수 회/시간)
- 시간: 김 (수백 ms ~ 수 초)
- 영향: 발생 시 명확한 지연
```

**실제 로그:**

```
2026-01-17T10:30:15.123: [Full GC (Allocation Failure) 1800M->800M(2G), 1.2345678 secs]
```

**1.23초 동안 애플리케이션 정지 (Stop-The-World)!**

## GC 자체가 CPU 사용의 중요한 비중을 차지

### CPU 사용량 분석

```
전체 CPU 사용량: 60%
  ├─ 실제 비즈니스 로직: 40%
  ├─ GC: 15%
  ├─ I/O 대기: 3%
  └─ 기타: 2%
```

**GC가 전체 CPU의 25% (15/60)를 차지!**

### GC CPU 사용량이 높은 경우

```
전체 CPU 사용량: 80%
  ├─ 실제 비즈니스 로직: 45%
  ├─ GC: 30%
  └─ 기타: 5%
```

**GC가 전체 CPU의 37.5%를 차지!**

**이 경우:**
- 트래픽을 줄여도 GC 부담이 남아있음
- CPU가 쉽게 내려가지 않음
- **객체 생성을 줄이는 최적화가 필요**

# 핵심 요약

## CPU 사용량은 RPS보다 코드 경로와 GC의 영향을 더 크게 받음

**잘못된 가정:**
```
RPS ↓ = CPU ↓
```

**실제:**
```
CPU 사용량 = f(RPS, 코드 경로, GC, 스레드 관리)
```

**예시:**
- RPS가 절반으로 줄어도
- 캐시 미스로 무거운 코드 경로를 타면
- CPU는 오히려 증가할 수 있음

## TPS는 실제 CPU 부담을 더 잘 반영

**RPS vs TPS:**

```
서비스 A:
- 1000 RPS
- 각 요청당 1 TPS
- 총 1000 TPS
- CPU: 20%

서비스 B:
- 200 RPS (1/5 수준)
- 각 요청당 20 TPS
- 총 4000 TPS (4배!)
- CPU: 60% (3배!)
```

**교훈:**
- **TPS가 실제 작업량을 반영**
- **CPU 예측 시 TPS를 기준으로**

## RPS 감소가 CPU 감소로 직결되지 않음

**이유:**

1. **코드 경로 변화**
   - 캐시 히트율 하락
   - 무거운 로직으로 분기

2. **GC는 지연되어 발생**
   - 과거 트래픽의 영향
   - Old GC는 트래픽과 무관

3. **백그라운드 작업**
   - 스케줄링된 배치
   - 메트릭 수집
   - 로그 처리

## BFF / Gateway는 구조적으로 GC-heavy한 서비스

**이유:**

1. **JSON 파싱/직렬화**
   - 문자열 ↔ 객체 변환
   - 임시 객체 대량 생성

2. **DTO 변환**
   - 도메인 객체 → DTO
   - 여러 소스 데이터 결합

3. **외부 API 응답 가공**
   - 여러 서비스 호출
   - 데이터 매핑 및 변환

4. **임시 객체 대량 생성**
   - Stream API 사용
   - Collection 변환
   - String 조작

**결과:**
- 요청 수보다 **객체 생성 수**가 중요
- GC 튜닝이 필수
- 객체 풀링, 재사용 고려 필요

# 실전 최적화 가이드

이제 이론을 실전에 적용해보자.

## 1. 모니터링 설정

### GC 로그 활성화

```bash
# JVM 옵션 추가
-Xlog:gc*:file=/var/log/gc.log:time,uptime,level,tags
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

### 메트릭 수집

```yaml
# Prometheus + Micrometer 설정
management:
  metrics:
    export:
      prometheus:
        enabled: true
    enable:
      jvm: true
      process: true
      system: true
```

**주요 메트릭:**
- `jvm.gc.pause`: GC 일시정지 시간
- `jvm.gc.memory.allocated`: 할당된 메모리
- `jvm.gc.memory.promoted`: Old로 승격된 메모리
- `process.cpu.usage`: CPU 사용률

## 2. 코드 최적화

### 객체 생성 줄이기

**Before:**
```java
public List<ProductDto> getProducts() {
    return products.stream()
        .map(p -> {
            ProductDto dto = new ProductDto();
            dto.setId(p.getId());
            dto.setName(p.getName());
            // ... 많은 필드 설정
            return dto;
        })
        .collect(Collectors.toList());
}
```

**After:**
```java
// 객체 풀 사용
private final ObjectPool<ProductDto> dtoPool = new ObjectPool<>(ProductDto::new);

public List<ProductDto> getProducts() {
    List<ProductDto> result = new ArrayList<>(products.size());
    for (Product p : products) {
        ProductDto dto = dtoPool.borrow();
        dto.reset();  // 초기화
        dto.setId(p.getId());
        dto.setName(p.getName());
        result.add(dto);
    }
    return result;
}
```

### String 최적화

**Before:**
```java
String result = "";
for (String item : items) {
    result += item + ",";  // 매번 새 String 객체!
}
```

**After:**
```java
StringBuilder sb = new StringBuilder(items.size() * 10);
for (String item : items) {
    sb.append(item).append(',');
}
String result = sb.toString();
```

### Collection 사이즈 미리 지정

**Before:**
```java
List<Item> result = new ArrayList<>();  // 기본 크기 10
// 100개 추가 → 여러 번 resize → 배열 복사
for (int i = 0; i < 100; i++) {
    result.add(item);
}
```

**After:**
```java
List<Item> result = new ArrayList<>(100);  // 크기 미리 지정
for (int i = 0; i < 100; i++) {
    result.add(item);  // resize 없음
}
```

## 3. 캐싱 전략

### 응답 캐싱

```java
@Cacheable(value = "products", key = "#id")
public ProductDto getProduct(Long id) {
    // 무거운 로직
    // 캐시 히트 시 이 로직 실행 안 됨
    return expensiveOperation(id);
}
```

### 계산 결과 캐싱

```java
private final LoadingCache<String, ExpensiveResult> cache =
    CacheBuilder.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .build(key -> computeExpensiveResult(key));

public ExpensiveResult getResult(String key) {
    return cache.get(key);  // 캐시 미스 시에만 계산
}
```

## 4. GC 튜닝

### Young Generation 크기 조정

```bash
# Young을 크게 → GC 빈도 감소
-XX:NewRatio=1  # Young:Old = 1:1 (기본 1:2)
-XX:SurvivorRatio=8  # Eden:Survivor = 8:1

# 또는 직접 지정
-XX:NewSize=1G
-XX:MaxNewSize=1G
```

### G1GC 튜닝

```bash
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200  # 목표 일시정지 시간
-XX:G1HeapRegionSize=16M  # Region 크기
-XX:InitiatingHeapOccupancyPercent=45  # Old GC 시작 임계값
```

## 5. 코드 경로 최적화

### 조기 반환 (Early Return)

**Before:**
```java
public Response process(Request req) {
    Data data = loadData(req);
    if (data != null) {
        ValidatedData validated = validate(data);
        if (validated.isValid()) {
            ProcessedData processed = process(validated);
            if (processed.isSuccess()) {
                return createResponse(processed);
            }
        }
    }
    return errorResponse();
}
```

**After:**
```java
public Response process(Request req) {
    Data data = loadData(req);
    if (data == null) return errorResponse();

    ValidatedData validated = validate(data);
    if (!validated.isValid()) return errorResponse();

    ProcessedData processed = process(validated);
    if (!processed.isSuccess()) return errorResponse();

    return createResponse(processed);
}
```

### 불필요한 변환 제거

```java
// 캐시된 데이터는 이미 DTO 형태로 저장
@Cacheable(value = "products", key = "#id")
public ProductDto getProduct(Long id) {
    Product product = repository.findById(id);
    return convertToDto(product);  // 변환은 한 번만
}
```

# 마무리

애플리케이션 성능을 최적화하려면 단순히 트래픽 수치만 볼 것이 아니라, **실제 CPU를 사용하는 요소들을 이해**해야 한다.

## 핵심 개념 정리

### 1. RPS vs TPS
- **RPS**: 클라이언트 요청 수 (외부 관점)
- **TPS**: 실제 트랜잭션 수 (내부 관점)
- **TPS가 CPU 부담을 더 정확히 반영**

### 2. CPU 사용량 결정 요소
1. 요청 처리량 (RPS/TPS)
2. **코드 경로** (캐시 히트율, 분기 로직)
3. **GC** (객체 생성, Young/Old GC)
4. 스레드 관리 (Context Switching)

### 3. RPS 감소 ≠ CPU 감소
- 코드 경로가 무거워지면 RPS 감소해도 CPU 증가
- GC는 과거 트래픽의 영향으로 지연 발생
- Old GC는 트래픽과 무관하게 발생 가능

### 4. BFF/Gateway는 GC-heavy
- JSON 파싱/직렬화
- DTO 변환
- 외부 API 응답 가공
- **객체 생성 수가 CPU 사용량 결정**

## 실전 적용 체크리스트

### 모니터링
- [ ] GC 로그 활성화
- [ ] JVM 메트릭 수집 (Prometheus/Grafana)
- [ ] CPU/메모리 프로파일링 (JProfiler, YourKit)
- [ ] APM 도구 (Datadog, New Relic)

### 코드 최적화
- [ ] 불필요한 객체 생성 제거
- [ ] String 연산 최적화 (StringBuilder)
- [ ] Collection 사이즈 미리 지정
- [ ] Stream API 신중히 사용
- [ ] 캐싱 전략 수립

### GC 튜닝
- [ ] Young Generation 크기 조정
- [ ] GC 알고리즘 선택 (G1GC, ZGC)
- [ ] GC 목표 설정 (MaxGCPauseMillis)
- [ ] 메모리 크기 적절히 설정

### 아키텍처
- [ ] 무거운 로직 비동기 처리
- [ ] 캐시 계층 추가
- [ ] 배치 처리 고려
- [ ] 서비스 분리 (마이크로서비스)

## 다음 단계

성능 최적화는 지속적인 과정이다:

1. **측정** → 현재 상태 파악
2. **분석** → 병목 지점 찾기
3. **최적화** → 개선 적용
4. **검증** → 효과 측정
5. **반복** → 지속적 개선

**기억할 점:**
- 추측하지 말고 측정하라
- 전체 시스템을 고려하라
- 과도한 최적화를 경계하라
- 가독성과 유지보수성도 중요하다

이러한 이해를 바탕으로 실제 서비스의 성능을 체계적으로 개선할 수 있다.

# 참고 자료

- [Java Garbage Collection Basics](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
- [Understanding G1GC](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-g1-garbage-collector1.html)
- [JVM Performance Optimization](https://www.baeldung.com/jvm-performance-optimization)
- [Micrometer Documentation](https://micrometer.io/docs)
- [The Garbage Collection Handbook](https://gchandbook.org/)
