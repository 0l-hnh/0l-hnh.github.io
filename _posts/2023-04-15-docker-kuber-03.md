---
layout: single
title:  "[Docker] Docker 강의 정리 (3) - Docker 이미지, 쿠버네티스"
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

Docker(도커), 쿠버네티스 관련 재직자 지원 수업의 세 번째 강의 내용을 정리한다.  

## 2023-04-15 강의 노트  
### 07. Docker scratch image
* 이미지를 최대한 작게 만들기 위해 베이스 이미지로 'scratch'를 사용  
* [참고를 위한 Docker 공식 사이트](https://hub.docker.com/_/scratch)  

scratch 이미지를 사용하는 가장 간단한 Dockerfile 예제는 아래와 같다.  
```Ini
FROM scratch
COPY hello /
#대괄호 안에 들어간 "/hello"는 exec 형식
CMD ["/hello"]
```
이 때 hello는 '.c' 파일로 컴파일한 바이너리 파일이며, 아래와 같이 환영 문구를 printf 하는 가장 간단한 소스로 테스트 해 본다.  
```c
#include <stdio.h>
int main()
{
        printf("Hello My Container! \n");
        return 0;
}
```  
gcc 로 hello.c 를 컴파일 한 뒤 해당 바이너리 파일을 내부로 copy 하여 docker build 한다.  
```bash
$ gcc -o hello hello.c
$ file hello
hello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=0bcce3f093e3e43933efb6645a0e3ed5796b10a1, not stripped
$ docker build -t myhello .
[+] Building 0.3s (5/5) FINISHED                                                     
 => [internal] load build definition from Dockerfile                            0.1s
 => => transferring dockerfile: 137B                                            0.0s
 => [internal] load .dockerignore                                               0.1s
 => => transferring context: 2B                                                 0.0s
 => [internal] load build context                                               0.0s
 => => transferring context: 8.45kB                                             0.0s
 => [1/1] COPY hello /                                                          0.1s
 => exporting to image                                                          0.1s
 => => exporting layers                                                         0.1s
 => => writing image sha256:28c4dbd8f38e207e2590e236f9d641595afd83ade7520396b9  0.0s
```
이제 hello가 잘 찍히는지 확인한다.  
```bash
$ docker run myhello
exec /hello: no such file or directory
```
오... 안 된다. 왜 안 되는지 확인을 해보자  
'hello' 라는 파일이 동작하기 위해 참조하는 라이브러리 파일을 알아본다.  
```bash
$ ldd hello
        linux-vdso.so.1 =>  (0x00007ffc0af3a000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f482c8de000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f482ccac000)
```
컨테이너 내에서 로컬 커널에 있는 라이브러리를 참조하지 못 해서 발생하는 문제로 추정된다.  
파일을 정적으로 컴파일 하면 해당 이슈가 없지만, 동적으로 컴파일 했기 때문에 이런 문제가 발생한다.  
해당 이슈가 맞는지 확인하기 위하여, 위의 '.so' 파일들을 docker build 시 사용할 수 있도록 build 위치에 복사한 뒤 다시 진행한다.  
```bash
$ cp /lib64/ld-linux-x86-64.so.2 lib64/.
$ cp /lib64/libc.so.6 lib64/.
```
Dockerfile은 아래와 같이 수정한다.
```Ini
FROM scratch
COPY hello /
COPY lib64 /lib64
CMD ["/hello"]
```
이제 실행한다.  
```bash
$ docker build -t myhello .
[+] Building 0.4s (6/6) FINISHED                                                     
 => [internal] load build definition from Dockerfile                            0.1s
 => => transferring dockerfile: 150B                                            0.0s
 => [internal] load .dockerignore                                               0.0s
 => => transferring context: 2B                                                 0.0s
 => [internal] load build context                                               0.1s
 => => transferring context: 2.32MB                                             0.1s
 => CACHED [1/2] COPY hello /                                                   0.0s
 => [2/2] COPY lib64 /                                                          0.1s
 => exporting to image                                                          0.1s
 => => exporting layers                                                         0.1s
 => => writing image sha256:1431f69860154ff5c4809e172914ee7a1ecb354f9484ef7ef0  0.0s
 => => naming to docker.io/library/myhello                                      0.0s
[vagrant@serverx 230415]$ vi Dockerfile 
[vagrant@serverx 230415]$ docker build -t myhello .
[+] Building 0.4s (6/6) FINISHED                                                     
 => [internal] load build definition from Dockerfile                            0.1s
 => => transferring dockerfile: 155B                                            0.0s
 => [internal] load .dockerignore                                               0.1s
 => => transferring context: 2B                                                 0.0s
 => [internal] load build context                                               0.0s
 => => transferring context: 372B                                               0.0s
 => CACHED [1/2] COPY hello /                                                   0.0s
 => [2/2] COPY lib64 /lib64                                                     0.1s
 => exporting to image                                                          0.1s
 => => exporting layers                                                         0.1s
 => => writing image sha256:24c9e62ca272cf7fc099e7b9ebe973b68a756d5b5c933dd689  0.0s
 => => naming to docker.io/library/myhello
$ docker run myhello
Hello My Container! 
```
정상적으로 실행 되는 것을 확인했다.  
scratch 이미지는 정말 아무 것도 없는 이미지로, 필요한 바이너리 파일만 담아서 빠르게 실행할 수 있게 한다.  
```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
myhello       latest    24c9e62ca272   6 minutes ago    2.33MB
```
이미지 사이즈를 확인하면, 거의 바이너리 파일 크기만 갖고 있는 아주 작은 이미지인 점을 확인할 수 있다.  
정적 (static) 바이너리 파일을 빌드하면 컨테이너 내에 필요한 라이브러리를 COPY 하지 않아도 된다.  
```bash
# -static 옵션으로 빌드가 안 될 시, yum install glibc-static 하고 실행한다. 
$ gcc -static -o hello hello.c 
$ ldd hello
        not a dynamic executable
$ docker build -t myhello:static .
[+] Building 0.6s (6/6) FINISHED                            
 => [internal] load build definition from Dockerfile   0.0s
 => => transferring dockerfile: 155B                   0.0s
 => [internal] load .dockerignore                      0.1s
 => => transferring context: 2B                        0.0s
 => [internal] load build context                      0.1s
 => => transferring context: 861.82kB                  0.0s
 => [1/2] COPY hello /                                 0.1s
 => [2/2] COPY lib64 /lib64                            0.2s
 => exporting to image                                 0.2s
 => => exporting layers                                0.1s
 => => writing image sha256:1dc65c3cefbd29db13207e733  0.0s
 => => naming to docker.io/library/myhello:static      0.0s
$  docker run myhello:static
Hello My Container! 
```
잘 실행이 되었다.  
```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
myhello       static    1dc65c3cefbd   55 seconds ago   3.18MB
myhello       latest    24c9e62ca272   14 minutes ago   2.33MB
```
이미지 사이즈를 확인하면 위와 같다.  
docker image history 명령어로 빌드 과정에서 늘어난 이미지 사이즈를 확인할 수 있다.  
```bash
$ docker image history myhello:static
IMAGE          CREATED         CREATED BY                     SIZE      COMMENT
1dc65c3cefbd   2 minutes ago   CMD ["/hello"]                 0B        buildkit.dockerfile.v0
<missing>      2 minutes ago   COPY lib64 /lib64 # buildkit   2.32MB    buildkit.dockerfile.v0
<missing>      2 minutes ago   COPY hello / # buildkit        861kB     buildkit.dockerfile.v0
```  

### 08. Docker Multi-stage build
* Docker를 빌드할 때 바이너리 파일을 빌드한 다음, 해당 바이너리 파일을 다른 Docker 베이스 이미지를 사용해서 동작시키는 방법
* 실행 이미지의 크기가 경량화 됨  
  
일단 아래와 같은 Dockerfile을 작성해본다.  
```Ini
FROM ubuntu:18.04 
RUN apt-get update
RUN apt-get install -y gcc
COPY hello.c /tmp
WORKDIR /tmp
RUN gcc -o hello-world hello.c
CMD ["/tmp/hello-world"]
```
해당 Dockerfile은 'single stage' 이다.  

```bash
$ docker build -t single-stage .
[+] Building 82.6s (11/11) FINISHED                                                  
 => [internal] load build definition from Dockerfile                            0.1s
 => => transferring dockerfile: 251B                                            0.0s
 => [internal] load .dockerignore                                               0.1s
 => => transferring context: 106B                                               0.0s
 => [internal] load metadata for docker.io/library/ubuntu:18.04                 0.4s
 => [1/6] FROM docker.io/library/ubuntu:18.04@sha256:8aa9c2798215f99544d1ce743  4.2s
 => => resolve docker.io/library/ubuntu:18.04@sha256:8aa9c2798215f99544d1ce743  0.1s
 => => sha256:0779371f96205678dbcaa3ef499be2e5f262c8b09aadc11754bf 424B / 424B  0.0s
 => => sha256:3941d3b032a8168d53508410a67baad120a563df67a79595 2.30kB / 2.30kB  0.0s
 => => sha256:0c5227665c11379f79e9da3d3e4f1724f9316b87d259ac 25.69MB / 25.69MB  1.4s
 => => sha256:8aa9c2798215f99544d1ce7439ea9c3a6dfd82de607da1ce 1.33kB / 1.33kB  0.0s
 => => extracting sha256:0c5227665c11379f79e9da3d3e4f1724f9316b87d259ac0131628  2.4s
 => [internal] load build context                                               0.1s
 => => transferring context: 174B                                               0.0s
 => [2/6] RUN apt-get update                                                   11.6s
 => [3/6] RUN apt-get install -y gcc                                           62.4s
 => [4/6] COPY hello.c /tmp                                                     0.4s
 => [5/6] WORKDIR /tmp                                                          0.2s
 => [6/6] RUN gcc -o hello-world hello.c                                        1.4s
 => exporting to image                                                          1.8s
 => => exporting layers                                                         1.8s
 => => writing image sha256:11e1548881285049262cd7b5b1c2badf170b9afd1ffbd2a54f  0.0s
 => => naming to docker.io/library/single-stage     
$ docker run single-stage
Hello My Container! 
```
내부에서 hello.c 가 잘 컴파일 되어서, 정상적으로 해당 파일이 실행되었음을 볼 수 있다.  
```bash
$ docker images
REPOSITORY     TAG       IMAGE ID       CREATED              SIZE
single-stage   latest    11e154888128   About a minute ago   220MB
```
생성된 이미지의 총 크기는 220 MB 이다.  
컴파일러 크기까지 포함되어서 사이즈가 커진 것이다.  

이제 베이스 이미지를 두 개 사용해서 빌드하는, multi-stage 빌드를 해 본다.  
```Ini
FROM ubuntu:18.04 AS build-image
RUN apt-get update
RUN apt-get install -y gcc
COPY hello.c /tmp
WORKDIR /tmp
RUN gcc -o hello-world hello.c

FROM ubuntu:18.04 
COPY --from=build-image /tmp/hello-world .
CMD ["./hello-world"]
```
빌드 이미지를 쓰면, 빌드 이미지에서 만든 것을 두 번째 이미지로 복사한다.  
따라서 컴파일러가 설치되지 않은 ubuntu 이미지에서 바이너리 파일을 실행하는 것과 같은 효과를 보여서, 이미지 사이즈가 작아진다.  
빌드하여 확인 해 본다.  
```bash
$  docker build -t multi-stage .
[+] Building 1.3s (12/12) FINISHED                                                   
 => [internal] load build definition from Dockerfile                            0.0s
 => => transferring dockerfile: 324B                                            0.0s
 => [internal] load .dockerignore                                               0.1s
 => => transferring context: 106B                                               0.0s
 => [internal] load metadata for docker.io/library/ubuntu:18.04                 0.9s
 => CACHED [build-image 1/6] FROM docker.io/library/ubuntu:18.04@sha256:8aa9c2  0.0s
 => [internal] load build context                                               0.0s
 => => transferring context: 174B                                               0.0s
 => CACHED [build-image 2/6] RUN apt-get update                                 0.0s
 => CACHED [build-image 3/6] RUN apt-get install -y gcc                         0.0s
 => CACHED [build-image 4/6] COPY hello.c /tmp                                  0.0s
 => CACHED [build-image 5/6] WORKDIR /tmp                                       0.0s
 => CACHED [build-image 6/6] RUN gcc -o hello-world hello.c                     0.0s
 => [stage-1 2/2] COPY --from=build-image /tmp/hello-world .                    0.1s
 => exporting to image                                                          0.1s
 => => exporting layers                                                         0.1s
 => => writing image sha256:66c3ba5557ae0cfaef2add249d5756fb4bd0b1f4f8462e1ab6  0.0s
 => => naming to docker.io/library/multi-stage         
$ docker run multi-stage
Hello My Container!
$ docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
multi-stage    latest    66c3ba5557ae   27 seconds ago   63.2MB
single-stage   latest    11e154888128   9 minutes ago    220MB
```
동일하게 '.c' 파일이 컴파일 되는 것을 알 수 있으며, multi-stage로 빌드했을 때 이미지 크기가 약 70% 줄어든 것을 확인 가능하다.  

### 09. Docker Registry 
* docker image를 올려두는 곳이며, public과 private 이 각각 있음  

#### Docker hub에 업로드
1. tag 설정
docker tag 이미지이름:tag docker registry url/이미지이름:tag
2. 비보안(insecure) 레지스트리추가
```bash
$ vi /etc/docker/daemon.json
#daemon.json 수정
{"insecure-registries": ["registry.example.com:5000"] }
$ systemctl restart docker
```
3. docker image upload
```bash
$ docker tag hello-world registry.example.com:5000/hello-world
$ docker images
$ docker push registry.example.com:5000/hello-world
```
registry 를 사용하여서 해당 이미지를 docker hub 본인 계정에 업로드 하였다.  

#### Registry 서버를 사용
1. registry server 에 docker 설치
```bash
$ sudo hostnamectl set-hostname registry.example.com
```
docker 가 설치안되어 있으면 먼저 docker 부터 설치한다.  
```bash
$ sudo yum install -y yum-utils
$ sudo yum-config-manager     --add-repo     https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install docker-ce docker-ce-cli containerd.io
$ sudo systemctl start docker
```
2. docker registry 이미지 다운로드 
```bash
$ docker pull registry:latest
```
3. registry container 실행  
```
$ docker run -d --name registry -p 5000:5000 --restart=alwyas registry
```
