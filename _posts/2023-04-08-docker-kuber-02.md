---
layout: single
title:  "[Docker] Docker 강의 정리 (2)"
date:   2023-04-08 10:10:00 +0900

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

Docker(도커), 쿠버네티스 관련 재직자 지원 수업의 두 번째 강의 내용을 정리한다.  

## 2023-04-08 강의 노트  
### 04. Docker 실습 이어서 
시작 전에 Network 문제가 없는지 확인한다.  
* NetworkManager로 확인하는 방법 
```bash
$ sudo yum install NetworkManager-tui
$ sudo mntui
```
* ping을 보내는 방법
```bash
$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=54.4 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=38.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=34.5 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=114 time=35.8 ms
```
잘 되는 것을 확인했다.  

Docker 의 bridge network type을 Host 로 하는 것을 해보자  
```bash
$ docker network create -d host myhost
Error response from daemon: only one instance of "host" network is allowed
```
"host"는 하나밖에 쓰지 못 하며, 삭제도 하지 못 한다.  
만들어진 host 를 사용한다.  

```bash 
$ docker run -it --network host --name centos8 centos:8 /bin/bash
```

### 05. Docker image
#### 컨테이너에서 docker 이미지 만들기  
apache 컨테이너를 사용해서 실습해본다.
```bash
$ docker run --name new_apache -d httpd:2.4
$ docker exec -it new_apache /bin/bash
```
컨테이너 내부에서 파일을 편집하려고 하는데, 만약 편집기가 없다면 설치한 뒤 진행한다.  
apache 서버는 debian 계열이므로, apt update 후 install vim 실행한다.  
```bash
$ vi /usr/local/apache2/htdocs/index.html 
$ cat /usr/local/apache2/htdocs/index.html
```
위에서 수정한 내용은 컨테이너 삭제 시 사라진다.  

```bash
docker stop $(docker ps -q); docker rm $(docker ps -aq)
94f2fea86c41
94f2fea86c41
b8fb32487b6b
```
다시 apache 이미지를 올려서 확인을 하면, 이전 컨테이너에서 수정한 내용이 없다는 것을 알 수 있다.  
컨테이너 내에서 수정한 내역을 계속 사용하고 싶다면, 컨테이너를 커밋해서 이미지로 생성한다.  

```bash
$ docker run --name new_apache -d httpd:2.4
$ docker exec -it new_apache /bin/bash

#컨테이너 내부에서 필요 패키지를 다시 설치한다
$ apt update
$ apt install vim
$ apt install net-tools
$ vi /usr/local/apache2/htdocs/index.html 
$ exit

#컨테이너 커밋
# docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
$ docker container commit new_apache test_image:v1.0
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
test_image    v1.0      1cef1b917867   6 seconds ago   200MB
mysql         5.7       3f3447deacaa   12 hours ago    455MB
httpd         2.4       192d41583429   9 days ago      145MB
httpd         latest    192d41583429   9 days ago      145MB
hello-world   latest    feb5d9fea6a5   18 months ago   13.3kB
centos        8         5d0da3dc9764   18 months ago   231MB
```
원하는 이미지가 생성된 것을 알 수 있다.  
