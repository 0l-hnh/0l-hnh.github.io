---
layout: single
title:  "[Docker] Docker 강의 정리 (5) - Deployment, Storage, Application 배포"
date:   2023-04-29 10:10:00 +0900

categories:
  - docker
tags: [docker, kubernetes, linux]

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
### 16. 쿠버네티스 명령어와 주요 개념 (2)
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
-출처 : [쿠버네티스 공식 페이지](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)  

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
```bash
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
'kubectl rollout undo deployment nginx --to-revision=2' 에서 --to-revision 옵션이 없다면, 바로 이전 버전으로 롤 백 된다.  

#### Service : LoadBalancer type  
먼저 지난 주에 배웠던 service type 두 개에 대해서 복습한다.  
label 을 가진 pod 생성 후, service 를 생성하면 해당 service 에 Cluster IP로 접속이 가능하다. 또한, 외부 접속이 가능하기 위해서는 'NodePort' type으로 변경하여 Node IP (cat /etc/hosts의 IP)와 Port 번호로 접근할 수 있다.  
```bash
$ kubectl get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP          6d20h
myweb-service   NodePort    10.105.77.228   <none>        8001:30632/TCP   6d19h
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
:1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.14.50   m.example.com      m
192.168.14.51   w1.example.com     w1
192.168.14.52   w2.example.com     w2
# 외부에서는 '192.168.14.50:30632' 로 접근 가능하다. 
```  
또, 이 방법 외에 'LoadBalacer' type으로 버전을 설정할 수가 있다. 이 type은 클라우드 환경인 gcp나 aws 에서는 기본으로 지원하지만 Local PC인 경우 오픈소스 프로젝트인 metal lb 를 설치해야 한다.  

#### Storage Setting   
쿠버네티스의 볼륨 설정에 대해 알아본다.  
[쿠버네티스 공식 페이지](https://kubernetes.io/ko/docs/concepts/storage/) 에서 다양한 종류의 스토리지를 확인할 수 있다.  
오늘 수업에서는 스토리지의 기본적인 사용 방식을 이해하고, 몇 가지 타입을 알아보도록 한다.  

##### hostPath
* 호스트 노드의 파일시스템에 있는 파일이나 디렉토리에 직접 마운트 
* pod이 삭제되어도 볼륨의 데이터는 유지
* 기존 pod 삭제되고 새로운 pod가 스케쥴링 될 때, 만약 기존 노드에 pod가 스케쥴링 되지 않으면 기존 노드 데이터는 사용할 수 없음
* 공식에서는 **보안상 hostPath 사용하지 않는 것**을 권장  

![hostPath 이미지](https://i.ibb.co/kSJQYJq/2023-04-29-115352.png)  

아래와 같이 예시 yaml 파일을 작성한다.  
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: myapache-new
  labels:
    app: myweb-svc
spec:
  containers:
  - name: myapache-container
    image: httpd:2.4
    ports:
    - containerPort: 80
    volumeMounts:
    - name: hostpath-volume
      mountPath: /usr/local/apache2/htdocs
  volumes:
  - name: hostpath-volume
    hostPath:
      path: /var/tmp/web_docs
```  
```bash
$ kubectl get pods 
NAME           READY   STATUS    RESTARTS   AGE
myapache-new   1/1     Running   0          6m32s
```
잘 실행 되었다. 
```bash
$ kubectl exec -it myapache-new -- /bin/bash
(apache)$ df -ha
Filesystem                   Size  Used Avail Use% Mounted on
overlay                      125G  4.2G  121G   4% /
proc                            0     0     0    - /proc
tmpfs                         64M     0   64M   0% /dev
devpts                          0     0     0    - /dev/pts
mqueue                          0     0     0    - /dev/mqueue
sysfs                           0     0     0    - /sys
tmpfs                        909M     0  909M   0% /sys/fs/cgroup
cgroup                          0     0     0    - /sys/fs/cgroup/systemd
cgroup                          0     0     0    - /sys/fs/cgroup/net_cls,net_prio
cgroup                          0     0     0    - /sys/fs/cgroup/perf_event
cgroup                          0     0     0    - /sys/fs/cgroup/cpu,cpuacct
cgroup                          0     0     0    - /sys/fs/cgroup/rdma
cgroup                          0     0     0    - /sys/fs/cgroup/hugetlb
cgroup                          0     0     0    - /sys/fs/cgroup/pids
cgroup                          0     0     0    - /sys/fs/cgroup/freezer
cgroup                          0     0     0    - /sys/fs/cgroup/devices
cgroup                          0     0     0    - /sys/fs/cgroup/memory
cgroup                          0     0     0    - /sys/fs/cgroup/cpuset
cgroup                          0     0     0    - /sys/fs/cgroup/blkio
/dev/mapper/cl_centos8-root  125G  4.2G  121G   4% /etc/hosts
/dev/mapper/cl_centos8-root  125G  4.2G  121G   4% /dev/termination-log
/dev/mapper/cl_centos8-root  125G  4.2G  121G   4% /etc/hostname
/dev/mapper/cl_centos8-root  125G  4.2G  121G   4% /etc/resolv.conf
shm                           64M     0   64M   0% /dev/shm
/dev/mapper/cl_centos8-root  125G  4.2G  121G   4% /usr/local/apache2/htdocs
tmpfs                        1.7G   12K  1.7G   1% /run/secrets/kubernetes.io/serviceaccount
proc                            0     0     0    - /proc/bus
proc                            0     0     0    - /proc/fs
proc                            0     0     0    - /proc/irq
proc                            0     0     0    - /proc/sys
proc                            0     0     0    - /proc/sysrq-trigger
tmpfs                        909M     0  909M   0% /proc/acpi
tmpfs                         64M     0   64M   0% /proc/kcore
tmpfs                         64M     0   64M   0% /proc/keys
tmpfs                         64M     0   64M   0% /proc/timer_list
tmpfs                         64M     0   64M   0% /proc/sched_debug
tmpfs                        909M     0  909M   0% /proc/scsi
tmpfs                        909M     0  909M   0% /sys/firmware
(apache)$ echo "welcome apache container" > index.html
(apache)$ ls
index.html
```
해당 pod로 접속하여 host의 디렉토리가 잘 마운트 되어 있는 것을 확인하였다. 그 후 'index.html' 페이지를 생성하였다. 이제 해당 pod이 실행 중인 w1.example.com node에 접속하면, 볼륨으로 준 디렉토리에 해당 파일이 생성되어 있다.  
```bash
$ ssh w1.example.com 
(w1)$ ls /var/tmp/web_docs/
index.html
```  
현재 pod가 삭제되어도, 동일한 w1 node에 할당된 pod는 기존과 동일한 페이지를 볼 수 있다. 단, 다른 node에 pod가 할당된다면 해당 페이지를 볼 수 없다.  
예를 들어, 아래와 같이 apache 서버를 몇 개 더 띄운 뒤 접속을 시도할 시 기존 pod과 동일한 node를 사용할 때는 새로 만들어 준 페이지를 볼 수 있으나, 그렇지 않을 때는 다른 페이지가 난다는 것을 확인할 수 있다.  
```bash
$ kubectl get pods -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP            NODE             NOMINATED NODE   READINESS GATES
myapache-new    1/1     Running   0          82s   10.244.2.85   w2.example.com   <none>           <none>
myapache-new2   1/1     Running   0          17s   10.244.1.58   w1.example.com   <none>           <none>
myapache-new3   1/1     Running   0          9s    10.244.2.86   w2.example.com   <none>           <none>
$ curl http://10.244.2.85
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /</title>
 </head>
 <body>
<h1>Index of /</h1>
<ul></ul>
</body></html>
$ curl http://10.244.1.58
welcome apache container
```  
이런 문제를 방지하기 위하여 yaml 파일에 'nodeName'을 쓰거나, 'nodeSelector' 를 설정할 수 있다. nodeName을 쓰는 방식으로 지정을 해 보자,   
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: myapache-new4
  labels:
    app: myweb-svc
