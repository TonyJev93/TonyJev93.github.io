---
title: "[Java] GC(Garbage Collection) 핵심 개념 - 기본부터 실전까지"
last_modified_at: 2026-01-17T10:00:00+09:00
categories:
    - Java
tags:
    - Java
    - GC
    - Garbage Collection
    - JVM
    - Memory
    - Performance
toc: true
toc_sticky: true
toc_label: "목차"
---

Java의 자동 메모리 관리 시스템인 GC(Garbage Collection)의 핵심 개념을 이해하고, 실제 서비스 성능에 어떤 영향을 미치는지 알아본다.
{: .notice--info}

# 들어가며

많은 개발자들이 "GC가 알아서 메모리를 관리해준다"는 사실은 알고 있지만, 실제로 GC가 어떻게 동작하는지, 왜 CPU를 많이 사용하는지, 어떻게 최적화해야 하는지는 잘 모르는 경우가 많다.

이 글에서는 GC의 **가장 기본적인 개념**부터 시작하여, 실제 서비스 운영에서 마주하는 상황들을 이해할 수 있도록 설명한다.

---

# 1. GC란 무엇인가

## 정의

**Garbage Collection(GC)**은 사용되지 않는 객체를 메모리에서 자동으로 제거하는 작업이다.

```
개발자가 직접 메모리 관리 (C/C++)
─────────────────────────────────
malloc() → 메모리 할당
free()   → 메모리 해제 (개발자가 직접!)
         → 잊으면 메모리 누수 발생

JVM의 자동 메모리 관리 (Java)
─────────────────────────────────
new 객체 → 메모리 할당
(자동)   → GC가 알아서 제거
         → 개발자는 신경 쓰지 않아도 됨
```

## GC의 목적

### 1. 메모리 누수 방지

```java
// C/C++에서는 위험한 코드
void createObjects() {
    Object* obj = new Object();
    // delete obj; 를 깜빡하면 메모리 누수!
}

// Java에서는 안전
void createObjects() {
    Object obj = new Object();
    // GC가 알아서 정리
}
```

### 2. 안정적인 메모리 재사용

- 사용하지 않는 메모리를 회수
- 새로운 객체 생성을 위한 공간 확보
- 메모리 부족(OutOfMemoryError) 방지

## JVM이 수행하는 자동 메모리 관리

중요한 점은 **GC는 애플리케이션 대신 JVM이 수행**한다는 것이다.

```
애플리케이션 (개발자 코드)
    │
    ├─ 객체 생성
    ├─ 비즈니스 로직 수행
    └─ 객체 사용 종료
        ↓
JVM (Garbage Collector)
    │
    ├─ 사용되지 않는 객체 탐지
    ├─ 메모리에서 제거
    └─ 메모리 공간 확보
```

**대신 이 작업에는 CPU 비용이 발생한다!**

---

# 2. GC의 판단 기준

GC는 어떻게 "사용되지 않는 객체"를 판단할까?

## 참조(Reference)의 존재 여부

핵심은 **객체에 참조가 있는지 없는지**다.

```java
// 1. 참조가 있는 객체 (살아있음)
String name = "Tony";  // name이 참조하고 있음 → GC 대상 아님

// 2. 참조가 없는 객체 (죽은 객체)
new String("Temp");    // 아무도 참조하지 않음 → GC 대상!

// 3. 참조가 끊긴 객체
String data = new String("Data");
data = null;           // 참조 끊김 → GC 대상!
```

## GC의 판단 기준

```
객체 생성
    ↓
참조가 있는가?
    ├─ Yes → 살아있는 객체 (Live Object)
    │         GC 대상 아님
    │
    └─ No  → 죽은 객체 (Dead Object)
              GC 대상!
```

## Reachability (도달 가능성)

더 정확히는 **GC Root에서 도달 가능한지**를 확인한다.

```
GC Root (시작점)
    ├─ Stack의 로컬 변수
    ├─ Static 변수
    └─ JNI 참조
        ↓
    [객체 A] ─→ [객체 B] ─→ [객체 C]
        │
        └─→ [객체 D]

도달 가능: A, B, C, D → GC 대상 아님
도달 불가능: [객체 X] (참조 끊김) → GC 대상!
```

---

# 3. JVM 메모리 구조 (개념 수준)

GC가 동작하는 공간을 이해해야 한다.

## Heap 메모리 구조

