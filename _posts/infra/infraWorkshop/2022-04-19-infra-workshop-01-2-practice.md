---
title: "[인프라 공방 5기] 1주차 - 그럴듯한 인프라 만들기(2) : 실습"
last_modified_at: 2022-04-24T00:50:00+09:00
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

인프라 공방 5기 : NEXTSTEP 에서 진행하는 '인프라 공방 5기' 1주차 '그럴듯한 인프라 만들기' 실습에 대한 내용을 정리한다.
{: .notice--info}

---

[⬅️ 인프라 공방 5기 목차보기](/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/)

---

# 학습 목표

1. AWS 상에서 네트워크를 구성하며, 네트워크 기본 개념들을 학습한다.
2. 컨테이너를 학습하고 3 tier로 운영환경을 구성한다.
3. 개발 환경을 구성해보고 지속적 통합을 경험한다.

<br>

# 0단계 - pem 키 생성하기

## 요구사항

- [x] pem 키 생성하기

## 방법

- `AWS > EC2 > 키 페어` 접근
- [키 페어 생성] 버튼 클릭

![img](https://user-images.githubusercontent.com/53864640/164216439-e4e7a73c-05c8-424c-ae32-570d6d9636a3.png){: .align-center}

- 생성 완료 후 pem 파일 다운로드 및 개인 저장소에 보관

## pem 키를 어디에 사용할까?

- 생성된 pem 키는 앞으로 EC2에 SSH를 통해 원격으로 접근할 때 이용 된다.
- 접근할 원격 EC2 에는 생성된 pem 키의 public key가 `~/.ssh` 폴더 하위에 들어가게 되고, private 키는 원격 EC2에 접근하고자 하는 단말기 내에 존재해야 한다.

<br>

# 1단계 - 망 구성하기

## 요구사항

![img](https://techcourse-storage.s3.ap-northeast-2.amazonaws.com/b3c9ac82a4e94f2d95e1db822716408c){: .align-center}

- [x] 웹 서비스를 운영할 네트워크 망 구성하기
- [x] 웹 애플리케이션 배포하기

## 망구성

### VPC 생성
  - CIDR은 C class(x.x.x.x/24)로 생성. -> `192.168.10.0/24`

![image](https://user-images.githubusercontent.com/53864640/164218682-30cf8397-89f5-4828-90c3-1fc02b6d98f4.png){: .align-center}

![image](https://user-images.githubusercontent.com/53864640/164218346-de1e32eb-4716-47ed-ae0c-3b32c97781d2.png){: .align-center}

<br>

### Subnet 생성
- [x] 외부망으로 사용할 Subnet : 64개씩 2개 (AZ 다르게 구성)
- [x] 내부망으로 사용할 Subnet : 32개씩 1개
- [x] 관리용으로 사용할 Subnet : 32개씩 1개
  
![image](https://user-images.githubusercontent.com/53864640/164221015-086f4397-0ba5-4a65-aa9e-16a899a4b09c.png){: .align-center}

| 구분  | 서브넷 이름                | 가용 영역(AZ)       | IPv4 CIDR 블록      |
|-----|-----------------------|-----------------|-------------------|
|외부망| tonyjev93-external-01 | ap-northeast-2a | 192.168.10.0/26   |
|외부망| tonyjev93-external-02 | ap-northeast-2c | 192.168.10.64/26  |
|내부망| tonyjev93-internal-01 | ap-northeast-2a | 192.168.10.128/27 |
|관리용| tonyjev93-admin-01 | ap-northeast-2c | 192.168.10.160/27 |

<br>

### Internet Gateway 연결
- [x] Internet Gateway 생성
  - ![image](https://user-images.githubusercontent.com/53864640/164222337-2a8b2ec9-4cc7-4454-8e98-2c9dafd958c8.png)

- [x] Internet Gateway <-> VPC 연결
  - ![image](https://user-images.githubusercontent.com/53864640/164222495-24fa67f8-7e93-423b-b0f0-0a49321b81e2.png)

<br>

### Route Table 생성

![image](https://user-images.githubusercontent.com/53864640/164222904-41e1358e-d6fc-4a4e-8262-7b36c4a688b0.png)

- 외부망 전용 Route
  - 0.0.0.0/0 : Internet Gateway 연결 (내부 <-> 외부 양방향 통신)
  - 서브넷 연결 : tonyjev93-external-01, tonyjev93-external-02, tonyjev93-admin-01

![image](https://user-images.githubusercontent.com/53864640/164243171-81d009eb-a693-458e-a7ae-31b35254b09c.png)

- 내부망 전용 Route
  - 0.0.0.0/0 : NAT 연결 (내부 -> 외부 단방향 통신)
  - 서브넷 연결 : tonyjev93-internal-01

![image](https://user-images.githubusercontent.com/53864640/164244178-18d45b0e-c529-41f1-b984-a35e0b640032.png)

<br>

### Security Group 설정

- [x] 외부망 (tonyjev93-external)
  - 전체 대역 : 8080 포트 오픈
  - 관리망 : 22번 포트 오픈
  - ![image](https://user-images.githubusercontent.com/53864640/164227189-ca4f23ae-384b-40d4-a0d6-0c2980e0a4ea.png)
- [x] 내부망 (tonyjev93-internal)
  - 외부망 : 3306 포트 오픈
  - 관리망 : 22번 포트 오픈
  - ![image](https://user-images.githubusercontent.com/53864640/164228458-59f8000e-edf6-498c-8034-d7cc134b6ece.png)
- [x] 관리망 (tonyjev93-admin)
  - 자신의 공인 IP : 22번 포트 오픈
  - ![image](https://user-images.githubusercontent.com/53864640/164225975-5009d275-7619-4dcd-a061-73fa17337232.png)

<br>

### 서버 생성

- [x] 외부망에 웹 서비스용도의 EC2 생성
- [x] 내부망에 데이터베이스용도의 EC2 생성
- [x] 관리망에 베스쳔 서버용도의 EC2 생성
- [x] 고정 IP 할당(탄력적 IP 할당)
  - ![image](https://user-images.githubusercontent.com/53864640/164457913-0a5d67c9-54e5-42c6-b98e-e66fb528d1a0.png)
  - ![image](https://user-images.githubusercontent.com/53864640/164458114-dea303af-0671-4e9e-9c68-85e9ff90a680.png)
  
- **Spec**
  - Amazone Machine Image : Ubuntu 64bit(Ubuntu Server 18.04 LTS)
  - Instance Type : t3.medium
  - Key Pair : KEY-tonyjev93
  - VPC : tonyjev93-vpc
  - 퍼블릭 IP 자동할당 : 활성화

| 구분  | 인스턴스 이름               | 서브넷                   | 보안 그룹              | Private IP     | Public IP        |
|-----|-----------------------|-----------------------|--------------------|----------------|------------------|
| 외부망 | EC2-tonyjev93-web     | tonyjev93-external-01 | tonyjev93-external | 192.168.10.26  | 52.78.72.73(고정)  |
| 내부망 | EC2-tonyjev93-db     | tonyjev93-internal-01 | tonyjev93-internal | 192.168.10.144  | -                |
| 관리망 | EC2-tonyjev93-bastion     | tonyjev93-admin-01 | tonyjev93-admin | 192.168.10.168  | 52.79.68.166(고정) |


<br>

- [x] 베스쳔 서버에 Session Timeout 600s 설정

```shell
$ sudo vi ~/.profile
  HISTTIMEFORMAT="%F %T -- "    ## history 명령 결과에 시간값 추가
  export HISTTIMEFORMAT
  export TMOUT=600              ## 세션 타임아웃 설정 
    
$ source ~/.profile
$ env
```
- Timeout 으로 자동 logout
  - ![img](https://user-images.githubusercontent.com/53864640/164260424-ecbd8b1e-322c-43d4-8405-c7680141b82d.png)

<br>

- [x] 베스쳔 서버에 shell prompt 변경하기
  - 구분해야 하는 서버의 Shell Prompt를 설정하여 관리자의 인적 장애 예방

```shell
$ sudo vi ~/.bashrc
  USERNAME=BASTION
  PS1='[\e[1;31m$USERNAME\e[0m][\e[1;32m\t\e[0m][\e[1;33m\u\e[0m@\e[1;36m\h\e[0m \w] \n\$ \[\033[00m\]'

$ source ~/.bashrc
```
- 반영 후 prompt 모습
  - ![image](https://user-images.githubusercontent.com/53864640/164254496-22ef31ff-a585-46a6-954a-be0245d8ac01.png)

<br>

- [x] 베스쳔 서버에 Command 감사로그 설정
  - 서버에 직접 접속하여 작업할 경우, 작업 이력 히스토리를 기록해두어야 장애 발생시 원인을 분석 할 수 있음.

```shell
$ sudo vi ~/.bashrc
  tty=`tty | awk -F"/dev/" '{print $2}'`
  IP=`w | grep "$tty" | awk '{print $3}'`
  export PROMPT_COMMAND='logger -p local0.debug "[USER]$(whoami) [IP]$IP [PID]$$ [PWD]`pwd` [COMMAND] $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" )"'

$ source  ~/.bashrc
```

```shell
$ sudo vi /etc/rsyslog.d/50-default.conf
  local0.*                        /var/log/command.log
  # 원격지에 로그를 남길 경우 
  local0.*                        @원격지서버IP
    
$ sudo service rsyslog restart
$ tail -f /var/log/command.log
```

- 사용한 명령어 이력
  - ![img](https://user-images.githubusercontent.com/53864640/164261301-ad1a5cc2-d1c0-4c8a-ad7e-098a0601a5f7.png)

<br>

## 웹 애플리케이션 배포

- [x] 외부망에 [웹 애플리케이션](https://github.com/next-step/infra-subway-deploy) 배포
- [x] DNS 설정 - [http://www.tonyjev93.kro.kr:8080](http://www.tonyjev93.kro.kr:8080) 로 설정 완료
  - [무료 DNS 설정 사이트](https://xn--220b31d95hq8o.xn--3e0b707e/)
    - ![image](https://user-images.githubusercontent.com/53864640/164451650-4142cab0-59cb-4d39-9052-0f7cd2627683.png)
    - ![image](https://user-images.githubusercontent.com/53864640/164451472-e5929ed8-b64d-499e-879e-5c608bf3c636.png)
    
<br>

# 2단계 - 배포하기

## 요구사항

- [x] 운영 환경 구성하기
- [x] 개발 환경 구성하기

### docker 설치

```shell
$ sudo apt-get update && \
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
sudo apt-key fingerprint 0EBFCD88 && \
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" && \
sudo apt-get update && \
sudo apt-get install -y docker-ce && \
sudo usermod -aG docker ubuntu && \
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && \
sudo chmod +x /usr/local/bin/docker-compose && \
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### docker 명령어 권한 부여

```shell
# docker 명령어 권한 부여
$ sudo usermod -aG docker $(whoami)
$ sudo reboot
```

## 운영 환경 구성하기

- [x] 웹 애플리케이션 앞단에 Reverse Proxy 구성하기
  - [x] 외부망에 Nginx로 Reverse Proxy를 구성
    - [nginx 설정값](https://prohannah.tistory.com/136)
  - [x] Reverse Proxy 에 TLS 설정
    - letsencrypt를 활용한 무료 TLS 인증서 사용
    - ```shell
  $ docker run -it --rm --name certbot \
  -v '/etc/letsencrypt:/etc/letsencrypt' \
  -v '/var/lib/letsencrypt:/var/lib/letsencrypt' \
  certbot/certbot certonly -d 'tonyjev93.kro.kr' --manual --preferred-challenges dns --server https://acme-v02.api.letsencrypt.org/directory
  ```

![image](https://user-images.githubusercontent.com/53864640/164715016-71cef179-c8b2-4975-b13f-504ea10508c5.png)

- [무료 DNS 설정 사이트](https://xn--220b31d95hq8o.xn--3e0b707e/) 에서 DNS TXT 레코드로 추가한다.

![image](https://user-images.githubusercontent.com/53864640/164715140-2def3578-a325-41f4-b7b2-a8ec8eff3c62.png)

- 생성된 인증서 현재 경로로 옮기기

```shell
$ cp /etc/letsencrypt/live/[도메인주소]/fullchain.pem ./
$ cp /etc/letsencrypt/live/[도메인주소]/privkey.pem ./
```

- nginx Dockerfile 수정

```
FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf 
COPY fullchain.pem /etc/letsencrypt/live/[도메인주소]/fullchain.pem
COPY privkey.pem /etc/letsencrypt/live/[도메인주소]/privkey.pem
```

- nginx.conf 파일 수정

```
...

http {       
  upstream app {
    server 172.17.0.1:8080;
  }
  
  # Redirect all traffic to HTTPS
  server {
    listen 80;
    return 301 https://$host$request_uri;
  }

  server {
    listen 443 ssl;  
    ssl_certificate /etc/letsencrypt/live/[도메인주소]/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/[도메인주소]/privkey.pem;

    # Disable SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    # 통신과정에서 사용할 암호화 알고리즘
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;

    # Enable HSTS
    # client의 browser에게 http로 어떠한 것도 load 하지 말라고 규제합니다.
    # 이를 통해 http에서 https로 redirect 되는 request를 minimize 할 수 있습니다.
    add_header Strict-Transport-Security "max-age=31536000" always;

    # SSL sessions
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;      

    location / {
      proxy_pass http://app;    
    }
  }
}
```

- Docker 재빌드 및 재실행

```shell
$ docker stop proxy && docker rm proxy
$ docker build -t nextstep/reverse-proxy .
$ docker run -d -p 80:80 -p 443:443 --name proxy nextstep/reverse-proxy
```

- [x] 운영 데이터베이스 구성하기

```shell
# 실습용 DB (id : root / password: masterpw)
$ docker run -d -p 3306:3306 brainbackdoor/data-subway:0.0.1
```

## 개발 환경 구성하기

- [x] 설정 파일 나누기
  - Junit : h2
  - Local : docker(mysql)
  - Prod : 운영 DB를 사용하도록 설정

``` 
application-local.properties
application-prod.properties
application-test.properties
```

- 위와 같이 properties 파일들을 배포 환경 별로 나눌 수 있음
- jar 실행 시 `-Dspring.profiles.active=prod` 옵션을 통해 설정 파일을 결정할 수 있다.

```shell
$ java -jar -Dspring.profiles.active=prod [jar파일명]
```

<br>

# 3단계 - 배포 스크립트 작성하기

- [x] 배포 스크립트 작성하기 
  - 반복적으로 사용하는 명령어를 Script로 작성해 본다.

## 변수 설정

```shell
# ./color.sh

## 변수 설정
txtrst='\033[1;37m' # White
txtred='\033[1;31m' # Red
txtylw='\033[1;33m' # Yellow
txtpur='\033[1;35m' # Purple
txtgrn='\033[1;32m' # Green
txtgra='\033[1;30m' # Gray
```

```shell
# ./colorTest.sh
source $(dirname $0)/color.sh

echo -e "${txtylw}=======================================${txtrst}"
echo -e "${txtgrn}  << 색깔 변수들 🧐 >>${txtrst}"
echo -e "${txtrst}txtrst='\033[1;37m' # White${txtrst}"
echo -e "${txtred}txtred='\033[1;31m' # Red${txtrst}"
echo -e "${txtylw}txtylw='\033[1;33m' # Yellow${txtrst}"
echo -e "${txtpur}txtpur='\033[1;35m' # Purple${txtrst}"
echo -e "${txtylw}txtgrn='\033[1;32m' # Green${txtrst}"
echo -e "${txtgra}txtgra='\033[1;30m' # Gray${txtrst}"
echo -e "${txtylw}=======================================${txtrst}"
```

<<출력 결과>>
{: .text-center}
![image](https://user-images.githubusercontent.com/53864640/164912098-a02b7845-dc24-4e68-93e7-4f090374f1d2.png){: .align-center}

## 기능 단위 함수선언

```shell
# ./functionTest.sh
source $(dirname $0)/color.sh

function pull() {

  echo -e ">> Pull Request 🏃"
  git pull origin tonyjev93
}

pull;
```

## 파라미터 전달

```shell
# ./parameterTest.sh
source $(dirname $0)/color.sh

# 변수 선언
EXECUTION_PATH=$(pwd) # 해당 스크립트를 실행시킨 경로
SHELL_SCRIPT_PATH=$(dirname $0) # 해당 스크립트의
FIRST=$1 # 1 번째 입력 값
SECOND=$2 # 2 번째 입력 값

# 변수 출력
echo -e "${txtylw}============= 변수 출력 ===============${txtrst}"
echo -e "${txtgrn} EXECUTION_PATH: ${EXECUTION_PATH}"
echo -e "${txtgrn} SHELL_SCRIPT_PATH: ${SHELL_SCRIPT_PATH}"
echo -e "${txtgrn} FIRST: ${FIRST}"
echo -e "${txtgrn} SECOND: ${SECOND}"
echo -e "${txtgrn} \$#: $#"  # 입력 인자 개수

## 조건 설정
if [[ $# -ne 0 ]] # 입력 인자 개수($#) 가 0이 아닌 경우(-ne)
then
    echo -e "${txtylw}=======================================${txtrst}"
    echo -e "${txtgrn}  << 스크립트 🧐 >>${txtrst}"
    echo -e ""
    echo -e "${txtgrn} $0 브랜치이름 ${txtred}{ prod | dev }"
    echo -e "${txtylw}=======================================${txtrst}"
    exit
fi

## ...
```

- 실행 명령어
```shell
# $0 = ./parameterTest.sh, $1 = first, $2 = second
$ ./parameterTest.sh first second 
```

## 반복적으로 동작(crontab)

cron 
: 소프트웨어 데몬
: 유닉스 계열 시간 기반 작업 스케쥴러
: crontab 이라는 파일에 의해 구동 됨
: 리눅스 User 마다의 crontab 파일을 가질 수 있음

- 두 가지 방식
  1. /etc/crontab 파일
  2. crontab -e 명령어

### 1. /etc/crontab 파일

- 관리자 권한 접근 필요

```shell
$ sudo -i
```

- crontab 파일 수정

```shell
$ sudo vi /etc/crontab
```

```shell
- Crontab 규칙

* * * * *  유저이름 명령어
┬ ┬ ┬ ┬ ┬
│ │ │ │ └─ 요일 (0 - 6) (0:일요일, 1:월요일, 2:화요일, …, 6:토요일)
│ │ │ └─ 월 (1 - 12)
│ │ └─일 (1 - 31)
│ └─ 시 (0 - 23)
└─ 분 (0 - 59)
```

- 예시

![image](https://user-images.githubusercontent.com/53864640/165104891-642df82f-eff4-481e-94e2-6dd32db25b46.png)

```shell
# m h dom mon dow user  command
  * *  *   *   *  root  /home/ubuntu/nextstep/infra-subway-deploy/test.sh >> /var/log/cron.log
```

- 매분 마다 `/home/ubuntu/nextstep/infra-subway-deploy/test.sh` 명령어를 수행하며, `/var/log/cron.log` 파일에 log 를 남긴다.
  - `>` : 기존 로그를 삭제(truncate)하고 신규 로그 추가
  - `>>` : 기존 로그 + 신규 로그(누적)

### 2. crontab -e 명령어

- crontab 관련 명령어들
  - `crontab -l` : 사용중인 크론탭 리스트 확인
  - `crontab -e` : 크론탭 수정
  - `crontab -r` : 모든 크론탭 삭제하기
  - `service cron start` : 크론탭 실행하기
  - `service cron status` : 크론탭 상태 확인하기
  - `service cron stop` : 크론탭 중지하기
  - `service cron restart` : 크론탭 재시작하기

**[사용 방법]**
- 계정 별 crontab 설정이 가능하므로 실행시킬 계정으로 접속
- `crontab -e` 명령어 수행 - 최초에 편집기 선택하도록 나옴(default : `nano`)
  - 추 후, vim 편집기로 변경을 원한다면 `export VISUAL=vim; crontab -e` 명령어 수행
- crontab 파일 수정
  
```shell
# m h  dom mon dow   command
  * *  *   *   *    /home/ubuntu/nextstep/infra-subway-deploy/test.sh > /home/ubuntu/nextstep/infra-subway-deploy/cron-by-e.log
```
- 크론탭 실행 : `service cron start`

**[참고]**

- crontab 수행이 안될 때
  - `crontab status` 명령어 수행 시 `(CRON) info (No MTA installed, discarding output)`이라는 로그가 존재한다면,
  - `sudo apt-get install postfix` 명령어 수행
  - ![image](https://user-images.githubusercontent.com/53864640/165108137-ba55d988-9617-403c-a5e6-3a93272404a8.png)
  - 위 와 같은 화면이 나타나면 `No configuration` 선택 후 OK 누르면 됨.(email 서버 선택하라는 화면인데 무시.)
- Ubuntu 서버 시간설정
  - `date` 명령어로 현재 세팅된 시스템 시간 확인(기본 세팅 : UTC)
  - UTC -> KST 변경
    - 관리자 권한으로 변경 : `sudo -i`
    - `dpkg-reconfigure tzdata` 명령어 수행
    - ![image](https://user-images.githubusercontent.com/53864640/165113721-86fdde86-3255-46b4-89e1-00085f1129e3.png)
    - 위와 같은 화면이 등장 하면 **Asia** 선택 후 OK
    - ![image](https://user-images.githubusercontent.com/53864640/165112706-75fd478c-3436-47e8-b1ef-34a3cd2bffa1.png)
    - 위와 같이 **Seoul** 선택 후 OK

<br>

# 고찰

NEXTSTEP 의 `인프라 공방 5기` 첫 주차 수업을 완료하였다. 본인은 인프라 관련 지식과 경험이 부족 했기에 일부러 인프라 관련 학습을 피해왔었다.<br>
그러나 언제까지나 계속 피할 수는 없었기에 양질의 수업과 강의 자료를 제공해주는 이번 수업을 통해서 인프라 관련 학습과 경험을 얻어 가고자 한다.<br><br>
첫 주차 주제는 '그럴듯한 인프라 만들기'였다.<br>
이번 주제를 통해 학습목표에 맞게 AWS를 통한 네트워크 구성을 할 수 있게 되었고, 네트워크의 기본 개념에 대해 학습할 수 있었다.<br>
AWS를 처음 접해본 터라 처음에는 수 많은 메뉴에 우왕좌왕 했지만 점차 익숙해지며 AWS의 편리함에 매력을 느끼게 되었다. 그리고 직접 서버를 구성해가며 네트워크와 보안 설정을 세팅한 결과가 즉각적으로 반영되어 확인 할 수 있다는 점에서 실습을 통한 이론공부가 잘 이루어졌다고 생각한다.<br><br>
그리고 웹서버(nginx)를 직접 구현해 가며 무료 SSL과 무료 DNS 등록과 같은 새로운 서비스들에 대해 알게 되었고, nginx에 대해서도 나름의 학습을 할 수 있게 되었다.<br><br>
docker 컨테이너를 통한 서비스 배포는 기존에도 해왔던 것이라 그렇게 새롭지는 않았다.<br><br>
'배포 스크립트 작성하기' 미션은 `shell script`를 작성하는 재미를 느끼게 해주었다. 그 동안 직접 shell script를 작성할 기회가 없었기에 이번 활동이 신선하고 재미있었다.<br>
crontab을 통해 script를 주기적으로 수행하는 방법에 대해 이론으로만 알고있었던 것을 실습과 학습을 통해 많은 경험을 얻게 되었다.<br><br>
이번 주차의 활동을 통해 서비스 준비를 위해 필요한 기본적인 개발 환경 구축에 대해 전반적으로 살펴볼 수 있었고, 앞으로 새로운 서비스를 시작하게 된다면 '그럴듯한 인프라'를 직접 만들 수 있을 것이라는 자신감을 얻게 되었다.
{: .notice--primary}

## 미션수행 PR
- [1주차 - 0단계](https://github.com/next-step/infra-subway-deploy/pull/354)
- [1주차 - 1단계](https://github.com/next-step/infra-subway-deploy/pull/379)
- [1주차 - 2단계](https://github.com/next-step/infra-subway-deploy/pull/391)
- [1주차 - 3단계](https://github.com/next-step/infra-subway-deploy/pull/411)

<br>

# 참고

- [인프라 공방 5기](https://edu.nextstep.camp/c/VI4PhjPA/)

<br>

---

[⬅️ 인프라 공방 5기 목차보기](/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/)

---