---
title: "[인프라 공방 5기] 2주차 - 성능 진단하기(2) : 실습"
last_modified_at: 2022-04-29T23:00:00+09:00
categories:
    - Infra
    - NEXTSTEP
    - 인프라 공방 5기 
tags:
    - Infra
    - NEXTSTEP
    - 인프라공방 5기
toc: true
toc_sticky: true
toc_label: "목차"
---

인프라 공방 5기 : NEXTSTEP 에서 진행하는 '인프라 공방 5기' 2주차 '성능 진단하기' 실습에 대한 내용을 정리한다.
{: .notice--info}

---

<div style="
  display: flex;
  justify-content: space-between;
">
  <a href="/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/">⬅️ 인프라 공방 5기 목차보기</a>
</div>

---

# 학습 목표

- USE 방법론과 쓰레드 덤프 활용을 통한 서버 진단
- webpagetest, pagespeed 활용을 통한 웹 성능 예산 고민
- 성능 목표치 설정 및 부하테스트 수행 

<br>

# 부하테스트

## 요구사항

- [x] 테스트 전제조건 정리
  - [x] 대상 시스템 범위
  - [x] 목푯값 설정 (latency, throughput, 부하 유지기간)
  - [x] 부하 테스트 시 저장될 데이터 건수 및 크기
- [x] 각 시나리오에 맞춰 스크립트 작성
  - [x] 접속 빈도가 높은 페이지
  - [x] 데이터를 갱신하는 페이지
  - [x] 데이터를 조회하는데 여러 데이터를 참조하는 페이지
- [x] Smoke, Load, Stress 테스트 후 결과를 기록

## k6 사용방법

- k6 : 부하테스트 툴

### k6 설치

```shell
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
$ echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
$ sudo apt-get update
$ sudo apt-get install k6
```

### Test script 작성

- k6는 javascript 기반으로 테스트 script를 작성한다.

```javascript
// script.js
import http from 'k6/http';
import { sleep } from 'k6';

export default function () {
  //http.get('http://test.k6.io');
  http.get('http://본인이 부하테스트를 희망하는 사이트를 여기에 입력해주세요.');
  sleep(1);
}
```

### 실행

```shell
$ k6 run --vus 10(가상 사용자 수) --duration 30s(부하시간) --out csv=test.csv(출력되는 결과물 형식) script.js(실행할 스크립트)
```

## 테스트 설정값 구하기

### 목표 rps 구하기
  - 1일 예상 사용자 수(DAU) 정의
  - 피크 시간대 집중률 예상 (최대 트래픽 / 평소 트래픽)
  - 1명당 1일 평균 접속 혹은 요청수 예상
  - Throughput 계산(1일 평균 rps ~ 1일 최대 rps)
    - 1일 총 접속 수 = 1일 사용자 수(DAU) x 1명당 1일 평균 접속 수
    - 1일 평균 rps = 1일 총 접속 수 / 86,400 (초/일)
    - 1일 최대 rps = 1일 평균 rps x 피크 시간대 집중률(최대 트래픽 / 평소 트래픽)

### VUser 구하기
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

### 테스트 기간
- 일반적으로 Load Test는 보통 30분 ~ 2시간 사이로 권장
- 부하가 주어진 상황에서 DB Failover, 배포 등 여러 상황을 부여하며 서비스의 성능을 확인한다.


## 대시보드 구성

### influx db

- 특징
  - influx는 오픈 소스 기반의 시계열 데이터베이스이다.(시간 흐름을 중심으로 데이터 관리)
  - 운영 모니터링, 애플리케이션 매트릭스, 사물인터넷 센서 데이터, 실시간 분석 등 분야에서 최적화 되어있다.
  - influx db는 8086 포트 점유
- 설치

```shell
$ sudo curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
$ sudo echo "deb https://repos.influxdata.com/ubuntu bionic stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
$ sudo apt update
$ sudo apt install influxdb
```
 
- 실행

```shell
# 서비스 중지
$ sudo systemctl stop influxdb

# 서비스 시작
$ sudo systemctl start influxdb

# 서버 재시작 후 자동 실행
$ sudo systemctl enable --now influxdb
$ sudo systemctl is-enabled influxdb
  
# 서비스 상태 확인
$ sudo systemctl status influxdb

# 서비스 헬스 체크
$ curl {server_ip}:8086/health
```

- 설정 

```shell
$ sudo nano /etc/influxdb/influxdb.conf
```

```
[http]
# Determines whether HTTP endpoint is enabled.
enabled = true <-- 주석 제거
# Determines whether the Flux query endpoint is enabled.
# flux-enabled = false
```

- 계정 생성 및 접속

