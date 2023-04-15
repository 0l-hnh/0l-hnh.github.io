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
[참고를 위한 Docker 공식 사이트](https://hub.docker.com/_/scratch)  
Scratch Image란 이미지를 최대한 작게 만들기 위해서 사용하는 것이다.  
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

#### 