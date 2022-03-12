---
layout: single
title:  "[Linux] awk 명령어 사용법"
date:   2022-03-12 23:32:00 +0900

categories:
  - os
tags: [os, linux]

author_profile: true
sidebar:
  nav: "docs"

toc: true
toc_label: "목록"
toc_icon: "bars"
toc_sticky: true
---

유닉스 계열 운영체제에서 사용 가능한 스크립트 언어 awk 의 사용 방법에 대해 정리한다.  

## 기본 기능
* 텍스트 형태로 되어있는 입력 데이터를 행과 단어 별로 처리해 출력한다.

## 자주 사용하는 문법
* 첫 번째 필드 출력
```bash
awk '{print $1}' input
```
* 마지막 필드 출력
```bash
awk '{print $NF}' input
```
* 첫 번째 필드에 prefix 추가해서 출력
```bash
awk '{print "prefix_"$1}' input
```
*  -F 로 구분자를 지정
```bash
awk -F '\t' '{print $1}' input
```
