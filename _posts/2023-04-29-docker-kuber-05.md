---
layout: single
title:  "[Docker] Docker 강의 정리 (5)"
date:   2023-04-29 10:10:00 +0900

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

Docker(도커), 쿠버네티스 관련 재직자 지원 수업의 마지막 강의 내용을 정리한다.  
오늘은 deployment와, 쿠버네티스의 storage 및 실제 어플리케이션을 개발 및 배포하는 방법에 대해 배울 예정이다.  

## 2023-04-29 강의 노트  
### 16. 쿠버네티스 명령어와 주요 객체 (2)
강의 시작 전에, VM을 실행하고 모두 잘 연결되어 있는지 확인한다.  
```bash
$ kubectl get nodes
NAME             STATUS   ROLES           AGE   VERSION
m.example.com    Ready    control-plane   9d    v1.27.1
w1.example.com   Ready    <none>          9d    v1.27.1
w2.example.com   Ready    <none>          9d    v1.27.1
```  
잘 되어 있다.  

#### Deployment - Replica set  
namespace 를 변경하여 실습하는 것도 가능하지만, 실습 환경인 노트북에서 cpu 개수를 할당하지 못 할 것 같아서 default 로 진행하였다.  
Deployment 객체를 생성하기 위하여 yaml 파일을 작성하였다.  
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14
        ports:
        - containerPort: 80
```  
지난 주 작성한 replica yaml 파일과 굉장히 유사하다. 실제로, deployment는 replica set이 하는 기능을 사용할 수 있으며, roll-in 과 같은 추가 기능도 사용 가능하다.  
Deployment 객체를 생성하고, Replica set이 pod을 제대로 생성하는지 확인 해본다.  
```bash
$ kubectl apply -f deploy.yaml
deployment.apps/nginx created
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7d5c8d9554-6sj6l   1/1     Running   0          5m18s
nginx-7d5c8d9554-7zxwp   1/1     Running   0          5m18s
nginx-7d5c8d9554-dbzjh   1/1     Running   0          5m18s
nginx-7d5c8d9554-dpz7f   1/1     Running   0          5m18s
nginx-7d5c8d9554-fnd2n   1/1     Running   0          5m18s
nginx-7d5c8d9554-grknc   1/1     Running   0          5m18s
nginx-7d5c8d9554-j5jk8   1/1     Running   0          5m18s
nginx-7d5c8d9554-s66wl   1/1     Running   0          5m18s
nginx-7d5c8d9554-tssfk   1/1     Running   0          5m18s
nginx-7d5c8d9554-zfkc5   1/1     Running   0          5m18s
$ kubectl delete pods --all
pod "nginx-7d5c8d9554-6sj6l" deleted
pod "nginx-7d5c8d9554-7zxwp" deleted
pod "nginx-7d5c8d9554-dbzjh" deleted
pod "nginx-7d5c8d9554-dpz7f" deleted
pod "nginx-7d5c8d9554-fnd2n" deleted
pod "nginx-7d5c8d9554-grknc" deleted
pod "nginx-7d5c8d9554-j5jk8" deleted
pod "nginx-7d5c8d9554-s66wl" deleted
pod "nginx-7d5c8d9554-tssfk" deleted
pod "nginx-7d5c8d9554-zfkc5" deleted
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7d5c8d9554-46dmw   1/1     Running   0          39s
nginx-7d5c8d9554-9x48b   1/1     Running   0          39s
nginx-7d5c8d9554-nwq4c   1/1     Running   0          39s
nginx-7d5c8d9554-pncsz   1/1     Running   0          39s
nginx-7d5c8d9554-pplj8   1/1     Running   0          39s
nginx-7d5c8d9554-qd65x   1/1     Running   0          39s
nginx-7d5c8d9554-vqvt4   1/1     Running   0          39s
nginx-7d5c8d9554-wpg6k   1/1     Running   0          40s
nginx-7d5c8d9554-wxlmq   1/1     Running   0          39s
nginx-7d5c8d9554-xj569   1/1     Running   0          39s
$ kubectl get replicasets.apps
NAME               DESIRED   CURRENT   READY   AGE
nginx-7d5c8d9554   10        10        10      6m58s
```  
정상적으로 동작 중인 것을 확인하였다. 이제 replica set에 의하여 pods는 설정한 갯수만큼 계속 유지된다.  

Replica set 객체를 생성했을 때 pod을 삭제하는 방법은, pod의 개수를 0으로 설정하거나 replica set을 삭제하는 것이었다.  
그러나 deployment 에 의해 생성된 pods은 replica set을 삭제해도 다시 생성된다. deployment가 안정적으로 replica set을 유지하여, 해당 객체가 다시 생성되기 때문이다.  
따라서 해당 pod을 삭제하고 싶을 때에는 가장 상위 객체인 deployment를 삭제한다.  
```bash
$ kubectl delete replicasets nginx-7d5c8d9554 
replicaset.apps "nginx-7d5c8d9554" deleted
$ kubectl get replicasets.apps
NAME               DESIRED   CURRENT   READY   AGE
nginx-7d5c8d9554   10        10        5       64s
# replicaset이 다시 올라오지 않도록 deployment를 삭제한다. 
$ kubectl delete deployments.apps nginx 
deployment.apps "nginx" deleted
$ kubectl get replicasets.apps
No resources found in default namespace.
```  

#### Deployment - Roll-in update  
