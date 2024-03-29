---
layout: single
title:  "[DevLog] AWS 교육 (General Immersion Day) 참가"
date:   2023-06-02 10:10:00 +0900

categories:
  - devlog
tags: [devlog]

author_profile: true
sidebar:
  nav: "docs"

toc: true
toc_label: "목록"
toc_icon: "bars"
toc_sticky: true

sitemap:
  changefreq: daily
  priority : 1.0
---

## 개요  
바야흐로 세상은 대-클라우드 시대... AWS 교육을 듣기 위하여 역삼 센터필드에 왔다.  
순서 : Network - 실습 - Compute - 실습 - Database -  실습 - Storage - 실습

## Intro  
AWS 리전은 더 높은 가용성, 확장성, 내결함성을 위해서 다중의 가용영역으로 구성됨.  
한국에는 4번째 가용영역을 열었고 지금도 운영되고 있음.  
콘텐츠 전송 네트워크 (CDN) -  최종 사용자에게 더 짧은 지연 시간으로 콘텐츠를 전송하기 위해 글로벌 네트워크를 이용  

AWS에 다양한 서비스가 있음  
AWS 보안 - 고객은 클라우드 환경 위에 올라가는 보안에 대한 책임이 있으며, AWS는 클라우드 운영 자체에 책임이 있음  
AWS Global INfrastructure 의 이점 : 보안, 가용성, 확장  

## 첫 번째 세션 : Network  
- VPC 란?  
- 엑세스 제어  
- VPC 연결, End-point  

### VPC 란?  
- 사용자가 정의한, 논리적으로 격리된 가상의 프라이빗 네트워크 환경  
- 리전 별 디폴트 VPC 가 생성되어 있음
  - 정해진 IP 주소가 있음.
  - 유연한 확장, 네트워크 관리 등이 어려울 수 있음.  
  - 되도록 새로운 VPC 를 만드는 것을 권장.   
- 다양한 bit 지원 (/16 ~ /28) -> IP range 를 원하는 만큼 생성  
  - 2^bit - 5개 (5개는 AWS에서 사용하는 개수)
  - IP range 를 추가할 수는 있지만, VPC 생성했을 때의 지정한 것은 변경이 불가능  
- 가상 네트워크 제어 기능 : IP 주소 범위, 서브네팅, 라우팅, 보안영역 등  
- 여러 개의 가용영역에 대하여 VPC 생성이 가능  
- 리전-가용영역 내에 VPC 가 생성되며 EC2 인스턴스나, RDS 인스턴스 등을 생성 (지정한 가용영역에 생성)  
  - S3, DynamoDB는 리전 서비스 영역
  - Amazon Route 53은 글로벌 서비스 영역  
- 리전 별로 생성 가능한 서비스가 5개 Limit -> Soft Limit 이기 때문에 신청을 통해 늘릴 수 있음  

### VPC 의 구성  
#### 서브넷
- VPC를 만들 때 내부에 만들 수 있는 것들을 동시에 만들 수 있도록 기능 제공  
- VPC 의 IP range 안에서 쪼개서 서브넷을 만들어야 함. (서브넷은 IP 신규 신청을 통한 확장도 어렵고, 할당 시 지정 range 를 벗어나면 deployment 어려움) 
 