spec:
  containers:
  - name: myapache-container
    image: httpd:2.4
    ports:
    - containerPort: 80
    volumeMounts:
    - name: hostpath-volume
      mountPath: /usr/local/apache2/htdocs
  nodeName: w1.example.com
  # nodeName은 'containers'와 정렬을 맞춰야 한다. 
  volumes:
  - name: hostpath-volume
    hostPath:
      path: /var/tmp/web_docs
```  
```bash
$ kubectl get pods -o wide
NAME            READY   STATUS    RESTARTS   AGE     IP            NODE             NOMINATED NODE   READINESS GATES
myapache-new    1/1     Running   0          7m5s    10.244.2.85   w2.example.com   <none>           <none>
myapache-new2   1/1     Running   0          6m      10.244.1.58   w1.example.com   <none>           <none>
myapache-new3   1/1     Running   0          5m52s   10.244.2.86   w2.example.com   <none>           <none>
myapache-new4   1/1     Running   0          7s      10.244.1.59   w1.example.com   <none>           <none>
$ curl http://10.244.1.59
welcome apache container
```  
myapache-new4가 지정한 node에 할당된 것을 확인 가능하다. 이제 myapache-new4는 원하는 스토리지에 저장된 데이터를 사용 가능하다.  
해당 방법은 실제 프로덕션 배포 환경에서 사용하기에는 부적절할 수 있다.  

##### emptyDir
* 영구 스토리지는 아니며, pod이 실행될 동안만 존재  
* emptyDir의 생명 주기는 pod 단위이기 때문에, 컨테이너가 삭제되거나 재시작 되어도 삭제되지 않고 계속해서 사용 가능  
* 디스크 대신 메모리를 사용하는 것이 가능함  

자세한 설명 : [공식 사이트 링크](https://kubernetes.io/ko/docs/tasks/configure-pod-container/configure-volume-storage/)  

redis 이미지를 사용하고, emptyDir 볼륨을 'memory'로 사용하는 예시 yaml 을 아래와 같이 작성하였다.  
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir:
      medium: Memory
      sizeLimit: 1Gi # sizeLimit 지정하지 않으면 가용 메모리 모두 사용할 수 있음
```  
```bash
$ kubectl apply -f redis.yaml 
pod/redis created
$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE             NOMINATED NODE   READINESS GATES
redis   1/1     Running   0          86s   10.244.1.60   w1.example.com   <none>           <none>
```  
w1.example.com 에 pod이 배치 되었다. 해당 node 에 스토리지가 배정 되었을 것이다.  
```bash
$ kubectl exec -it redis -- /bin/bash
(redis)$ df -ah | grep redis
tmpfs                        1.0G     0  1.0G   0% /data/redis
(redis)$ echo "hello redis" > /data/redis/redis.txt
(redis)$ exit
$ ssh w1.example.com
(w1)$ sudo find / -name redis.txt
/var/lib/kubelet/pods/569c5636-8dea-42f3-8ebb-a64e8e3b0de0/volumes/kubernetes.io~empty-dir/redis-storage/redis.txt
(w1)$ sudo cat /var/lib/kubelet/pods/569c5636-8dea-42f3-8ebb-a64e8e3b0de0/volumes/kubernetes.io~empty-dir/redis-storage/redis.txt
hello redis
```  
해당 pod에서 생성한 데이터가 node 디렉토리에 저장되어 있으며, 할당한 메모리만큼 크기가 잡힌 것을 확인 가능하다.  
단, 해당 데이터는 pod이 삭제되면 함께 삭제되는 pod의 sub-directory 에 위치해 있다. 따라서 pod이 삭제된 후에는 해당 파일을 확인할 수 없다.  
```bash
$ kubectl delete pods redis
pod "redis" deleted
$ ssh w1
(w1)$ sudo cat /var/lib/kubelet/pods/569c5636-8dea-42f3-8ebb-a64e8e3b0de0/volumes/kubernetes.io~empty-dir/redis-storage/redis.txt
cat: /var/lib/kubelet/pods/569c5636-8dea-42f3-8ebb-a64e8e3b0de0/volumes/kubernetes.io~empty-dir/redis-storage/redis.txt: No such file or directory
```  
해당 pod을 삭제 후 동일 node 내 디렉토리에 접근을 시도하니, 오류가 발생했다.  
/tmp 와 /var/tmp 에 위치한 파일은 일정 기간 동안 접근하지 않으면 삭제되니 주의해야 한다.  

