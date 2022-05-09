---
title: "[인프라] 웹 성능 - 부하 테스트"
last_modified_at: 2022-05-09T23:00:00+09:00
categories:
    - Infra
    - 웹 성능 테스트
tags:
    - Infra
    - 웹 성능 테스트
    - k6
toc: true
toc_sticky: true
toc_label: "목차"
---

웹 성능 테스트 : 인프라 성능 진단을 위해 사용되는 웹 성능 부하테스트에 대한 내용을 정리한다.
{: .notice--info}

# k6

- 시나리오 기반 테스트 툴
- 자바스크립트를 통한 테스트 로직 작성

<br>

# 테스트 종류

- Smoke Test
  - 최소한의 부하로 구성된 테스트 -> 테스트 시나리오 정상 동작여부 확인
  - VUser 1~2로 구성하여 테스트
- Load Test
  - 서비스의 평소 트래픽과 최대 트래픽 상황에서 성능이 어떤지 확인. 이 때 정상 동작여부도 확인
  - 애플리케이션 배포 및 인프라 변경(scale out, DB failover 등)시 성능 변화 확인
  - 외부 요인(결제 등)에 따른 예외 상황 확인
- Stress Test
  - 서비스가 극한의 상황에서 어떻게 동작하는지 확인
  - 장기간 부하 발생 시 한계치 확인. (기능 정상동작 여부 확인)
  - 최대 사용자, 최대 처리량 확인
  - 스트레스 테스트 후 수동작업 없이 자동복구 가능 여부 확인

<br>

# 테스트 설정값 구하기

## 목표 rps 구하기
- 1일 예상 사용자 수(DAU) 정의
- 피크 시간대 집중률 예상 (최대 트래픽 / 평소 트래픽)
- 1명당 1일 평균 접속 혹은 요청수 예상
- Throughput 계산(1일 평균 rps ~ 1일 최대 rps)
  - 1일 총 접속 수 = 1일 사용자 수(DAU) x 1명당 1일 평균 접속 수
  - 1일 평균 rps = 1일 총 접속 수 / 86,400 (초/일)
  - 1일 최대 rps = 1일 평균 rps x 피크 시간대 집중률(최대 트래픽 / 평소 트래픽)

## VUser 구하기
- Request Rate(rps) : 1초당 요청 수
- VU : 가상 사용자 수
- R : VU 반복당 요청 수
- T : VU 반복을 완료하는 데 필요한 시간보다 큰 값

```
T = (R * http_req_duration) (+ 1s) // 내부망에서 테스트할 경우 예상 latency를 추가한다, http_req_duration : 요청 왕복 시간
VUser = (목표 rps * T) / R
```

가령, 두개의 요청(R=2)이 있고, 왕복시간(http_req_duration)이 0.5s, 지연시간이 1초라고 가정할 때, T=2 이다.
목표 rps를 300이라 할 때, `VU = (300 * 2) / 2 = 300`이다.

<br>

# 테스트 기간
- 일반적으로 Load Test는 보통 30분 ~ 2시간 사이로 권장
- 부하가 주어진 상황에서 DB Failover, 배포 등 여러 상황을 부여하며 서비스의 성능을 확인한다.

<br>

# 테스트 스크립트 작성

- 예시(smoke 테스트)

```javascript
// somke.js
import http from 'k6/http';
import {check, group, sleep, fail} from 'k6';

export let options = {
  vus: 1, // 1 user looping for 1 minute
  duration: '10s',

  thresholds: {
    http_req_duration: ['p(99)<1500'], // 99% of requests must complete below 1.5s
  },
};

const BASE_URL = '[Target URL]';
const USERNAME = 'test id';
const PASSWORD = 'test password';

export default function () {

  var payload = JSON.stringify({
    email: USERNAME,
    password: PASSWORD,
  });

  var params = {
    headers: {
      'Content-Type': 'application/json',
    },
  };


  let loginRes = http.post(`${BASE_URL}/login/token`, payload, params);

  // 로그인 성공 여부 테스트
  check(loginRes, {
    'logged in successfully': (resp) => resp.json('accessToken') !== '',
  });


  let authHeaders = {
    headers: {
      Authorization: `Bearer ${loginRes.json('accessToken')}`,
    },
  };

  let myObjects = http.get(`${BASE_URL}/members/me`, authHeaders).json();

  // 나의 정보 조회 성공여부 테스트
  check(myObjects, {'retrieved member': (obj) => obj.id != 0});
  sleep(1); // 지연 시간
};
```

- [소스 설명](https://k6.io/docs/using-k6/metrics/)
  - options
    - vus = 가상 사용자 수
    - duration = 진행 시간
    - Thresholds > http_req_duration = 목표 설정. ex) 'p(99)<1500' : 모든 요청의 99% 가 1500ms 안에 들어야 함
  - check
    - 테스트 결과 도출 방법
    - 테스트 결과를 기록
  - sleep
    - 지연시간 가정

## load 테스트

- 평소 & 예상 최대 트래픽 상황에서 버티는지 확인 (동일 VUser 로 지속적으로 부하 발생)

```javascript
// load.js
...

export let options = {
  stages: [
    {duration: '5m', target: 100},
    {duration: '20m', target: 100},
    {duration: '5m', target: 0},
  ],

  thresholds: {
    http_req_duration: ['p(99)<1500'],
  },
};

...
```

- target = vus
- target 이 변경된 경우 : duration 동안 설정된 target 만큼 VUser 수가 변경됨
- target 이 동일한 경우 : 해당 VUser 수 유지  

## stress 테스트

- 극한의 상황에서 어떻게 동작하는지 확인 및 한계치 확인 (VUse 점진적 증가)

```javascript
// stress.js
...

export let options = {
  stages: [
    {duration: '5m', target: 100},
    {duration: '10m', target: 100},
    {duration: '5m', target: 200},
    {duration: '10m', target: 200},
    {duration: '5m', target: 300},
    {duration: '10m', target: 300},
    {duration: '5m', target: 0},
  ],
  thresholds: {
    http_req_duration: ['p(99)<1500'],
  },
};

...
```

<br>

# 테스트 시작

- script 경로 내에서 아래 명령어 수행

```shell
$ k6 run smoke.js
```

<br>

# 참고

- [인프라 공방 5기](https://edu.nextstep.camp/c/VI4PhjPA/) - 강의자료