#### 라우팅 테이블
[참고 링크 : 라우팅이란 무엇인가요?](https://aws.amazon.com/ko/what-is/routing/)  
- VPC 대역 내에서 라우팅  
- 서브넷 레벨로 라우팅 테이블에서 매핑하게 됨  
  - 하나의 라우팅 테이블에 여러 개의 서브넷을 붙일 수 있음  
  - 하나의 서브넷에서는 여러 개의 라우팅 테이블을 볼 수 없음  
- 대외 서비스 등이 필요할 때 목적에 따른 라우팅 테이블을 만들어서 사용할 수 있음  
  - 생성하지 않을 시 디폴트 라우팅 테이블을 사용한다고 생각하면 됨  
- 인터넷 통신을 위해서는, 라우팅 테이블에 Destination (모든 IP) & Target (인터넷게이트웨이의 ID) 이 설정 되어야 함  

#### 프라이빗 서브넷
- 퍼블릭 서브넷과 프라이빗 서브넷  
  - 보안을 위해, Public에 두는 리소스는 최소화 하기를 권장  
  - Public 라우팅 테이블은 Destination (모든 IP) & Target (인터넷게이트웨이의 ID) 으로
  - Private 라우팅 테이블은 Destination (모든 IP) & Target (local) 
- NAT 게이트웨이 지원  
  - Private 에서 NAT 게이트웨이의 ID를 지정하면, 프라이빗 서브넷에서 NAT 게이트웨이를 통해서 인터넷에 접속 가능함  

#### DNS와 DHCP
- DHCP 란? 자동으로 IP를 만들어주는 서비스
- AWS 에서는 DHCP를 통해 새 리소스에 자동으로 사설 IP를 할당해주게 된다
- 빌트인 DNS 를 사용하여서 프라이빗 DNS 이름을 자동 할당한다  

#### Public IP  
- EC2 인스턴스를 생성할 때 자동으로 Public IP를 생성하는 방법이 있음
  - Stop 했다가 올라오면 Pubilc IP가 변경될 수 있음  
  - 고정 IP 가 필요할 시 EIP 를 사용  

### VPC 엑세스 제어
#### NACL과 보안그룹  
- VPC에서 엑세스를 제어하는 방법...  
- 보안그룹  
  - 인스턴스(ENI) 단위 방화벽  
  - 상태 저장  
  - Allow 만 설정 가능  
- Network Access Control List(NACL)  
  - 서브넷 단위 방화벽
  - 상태 비저장
  - Allow, Deny 설정 가능  
- 상태 저장이란?
  - 상태 저장 : outbound 에 대해 port 설정이 되어 있다면, inbound 는 설정 하지 않음
  - 상태 비저장 : outbount 에 대해 port 설정이 되어 있어도, inbound 설정을 해야 함 (별도 설정)  
    - 상태 비저장일 시, Inbound에서 허락된 트래픽 응답도 outbound 내 명시적 allow 가 있어야 리턴
    - outbound rules : Protocol 주로 80 혹은 443 port  
- 보안그룹 설정
  - Inbound 로 허용된 트래픽의 Outbound 는 자동 허용
  - 다른 보안그룹을 참조하여 Source 로 지정 가능
  - ex : DB 서버 등은 애플리케이션 서버에서 주로 접근을 할 텐데, 해당 서버들의 보안그룹을 port 에 허용하면 해당 보안그룹을 사용하는 서버들이 모두 접근 가능
보안그룹은 많이 쓰는데, 상황에 따라 NACL 활용도 좋음.  

#### VPC 엑세스 제어 : Default NACL  
- 아무 설정 하지 않을 시 Inbound & Outbound 모두 Allow  
- NACL 을 통해서 엑세스 제어 필요할 시에는 default NACL 이 아닌 별도 NACL 생성 권고  

### 운영환경을 위한 VPC 디자인
- 적절한 크기의 대역 확보
- Custom VPC 사용  

### VPC 연결
- 인터넷 게이트웨이를 통한 연결 -> 권고하지 않음
- VPC Peering 을 권고  
  - 라우팅 테이블에 Target 을 VPC 로 설정하면, 해당 VPC 에 연결 가능  
  - 다른 VPC 간에 통신을 하려면, IP가 달라야 하기 때문에 Default VPC 로는 불가능함 (IP 고려를 잘 해야 함)  
  - 전이적 라우팅 불가  
  - 여러 대 VPC 연결할 시 AWS Transit Gateway 권장  

#### Site-to Site VPN
- 온프로미스와 연결 할 시 사용  

#### AWS Direcct Connect  
- 파트너사 전용 선을 사용  

### VPC Endpoint - 게이트웨이 타입
- 아무 설정을 하지 않을 시, VPC 인스턴스에서 리전영역에 있는 S3 에 접속을 할 때 라우팅 테이블을 통해 외부 인터넷에 접속하여, 다시 S3에 접속
- 외부로 나가지 않고 바로 S3 혹은 DynamoDB 등에 접속할 때 엔드포인트를 통해 접속 가능  

### 실습 순서  
본 실습에서는 VPC Wizard를 통하여 Public Subnet 및 Private Subnet을 2개의 가용영역(AZ-a, AZ-c)에 각각 하나씩 생성하고, 하나의 Public Subnet에 NAT 게이트웨이를 구성함.  
이후 라우팅 테이블을 설정하여 트래픽 흐름을 정의하고, 이와 같은 작업을 통해 추후 고가용성과 확장성을 가진 웹 서비스 환경을 만들기 위한 기본 네트워크 구성을 완료하게 됨.  

## 두 번째 세션 : Compute  
### EC2  
- EC2 : 클라우드에 위치한 가상 서버 인스턴스  
- ECS, EKS, FARGATE : Docker 컨테이너를 대규모로 실행하고 관리하는 서비스  
- AWS LAMBDA : 이벤트에 대한 응답으로 코드를 실행하는 서버리스 컴퓨팅  

#### Amazon Elastic Compute Cloud (EC2)  
- 하이퍼바이저는 원래 Zen 을 사용하다가, 지금은 자체 개발한 가상화 머신&소프트웨어 사용  
- Linux, Windows, Mac  
- Arm 및 x86 아키텍쳐  
- 베어 메탈, 디스크, 네트워킹  
- 가상 머신이 가상 블록 디스크에 들어가서, 실행이 된다  

#### EC2 운영 체제
- Windows Server 
- Amazon Linux
- Debian
- CentOS
- 등등...  
 
#### Amazon Machine Image (AMI)  
- 인스턴스 시작에 필요한 정보 제공  
- 사용 방법은 주로 
  1. 기본 AMI 로 인스턴스 생성
  2. 변경 및 구성 후, 사용자 정의 AMI 작성
  3. 사용자 정의 AMI 를 사용하여 필요한 인스턴스들 생성  
- 한 번 생성한 AMI 를 다시 사용하게 됨
- Auto Scaling 시...  

#### 수명 주기
- 실행중(Running) : 실행 중인 상태. 과금 발생 -> 정지됨(Stopped) : 중지된 상태. 과금 안 됨 -> 종료됨(Terminated) : 완전히 제거된 상태. 과금 안 됨
- 중간에 'Hibernated' 라는 최대절전상태가 있음  

### Storage
#### Elastic Block Store (EBS)
- 블록 영구 스토리지
- 인스턴스 수명에 관계없이 지속
- API 를 이용하여 생성, 연결, 수정
- 워크로드에 따라 스토리지 선택
- 볼륨 암호화 지원  

#### EC2 인스턴스 스토어
- 블록 수준 임시 스토리지
- Snapshot 기능 미지원
- 물리적으로 연결된 디스크를 사용 (서버 내 SSD 또는 HDD)  
- 디폴트로 할당 되어 있는 스토리지가 있기 때문에, fs만 올리면 쓸 수 있음
- 추가 할당은 불가능  

### 인스턴스 유형
#### 유형
[참고 링크 : 인스턴스 유형](https://aws.amazon.com/ko/ec2/instance-types/)  
컴퓨팅 최적화 유형 인스턴스는 'Core 와 Memory 의 비율' 로 이해하면 될 것 같음  
ex : T3, M5처럼 범용 인스턴스와 컴퓨팅 최적화 된 인스턴스인 C5 는 Core-Memory 비율이 다름  

리전마다 사용할 수 있는 인스턴스 유형이 다르며, 확인하는 방법은 콘솔에서 생성할 때 확인하거나, 혹은 요금 페이지에서 볼 수 있음  
GPU 서버는 금액 차이가 많이 남  
학계에서 가장 많이 쓰는 리전은 미국의 오레곤, 노스버지니아  

#### 인스턴스의 세대  
[참고 링크 : 인스턴스 추가 기능](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/instance-types.html)  
ex : c7gn.xlarge
- 맨 앞 - 인스턴스 패밀리  
- 두번째 - 인스턴스 세대  
- 그 다음 : 추가 기능 (ex : CPU type, n이 있으면 network 에 최적화 된 타입)  
- '.' 뒤 : 인스턴스 크기  
팁 : 인스턴스 세대는 무조건 높은 숫자를 사용하는 것이 좋음  
팁2 : 최적인 인스턴스를 선택하는 방법은 언제나 "적당한 것이 최고"  

### 컴퓨팅 관련 서비스
아래와 같은 유용한 기능들이 있음  
- Elastic Load Balancing
- Amazon EC2 Auto Scaling
- CloudWatch 
각각을 설정하여 웹 서비스를 효율적으로 구성할 수 있다!  

### 실습 순서  
- 웹 서버 인스턴스 시작(Launching) 및 사용자 지정 데이터(User Data)의 실행  
- 보안 그룹(Security Group)의 설정  
- 커스텀 AMI(Amazon Machine Image) 생성  
- ALB(Application Load Balancer) 생성  
- 시작 템플릿(Launch Template) 구성  
- Auto Scaling Group 구성  
- Auto Scaling 테스트 및 수동 설정 변경  

중간에 부하 테스트가 있는데, 부하테스트는 아래 원리로 진행하게 된다.  

> CPU 부하를 발생시키는 원리는 CPU Idle 값이 30이 넘어가면, 임의의 파일을 만들고 압축하고 압축을 해제하는 작업을 하도록 PHP 코드가 5초 주기로 동작합니다. ALB에 의해서 트래픽이 분산되어 동작되기 때문에 부하를 걸어주고 나면 계속적으로 다른 인스턴스에도 부하가 발생하게 됩니다.  

## 세 번째 세션 : Database  
### Background History & 기본 개념  
- 앱 아키텍쳐는 시간 순으로 Mainframe -> Client-Server -> 3-tier -> Microservice 진화함 
- Microservice 에서 요구하는 형식의 DB 를 서비스 하기 위한 AWS의 기술이 무엇인지 알아볼 것
- 기존 방식 대비 자체 관리 Database 의 장점이 있음 (자체 관리 Database : mySQL 등)
  --> 그러나 이런 방식은 하드웨어 및 소프트웨어 설치, 구성, 패치, 백업 등에 문제가 있을 수 있으며 보안 규정 지키기도 힘듦
- AWS 는 완전 관리형으로, 모니터링이나 격리, 보안 등을 담당하게 됨  

### AWS의 DB
[참고 링크 : 목적에 맞게 설계된 Database](https://aws.amazon.com/ko/products/databases/)  
ex : 기존 EC2에서 DB 를 자체 설치해서 사용한다면, 데이터센터 이슈나 fail-over 등을 직접 고민했어야 함  
하지만 AWS 에서 제공하는 서비스를 사용할 때는, 스키마 설계 및 쿼리 생성, 쿼리 최적화까지만 고민하면 됨.  
AWS Database는 목적에 맞게 설계되어 있음.  
RDS, Aurora 는 아래에서 더 자세히 서술할 예정이며, non-Relational DB (ex : mongoDB 등) 은 Amazon DocumnetDB 등으로 서비스 하게 됨  

#### Amazon RDS  
- 6가지 DB 엔진을 갖춘 관게형 Database
- MySQL, PosgreSQL, MariaDB 등이 RDS 라는 서비스로 존재  
- Multi-AZ 배포 가능 (AZ 사이에서 복제본을 유지하여 안정성 보장)
- 읽기 전용 복제본으로, 비동기식 복제를 사용. read Replica 를 사용하여 기존 DB에서 장애가 발생할 경우 복제로 DB Instance 를 사용  
- 성능 개선 도우미 (Performance Insights) 를 사용하면, 로드를 유발하는 SQL 문과 이유를 파악할 수 있으며 부하 필터링도 확인 가능함  
  (단, DB 엔진 외에 프로세스가 돌아가기 때문에 자원을 일부 사용하게 됨.)  

#### Amazon Aurora  
- 클라우드용으로 구축된 MySQL 및 PostgreSQL 호환 관게형 Database
- 내부적으로 수정을 거쳤기 때문에, 성능 면에서 표준 MySQL 의 5배 처리량을 보인다고 함 
- 구조부터 분산 처리를 가정하고 설계되었기 때문에, 데이터 복제 시 기존 Database (MySQL) 보다 빠름
- 최대 128 TB 까지 저장 공간 자동 증분  

### 다양한 Databbase Feature  
- Database Backtrack : Backup과 Restore 없이 Database의 시점 복구 기능 사용 가능  
- Zero Downtime 패치 : 패치 중에도 Client Session 유지하며, 가동 중단 없이 Aurora Cluster 패치 적용  
- Database 복제 : 데이터 복사 없이 복제본을 생성하여 다른 AWS 계정/조작과 Database 공유  

### 다양한 데이터 서비스
- DMS
- SCT  

해당 DB는 아마존에서 자체 개발한 '그래비톤' 칩 위에서 구동하면 최적의 성능을 얻을 수 있을 것...  

### 실습 순서  
- VPC 보안 그룹 생성
- RDS 인스턴스 생성
- RDS 크레덴셜 저장
- 웹앱 서버와 RDS 연결
- (옵션) RDS 관리 기능
- (옵션) RDS Aurora 연결  

## 네 번째 세션 : Storage  
[참고 : AWS 클라우드 스토리지](https://aws.amazon.com/ko/products/storage/)
### 스토리지 타입
- 블록 스토리지 : 데이터를 일정 크기 블록으로 나누어 저장, 호스트에서 파일 시스템을 생성 -> Storage Area Network (SAN) 사용
- 파일 스토리지 : 데이터를 파일 및 폴더의 계층 구조로 저장, 스토리지 단에서 파일 시스템을 생성 -> Network Attached Storage (NAS) 사용
- 객체 스토리지 : 데이터를 객체라고 하는 비정형 형식으로 저장, REST 기반의 API 호출을 통해 데이터에 접근 -> HTTP 프로토콜 사용  

AWS는 업계에서 가장 광범위한 스토리지 포트폴리오를 제공한다 (고 한다...)  
파일 스토리지로는 AWS Snowcone, Snowball Edge, Snowmobile 등 디바이스에 담아서 AWS에 보내면 오프라인으로 데이터를 전송  

#### AWS 블록 스토리지
- 인스턴스 스토리지 : 호스트 하드웨어에 연결된 임시 블록 레벨 스토리지
- EBS : 사용하기 쉽고 고성능의 블록 스토리지 서비스, EC2와 함께 사용  
- 스냅샷 : 이미지 등 EBS 데이터의 증분, 특정 시점 복제본  

#### Amazon Simple Storage Service (S3)  
[참고 : S3의 다양한 스토리지 클래스](https://aws.amazon.com/ko/s3/storage-classes/)  
- 비용 절감, DataLake 구축, Gacier Deep Archive 를 사용  
- 다양한 객체 스토리지 클래스가 존재  
- AWS에서 굉장히 초창기에 나온 솔루션  

### 공유 파일 시스템 
- Amazon Elastic File System(EFS)
  - 서버리스 기반 공유 파일 시스템을 제공  
  - 수명 주기에 따른 파일 시스템 관리 가능  
- Amazon FSx for Windows File Server   
  - 윈도우 서버에 구축되는 확장 가능한 파일 스토리지 서비스  
  - 단일 가용영역, 멀티 가용영역 둘 다 사용 가능
- Amazon File Cache  

### 데이터 백업, 전송
- 백업에 필요한 다양한 기능 또한 제공 가능함
- Storage Gateway 를 사용하여 객체 스토리지 클래스에 객체를 저장하고, 접근 가능함
  - S3, FSx 등에 파일을 저장하기 위한 게이트웨이가 있음
  - 사용자는 마치 온프레미스에 데이터가 있는 것처럼 사용 가능
  - 볼륨 게이트웨이를 사용하면, 온프레미스 데이터를 클라우드로 백업  

### DataSync
- 온프레미스 서버와 AWS 간의 전송을 간소화, 자동화 및 가속화  

### 실습 순서
- S3에 Bucket 생성
- 버킷에 오브젝트 추가하기
- 오브젝트 보기
- 정적 웹 사이트 호스팅 사용
- 오브젝트 이동
- 버킷 버저닝 활성화
- 오브젝트 및 버킷 삭제  

## 감상
1. AWS 에서 구축한 자체 클라우드 생태계를 맛보기나마 처음부터 끝까지 구축하는 것을 실습으로 진행할 수 있어서 유용했음.
2. 단, 개발 시 제공하는 Console 이 비교적 high-level 로 만들어져 있는데, 컨트롤 해야 하는 객체가 많다 보니 오류가 발생할 시 혼란이 올 수 있음...  
3. 웹 기반 서비스를 클라우드화 하여 네트워크, DB, 스토리지 등 전체 구조를 AWS 통해 구축한다면, 요금 계산을 잘 해야 할 것 같음  

그 외. 버킷을 mount 해서 쓸 수 있도록 하는 서드파티 등을 현재처럼 open-source 가 아니라 자체 솔루션으로 제공할 수 있도록 개발을 진행 중에 있음  