##### nfs
* 리눅스에서 가장 많이 쓰는 타입 (aws에서는 efs를 사용하는데, nfs와 굉장히 유사하기 때문에 nfs를 이해하면 잘 사용할 수 있음)  

기존 Docker VM 으로 사용했던 192.168.51.10 VM을 nfs 서버로 사용한다.  
VM끼리 통신이 가능하도록 ip 대역과 LAN 카드를 설정한 다음, nfs 서버로 사용할 docker 서버에 아래와 같이 nfs-utils을 설치하고 방화벽을 내린다. 
```bash
(dkr)$ sudo yum -y install nfs-utils
(dkr)$ sudo firewall-cmd --list-alls
```  
그리고 node 에 /var/nfs_storage 를 마운트하여 사용하고, pod의 컨테이너는 생성될 때 node의 로컬 디렉토리에 마운트한다.  
nfs-server를 설치한 VM의 /etc/exports에 아래와 같이 연결할 서버를 설정한다.  
```bash
(dkr)$ mkdir /var/nfs_storage
(dkr)$ cat exports
/var/nfs_storage *(rw,sync,insecure,no_root_squash)
#권한 설정을 잘못 하면 연결이 되지 않으니 주의한다. 
$ exportfs -a
```  
nfs 서버로 쓸 VM 세팅이 완료되었다. work node와 master node에도 nfs-utils를 설치하고, 명령어를 사용하여 mount 한다. mount 시에는 root로 명령어를 실행해야 편하다.  
```bash
$ sudo yum -y install nfs-utils
$ mount -t nfs -o rw 192.168.51.10:/var/nfs_storage /mnt/nfs
```
이제 nfs 서버에 /mnt/nfs 디렉토리가 마운트 되었다. pod의 yaml 파일 내에 nfs-volume 설정을 해 본다.  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-storage-test
spec:
  containers:
  - name: nfs-container-test
    image: centos:7
    command: ['sh', '-c', '/usr/bin/sleep 3600s']
    volumeMounts:
    - name: nfs-volume
      mountPath: /mnt
  volumes:
  - name: nfs-volume
    nfs:
      path: /var/nfs_storage
      server: 192.168.51.10