```
┌─────────────────────────────────────────┐
│              JVM Heap                    │
├─────────────────────────────────────────┤
│         Young Generation                │
│  ┌──────────┬────────┬────────┐         │
│  │  Eden    │ Survivor 0 │ Survivor 1 │ │
│  │          │  (S0)  │  (S1)  │         │
│  └──────────┴────────┴────────┘         │
├─────────────────────────────────────────┤
│         Old Generation                  │
│  (Tenured)                              │
│                                         │
└─────────────────────────────────────────┘
```

### Young 영역 (Young Generation)

**새로 생성된 객체**가 할당되는 공간

- **Eden**: 객체가 최초로 생성되는 곳
- **Survivor 0/1**: Minor GC에서 살아남은 객체가 이동하는 곳

특징:
- 객체가 빠르게 생성되고 소멸됨
- Minor GC가 자주 발생
- GC 속도가 빠름

### Old 영역 (Old Generation)

**오래 살아남은 객체**가 저장되는 공간

특징:
- Young 영역에서 여러 번 살아남은 객체가 이동
- Major/Full GC가 발생
- GC 속도가 느림

---

# 4. 객체 생명주기 흐름

객체는 다음과 같은 생명주기를 거친다.

## 전체 흐름

```
[1단계: 객체 생성]
new Object()
    ↓
Eden 영역에 할당

[2단계: Minor GC 발생]
Eden이 가득 참
    ↓
Minor GC 실행
    ├─ 살아있는 객체 → Survivor 0로 이동 (age = 1)
    └─ 죽은 객체 → 메모리에서 제거

[3단계: 반복적인 Minor GC]
Eden이 다시 가득 찬 후 Minor GC
    ├─ 살아있는 객체 → Survivor 1로 이동 (age = 2)
    └─ 죽은 객체 → 제거

다시 Minor GC
    ├─ 살아있는 객체 → Survivor 0로 이동 (age = 3)
    └─ 죽은 객체 → 제거

[4단계: Old 영역으로 승격]
age가 임계값(예: 15) 도달
    ↓
Old 영역으로 이동 (Promotion)

[5단계: Major/Full GC]
Old 영역이 가득 참
    ↓
Major/Full GC 발생
    ├─ 살아있는 객체 → Old에 유지
    └─ 죽은 객체 → 제거
```

## 코드 예제

```java
public class GCLifecycleExample {
    public void processRequests() {
        // 1. Eden에 생성
        String tempData = new String("임시 데이터");

        // 2. 메서드 종료 시 참조 끊김 → Minor GC 대상
    }

    // 3. 오래 살아남는 객체
    private static Cache cache = new Cache(); // Old 영역으로 이동

    public static void main(String[] args) {
        GCLifecycleExample example = new GCLifecycleExample();

        for (int i = 0; i < 10000; i++) {
            example.processRequests(); // 많은 임시 객체 생성
        }
        // → Minor GC 빈번히 발생
    }
}
```

---

# 5. Minor GC vs Major GC

## Minor GC

**Young 영역**을 정리하는 GC

```
특징
─────────────────────
영역: Young Generation (Eden + Survivor)
발생 빈도: 매우 빈번
소요 시간: 짧음 (수 ms ~ 수십 ms)
영향: 비교적 적음
대상: 짧은 생명의 객체
```

### Minor GC 발생 시나리오

```java
public void handleRequest(HttpRequest request) {
    // 요청마다 생성
    String requestId = UUID.randomUUID().toString();
    Map<String, Object> data = new HashMap<>();
    Response response = processRequest(request);

    // 메서드 종료 시 모두 GC 대상
}

// 1초에 1000번 호출되면?
// → Eden이 빠르게 차고
// → Minor GC가 자주 발생
```

## Major / Full GC

**Old 영역**을 정리하는 GC

```
특징
─────────────────────
영역: Old Generation
발생 빈도: 낮음
소요 시간: 김 (수백 ms ~ 수 초)
영향: 큼 (Stop-The-World)
대상: 오래 살아남은 객체
```

### Major GC 발생 시나리오

```java
public class CacheManager {
    // Old 영역에 저장되는 큰 객체
    private static Map<String, Object> cache = new HashMap<>();

    public void addToCache(String key, Object value) {
        cache.put(key, value);
        // cache는 계속 살아있음 → Old 영역
    }

    // cache가 계속 커지면
    // → Old 영역이 가득 참
    // → Major GC 발생
}
```

