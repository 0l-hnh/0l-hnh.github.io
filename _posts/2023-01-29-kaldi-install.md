---
layout: single
title:  "[ASR] Kaldi toolkit 설치"
date:   2023-01-29 11:10:00 +0900

categories:
  - asr
tags: [asr, kaldi]

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
Kaldi는 음성인식을 위한 toolkit으로, C++ 로 작성되었으며 Apache v2.0 라이센스를 따른다.  
사용하는 주요 외부 라이브러리는 srilm, openFST이며 nnet3 dnn와 HCLG.fst를 결합한 그래프로 디코딩을 수행하는 것이 특징이다.  
phone의 전이 확률을 계산하는 AM과 n-gram LM을 사용하는 방식을 통해 높은 정확도를 보인다.  

## 상세  
### Kaldi 설치  
아래 설치 과정은 CentOS7을 기준으로 하며, Kaldi 설치 시 요구되는 최소 사항을 모두 만족하였다고 가정한다.  
Kaldi 컴파일을 위해서는 Cmake, gcc, python2, python3, nvidia 드라이버 및 사용 GPU에서 지원하는 버전의 CUDA가 커널에 설치되어 있어야 한다.  
또한 수학 라이브러리인 MKL 혹은 ATLAS를 필요로 한다.  
위 필수 설치 라이브러리들을 갖춘 후 아래와 같이 Kaldi를 설치할 수 있다.  

1. git 레포지토리에서 최신 소스를 다운로드한다.  
```bash
$ git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream
$ cd kaldi
```
위의 kaldi 디렉토리를 ${KALDI_ROOT}로 사용한다.  
2. tools 디렉토리의 외부 라이브러리들을 설치한다.
```bash
$ cd tools
$ make 
```  
3. srilm의 경우, 라이센스 때문에 별도 설치를 해야 한다. 
srilm 공식 사이트에서 설치 파일을 다운로드 받은 후 tools 디렉토리에 위치한 뒤 extras/install_srilm.sh 을 실행하여 진행한다.  
공식 사이트 링크 : http://www.speech.sri.com/projects/srilm/srilm_download.php  
4. 외부 패키지가 모두 설치 되었으면 칼디 소스를 컴파일 한다.
```bash
$ cd ${KALDI_ROOT}/src
$ ./configure --shared
$ make -j 4 
```  
전체 소스 컴파일에는 상당 시간이 소요된다. 에러 메시지 없이 설치가 끝나면 각 예제의 'path.sh' 파일을 실행하는 것으로 바이너리 파일들을 커맨드 라인에서 바로 실행 가능하다.  

## 참고 사이트  
[Kaldi 공식 문서](http://kaldi-asr.org/  )  
