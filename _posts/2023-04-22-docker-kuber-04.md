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
### 11. 쿠버네티스 (k8s)
지난 시간 종료 때 Vagrantfile 로 VM을 build 하고, ssh 설정 및 클러스터 설치를 진행하였다.  
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