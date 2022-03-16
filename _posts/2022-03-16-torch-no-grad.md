---
layout: single
title:  "[Pytorch] 훈련 시 메모리 절약을 위한 torch.no_grad 사용 "
date:   2022-03-16 23:57:00 +0900

categories:
  - python
tags: [python, pytorch]

author_profile: true
sidebar:
  nav: "docs"

toc: true
toc_label: "목록"
toc_icon: "bars"
toc_sticky: true 
---

Torch 로 선언된 신경망의 변수들 중 조정할 필요가 없는 파라미터를 컨텍스트 매니저(context manager)의 no_grad로 제어하는 방법에 대하여 서술한다.
## 설명
인공신경망은 모델의 출력과 데이터 레이블 간의 손실을 계산한 뒤, 오차를 역전파하여 파라미터를 최적화 한다. 이는 모델 훈련에 핵심적인 부분이지만, 훈련 모델의 성능 측정을 위한 Validation, Evaluation에는 불필요하다.  따라서 연산량이 많은 모델 훈련 시,  Backward 를 할 필요 없는 부분에 no_grad를 선언하면 메모리를 아낄 수 있다. 예를 들어, 용량이 크지 않은 GPU로 딥 러닝 학습을 수행할 시 흔히 발생하는 OOM Error 방지에 도움을 준다.
### Class 명
```python
class torch.no_grad
```
### 예제
input tensor x에 대한 연산 처리 중 역전파 하지 않을 부분을 with torch.no_grad(): 로 블록 처리하는 예시  
```python
>>> x = torch.tensor([1], requires_grad=True)
>>> with torch.no_grad():
...   y = x * 2
>>> y.requires_grad
False
```
@를 달아서 데코레이터로 사용 가능
```python
>>> x = torch.tensor([1], requires_grad=True)
>>> @torch.no_grad()
... def doubler(x):
...     return x * 2
>>> z = doubler(x)
>>> z.requires_grad
False
```
### 출처
[출처 (Torch 공식 페이지) 링크](https://pytorch.org/docs/stable/generated/torch.no_grad.html)
