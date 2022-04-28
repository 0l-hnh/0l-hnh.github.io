---
layout: single
title:  "[Linux] cat, paste 명령어 사용법"
date:   2022-04-27 12:10:00 +0900

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

sitemap:
  changefreq: daily
  priority : 1.0
---

텍스트 데이터를 처리할 때 유용한 스크립트 언어 cat, paste 의 사용 방법에 대해 정리한다.  

## 설명
cat, paste는 유닉스 계열 운영체제에서 사용 가능한 스크립트 언어이다. 텍스트 형태로 되어있는 입력 데이터를 편집할 때 유용하다.   

## 명령어
* 여러 파일을 수직적으로 concat 하기 위해서는 cat을 사용
```bash
cat [options] [file1 ..]
```
file1이 'AAA', file2가 'BBB' 일 시 
AAA  
BBB  
위와 같은 형태로 output 생성  
* 여러 파일을 수평적으로 merge 하기 위해 paste를 사용
```bash
paste [options] [file1 ..]
```
file1이 'AAA', file2가 'BBB'일 시   
AAA BBB  
구분자를 기준으로 위와 같이 output 생성  

## 참고
[cat 페이지 링크 (1)](http://www.incodom.kr/Linux/%EA%B8%B0%EB%B3%B8%EB%AA%85%EB%A0%B9%EC%96%B4/cat)
[paste 참고 블로그 링크 (2)](http://hyeonjae-blog.logdown.com/posts/654302)