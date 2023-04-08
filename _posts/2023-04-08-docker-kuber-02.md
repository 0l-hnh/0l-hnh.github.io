---
layout: single
title:  "[Docker] Docker 강의 정리 (2) - Docker 이미지"
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

### 05. Docker image 생성 : Container
#### 컨테이너로 docker 이미지 커밋  
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
이미지를 실행 후 접속해서, 수정된 컨테이너 설정이 잘 저장되었는지 확인한다.  
```bash
$ docker run --name test_image -d test_image:v1.0
$ docker exec -it test_image /bin/bash

#컨테이너 내부 설정 파일 확인
$ cat /usr/local/apache2/htdocs/index.html 
<html><body><h1>Hello This is my Apache Server</h1></body></html>
```
이전 수정 내역이 남아있는 것을 확인했다.  
만든 이미지를 본인의 Dockerhub에 업로드 할 수도 있다.  

#### 컨테이너를 tar 파일로 출력  
실행 중인 docker 컨테이너를 tar 파일로 저장하는 방법도 있다.  
```bash
#일단 컨테이너를 실행한다
$ docker run --name new_apache2 -d httpd:2.4
$ docker container export new_apache2 -o httpd.tar
$ ll
total 143164
-rw-rw-r--. 1 vagrant vagrant 146595328 Apr  1 21:23 httpd.tar

#tar -tf로 tar 파일 확인한다. 
$ tar -tf httpd.tar
```
.tar 형식 파일이 저장된 것을 확인 가능하다.  
이 tar 파일은 컨테이너를 백업한 것과 유사하다. 해당 파일을 이미지로 만들어서 실행한다.  
```bash
$ cat httpd.tar | docker image import - test_image:v1.1
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED              SIZE
test_image    v1.1      55884d1eafbc   About a minute ago   142MB
```
tar파일을 표준 출력으로 넘겨서, docker image import를 사용하는 방식이다.  
만들어진 이미지가 컨테이너로 실행 되는지 확인한다.  
```bash
$ docker run -d --name test_image2 test_image:v1.1
docker: Error response from daemon: No command specified.
```
오... 안 된다.  

오류 원인은 컨테이너 실행 시 command를 지정하지 않았기 때문이다.  
컨테이너 실행 시 httpd 를 띄우는 커맨드를 뒤에 붙여서 확인해보자.  
```bash
$ docker run --name test_image2 -d test_image:v1.1 /usr/local/apache2/bin/httpd -DFOREGROUND
#실행이 되었으면, apache 컨테이너의 ip 정보를 확인해서 접속 가능한지 확인한다
$ docker continer inspect test_image2 | grep 172
            "Gateway": "172.17.0.1",
            "IPAddress": "172.17.0.5",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.5",
$ curl http://172.17.0.5
<html><body><h1>It works!</h1></body></html>
```
정상적으로 동작하는 것을 확인하였다.  

#### dockear image save  
docker image 를 save 하는 방법이다.  
```bash
$ docker image save -o mysql.tar mysql:5.7
```
저장된 docker 의 tar 파일로 부터 docker 이미지를 읽을 때는 아래와 같이 실행한다.
```bash
$ docker image load -i mysql.tar
```

### 06. Docker image 생성 : Dockerfile
#### 주 사용 명령어  
추후 정리 예정

