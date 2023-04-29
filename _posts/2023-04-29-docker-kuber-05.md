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

#### Deployment - Replicas  
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
현재 실행 중인 nginx 의 ip를 확인한다.  
```bash 
$ ip a s cni0
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether f6:54:9c:67:3e:4a brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/24 brd 10.244.0.255 scope global cni0
       valid_lft forever preferred_lft forever
```

#### Deployment - Rolling Update, Rollout
쿠버네티스에서 지원하는 무중단 배포 방식에 대하여 알아본다.  

예시로, 현재 사용하는 nginx 버전을 1.14에서 1.15로 업데이트 해보자. yaml 파일을 수정한 다음 다시 apply 하면 이미 생성된 객체가 수정된다.  
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
        image: nginx:1.15
        ports:
        - containerPort: 80
```  
image의 태그를 수정하였다. 이제 적용한 다음 pods들의 상태를 watch로 관찰한다.  
```bash
$ kubectl apply -f deploy.yaml 
deployment.apps/nginx configured
$ watch -n 1 -d kubectl get pods
```  
실행 중인 pods 전체가 중단되는 것이 아니라 일부만 중단되고, 새로운 pods들이 생기는 것을 확인 가능하다. 이 동작에 대해 아래와 같은 설명을 찾을 수 있다.  

> 디플로이먼트는 업데이트되는 동안 일정한 수의 파드만 중단되도록 보장한다. 기본적으로 적어도 의도한 파드 수의 75% 이상이 동작하도록 보장한다(최대 25% 불가).  
- 출처 : [쿠버네티스 공식 페이지](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)  

이 설정은 depolyment.app의 설정에서도 찾을 수 있다.  
```bash
$ kubectl describe deployments.apps
Name:                   nginx
Namespace:              default
CreationTimestamp:      Sat, 29 Apr 2023 10:16:28 +0900
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 4
Selector:               app=nginx
Replicas:               10 desired | 10 updated | 10 total | 10 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
(후략)
```  

yaml 파일이 아니라 명령어로도 업데이트를 진행할 수 있다.  
업데이트 후 kubectl get pods -o wide 로 ip 정보를 알아내어 curl로 접속 시도한 뒤, 버전을 확인한다.  
```
$ kubectl set image deployment nginx nginx=nginx:1.16
deployment.apps/nginx image updated
$ kubectl get pods -o wide
NAME                     READY   STATUS              RESTARTS     AGE     IP            NODE             NOMINATED NODE   READINESS GATES
nginx-57d98f69f6-4jlzw   0/1     ContainerCreating   0            13s     <none>        w2.example.com   <none>           <none>
nginx-57d98f69f6-cjwdm   0/1     ContainerCreating   0            13s     <none>        w1.example.com   <none>           <none>
nginx-57d98f69f6-hhpm6   0/1     ContainerCreating   0            13s     <none>        w2.example.com   <none>           <none>
nginx-57d98f69f6-r5gq6   0/1     ContainerCreating   0            13s     <none>        w1.example.com   <none>           <none>
nginx-57d98f69f6-v8hhf   0/1     ContainerCreating   0            13s     <none>        w1.example.com   <none>           <none>
nginx-6dccb7ff87-6cm6p   1/1     Running             1 (9h ago)   4m12s   10.244.2.65   w2.example.com   <none>           <none>
nginx-6dccb7ff87-9jlff   1/1     Running             1 (9h ago)   4m12s   10.244.2.63   w2.example.com   <none>           <none>
nginx-6dccb7ff87-gb8jc   1/1     Running             1 (9h ago)   4m12s   10.244.2.64   w2.example.com   <none>           <none>
nginx-6dccb7ff87-nd52r   1/1     Running             1 (9h ago)   4m12s   10.244.2.58   w2.example.com   <none>           <none>
nginx-6dccb7ff87-qgdbp   1/1     Running             1 (9h ago)   4m12s   10.244.2.57   w2.example.com   <none>           <none>
nginx-6dccb7ff87-sk84r   1/1     Running             1 (9h ago)   4m12s   10.244.2.60   w2.example.com   <none>           <none>
nginx-6dccb7ff87-tsqrh   1/1     Running             1 (9h ago)   4m12s   10.244.2.61   w2.example.com   <none>           <none>
nginx-6dccb7ff87-wm4wf   1/1     Running             1            4m12s   10.244.2.59   w2.example.com   <none>           <none>
$ curl http://10.244.2.69/a.html
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.16.1</center>
</body>
</html>
```
'a.html' 페이지는 없기 때문에 접속되지 않았지만, 버전이 정상적으로 바뀌었다는 걸 확인할 수 있다.  
yaml 파일에서 maxUnavailable 은 rollinig update 동안 동작하지 않아도 되는 pod의 개수이며, maxSurge는 rolling update 동안 추가로 실행될 pod의 개수이다.  

kubectl 명령어로 업데이트 한 deployment 에 대해서, 아래와 같이 rollout 해본다.  
history 로 deployment의 업데이트 내역을 확인할 수 있으며, 원하는 리비전 번호를 지정하여 rollout, 즉 롤백 할 수 있다.  
```bash
$ kubectl set image deployment nginx nginx=nginx:1.17 --record=true
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx image updated
$ kubectl rollout history deployment nginx 
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image deployment nginx nginx=nginx:1.17 --record=true
$ kubectl rollout undo deployment nginx --to-revision=2
deployment.apps/nginx rolled back
$ curl http://10.244.2.77/a.html
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.16.1</center>
</body>
</html>
$ kubectl get replicasets.apps 
NAME               DESIRED   CURRENT   READY   AGE
nginx-57d98f69f6   10        10        10      11m
nginx-6cff568d77   0         0         0       8m19s
nginx-6dccb7ff87   0         0         0       15m
```  
1.17로 업데이트 하였던 버전이 다시 1.16으로 롤 백 되었다. 이 상태에서 Replica set을 확인하면, 사용하지 않는 버전이 삭제 (pods 개수가 0) 되었음을 알 수 있다.  
