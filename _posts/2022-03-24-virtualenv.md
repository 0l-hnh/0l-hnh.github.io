---
layout: single
title:  "[Python] 독립적 개발 환경을 위한 가상 환경 구축"
date:   2022-03-24 01:08:00 +0900

categories:
  - python
tags: [python]

author_profile: true
sidebar:
  nav: "docs"

toc: true
toc_label: "목록"
toc_icon: "bars"
toc_sticky: true 
---

python의 가상 환경을 virtualenv로 관리하는 방법에 대하여 서술한다. 본 문서는 CentOS 7 환경을 기반으로 작성되었다.
## 설명
python으로 개발할 때, 각 개발 프로젝트의 라이브러리 버전 등 개발 환경을 독립적으로 유지하고 싶다면 가상 환경을 생성하여 관리하는 것이 편리하다. 예를 들어, python2와 python3 개발 환경을 동일 서버에 각각 구축하고 싶다면 가상 환경을 생성할 필요가 있다. 또한 virtualenv를 사용하는 것은 Make file을 작성할 때도 유용하다.
## 명령어
python 2, 3 모두 사용 가능한 virtualenv 를 사용하였다.
```bash
virtualenv
```
## 예제 
* virtualenv 설치
```bash
pip install virtualenv
```
* 가상 환경 생성
새로운 환경 설치 위치가 프로젝트 경로 /data/project 라고 할 때, 아래 명령어로 python 가상 환경을 생성할 수 있다. 
```bash
virtualenv /data/project/venv 
```
* python 버전 지정하여 가상 환경 생성
만약 가상 환경 생성 시 특정 python 버전을 지정하고 싶다면, 아래와 같은 명령어를 입력한다.
```bash
virtualenv venv --python=python2 #local python2 버전으로 설치됨
virtualenv venv --python=python3 #local python3 버전으로 설치됨
```
* 가상 환경 실행
가상 환경 실행 시 아래와 같이 activate 한다.
```bash
source ./venv/bin/activate
```
venv 를 activate 한 뒤 원하는 python 패키지를 pip으로 설치하여 사용한다.
* 가상 환경 종료
```bash
deactivate
```

## 참고 자료
[참고 페이지 링크 (제타 위키)](https://zetawiki.com/wiki/Virtualenv_%EC%82%AC%EC%9A%A9%EB%B2%95)
