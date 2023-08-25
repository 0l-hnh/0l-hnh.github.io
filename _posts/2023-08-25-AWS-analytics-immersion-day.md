---
layout: single
title:  "[DevLog] AWS 교육 (General Immersion Day) 참가"
date:   2023-08-25 10:10:00 +0900

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
AWS Analytics Immersion Day 를 듣기 위해 역삼에 왔다.

## Intro  
AWS Analytics IMD - Intro  

### Q&A About Analytics  
- Why Analytics?
  - Q1. 왜 DB만으로는 안 되고 Analytics 를 사용해야 하는지, 예를 들어 PS 고객 중에는 사용자 activity log 를 아직도 rdb에 구축하는 경우가 있음. 근본적으로 data warehouse 를 구축하면 더 할 수 있는 것이 무엇인지.  
  - A1. Database 는 single node system 이고, Analytics 는 muliple nodes 로 병렬 처리가 가능하다. 그리고 database 는 row storage 형식을 취하지만, analytics database 는 column-storage format 이기 때문에, DB 엔진이 처음부터 끝까지 읽어서 처리할 필요가 없다. 그리고, Analytics 라고 하면 복잡한 시스템을 구축해야 할 것 같지만 AWS 에서는 서버-리스 서비스를 지원해서 소규모로 시작할 수 있는 것이 강점이다.  

- How to Start?
  - Q2. 뭐 부터 시작해야 하는지, 어떤 데이터를 어떻게 수집해야 하는지.  
  - A2. 모든 데이터를 다 웨어하우스에 옮기는 것 대신, 가장 비즈니스의 kpi 를 예측 (measure) 하기 위해 사용하고 싶은 데이터를 정리하고 working backwards 하는 것을 추천한다. 그리고 data-driven-everything 이라고 부르는 B2E 프로그램을 사용해봐도 좋다.   

- Global CSP compet
  - Q3. Azure, Google 등과 비교해서 AWS Datalake 의 차이는?
  - A3. 해당 서비스를 가장 오래 운영함 (얼마 전에 10주년을 맞았다) 아마존에서는 딥 러닝을 하기 위해 전혀 코드를 작성하지 않아도 데이터를 옮길 수 있다. Google 은 최근 가격을 인상. Azure 이 현재 제시하는 올-인-원 솔루션은 새로운 컨셉이 아님. AWS는 다양한 도메인에 대한 분석이 가능하도록, 예를 들어 Healthcare 분야라면 Health-Lake 를 제공.  

- AI/ML
  - Q4. Analytics 로 AI/ML 과 시너지를 내는 방법.
  - A4. Data 가 좋지 않다면, 아무리 모델을 optimize 하려고 해도 잘 되지 않을 것이다. Right Data 에 Rigth time 에 접근할 수 있도록 하는 것이 중요. 데이터 퀄리티를 높이기 위해 체크하는 작업과, 통합 작업 등을 편하게 하기 위해서는 데이터에 대한 정확한 이해가 필요하다. AWS Glue 라는 서비스는 데이터의 적재 이전에 그것이 정말 필요한 정보를 담고 있는지 확인하고, 피드백을 줄 수 있도록 한다.  

- Local traditional analytic solution
  - Q5. 전통적인 방법과 Cloud 에 구축하는 방법은 무엇이 다른가?
  - A5. 데이터 복제 등을 고려했을 때, Cloud 에 올리는 것이 Cost-effective 함. 또한 Share 가 편리하다.  

### AWS Analytics 소개
- Data Lake : 간단히 말하면, 'S3에 데이터를 그냥 넣어놓자. 그리고 카탈로그를 잘 하면 되지 않을까?' 하는 개념 -> S3 + Glue 로 Database 처럼 사용할 수 있으며, 서버리스 쿼리 서비스인 Athena 를 함께 사용한다.  
- AWS Glue Data Catalog Crawlers : 자동으로 데이터 카탈로그를 구축, 동기화 상태를 유지  
- Real Time Analysis : Amazon Redshift 와 Amazon Aurora 제로 ETL 통합, S3에서 RedShift 로 자동화된 파일 수집 (현재 개발 진행 중인 서비스)  
  
## Lab#1  
환경구성 확인 및 실습

