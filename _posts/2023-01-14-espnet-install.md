---
layout: single
title:  "[ASR] 도커를 사용한 ESPNet 간편 설치 및 예제 실행"
date:   2023-01-14 11:10:00 +0900

categories:
  - asr
tags: [asr, espnet, docker]

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
ESPNet이란 음성 인식, 음성 합성 관련 레시피를 제공하는 Toolkit 이다. 다양한 모듈 및 훈련에 필요한 도구가 포함되어 있다. 'espnet' 실행을 위해서는 KALDI 설치가 필수이며, 'espnet2'는 pytorch 설치 만으로 실행 가능하다.  
본 문서에서는 nvidia-docker로 ESPnet2 훈련 예제 실행 방법을 서술한다.  

## 상세  
### ESPNet 설치  
1. git 레포지토리에서 최신 소스를 다운로드한다.  
```bash
$ cd <any-place>
$ git clone https://github.com/espnet/espnet
```
2. espnet/docker 디렉토리의 실행 스크립트를 실행한다.  

### 간단한 예제 실행  
yes/no 두 개 단어를 인식하는 가장 간단한 예제를 실행하여 정상 설치를 확인한다.  
특징 추출부에서 KALDI IO를 사용하지 않는 ESPnet2 레시피 실행을 원할 시 –is-egs2 플래그를 추가하여 아래와 같이 실행한다.  
```bash
$ cd espnet/docker
$  ./run.sh --docker-gpu 0 --is-egs2 --docker-egs yesno/asr1 --ngpu 1
```
외부 인자는 실행 스크립트 내에서 정의할 수도 있다.  
이 때 레시피 내에서 구동되는 run.sh 내용은 다음과 같다.  
```bash
$ NV_GPU='0' nvidia-docker run -i --rm --name ${name} -v /data/espnet/egs:/espnet/egs -v /data/espnet/espnet:/espnet/espnet -v /data/espnet/test:/espnet/test -v /data/espnet/utils:/esp       net/utils -v /data/espnet/egs2:/espnet/egs2 -v /data/espnet/espnet2:/espnet/espnet2 -v /dev/shm:/dev/shm espnet/espnet:gpu-latest-user-hanni /bin/bash -c 'cd /espnet/egs2/yesno/asr1; ./run.sh --ngpu 1’
```
정상 실행 시 출력되는 로그 파일은 추후 추가 예정  

## 시행 착오
### nvcc: command not found CUDA was not found in your system
분명히 nvidia-docker 까지 잘 설치한 것 같은데 위와 같은 오류를 만나면 슬프다. 이 경우 오류를 뱉는 run.sh 의 line 71을 아래와 같이 주석 처리 하여도 동작에 이상이 없음을 확인하였다.  
```bash
from_tag="cpu"
if [ ! "${docker_gpu}" == "-1" ]; then
    #docker_cuda=$( nvcc -V | grep release )
    docker_cuda=${docker_cuda#*"release "}
    docker_cuda=${docker_cuda%,*}

    # After search for your cuda version, if the variable docker_cuda is empty the program will raise an error
    #if [ -z "${docker_cuda}" ]; then
    #    echo "CUDA was not found in your system. Use CPU image or install NVIDIA-DOCKER, CUDA for GPU image."
    #    exit 1
    #fi
        from_tag="gpu"
fi
```  

### /bin/nvidia-docker: line 34: /bin/docker: Permission denied
다음 명령어로 해결한다.
```bash
$ sudo semanage fcontext -a -t container_runtime_exec_t /usr/bin/nvidia-docker
$ sudo restorecon -v /usr/bin/nvidia-docker
```

## 참고 사이트  
[Docker 설치 관련 포스트](https://0l-hnh.github.io/docker/docker-install  )  
[ESPNet 공식 문서](https://espnet.github.io/espnet/docker.html  )  
[nvidia-docker 권한 에러](https://github.com/NVIDIA/nvidia-docker/issues/814  )