#### 실습  
Docker 이미지를 빌드하는 방법으로, Dockerfile을 작성한 뒤 docker build 로 빌드할 수 있다.  
예를 들어, CentOs8을 베이스 이미지로 사용하는 어떤 이미지를 아래와 같이 만들 수 있다.  
```Ini
FROM centos:8
RUN cal
RUN touch /tmp/test.txt
RUN touch /var/test2.txt
CMD sleep 500s
```
작성 후, Dockerfile이 존재하는 위치에서아래 명령어로 빌드한다.  
```bash
#'t'는 tag이다
$ docker build -t sample .  
[+] Building 4.4s (8/8) FINISHED                                                        
 => [internal] load .dockerignore                                                  0.2s
 => => transferring context: 2B                                                    0.0s
 => [internal] load build definition from Dockerfile                               0.2s
 => => transferring dockerfile: 202B                                               0.0s
 => [internal] load metadata for docker.io/library/centos:8                        0.0s
 => [1/4] FROM docker.io/library/centos:8                                          0.0s
 => [2/4] RUN cal                                                                  2.2s
 => [3/4] RUN touch /tmp/test.txt                                                  0.8s
 => [4/4] RUN touch /var/test2.txt                                                 0.8s
 => exporting to image                                                             0.2s
 => => exporting layers                                                            0.2s
 => => writing image sha256:4d422c5e3d5a38356aa19fc11333126d91be7ac4a75ba3f0d39c4  0.0s
 => => naming to docker.io/library/sample
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED              SIZE
sample        latest    4d422c5e3d5a   About a minute ago   231MB
```
성공적으로 빌드된 것을 확인될 수 있다.  
ENTRYPOINT, CMD는 빌드될 때 가장 마지막에 있는 하나만 Command로 실행이 된다.  
단 ENTRYPOINT와 CMD는 각각 한 번씩 사용 가능하며, 이 경우 CMD가 ENTRYPOINT의 인수로 들어간다.  
```Ini
# CentOS 8 Images
FROM centos:8
RUN cal
RUN touch /tmp/test.txt
RUN touch /var/test2.txt

ENTRYPOINT cal
CMD 05 2023
```
과연 위와 같이 실행하면 "cal"에 대해 "05 2023" 변수가 들어갈까?
```bash
$ docker build -t sample .
[+] Building 0.1s (8/8) FINISHED                                           
 => [internal] load build definition from Dockerfile                  0.0s
 => => transferring dockerfile: 213B                                  0.0s
 => [internal] load .dockerignore                                     0.1s
 => => transferring context: 2B                                       0.0s
 => [internal] load metadata for docker.io/library/centos:8           0.0s
 => [1/4] FROM docker.io/library/centos:8                             0.0s
 => CACHED [2/4] RUN cal                                              0.0s
 => CACHED [3/4] RUN touch /tmp/test.txt                              0.0s
 => CACHED [4/4] RUN touch /var/test2.txt                             0.0s
 => exporting to image                                                0.0s
 => => exporting layers                                               0.0s
 => => writing image sha256:0f449c9acced44a57604b9e4559dfcd069c89a6d  0.0s
 => => naming to docker.io/library/sample                             0.0s
 $ docker run sample
     April 2023     
Su Mo Tu We Th Fr Sa
                   1
 2  3  4  5  6  7  8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30                  
```
되지 않았다... 이유를 알아보자  
```bash
$ /bin/sh -c cal 05 2023
     April 2023     
Su Mo Tu We Th Fr Sa
                   1
 2  3  4  5  6  7  8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30
```  
shell 형식으로 ENTRYPOINT와 CMD에 실행할 명령어를 주었더니, 마치 /bin/sh로 'cal'만 실행하는 것과 같은 결과를 얻은 것이다.  
```Ini
# CentOS 8 Images
FROM centos:8
RUN cal
RUN touch /tmp/test.txt
RUN touch /var/test2.txt

ENTRYPOINT ["cal"]
CMD ["05","2023"]
```  
```bash
$ docker build -t sample .
[+] Building 0.1s (8/8) FINISHED                                            
 => [internal] load build definition from Dockerfile                   0.0s
 => => transferring dockerfile: 223B                                   0.0s
 => [internal] load .dockerignore                                      0.1s
 => => transferring context: 2B                                        0.0s
 => [internal] load metadata for docker.io/library/centos:8            0.0s
 => [1/4] FROM docker.io/library/centos:8                              0.0s
 => CACHED [2/4] RUN cal                                               0.0s
 => CACHED [3/4] RUN touch /tmp/test.txt                               0.0s
 => CACHED [4/4] RUN touch /var/test2.txt                              0.0s
 => exporting to image                                                 0.0s
 => => exporting layers                                                0.0s
 => => writing image sha256:f9471e0d469de730ac1ea42193bf1c1e91191e367  0.0s
 => => naming to docker.io/library/sample          
$ docker run sample
      May 2023      
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30 31         
```
이제 원하던 결과를 얻었다.  
CMD에 들어간 인자는 docker 컨테이너 실행 시 변경 가능하다.  
```bash
$ docker run sample 07 2023
      July 2023     
Su Mo Tu We Th Fr Sa
                   1
 2  3  4  5  6  7  8
 9 10 11 12 13 14 15
16 17 18 19 20 21 22
23 24 25 26 27 28 29
30 31               
```
Dockerfile 내에 ENTRYPOINT \["cal","05","2023"\]으로 작성해도 동일한 결과를 얻었을 테지만, 그런 경우에는 docker run 시에 다른 변수를 커맨드로 줄 수 없다.  

ENTRYPOINT 내에 반드시 shell 형식으로 적어줘야 하는 명령어도 존재한다.  
예를 들어 아래와 같은 경우이다.  

```Ini
# CentOS 8 Images
FROM centos:8
RUN cal
RUN touch /tmp/test.txt
RUN touch /var/test2.txt

#ENTRYPOINT ["cal"]
#CMD ["05","2023"]
ENTRYPOINT ps -e | head -n 2
```
```bash
$ docker build -t sample .
[+] Building 0.1s (8/8) FINISHED                                            
 => [internal] load build definition from Dockerfile                   0.1s
 => => transferring dockerfile: 256B                                   0.0s
 => [internal] load .dockerignore                                      0.0s
 => => transferring context: 2B                                        0.0s
 => [internal] load metadata for docker.io/library/centos:8            0.0s
 => [1/4] FROM docker.io/library/centos:8                              0.0s
 => CACHED [2/4] RUN cal                                               0.0s
 => CACHED [3/4] RUN touch /tmp/test.txt                               0.0s
 => CACHED [4/4] RUN touch /var/test2.txt                              0.0s
 => exporting to image                                                 0.0s
 => => exporting layers                                                0.0s
 => => writing image sha256:a6c4a269a1b30c9b1141bb4fc77ab419d2ee12dfd  0.0s
 => => naming to docker.io/library/sample   
$ docker run sample
  PID TTY          TIME CMD
    1 ?        00:00:00 sh
```