```shell
# 계정 생성
$ curl -XPOST "http://{server_ip}:8086/query" --data-urlencode "q=CREATE USER {user_name} WITH PASSWORD '{your_password}' WITH ALL PRIVILEGES"

# 계정 접속
$ influx -username '{user_name}' -password '{your_password}'

# Query TEST
$ curl -G http://{server_ip}:8086/query -u {user_name}:{your_password} --data-urlencode "q=SHOW DATABASES" 
```

<br>

### grafana

**특징**

- grafana 시계열 데이터랑 메트릭 정보를 보여주기 위한 오픈소스 대시보드 툴
- Graphite, Elasticsearch, OpenTSDB, Prometheus, InfluxDB, Cloudwatch 등을 데이터 소스로 이용
- grafana는 3000 포트 점유

**설치**

```shell
$ sudo apt-get update && sudo apt-get upgrade -y 
$ sudo apt-get install -y apt-transport-https 
$ sudo apt-get install -y software-properties-common wget 
$ wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add - 
$ echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
$ sudo apt-get update 
$ sudo apt-get install grafana
```

**실행**

```shell
$ service grafana-server start
```
 
**접속** 
- 브라우저로 `{server_ip}:3000` 접속
- 초기 계정(id/password) : `admin / admin`

**Datasource 설정**

- Configuration > Data Source

