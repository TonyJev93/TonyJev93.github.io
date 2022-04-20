---
title: "[인프라 공방 5기] 1주차 - 그럴듯한 인프라 만들기(1) : 배경지식"
last_modified_at: 2022-04-19T23:00:00+09:00
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

<div style="
  display: flex;
  justify-content: space-between;
">
  <a href="/infra/nextstep/인프라%20공방%205기/infra-workshop-00-overview/">⬅️ 인프라 공방 5기 목차보기</a>
  <a href="/infra/nextstep/인프라%20공방%205기/infra-workshop-01-2-practice/"> 1주차 - 그럴듯한 인프라 만들기(2) : 실습 보러가기 ➡️️</a>
</div>

---

인프라 공방 5기 : NEXTSTEP 에서 진행하는 '인프라 공방 5기' 1주차 '그럴듯한 인프라 만들기' 실습에 앞서 이에 필요한 배경지식을 정리한다.
{: .notice--info}


# 1. 왜 Public Cloud 인가요?

## Cloud 란

- 인터넷을 통해 원격으로 접근 가능한 모든 것

## Cloud Computing 이란

- 서버, 데이터베이스, 네트워킹 등 컴퓨팅 리소스를 인터넷을 통해 관리하는 것

## 클라우드를 사용하는 이유

- 관심사 분리
  - 서비스 제공자가 집중해야 할 대상은 서비스의 **Core Value**
  - 데이터/서버/네트워크 관리는 Cloud 제공 업체에서 진행
  
<br>

# 2. 망 분리하기

## 네트워크 학습 이유

- 서비스는 **네트워크**를 통해 사용자에게 **가치를 전달**
- **네트워크 상의 이슈**는 서비스에 심각한 **문제를 발생**시킴
- 그러므로, **안정적인 서비스 운영**을 위해 미리 **네트워크를 학습**할 필요가 있음

## 1) 통신망

### 망분리 필요성

