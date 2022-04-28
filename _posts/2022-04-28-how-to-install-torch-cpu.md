---
layout: single
title:  "[Python] Could not find a version that satisfies the requirement 해결"
date:   2022-04-28 11:00:00 +0900

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

pip 으로 특정 라이브러리의 버전을 지정하여 설치할 때 가끔 마주치는 오류에 대하여 서술한다.  
## 설명
특정 버전을 사용하여 개발해야 할 때 pip으로 해당 라이브러리를 지정하여 설치하다가 아래와 같은 오류를 마주칠 때가 있다.   
## 시행착오
### 1) 'Could not find a version that satisfies the requirement torch==1.8.0+cpu
#### 에러 발생 원인
특정 torch 버전의 cpu only 사양을 설치해야 할 때 'ERROR: Could not find a version that satisfies the requirement' 라는 오류가 발생했다. 다운로드 페이지를 명시하지 않아서 발생한 오류이다.    
#### 해결 방법
-f https://download.pytorch.org/whl/torch_stable.html를 추가한다.  
``` bash
pip install torch==1.8.0+cpu -f https://download.pytorch.org/whl/torch_stable.html 
```