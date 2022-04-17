---
layout: single
title:  "[Python] 'ascii' codec can't encode characters in position: ordinal not in range 해결"
date:   2022-03-30 11:40:00 +0900

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

sitemap:
  changefreq: daily
  priority : 1.0
  
---

python으로 한글 데이터를 다룰 때 빈번히 일어나는 인코딩 문제 해결에 대하여 작성하였다.
## 설명
알파벳 이외의 문자를 입/출력해야 하는 개발자는 개발 시 인코딩 문제를 자주 마주하게 된다. 본 문서는 python으로 개발하는 도중 겪은 인코딩 관련 이슈를 정리하려는 용도로 작성되었다.
## 시행착오
### 1) 'ascii' codec can't encode characters in position: ordinal not in range
#### 에러 발생 원인
표준 입출력을 사용하여 터미널에 한글로 출력한 로그, 혹은 텍스트를 python에서 처리할 때, 서버의 python encoding이 utf-8이 아니면 위의 오류가 발생할 수 있다.
#### 해결 방법
쉘 스크립트 내부에 아래 line을 추가하여 python encoding을 utf-8로 선언하면 오류가 사라진다.
```bash
export PYTHONIOENCODING=UTF-8 
```