```  
```bash
$ apply -f nfs.yaml 
$ kubectl exec -it nfs-storage-test -- /bin/bash
(nfs-storage-test)$ touch /mnt/test
```  
nfs 서버로 사용하는 192.168.51.10 서버로 접속하면, 지정 위치에 해당 파일이 생성된 것을 확인할 수 있다.  

#### PersistentVolume
##### PV, PVC
* PV : 영구 스토리지 볼륨을 설정하기 위한 객체
* PVC : 영구 스토리지 볼륨 사용을 요청하기 위한 객체  

##### Storage's Lifecycle
1. Provisioning  
  - 볼륨으로 사용하기 위한 공간을 확보
  - PV 생성 (동적, 정적)
2. Binding
  - Provisioning으로 생성된 PV와 PVC를 연결
  - PVC를 통해 조건에 맞는 PV와 연결
  - PVC 한 개는 한 개의 PV에 bindings
3. 사용
  - PVC가 pod에 설정되고, pod는 PVC를 통해 불륨을 인식
4. 반환
  - PVC 사용이 끝난 뒤 자원이 반환됨.
  - 반환된 자원에 대한 사용 정책 : retain, delete, recycle 등

CLI에서 접근 모드에 대한 약어는 아래와 같다.
* ReadWriteOnce (RWO)
* ReadOnlyMany (ROX)
* ReadWriteMany (RWX)  

### 17. 웹 어플리케이션 배포 실습  
작업 순서 : (백엔드) DB -> 서버 -> wp (프론트엔드)
1. PersistentVolume (pv) 생성
  - 작성 시 storage에 대해 알 필요 있음 (용량, path, type 등)
  - 생성된 pv는 namespace와 상관 없이 다 보임 
2. PersistentVolumeClaim (pvc) 생성해서 pv 요청 
  - pv type에 대해 몰라도, 용량만 작성하여서 사용 가능함
```bash
# Binding 된 결과
$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                       STORAGECLASS   REASON   AGE
pv1    1Gi        RWO            Retain           Bound    default/mysql-volumeclaim                           4m45s
$ kubectl get pvc
NAME                STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-volumeclaim   Bound    pv1      1Gi        RWO                           9s
```
