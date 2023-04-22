---
layout: single
title:  "[Docker] Docker 강의 정리 (4) - 쿠버네티스"
date:   2023-04-15 10:10:00 +0900

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

Docker(도커), 쿠버네티스 관련 재직자 지원 수업의 네 번째 강의 내용을 정리한다.  

## 2023-04-22 강의 노트  
### 11. minikube  
minikube 는 간단한 테스트 용도로 사용 가능하나, 실제 운영 환경에서 쓰기에는 한계가 있다.  
아래 페이지에서 설치 없이 웹 기반으로 공부도 가능하다.  
[링크]()  

### 12. 쿠버네티스 (k8s)
지난 시간 종료 때 Vagrantfile 로 VM을 build 하고, ssh 설정 및 클러스터 설치를 진행하였다.  