- [Defense in depth](https://en.wikipedia.org/wiki/Defense_in_depth_%28computing%29)
  - 개인 정보를 다루는 DB 서버 등을 위한 내부망
  - 사용자가 접근하는 웹 서버를 위한 외부망

### 통신망 이란

- **노드**들과 이들 **노드**들을 연결하는 **링크**들로 구성된 하나의 **시스템**
    - 노드 : IP로 식별할 수 있는 대상
    - 링크 : 물리적 회선
- 하나의 **Subnet**을 **하나의 망**이라고 칭할 수 있음.


### AWS에서의 망

![image](https://user-images.githubusercontent.com/53864640/163997930-46ef57bc-4350-4b37-89e1-e2e857c46894.png){: .align-center}

- **Region** : 국가 / 지역
- **Availability Zone** : 데이터센터
- **VPC**
  - 하나의 Region 에 종속
  - 다수의 AZ 설정 가능
  - VPC IP 대역 내에서 망 구성

### L2 Switch

- Multiple Access를 위한 장비
- 서버에 있는 Network Interface Card의 MAC 주소를 통해 통신
- 통신방식
  - Forwarding : MAC 테이블에 정보가 있을 때
  - Flooding : MAC 테이블에 정보가 없을 때

### Router

- 서로 다른 네트워크간의 통신 중계
- 통신방식
    - Forwarding : MAC 테이블에 정보가 있을 때
    - Drop : MAC 테이블에 정보가 없을 때

### 인터넷 통신

- 외부 네트워크와 통신하기 위해 Public IP가 존재해야 함
- 라우터는 Private IP가 목적지일 때 인터넷 구간으로 보내지 않음
  - NAT 을 통해 Private IP -> Publlic IP 변환 작업 필요
- 자신이 속한 subnet 의 서버는 가상 스위치를 통해 직접 통신
- 자신이 속하지 않은 subnet 은 가상 라우터를 통해 직접 통신
- 그 외 전체 대역(0.0.0.0/0)은 Internet Gateway 로 통신


### Topology

![image](https://user-images.githubusercontent.com/53864640/163999548-c031ecec-cd68-47c3-8c18-4637e654996c.png){: .align-center}

## 2) 네트워크 장비

### Collision Domain & Broadcast Domain

- **Collision Domain**
  - 네트워크는 통신 전 보내고자 하는 단말이 다른 누군가와 통신 중인지 확인함 = Carrier Sense
  - 확인 결과 통신 중이지 않을 경우 데이터 전송을 함
  - 그런데 동시에 다른 단말에서도 통신 중이지 않음을 확인하고 데이터를 전송한 경우 충돌이 발생하게 됨 = Collision(충돌)
    - 따라서 데이터를 전송하고 나서 항상 정상적으로 보내졌는지 확인하는 절차를 가지고 충돌시에는 재전송을 하게 됨.
    - 이를 CSMA/CD(Carrier Sesnse Multiple Access / Collision Detect) 방식이라 하며, 이더넷이 사용하는 방식 (보내고 싶을 때 막 보내고 나중에 잘 갔는지 확인)
  - 즉, **Collision Domain**는 동시에 여러 단말에서 통신을 시도할 때 충돌이 발생할 가능성이 있는 단말들의 묶음을 나타냄
- **Broadcast Domain**
  - Boradcast 란 동일 통신망으로 연결된 전체 단말기에 데이터를 전송하는 것이다.
  - 스위치 이용 시 특정 단말이 전송 대상 단말에 대한 정보가 없는 상태에서 Broadcast를 통해 미지의 단말 정보를 얻어와 데이터를 전송 할 수 있게 된다.
  - 이렇게 Broadcast를 통해 데이터가 전달되어 나가는 영역을 Broadcast Domain이라고 한다.

### Hub VS Switch

- Hub
  - Layer 1 장비
  - 하나의 단말이 허브에 데이터를 보내고 있을 때 다른 단말에서 동시에 데이터를 허브에 전달할 경우 Collusion 발생
  - 즉, 동일 Hub 내에 묶여있는 모든 장비들이 Collision Domain에 속하게 됨. (Multi Access 불가능)
- Switch
  - Layer 2 장비
  - 2계층 장비로 헤더를 열어 MAC주소를 확인해 Port별로 Collusion Domain 을 나눌 수 잇음 (Multi Access 가능)

즉, Multi Access 구성할 경우 Switch를 사용해야 함.

## 3) VPC

- 하나의 서비스를 위한 네트워크를 다루는 단위
- 서브넷, 라우팅 테이블, 인터넷 게이트웨이 등 설정
  - 서브넷 : VPC에 설정한 네트워크 대역을 더 세부적으로 나눈 네트워크
  - 라우팅 테이블 : 서브넷이 다른 서브넷 혹은 외부망과 통신하기 위한 정보를 가지고 있음
  - 인터넷 게이트웨이 : 외부망과의 연결 담당
- Private IP 대역으로 CIDR을 설정하여 생성
- 하나의 Region 안에 VPC를 구성할 수 있으며 한국은 `ap-northeast-2` Region을 사용

## 4) 서브네팅

- VPC는 n개의 Subnet을 가질 수 있음.
- AWS의 AZ(Availablity Zone)는 물리적으로 나뉜 IDC 이다.
- Subnet 마다 AZ를 다르게 구성 = [재해 및 재난 대비 개인정보처리시스템의 물리적 안전조치](https://www.law.go.kr/%ED%96%89%EC%A0%95%EA%B7%9C%EC%B9%99/%EA%B0%9C%EC%9D%B8%EC%A0%95%EB%B3%B4%EC%9D%98%EC%95%88%EC%A0%84%EC%84%B1%ED%99%95%EB%B3%B4%EC%A1%B0%EC%B9%98%EA%B8%B0%EC%A4%80/(2019-47,20190607)/%EC%A0%9C12%EC%A1%B0) 고려
- 하나의 VPC 내에 구성된 Subnet 들은 물리적으로 다른 IDC에 구성되더라도 사설망을 통해 통신 가능
- [서브네팅 계산](https://www.ipaddressguide.com/cidr)

## 5) 외부 네트워크와 연결하기

- EC2는 Internet Gateway를 통해 외부 인터넷과 연결 가능
- 외부 통신을 위해 라우팅 테이블에 모든 대역(0.0.0.0/0)에 대한 통신을 Internet Gateway 로 요청하도록 설정해야 함
- 내부망은 별도의 라우팅 테이블 생성
- 내부망의 서버가 외부망에 접속해야 할 경우(외부 라이브러리 설치 등) NAT Gateway 활용
  - [Internet GW VS NAT GW](https://medium.com/awesome-cloud/aws-vpc-difference-between-internet-gateway-and-nat-gateway-c9177e710af6)


## 6) 접근 제어하기

- 서버 보안 패치 등은 AWS에 일임
- 우리는 네트워크 보안과 계정 보안(IAM)에 집중하면 됨
- AWS는 Security Group을 통해 특정 IP, Port에 대한 접근 제어 가능

<br>

# 3. 통신 확인하기

## 1) OSI 7 Layer

### 1계층. Physical Layer

- 역할 : Bit Stream(이진수 흐름) -> 아날로그(전기, 빛 등) 신호로 변환
- PDU : Bit
- 대표장비 : 케이블(LAN-UTP, WAN-Serial), 허브, 리피터, 커넥터(LAN-RJ45) 등
- 유형범위 : 로컬 장비간에 전송된 전기 또는 빛 신호

### 2계층. Data-Link Layer

- 역할 : MAC 주소를 이용한 노드간 연결
- PDU : Frame
- 대표장비 : 브릿지, L2 스위치 등
- 유형범위 : 로컬 장비간에 전송된 하위 수준 데이터 메시지
- 프로토콜 및 기술 : LAN-Ethernet Protocol, WAN-PPP, HDLC 등

### 3계층. Network Layer

- 역할 : 논리적 주소, 최초 출발지 -> 최종 목적지 최적경로 결정
- PDU : Packet 데이터그램
- 주소 : 논리적 주소 (IP(4Byte), IPX, Apple Talk)
- 대표장비 : 라우터, L3 스위치 등
- 유형범위 : 로컬 또는 원격 장비 간의 메시지
- 프로토콜 및 기술 : IPv4, IPv6, ARP, ICMP, IGMP, Routing Protocol(RIP, EIGRP, OSPF 등)

### 4계층. Transport Layer

- 역할 : 포트번호 통해 서비스 구분하여 데이터 전송
- PDU : Segment
- 주소 : Port(2Byte) (Well-known : 0 ~ 1023, 그 외 : 1024 ~ 65535)
- 대표장비 : L4 스위치 등
- 유형범위 : 소프트웨어 프로세스 간의 통신
- 프로토콜 및 기술 : TCP(확인응답-신뢰성), UDP(빠른속도), NetBEUI

### 5계층. Session Layer

- 역할 : 응용프로그램 간의 세션 수립/유지/종료
- 세션 : 두 사용자간 작업 시작부터 끝가지의 실시간 논리적 연결
- 유형범위 : 로컬 또는 원격 장비간의 세션
- 프로토콜 및 기술 : NetBIOS, 소켓, 네임드 파이프, RPC

### 6계층. Presentation Layer

- 역할 : 데이터의 표현(확장자 연결, 압축, 암호화, 변환)
- 유형범위 : 어플리케이션 데이터 표현
- 프로토콜 및 기술 : SSL, redirector, MIME

### 7계층. Application Layer

- 역할 : 사용자에게 인터페이스 제공, 원본데이터 생성
- PDU : 사용자 데이터
- 유형범위 : 어플리케이션 데이터
- 프로토콜 및 기술 : HTTP(TCP 80) / HTTPs(TCP 443) / SMTP(TCP 25) / POP3(TCP 110) / FTP(TCP 20, 21) / TFTP(UDP 69) / Telnet(TCP 23) / SSH(TCP 22) / DHCP(UDP 67, 68) / DNS(UDP 53) / SNMP(UDP 161, 162) 등

## 2) Ping check

- IP 정보만으로 서버에 요청이 가능한지 여부를 확인
- ICMP 프로토콜 사용

ICMP 란?
: IP가 신뢰성을 보장하지 않기 때문에 네트워크 장애나 중계 라우터 등의 에러에 대처할 수 없는데, ICMP는 오류정보를 발견하고 보고하는 기능을 담당하는 프로토콜이다.
: TCP가 아니기 때문에 Port 번호가 없음

그렇다면, **IP 정보로 통신 시 실제 서버의 위치는 어떻게 알 수 있을까?**

ARP(Address Resolution Protocol) 란?
: 논리적 주소인 IP주소 정보를 이용하여 물리적 주소인 MAC 주소를 알아와 통신이 가능하게 도와주는 프로토콜

ARP Request를 Broadcast로 요청하면 수신한 장비들 중 자신의 IP에 해당하는 장비가 응답을 한다.<br>
응답받은 NIC 포트 정보와 IP, MAC 주소를 기반으로 이후 통신을 진행한다.

```shell
$ ping {대상 IP}

# 통신 중간 거쳐가는 router 정보 출력
$ traceroute {대상 IP}
```

## 3) Port check

- 서비스의 정상 구동 여부를 확인할 수 있음

```shell
$ telnet {Target Server IP} {Target Service Port}
```

- 서버는 서비스에 하나의 포트번호를 오픈해두고도 많은 사용자와 연결을 맺을 수 있음
- 이를 가능하게 하는 이유는 [**소켓의 동작 방식**](http://jkkang.net/unix/netprg/chap2/net2_1.html)를 확인하면 됨.


<br>

# 4. 도커 컨테이너 이해하기

## 도커를 학습해야 하는 이유

- 우리가 원하는 것은 특정 환경에 종속되지 않은 상태로 어플리케이션을 띄우는 것
- 단순히 어플리케이션만 띄우고 싶을 뿐인데 OS 까지 띄우는 기존 가상 머신 방식은 엄청난 낭비
- 도커 컨테이너의 경우 필요한 커널은 호스트의 커널을 공유해 사용하고, 컨테이너 안에는 애플리케이션을 구동하기 위한 라이브러리 및 실행 파일만 존재하면 되기 때문에 이미지 용량이 가상 머신에 비해 대폭 줄어듦
- 이를 통해 애플리케이션의 개발과 배포가 편해지며, 여러 애플리케이션의 독립성과 확장성이 높아짐

## 컨테이너는 프로세스를 추상화

- 컨테이너는 이미지에 따라 실행되는 환경(파일 시스템)이 달라짐.
- 컨테이너가 서로 다른 파일시스템을 가질 수 있는 이유는 `chroot`를 활용하여 이미지(파일의 집합)를 루트 파일 시스템으로 강제로 인식시켜 프로세스를 실행하기 때문
- 컨테이너도 결국 **프로세스**

## 도커 이미지

- 이미지는 파일들의 집합
- 컨테이너는 이 파일들의 집합 위에서 실행되는 프로세스

## 도커 네트워크

### veth interface

- 랜카드에 연결된 실제 네트워크 인터페이스가 아닌, 가상으로 생성한 네트워크 인터페이스
- 패킷을 전달받으면 자신에게 연결된 다른 네트워크 인터페이스로 패킷을 보내주기 때문에 쌍(pair)으로 생성되어야 함.

### NET namespace

- 리눅스 격리 기술인 namespace 중 네트워크와 관련된 부분
- 네트워크 인터페이스를 각각 다른 namespace에 할당함으로써 서로가 서로를 모르게끔 설정

### 도커 네트워크 구조

![image](https://blog.kakaocdn.net/dn/bbz2B1/btqCF734kDK/E4w8EeBj3vZ1UHmNq1TdY1/img.png){: .align-center}

- 도커는 veth interface와 NET namespace를 사용해 네트워크를 구성
- 컨테이너는 namespace로 격리되고, 통신을 위한 네트워크 인터페이스(eth0)을 할당받음.
- host의 veth interface가 생성되고 컨테이너 내의 eth0과 연결 됨
- host의 veth interface는 docker0이라는 다른 veth interface와 연결 됨
  - docker0은 도커 실행 시 자동으로 생성되는 가상의 브릿지
  - 모든 컨테이너는 이 브릿지를 통해 서로 통신이 가능

## 도커 볼륨

- 도커 이미지는 컨테이너를 생성하면 이미지는 `읽기 전용`이 됨
- 볼륨 옵션을 통해 컨테이너 자체의 상태를 외부로부터 제공받도록 구성 가능

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
  <a href="/infra/nextstep/인프라%20공방%205기/infra-workshop-01-2-practice/"> 1주차 - 그럴듯한 인프라 만들기(2) : 실습 보러가기 ➡️️</a>
</div>

---