---
layout: single
title:  "[Docker] Dockerfile로 이미지 생성"
date:   2023-02-11 11:10:00 +0900

categories:
  - docker
tags: [docker, linux]

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

## 설명
Dockerfile은 도커 이미지를 정의하는 설정 파일로, 베이스 이미지 및 컨테이너 설정에 대한 정의를 포함한다.  
본 포스트에서는 Dockerfile을 작성하는 방법과, 작성할 때 사용하는 명령어를 정리한다.  

## 상세  
### Dockerfile 작성 방법  
간단하게 요약하면 아래와 같은 과정을 거친다.  

1. 베이스 이미지를 명시한다.     
2. 컨테이너 실행 시 필요한 몇 가지 옵션을 지정한다.   
3. 작성이 완료되면 저장 후, 빌드한다.  

위 과정을 따라 ubuntu18.04 베이스 이미지를 사용하는 간단한 Dockerfile 예시는 아래와 같다.  
```bash   
# syntax=docker/dockerfile:1
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```  
실무에서 Dockerfile을 작성하며 느낀 건데, Docker나 컨테이너 자체에 대한 지식도 물론 중요하지만 Backend, Network 및 OS에 대한 인사이트가 반드시 선행 되어야 좋은 Dockerfile을 작성할 수 있는 것 같다..  

### Dockerfile 작성 방법  
Dockerfile 빌드 시 하나의 layer를 생성하는 동작은 아래 4 가지이다.    

* FROM : 지정한 베이스 이미지로부터 layer를 생성한다.  
* COPY : Dockerfile이 위치한 local 디렉토리에서 특정 file을 이미지 내부로 copy 한다. (이미지의 리소스 사이즈가 증가함)  
* RUN : 이미지 생성 시 필요한 대부분의 동작을 RUN 이후 쉘 명령어로 작성한다. 해당 명령어는 베이스 이미지의 OS 문법을 따른다.  
* CMD : 생성된 이미지를 컨테이너로 실행할 시 실행할 명령어를 지정한다. Dockerfile 내 1회만 사용 가능.  

## 참고 사이트  
[Docker 공식 문서](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/  )  