## 비교표

| 구분 | Minor GC | Major / Full GC |
|------|----------|-----------------|
| **대상 영역** | Young Generation | Old Generation |
| **발생 빈도** | 매우 빈번 (초당 여러 번) | 낮음 (분/시간당) |
| **소요 시간** | 짧음 (수 ms) | 김 (수백 ms ~ 수 초) |
| **CPU 사용** | 비교적 낮음 | 높음 |
| **애플리케이션 영향** | 적음 | 큼 (일시 정지 가능) |
| **정상 범주** | ✓ 정상 | ⚠️ 주의 필요 |

## Stop-The-World (STW)

GC가 실행되는 동안 **애플리케이션이 일시 정지**되는 현상

```
애플리케이션 실행 중
    ↓
GC 시작 (STW 발생)
    │
    ├─ 모든 애플리케이션 스레드 정지
    ├─ GC만 실행
    └─ 메모리 정리
    ↓
GC 완료 (STW 종료)
    ↓
애플리케이션 재개

Minor GC STW: 수 ms (거의 느껴지지 않음)
Major GC STW: 수백 ms ~ 수 초 (사용자가 느낄 수 있음!)
```

---

# 6. "GC가 많은 서비스"의 의미

## 특징

### 1. 요청 처리 중 객체를 많이 생성

```java
@RestController
public class ApiController {

    @GetMapping("/search")
    public List<SearchResult> search(String keyword) {
        // 1. 요청마다 새로운 객체 대량 생성
        List<String> tokens = tokenize(keyword);  // 객체 생성
        List<Document> docs = findDocuments(tokens);  // 객체 생성
        List<SearchResult> results = new ArrayList<>();  // 객체 생성

        for (Document doc : docs) {
            SearchResult result = new SearchResult();  // 객체 생성
            result.setTitle(doc.getTitle());
            result.setContent(doc.getContent());
            results.add(result);
        }

        // 2. 메서드 종료 시 모두 GC 대상
        return results;
    }
}

// 초당 1000건 요청
// → 초당 수천~수만 개 객체 생성
// → Minor GC 빈번히 발생
```

### 2. 짧은 생명의 객체 비중이 높음

```
일반 서비스
───────────────────────
객체 생성: 100개
  ├─ 짧은 생명 (임시): 70개 → Minor GC
  └─ 긴 생명 (캐시 등): 30개 → Old 영역

GC가 많은 서비스
───────────────────────
객체 생성: 10,000개
  ├─ 짧은 생명 (임시): 9,900개 → Minor GC 폭증!
  └─ 긴 생명 (캐시 등): 100개 → Old 영역
```

### 3. GC가 지속적으로 CPU를 사용

```
CPU 사용량 구성
───────────────────────
일반 서비스:
  ├─ 비즈니스 로직: 80%
  └─ GC: 20%

GC가 많은 서비스:
  ├─ 비즈니스 로직: 50%
  └─ GC: 50% ← CPU의 절반을 GC가 사용!
```

## 실제 증상

### RPS가 낮아도 CPU가 높을 수 있음

```
서비스 A (정상)
───────────────────────
RPS: 1,000
CPU: 30%
GC: 정상 범위

서비스 B (GC 과다)
───────────────────────
RPS: 500 (더 낮음!)
CPU: 70% (더 높음!)
GC: 빈번한 Minor GC, 주기적인 Full GC
```

### 트래픽 변동 없이도 CPU 스파이크 발생

```
시간대별 CPU 사용량
───────────────────────
10:00 - CPU 30% (트래픽 100 RPS)
10:05 - CPU 80% (트래픽 100 RPS) ← 트래픽은 같은데 CPU 급증!
        └─ Full GC 발생

10:10 - CPU 35% (트래픽 100 RPS) ← 다시 정상
```

## 모니터링 지표

```
GC가 많은 서비스 판단 기준
───────────────────────────────

Minor GC
  ├─ 빈도: 초당 10회 이상
  ├─ 소요 시간: 평균 10ms 이상
  └─ → 정상 범주 가능 (하지만 주의)

Major/Full GC
  ├─ 빈도: 시간당 10회 이상
  ├─ 소요 시간: 평균 500ms 이상
  └─ → ⚠️ 성능 문제 발생 가능!

GC CPU 사용률
  ├─ 10% 이하: 정상
  ├─ 10~30%: 주의
  └─ 30% 이상: ⚠️ 최적화 필요
```

