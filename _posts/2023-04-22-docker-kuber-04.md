---
layout: single
title:  "[Docker] Docker 강의 정리 (4) - 쿠버네티스"
date:   2023-04-15 10:10:00 +0900

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

Docker(도커), 쿠버네티스 관련 재직자 지원 수업의 네 번째 강의 내용을 정리한다.  

## 2023-04-22 강의 노트  
### 11. 쿠버네티스 (k8s) 세팅 
지난 시간 종료 때 Vagrantfile 로 minikube 및 쿠버네티스를 설치하기 위한 VM을 build 하고, ssh 설정 및 클러스터 설치를 진행하였다.  
minikube는 간단하게 실행을 해볼 수 있고, [killercoda.com](https://killercoda.com/) 등의 사이트에 접속하면 미리 설정된 시나리오에 따라 생성된 가상 머신 (minikube)을 사용해서 웹 기반으로도 공부해볼 수 있다.  
하지만 본 실습에서는 적어도 세 개 이상의 클러스트를 사용하기 위하여 work node 두 개와 master node를 생성하여 진행할 예정이다.  
Vagarntfile 은 ruby 문법을 따르기 때문에, 해당 언어를 알고 있으면 좋다.  

#### 실습을 위한 VM 설정  
* master node ip 충돌이 나지 않게 ip 잘 설정  
* master node 의 크기는 cpu 개수가 2개 이상
* master node 의 경우, RAM 이 2 GB 이상 필수  

```bash
$ vagrant status
Current machine states:

w1.example.com            running (virtualbox)
w2.example.com            running (virtualbox)
m.example.com             running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```  
정상 설치되어 실행되는 것을 위와 같이 확인하였다.  

#### VM 실행 및 쿠버네티스 설치 확인  
master의 VM에 접속하여, work node가 잘 실행이 되고 있는지 확인한다.  
쿠버네티스 명령어는 'kubectl' 로 실행한다.  
```bash
$ kubectl get nodes
NAME             STATUS   ROLES           AGE     VERSION
m.example.com    Ready    control-plane   2d12h   v1.27.1
w1.example.com   Ready    <none>          2d12h   v1.27.1
w2.example.com   Ready    <none>          2d12h   v1.27.1
```  
Ready 상태라면 준비가 일단 된 것이다.  
쿠버네티스의 가장 기본적인 단위는 'pods' 이다.  

> 파드(Pod) 는 쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 가장 작은 컴퓨팅 단위이다. 파드 (고래 떼(pod of whales)나 콩꼬투리(pea pod)와 마찬가지로)는 하나 이상의 컨테이너의 그룹이다. 이 그룹은 스토리지 및 네트워크를 공유하고, 해당 컨테이너를 구동하는 방식에 대한 명세를 갖는다.  
출처 : [쿠버네티스 공식 문서](https://kubernetes.io/ko/docs/concepts/workloads/pods/)  

```bash
$ kubectl get pods
No resources found in default namespace. 
$ kubectl get pods -n kube-system
NAME                                    READY   STATUS    RESTARTS     AGE
coredns-5d78c9869d-ghdvp                1/1     Running   0            2d12h
coredns-5d78c9869d-hvn88                1/1     Running   0            2d12h
etcd-m.example.com                      1/1     Running   0            2d12h
kube-apiserver-m.example.com            1/1     Running   0            2d12h
kube-controller-manager-m.example.com   1/1     Running   0            2d12h
kube-proxy-9xf77                        1/1     Running   1 (9h ago)   2d12h
kube-proxy-gm5dd                        1/1     Running   0            2d12h
kube-proxy-rdlgr                        1/1     Running   0            2d12h
kube-scheduler-m.example.com            1/1     Running   0            2d12h
```
flannel namespace 를 설치하였다. 전부 Running 상태여야 pods 간에 통신이 정상적으로 이루어진다고 볼 수 있다.  
```bash
$ kubectl get ns
NAME              STATUS   AGE
default           Active   2d12h
kube-flannel      Active   2d12h
kube-node-lease   Active   2d12h
kube-public       Active   2d12h
kube-system       Active   2d12h
$ kubectl get pods -n kube-flannel
NAME                    READY   STATUS    RESTARTS     AGE
kube-flannel-ds-8pqh5   1/1     Running   1 (9h ago)   2d12h
kube-flannel-ds-jrszb   1/1     Running   0            2d12h
kube-flannel-ds-qr7tp   1/1     Running   0            2d12h
```

#### pod 생성 
* 쿠버네티스는 포그라운드, 백그라운드 개념이 없음 (work node 에서 실행되기 때문)  

```bash
$ kubectl run myapache --image httpd:2.4
pod/myapache created
```  
쿠버네티스 1.24부터 컨테이너 런타임으로 docker 를 지원하지 않는다.  
따로 지원하는 버전이 나왔기 때문에, 해당 런타임을 사용하면 가능하지만 본 실습에서는 Docker를 사용하지 않는다.  
```bash
$ kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE             NOMINATED NODE   READINESS GATES
myapache   1/1     Running   0          2m22s   10.244.2.2   w2.example.com   <none>           <none>
```  
pod의 ip 는 컨테이너 ip와 동일하다. 
컨테이너 ip가 pod ip를 사용하기 때문에, 해당 ip를 공유한다.  
```bash
$ curl http://10.244.2.2
It Works!
```  
해당 ip로 접속을 하니 잘 되었다. pod 통신이 되는 것을 확인하였다.  

### 12. 컨테이너 실행을 위한 런타임  
#### Docker 런타임  
Docker 런타임의 구성 요소 및 쿠버네티스 1.24부터 docker를 지원하지 않게 된 사유를 알아본다.  
Docker 런타임은 아래왕 같이 이해할 수 있다.  
* docker engine: 도커 이미지를 관리하고 컨테이너 실행을 위해 containerd 데몬과 통신하여 runc 기반으로 컨테이너를 실행  
* containerd: Docker 에서 OCI spec 을 준수하여 개발한 container runtime high level container run time. 도커레지스트리에서 이미지를 가져오고, 도커 네트워크및 스토리지 관리 그리고 컨테이너 실행을 위해서 저수준 런타임인 runc 를 실행하고 관리 
* runc : container 를 생성하고 실행하는 low level 런타임, OCI runtime spec 준수  

런타임 레벨은 (High) containerd -> runc (Low) 이다.  

#### CRI
* 쿠버네티스에서 만든 API (컨테이너 런타임 인터페이스)  
* 쿠버네티스가 각 런타임과 상호작용하는 방법을 설명하며, 주어진 컨테이너 런타임이 CRI API를 구현하면 런타임을 원하는대로 선택하여 컨테이너를 생성하고, 실행할 수 있음 (예제 : containerd, CRI-O 등)  
* CRI-O 는 Redhat, IBM 등이 개발한 쿠버네티스 전용 컨테이너 런타임으로, 컨테이너 생성 및 이미지 빌드 등을 할 수 없고, 실행에 중점을 둠.  
  
Docker는 CRI 표준을 준수하지 못 하기 때문에, 해당 인터페이스에 부합하는 Dockershim을 사용해야 kubelet에서 사용할 수 있다.  
즉, kubelet에서 거쳐야하는 단계가 많다.  

> kubelet ↔ Dockershim ↔ Docker ↔ containerd → runc  

쿠버네티스는 CRI 표준에 맞지 않는 Docker 를 사용하기 위하여 1.23 버전까지 중간 단계를 거쳐서 runc를 실행하였지만, 1.24부터는 containerd에 CRI 플러그인을 사용하여 컨테이너를 실행하게 되었다.  
단, 그 후 개발된 CRI-dockerd 를 컨테이너 런테임으로 사용하면 계속 docker를 사용할 수 있다.  
minikube의 경우 CRI-dockerd 를 사용한다.  