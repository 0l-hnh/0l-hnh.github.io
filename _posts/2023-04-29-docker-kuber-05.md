---
layout: single
title:  "[Docker] Docker 강의 정리 (5)"
date:   2023-04-29 10:10:00 +0900

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

Docker(도커), 쿠버네티스 관련 재직자 지원 수업의 마지막 강의 내용을 정리한다.  
오늘 : Deployment, storage 및 실제 어플리케이션을 개발 및 배포하는 방법  

## 2023-04-29 강의 노트  
### 16. 쿠버네티스 명령어와 주요 객체 (2)
강의 시작 전에, VM을 실행하고 모두 잘 연결되어 있는지 확인한다.  
```bash
$ kubectl get nodes
NAME             STATUS   ROLES           AGE   VERSION
m.example.com    Ready    control-plane   9d    v1.27.1
w1.example.com   Ready    <none>          9d    v1.27.1
w2.example.com   Ready    <none>          9d    v1.27.1
```  
잘 되어 있다.  

#### Deployment  
Deployment 형태로 객체를 생성하기 전에 yaml 파일을 수정한다.  