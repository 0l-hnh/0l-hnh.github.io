---
layout: single
title:  "[Docker] Linux에 nvidia-docker 설치하기 (CentOS6, CentOS7, Ubuntu18.04)"
date:   2023-01-08 12:10:00 +0900

categories:
  - docker
tags: [docker, container, linux]

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
딥 러닝 학습 등에 필요한 오픈소스 툴킷 사용 시 서버에 환경을 구축하는 것보다, Docker를 사용하여 어플리케이션 형식으로 동작시키는 것이 편리할 때가 있다.  
Docker란, OS-level로 어플리케이션을 격리 실행하게 하는 '컨테이너 가상화 기술'을 제공하는 무료 소프트웨어이다.
MLOps 사용을 이해할 때나 모델 서빙을 자동화하는 등 여러 모로 유용하게 쓰일 것 같아 Docker로 대표되는 컨테이너 가상화 기술에 대하여 지속적으로 포스팅하려 한다.  

![도커 이미지](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/Docker_%28container_engine%29_logo.svg/1920px-Docker_%28container_engine%29_logo.svg.png)  

본 문서에서는 Docker 사용에 앞서, Linux에 docker와, 컨테이너 가상 환경에서 GPU를 사용하기 위해 nvidia-docker를 설치하는 방법에 대하여 서술한다.  
GPU 사양에 맞는 nvidia-driver는 이미 설치되어 있다고 가정하여, 과정 서술을 생략하였다.

## CentOs7 nvidia-docker 설치
### Docker
docker를 먼저 설치한다.
```bash
$ curl -s https://get.docker.com | sudo sh
```
만약 curl이 설치되어 있지 않다면 'sudo yum install curl' 명령어를 터미널에 입력하여, curl 설치 후 진행한다.
정상 설치가 되었는지 확인하기 위해 도커 버전을 터미널에 출력한다.
```bash
$ docker -v
Docker version 20.10.12, build ####
```
위와 같이 도커 버전이 터미널에 출력되면 정상적으로 설치된 것이다.
### nvidia-docker
1. yum repo 추가
```bash
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | \
  sudo tee /etc/yum.repos.d/nvidia-docker.repo
```
2. daemon.json backup 및 설치
```bash
$ cp /etc/docker/daemon.json /etc/docker/daemon.json.bak
$ sudo yum -y install nvidia-docker2
```
3. daemon.json 확인
```bash
$ cat /etc/docker/daemon.json
```
4. docker restart
```bash
$ systemctl restart docker
```

## CentOS 6 Docker 설치
1. 사용하는 서버의 OS가 CentOS 6의 경우, 해당 버전의 공식 지원 종료 인해 CentOS 7에서 사용한 yum 설치 및 curl을 사용한 설치가 되지 않을 수 있다. 
```bash 
YumRepo Error: All mirror URLs are not using ftp, http[s] or file.
Eg. Invalid release/repo/arch combination/
removing mirrorlist with no valid mirrors: /var/cache/yum/x86_64/6/base/mirrorlist.txt
Error: Cannot find a valid baseurl for repo: base
```
아마 위와 같은 에러 로그를 마주칠 가능성이 높다. 이 경우 아래와 같이 미러리스트 텍스트 파일을 생성하고 진행한다.
```bash
$ echo "https://vault.centos.org/6.10/os/x86_64/" > /var/cache/yum/x86_64/6/base/mirrorlist.txt
$ echo "http://vault.centos.org/6.10/extras/x86_64/" > /var/cache/yum/x86_64/6/extras/mirrorlist.txt
$ echo "http://vault.centos.org/6.10/updates/x86_64/" > /var/cache/yum/x86_64/6/updates/mirrorlist.txt
$ echo "http://vault.centos.org/6.10/sclo/x86_64/rh" > /var/cache/yum/x86_64/6/centos-sclo-rh/mirrorlist.txt
$ echo "http://vault.centos.org/6.10/sclo/x86_64/sclo" > /var/cache/yum/x86_64/6/centos-sclo-sclo/mirrorlist.txt
```
2. 그 후 rpm 을 사용하여 설치한다.
```
$ curl -O -sSL https://get.docker.com/rpm/1.7.1/centos-6/RPMS/x86_64/docker-engine-1.7.1-1.el6.x86_64.rpm
$ rpm -Uvh docker-engine-1.7.1-1.el6.x86_64.rpm --nodeps
```
3. 이후 docker -v 로 버전 출력을 확인하면 설치 완료.  
nvidia-docker 는 CentOS 6에서 지원되지 않기 때문에 설치할 수 없다.  
[참고 페이지](https://github.com/NVIDIA/nvidia-docker/issues/743)

## Ubuntu 18.04 Docker 설치
우분투 유저일 시 상기 서술한 CentOS docker 설치 과정의 ‘yum’을 ‘apt-get’ 명령어로 대체하여 설치한다. 전체 명령어는 아래와 같다.  
### Docker
```bash
$ curl -s https://get.docker.com | sudo sh
$ docker -v
Docker version 20.10.12, build ####
``` 

### nvidia-docker
1. 저장소, 키 추가
```bash
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
  && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
  && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```
2. nvidia-docker2 설치
```bash
$ sudo apt-get install -y nvidia-docker2
```
3. docker restart
```bash
$ systemctl restart docker
```

## Docker 에 권한 부여
도커 설치 확인 후 실행 중 권한 문제가 발생한다면, 다음 명령어를 입력하여 현재 사용자에게 도커 사용 권한을 부여한다.
```bash
$ sudo usermod -aG docker $USER
$ sudo su - $USER
```
만약 usermod group 'docker' dose not exist 메시지가 나온다면 아래 명령어를 통해 group을 만들고 다시 명령어를 실행하면 된다.

```bash
$ sudo groupadd docker
```

## 참고 사이트
[Docker 공식 사이트](https://docs.docker.com/get-docker/)  
[CentOS7 nvidia-docker설치](https://hyunsoft.tistory.com/entry/centos-nvidia-docker-%EC%84%A4%EC%B9%98-1)  
[Ubuntu18.04 nvidia-docker설치](https://dongle94.github.io/docker/docker-nvidia-docker-install)