![전체 Workshop scenario](https://i.postimg.cc/9QSRvq8G/image.png)  

### 요약
- 데이터 인입부터 시각화까지 전 과정을 실습  
- 데모에 사용할 것은 Student Imformation 등 LMS 사용, S3에 저장되어 있는 raw data, curated data, query results 세 개의 버킷이 생성되어 있음.  
- 진행 예정 세션
  - API Call 해서 Database 에 데이터 적재하는 부분은 Skip  
  - Data Lake Storage 에 접근, Glue Crawler 로 생성된 카탈로그 확인  
  - Athena 로 data 확인  
  - QuickSight 로 시각화  
  - Amazon RedShift 를 생성하고 사용  

개인적으로 해보고 싶다면 GUID 를 생성해서 self-paced lab을 진행 가능.  

### AWS Glue, Amazon Athena  
#### Glue  
한 줄 요약 : 전처리 (ETL)
- 기존 ETL 과 비교
  - 전통적인 ETL 은 확장성이 제한되어 있고, 라이선스 비용이 높고, 특정 솔루션에 종속됨
  - DIY Big Data platform 은 관리가 어렵고, 전문성이 필요
  - 반면 Glue 는 서버리스 솔루션이고, 종속성이 없음
- AWS Glue data integration engines
  - 서버리스, 빠름, 비용 절감
  - Auto Scaling : 비용 절감을 위해 Worker 개수 자동 조정
  - Streaming : 작은 worker 유형 (예를 들어, 1 DPU 가 4 CPU 에 Core 16개 기준일 때, 0.25x 개 단위로 판매)
  - Flex : 예비 용량으로 실행
  - Scale, Reliability
    - Autoscaling 이 잘 된다고 함
  - Apache Spark, python
    - 개발자에게 친숙한 언어 사용
    - 데이터 처리 시 분산 환경 Pandas API 사용 
  - Build scalable applicatin with Ray
    - 단일 python node 사용 시와 달리, Ray Task 를 사용하여 큰 데이터에 적용 가능 (서울 지역은 현재 지원 X)
- Connectors
  - 빌트인 커넥터를 쓰거나, Open-source 기반 커넥터를 개발 가능
- Authoring Solutions
  - Visual ETL, Notebooks (Python command line 작성 가능), Code Editor 등 
  - Transformation 시 개발한 소스를 팀 간 공유 가능
- Operationalize
  - 다양한 Workflow 오케스트레이션 tools 이 있음
  - 예를 들어, S3에 Data Event 가 있을 시 Amazon EventBridge 로 Trigger가 되어, Glue Workflow 를 시작할 수 있음
- Data Management
  - 수용성이 좋고, 크롤러를 사용하면 카탈로그를 볼 수 있고, 민감한 데이터를 식별 및 마스킹 가능  

#### Athena  
한 줄 요약 : 쿼리 엔진  
- SQL 또는 Python 을 사용해서 데이터를 쿼리

#### 실습
준비된 raw data 를 S3에 Load 하는 것부터 시작  
1. Key Management Service (KMS)에 Key users 추가
2. Lambda 로 S3 에 raw data 추가 
3. Glue Crawler 실행 (AWS Cloudshell 에서 실행해도 되고, Crawlers 콘솔에서 실행해도 됨)
4. Athena 쿼리 날리고, 테이블 생성 확인
5. Athena Query Editor 에서 바로 쿼리 할 수 있음  

## Lab#2
### AWS Glue DataBrew 
요약 : '노코드'로 시각적 인터페이스를 제공하여, ETL 작업을 정의하고 쉽게 할 수 있도록 돕는 서비스  

#### AWS Glue DataBrew 
- Usage Flow
  1. 데이터 세트 정의
  2. 프로젝트를 만들고 레시피 (데이터 변환 단계) 정의
  3. 레시피 생성
  4. JOB 작성 & 실행  
- AWS EventBridge & Step Function 과 스케쥴 실행 연동 가능
- 특정 타이밍에 EventBridge 에 이벤트를 발행해, event-driven 처리를 구현 가능
- AWS 계정당 동시 JOB, 데이터 세트 등에 기본 할당량 값이 있으나 soft limit 이기 때문에 조절 가능  
- 과금은 사용 세션과 DataBrewJOB 을 합산하여 계산  

#### 실습

![Project 생성 예시 이미지](https://i.postimg.cc/QtLTyDRb/image.png)  

- Profile raw SIS datasets
  1. 데이터 셋을 생성
  2. Profile Job 생성
  3. Profile Job 실행  
- Combine raw SIS datasets
  1. 데이터 셋을 생성
  2. 레시피를 생성 (실습 시에는 Join, Rename 두 가지를 사용)
  3. 레시피를 Publish
  4. Recipe Job 을 생성하고, 실행  

이렇게 데이터를 추출하고 전처리하는 동작을 설계할 수 있음.

- Curating the student data lake using SQL
  1. LMS 데이터 셋을 생성
  2. Athena 쿼리 창에서 쿼리 날리기
  3. S3 에 데이터가 저장된 것을 확인 

- 쿼리문 예시

```java
CREATE TABLE 
    "db_cur_lmsdemo"."agg_pageview_count_by_user_course"
WITH (
  external_location = 's3://dataedu-curated-_guid_/lmsdemo/agg_pageview_count_by_user_course/',
  format = 'PARQUET',
  parquet_compression = 'SNAPPY'
)
AS
SELECT 
    user_id,
    course_id,
    timestamp_year,
    timestamp_month,
    timestamp_day,
    timestamp_hour,
    count(id) AS pageview_count
FROM 
    "db_raw_lmsdemo"."requests"
GROUP BY 
    user_id,
    course_id,
    timestamp_year,
    timestamp_month,
    timestamp_day,
    timestamp_hour

```  
