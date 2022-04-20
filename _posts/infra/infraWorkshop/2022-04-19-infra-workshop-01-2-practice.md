---
title: "[인프라 공방 5기] 1주차 - 그럴듯한 인프라 만들기(2) : 실습"
last_modified_at: 2022-04-20T23:00:00+09:00
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

---

[⬅️ 인프라 공방 5기 목차보기](/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/)

---

인프라 공방 5기 : NEXTSTEP 에서 진행하는 '인프라 공방 5기' 1주차 '그럴듯한 인프라 만들기' 실습에 대한 내용을 정리한다.
{: .notice--info}

# 학습 목표

1. AWS 상에서 네트워크를 구성하며, 네트워크 기본 개념들을 학습한다.
2. 컨테이너를 학습하고 3 tier로 운영환경을 구성한다.
3. 개발 환경을 구성해보고 지속적 통합을 경험한다.

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

- Spec
  - Amazone Machine Image : Ubuntu 64bit(Ubuntu Server 18.04 LTS)
  - Instance Type : t3.medium
  - Key Pair : KEY-tonyjev93
  - VPC : tonyjev93-vpc
  - 퍼블릭 IP 자동할당 : 활성화

| 구분  | 인스턴스 이름               | 서브넷                   | 보안 그룹              | Private IP     | Public IP     |
|-----|-----------------------|-----------------------|--------------------|----------------|---------------|
| 외부망 | EC2-tonyjev93-web     | tonyjev93-external-01 | tonyjev93-external | 192.168.10.26  | 3.35.169.198  |
| 내부망 | EC2-tonyjev93-db     | tonyjev93-internal-01 | tonyjev93-internal | 192.168.10.144  | 3.34.132.214  |
| 관리망 | EC2-tonyjev93-bastion     | tonyjev93-admin-01 | tonyjev93-admin | 192.168.10.168  | 13.124.44.222 |

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

# 참고

- [인프라 공방 5기](https://edu.nextstep.camp/c/VI4PhjPA/)

<br>

---

[⬅️ 인프라 공방 5기 목차보기](/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/)

---