![img](https://user-images.githubusercontent.com/53864640/165964409-9bcc198b-92c8-47be-bb41-d6a30d88a96f.png)

![image](https://user-images.githubusercontent.com/53864640/165978922-3eb4ee2a-d63b-40b2-b2e5-44aed8c8743a.png)

- URL에 influxDB 접속 URL 입력

![image](https://user-images.githubusercontent.com/53864640/165978965-181f820e-1291-4191-879b-1fdc85f3eb01.png)

- Database 에 본인의 influxDB에서 생성된 DB 이름 입력
- User/Password 에 본인이 생성한 계정을 입력

**Dashboard 설정**

- Dashboard > import
  - URL: [https://grafana.com/grafana/dashboards/2587](https://grafana.com/grafana/dashboards/2587) 입력

![img](https://user-images.githubusercontent.com/53864640/165978645-116b6295-355b-4653-a17c-df2e095141c7.png)

![image](https://user-images.githubusercontent.com/53864640/165979405-abe7b6bd-3a20-4eea-8622-b85da307875a.png)

- k6 > InfluxDB 선택

**테스트 시작**

```shell
$ k6 run --out influxdb=http://{server_ip}:8086/{database} {test_script.js}
```

- Dashboard 화면

![image](https://user-images.githubusercontent.com/53864640/165981757-6a85eec6-52ba-4906-afc0-12b15b1e76de.png)

<br>

# 로깅, 모니터링

## 요구사항

- [x] 애플리케이션 진단하기 실습을 진행해보고 문제가 되는 코드를 수정
- [x] 로그 설정하기
- [x] Cloudwatch로 모니터링

## 로그 설정하기

### Application Log 파일로 저장하기 

- [x] 회원가입, 로그인 등의 이벤트에 로깅을 설정
- [x] 경로찾기 등의 이벤트 로그를 JSON으로 수집

**주의사항**

- Avoid side effects
  - logging으로 인한 애플리케이션 동작에 영향을 미쳐서는 안된다.
  - ex) logging 시점에 NPE가 발생해 프로그램이 정상적으로 동작하지 않는 상황이 발생하면 안됨
- Be concise descriptive
  - 각 Logging에는 데이터와 설명이 모두 포함되어있어야 함
- Log method arguments and return values
  - 메소드의 Input, Output을 로그로 남기면 Debugger를 사용하지 않아도 됨.(Debugger를 사용할 수 없는 상황에서 유용)
  - AOP를 활용하여 메소드 앞, 뒤 부분에 발생할 중복 코드를 제거한다.
- Delete personal information
  - 로그에 사용자의 개인정보(전화번호, 계좌번호, 패스워드, 주소 등)를 남기지 않는다.

**logging level**

- `ERROR` : 예상하지 못한 심각한 문제가 발생하여 즉시 조치 필요
- `WARN` : 로직상 유효성 확인, 예상 가능한 문제로 인한 예외처리 등. 서비스 운영은 가능하지만 주의해야 함
- `INFO` : 운영에 참고할만한 사항으로 중요한 비즈니스 프로세스를 로깅
- `DEBUG/TRACE` : 개발 단계에서만 사용. 운영 단계에서는 사용하지 않음
    

**logback 설정 참고**
- [https://bamdule.tistory.com/29](https://bamdule.tistory.com/29) <- 정리 잘되어 있는 블로그

<br>

### Nginx Access Log 설정하기

- Docker의 volume 옵션을 통해 호스트 경로와 도커 경로를 마운트 한다.

```shell
$ docker run -d -p 80:80 -v /var/log/nginx:/var/log/nginx nextstep/reverse-proxy
```

<br>

**cAdvisor**

- 도커 상태 모니터링
- 설치
    ```shell
    docker run \
      --volume=/:/rootfs:ro \
      --volume=/var/run:/var/run:ro \
      --volume=/sys:/sys:ro \
      --volume=/var/lib/docker/:/var/lib/docker:ro \
      --volume=/dev/disk/:/dev/disk:ro \
      --publish=8080:8080 \
      --detach=true \
      --name=cadvisor \
      google/cadvisor:latest
    ```
- 모니터링에 필요한 디렉토리 볼륨 지정(보안을 위해 읽기전용 사용 `:ro`)
- 8080포트 개방 필요


## Cloudwatch로 모니터링

- IAM Role 설정 (in 모니터링 대상 EC2)

![image](https://user-images.githubusercontent.com/53864640/166231791-4483862c-8460-4769-99c5-8393e812c3f3.png)

![image](https://user-images.githubusercontent.com/53864640/166231811-1d3dc8af-c3b0-4428-ac06-9cb16c2624bf.png)

- 설치

```shell
# 설치
$ curl https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py -O

# 파이썬 설치
$ sudo apt install python

# 실행
$ sudo python ./awslogs-agent-setup.py --region  ap-northeast-2
# ㄴ Access Key, Secret Key 등 하지 말것! IAM Role 설정으로 충분함
```

### Cloudwatch로 로그 수집하기

- 설정파일 수정

```shell
$ sudo vi /var/awslogs/etc/awslogs.conf
```

```
[/var/log/syslog]
datetime_format = %b %d %H:%M:%S
file = /var/log/syslog
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = [로그그룹 이름]

[/var/log/nginx/access.log]
datetime_format = %d/%b/%Y:%H:%M:%S %z
file = /var/log/nginx/access.log
buffer_duration = 5000
log_stream_name = access.log
initial_position = end_of_file
log_group_name = [로그그룹 이름]

[/var/log/nginx/error.log]
datetime_format = %Y/%m/%d %H:%M:%S
file = /var/log/nginx/error.log
buffer_duration = 5000
log_stream_name = error.log
initial_position = end_of_file
log_group_name = [로그그룹 이름]
```

- `[로그그룹 이름]` : 개인의 github id로 지정

```shell
$ sudo service awslogs restart
```

### Cloudwatch로 메트릭 수집하기

```shell
$ wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb
$ sudo dpkg -i -E ./amazon-cloudwatch-agent.deb
```

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "metrics": {
    "metrics_collected": {
      "disk": {
        "measurement": [
          "used_percent",
          "used",
          "total"
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "*"
        ]
      },
      "mem": {
        "measurement": [
          "mem_used_percent",
          "mem_total",
          "mem_used"
        ],
        "metrics_collection_interval": 60
      }
    }
  }
}
```

```shell
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status

# -- status -- #
{
  "status": "running",
  "starttime": "2022-05-02T12:25:34+00:00",
  "configstatus": "configured",
  "cwoc_status": "stopped",
  "cwoc_starttime": "",
  "cwoc_configstatus": "not configured",
  "version": "1.247350.0b251780"
}
```

### USE 방법론을 활용하기 용이하도록 대시보드 구성

- AWS 검색창에 Cloudwatch 입력

![img](https://user-images.githubusercontent.com/53864640/166233892-3084b600-3024-46a7-9861-bb53eabc8ff4.png)

- 위젯 추가 > 행 선택

![img](https://user-images.githubusercontent.com/53864640/166233900-f923da18-81a9-4af1-a485-6a5ae02601de.png)

- 지표 선택

![img](https://user-images.githubusercontent.com/53864640/166236273-6ca84a98-88c5-4eac-95cc-9784d37fff07.png)

- 검색어에 모니터링을 원하는 필드 입력
  - CPU Utilization, Network In / Out, mem_used_percent, disk_used_percent 등 추가

![img](https://user-images.githubusercontent.com/53864640/166235947-bb4061d6-6907-4b35-976a-b74557753795.png)

- 최종 대시보드 구성 모습

<br>

# 참고

- [인프라 공방 5기](https://edu.nextstep.camp/c/VI4PhjPA/) - 강의자료

<br>

---

<div style="
  display: flex;
  justify-content: space-between;
">
  <a href="/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/">⬅️ 인프라 공방 5기 목차보기</a>
</div>

---