---

# 7. 핵심 요약

## 1. GC는 "안 쓰는 객체를 정리하는 CPU 작업"

```
메모리 관리의 대가
───────────────────────
이점: 개발자가 메모리 관리 안 해도 됨
대가: CPU 리소스 사용
```

## 2. Young GC는 정상 범주

```
✓ 빈번하게 발생 (정상)
✓ 빠르게 처리 (수 ms)
✓ 영향 적음
```

## 3. Old GC부터 성능 문제로 이어짐

```
⚠️ 발생 빈도 증가 → 문제 신호
⚠️ 긴 처리 시간 (수백 ms ~ 초) → 사용자 영향
⚠️ Stop-The-World → 애플리케이션 일시 정지
```

## 4. 객체 생성 패턴이 GC 성격을 결정

```
객체를 적게 생성하는 서비스
───────────────────────────
  ├─ Minor GC 적음
  ├─ Major GC 거의 없음
  └─ CPU 사용 낮음

객체를 많이 생성하는 서비스
───────────────────────────
  ├─ Minor GC 빈번
  ├─ Major GC 주기적 발생
  └─ CPU 사용 높음
```

## 전체 흐름 정리

```
[객체 생성]
    ↓
Eden에 할당
    ↓
Eden 가득 참
    ↓
Minor GC 발생 (빠름)
    ├─ 죽은 객체 제거
    └─ 살아남은 객체 → Survivor
        ↓
    여러 번 살아남음
        ↓
    Old 영역으로 이동
        ↓
    Old 영역 가득 차면
        ↓
    Major/Full GC 발생 (느림, 주의!)
        ├─ 죽은 객체 제거
        └─ 살아남은 객체 유지
```

---

# 실전 팁

## ✅ GC 모니터링 방법

```bash
# 1. GC 로그 활성화 (Java 11+)
java -Xlog:gc*:file=gc.log:time,uptime,level,tags

# 2. JVM 옵션으로 GC 튜닝
-Xms2g          # 초기 Heap 크기
-Xmx2g          # 최대 Heap 크기
-XX:NewRatio=2  # Young:Old 비율
```

## ✅ GC 문제 해결 방향

```
문제 상황
───────────────────────
Full GC가 자주 발생하고 시간이 오래 걸림

원인 분석
───────────────────────
1. Heap 메모리 부족
2. 메모리 누수
3. 불필요한 객체 과다 생성

해결 방법
───────────────────────
1. Heap 크기 증가
2. 메모리 프로파일링으로 누수 지점 찾기
3. 객체 재사용 (Object Pool, StringBuilder 등)
4. GC 알고리즘 변경 (G1GC, ZGC 등)
```

## ❌ 피해야 할 패턴

```java
// ❌ 반복문 안에서 객체 대량 생성
for (int i = 0; i < 1000000; i++) {
    String temp = new String("data" + i); // GC 부담
}

// ✅ 객체 재사용
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000000; i++) {
    sb.setLength(0);  // 초기화
    sb.append("data").append(i);
}

// ❌ 불필요한 객체 생성
String result = "";
for (String s : list) {
    result += s;  // 매번 새로운 String 생성
}

// ✅ StringBuilder 사용
StringBuilder sb = new StringBuilder();
for (String s : list) {
    sb.append(s);  // 객체 재사용
}
```

---

# 다음 단계

GC 기본 개념을 이해했다면, 다음 주제로 넘어가보자:

1. **GC 알고리즘 비교**
   - Serial GC, Parallel GC, CMS, G1GC, ZGC, Shenandoah

2. **GC 튜닝**
   - Heap 크기 설정
   - Young/Old 비율 조정
   - GC 알고리즘 선택

3. **메모리 프로파일링**
   - VisualVM, JProfiler 사용법
   - Heap Dump 분석
   - 메모리 누수 탐지

4. **실전 트러블슈팅**
   - OutOfMemoryError 대응
   - Full GC 최적화
   - GC 로그 분석

---

# 참고 자료

- [Oracle Java GC 튜닝 가이드](https://docs.oracle.com/en/java/javase/17/gctuning/)
- [Java Memory Management - Baeldung](https://www.baeldung.com/java-memory-management-interview-questions)
- [Understanding Java Garbage Collection](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)
- [G1GC 이해하기](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-g1-garbage-collector1.html)
