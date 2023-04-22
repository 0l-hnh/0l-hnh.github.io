---
layout: single
title:  "[Docker] Docker 강의 정리 (4) - 쿠버네티스 개념, CRI, 세팅 및 주요 오브젝트"
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
### 11. 쿠버네티스란?  
> 쿠버네티스란 각 컨테이너별 자원 제한, 문제발생시 자동시작 등 컨테이너를 배포/확장,제어,자동화 하기위한 다양한 기능을 지원하는 컨테이너 오케스트레이션 도구이다.  
\-출처 : [쿠버네티스 공식 문서](https://kubernetes.io/ko/)

Docker 로는 동일 장비에 존재하는 컨테이너만 node로 통신할 수 있기 때문에, 대규모 pods 관리에는 적합하지 않다.  
대규모 서비스 및 많은 장비를 사용하는 서비스 경우 '오케스트레이션' 이 중요한데, 이 역할로 쿠버네티스가 가장 보편적으로 사용된다.  
다만 쿠버네티스는 어렵고 완전히 마스터하기 쉽지 않다. 계속 공부를 해야 한다...  
본인이 실무 영역에 잘 적용하여 활용하는 것이 중요할 것 같다.  

### 12. 컨테이너 실행을 위한 런타임  
#### Docker 런타임  
Docker 런타임의 구성 요소 및 쿠버네티스 1.24부터 docker를 지원하지 않게 된 사유를 알아본다.  
Docker 런타임은 아래와 같이 이해할 수 있다.  
* docker engine: 도커 이미지를 관리하고 컨테이너 실행을 위해 containerd 데몬과 통신하여 runc 기반으로 컨테이너를 실행  
* containerd: Docker 에서 OCI spec 을 준수하여 개발한 container runtime high level container run time. 도커레지스트리에서 이미지를 가져오고, 도커 네트워크및 스토리지 관리 그리고 컨테이너 실행을 위해서 저수준 런타임인 runc 를 실행하고 관리 
* runc : container 를 생성하고 실행하는 low level 런타임, OCI runtime spec 준수  

런타임 레벨은 (High) containerd -> runc (Low) 이다.  

#### CRI, CRI-O
* CRI : 쿠버네티스에서 만든 API (컨테이너 런타임 인터페이스). 쿠버네티스가 각 런타임과 상호작용하는 방법을 설명하며, 주어진 컨테이너 런타임이 CRI API를 구현하면 런타임을 원하는대로 선택하여 컨테이너를 생성하고, 실행할 수 있음 (예제 : containerd, CRI-O 등)  
* CRI-O : Redhat, IBM 등이 개발한 쿠버네티스 전용 컨테이너 런타임으로, 컨테이너 생성 및 이미지 빌드 등을 할 수 없고, 실행에 중점을 둠.  
  
Docker는 CRI 표준을 준수하지 못 하기 때문에, 해당 인터페이스에 부합하는 Dockershim을 사용해야 kubelet에서 사용할 수 있다. 즉, kubelet에서 거쳐야하는 단계가 많다.  

> kubelet ↔ Dockershim ↔ Docker ↔ containerd → runc  

쿠버네티스는 CRI 표준에 맞지 않는 Docker 를 사용하기 위하여 1.23 버전까지 중간 단계를 거쳐서 runc를 실행하였지만, 1.24부터는 containerd에 CRI 플러그인을 사용하여 컨테이너를 실행하게 되었다.  

![에시이미지](https://i.ibb.co/BPwLYBv/2023-04-22-105639.png)  

단, 그 후 개발된 CRI-dockerd 를 컨테이너 런테임으로 사용하면 계속 docker를 사용할 수 있다. minikube의 경우 CRI-dockerd 를 사용한다.  
Docker를 대체할 수 있는 것 중 'podman'도 있다. Docker와 기본적인 명령어는 유사하며, RedHat 에서는 본인들이 개발한 Podman 사용을 권장한다.  

### 13. 쿠버네티스 (k8s) 세팅 
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
\-출처 : [쿠버네티스 공식 문서](https://kubernetes.io/ko/docs/concepts/workloads/pods/)  

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

### 14. 쿠버네티스 Pod 실행
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
apache가 실행된 상황에서 nginx를 하나 더 올려본다.  
```bash
$ kubectl run mynginx --image nginx:latest
$ kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
myapache   1/1     Running   0          86m   10.244.2.2   w2.example.com   <none>           <none>
mynginx    1/1     Running   0          43s   10.244.1.2   w1.example.com   <none>           <none>
```  
pods가 두 개 노드로 분배되었다.  

#### etcd, 스케쥴러, controller
쿠버네티스의 중요 컴포넌트들을 살펴본다.  
```bash
$ ps -ef | grep -w etcd | cat -n
     1  root        7284    7111  3 09:27 ?        00:05:01 etcd --advertise-client-urls=https://192.168.14.50:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --experimental-initial-corrupt-check=true --experimental-watch-progress-notify-interval=5s --initial-advertise-peer-urls=https://192.168.14.50:2380 --initial-cluster=m.example.com=https://192.168.14.50:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://192.168.14.50:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://192.168.14.50:2380 --name=m.example.com --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
     2  root        7319    7152  7 09:27 ?        00:09:15 kube-apiserver --advertise-address=192.168.14.50 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
$ ps -ef | grep -w kube-sched | cat -n
     1  vagrant    43783   12848  0 11:37 pts/9    00:00:00 grep --color=auto -w kube-sched
$ ps -ef | grep -w kube-controller-manaer | cat -n
     1  vagrant    44014   12848  0 11:38 pts/9    00:00:00 grep --color=auto -w kube-controller-manaer
```  

* etcd : 모든 클러스터 데이터를 저장하는 고가용성 키-값 저장소. node의 상태 (ex : pod의 실행 상태, 개수 등) 기록. 일종의 db 역할
* kube-scheduler : 생성된 pod를 node에 할당하는 컴포넌트. 가장 최적화 된 node에 pod를 배치하게 됨
* kube-controller manager : 컨트롤러 프로세스를 실행하고 클러스터의 실제 상태를 원하는 사양으로 조정
* kube-apiserver : 쿠버네티스 API 로, 외부/내부에서 관리자의 원격 명령을 받을 수 있는 컴포넌트. etcd, scheduler에서 결정된 node 에 신호를 보냄
* kubelet : docker 엔진 혹은 continaerd 등과 상호작용하여 컨테이너를 실행하며, 제공된 PodSpec 세트를 가져와 해당 컨테이너가 완전히 작동하는지 확인함

#### yaml 파일
쿠버네티스 관리 시에는 Menifest 파일이 필수이기 때문에 yaml 파일을 작성할 수 있어야 한다.  

### 15. 쿠버네티스 명령어와 주요 객체
쿠버네티스에서 사용하는 개념은 크게 '객체(Object)' 와 '컨트롤러(Controller)' 로 나눌 수 있다.  
(1) 객체는 사용자가 쿠버네티스에 바라는 상태(desired state)를 의미하며, (2) 컨트롤러는 객체가 원래 설정된 상태를 잘 유지할수있게 관리하는 역할이다.  

#### kubectl 실행 권한
kubectl 명령어가 root라도 바로 실행되지 않는다면, 권한 문제일 수도 있다.  
```bash
$ sudo su
(root)$ kubectl get nodes
E0422 12:02:20.048102   50984 memcache.go:265] couldn't get current server API group list: Get "http://localhost:8080/api?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```  
/etc/kubernetes/admin.conf 파일로 쿠버네티스 관리자 권한을 인증받아야 하는데, root 는 해당 파일이 PATH에 없기 때문이다. 해당 위치를 export 하면 아래와 같이 kubectl 사용이 가능하다.  
```bash
(root)$ export KUBECONFIG=/etc/kubernetes/admin.conf 
(root)$ echo $KUBECONFIG
/etc/kubernetes/admin.conf
$ kubectl get nodes
NAME             STATUS   ROLES           AGE     VERSION
m.example.com    Ready    control-plane   2d14h   v1.27.1
w1.example.com   Ready    <none>          2d14h   v1.27.1
w2.example.com   Ready    <none>          2d14h   v1.27.1
```  
단, 해당 방법으로도 일반 유저는 admin.conf를 참조할 수 없기 때문에 명령어 사용이 불가하다.  
```bash
# configuration for authorization to use kubecli command
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown vagrant:vagrant  /home/vagrant/.kube/config
$ echo "source <(kubectl completion bash)" >> ~/.bashrc #자동 완성 스크립트 만드는 부분
```
위와 같이 파일로 만들어서 유저 홈 디렉토리의 .bashrc 에 복사하면 사용 가능하다.  
자동 완성 기능을 사용하기 위해 만들어진 쉘 스크립트를 확인하고 싶다면, 아래와 같이 파일을 생성하여 볼 수 있다.  
```bash
$ kubectl completion bash > auto.sh
```  

#### Namespace  
##### Namespace 개념 및 관련 명령어
쿠버네티스에서 동일 namespace 에 이름이 동일한 pod을 생성하려고 하면 오류가 발생한다.  
이 때 -n 옵션으로 namespace 를 변경하면 생성할 수 있다.  
```bash
$ kubectl run myapache --image httpd:latest
Error from server (AlreadyExists): pods "myapache" already exists 
# ns로 현재 생성된 namespace를 확인한다.
$  kubectl get ns
NAME              STATUS   AGE
default           Active   2d14h
kube-flannel      Active   2d14h
kube-node-lease   Active   2d14h
kube-public       Active   2d14h
kube-system       Active   2d14h
$ kubectl run myapache --image httpd:latest -n kube-public
pod/myapache created
$ kubectl get pods -n kube-public 
NAME       READY   STATUS    RESTARTS   AGE
myapache   1/1     Running   0          55s
```
namespace로 동일 이름을 가진 pod이 격리되었다. 이런 방식으로 다양한 권한이 필요한 대규모 프로젝트 등을 관리할 수 있다.  
비유적으로 쉽게 생각하면, namespace는 일종의 pod를 담는 디렉토리라고 볼 수 있다.  

namespace 를 새로 만들어보자.  
```bash
$ kubectl create namespace myns
namespace/myns created
$ kubectl get ns
NAME              STATUS   AGE
default           Active   2d15h
kube-flannel      Active   2d15h
kube-node-lease   Active   2d15h
kube-public       Active   2d15h
kube-system       Active   2d15h
myns              Active   17s
$ kubectl config set-context --current --namespace myns
Context "kubernetes-admin@kubernetes" modified.
$ kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.14.50:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: myns
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
$ kubectl get pods
No resources found in myns namespace.
```  
성공적으로 생성이 되었고, 현재 namespace를 신규 생성한 namespace로 변경하였다. 신규 namespace로 생성된 pod는 없기 때문에, get pod 시에는 No resource 를 출력한다.  
namespace 를 삭제하고 싶을 때에는 delete를 사용한다.  
```bash
$ kubectl delete ns myns
namespace "myns" deleted 
```  
추가적으로 kubectx, kubens 를 설치하면 namespace를 쉽게 관리할 수 있다. 필수는 아니다.  

##### Name/space 메니페스트 파일
'*.yaml' 확장자를 사용하여 메니페스트 파일 작성 시 Namespace를 원하는 방식으로 생성할 수 있다.  

yaml 파일 작성 전에 api 버전을 확인한다. 
```bash
$ kubectl api-resources | grep -w ns
namespaces                        ns           v1                                     false        Namespace
```
v1이다. 해당 api 버전을 사용하여서 Namespace 파일을 작성해본다.   
```yaml
apiVersion: v1
kind: Namespace
metadata:
 name: testns 

```  
yaml 형식 파일은 대소문자 구분을 하며, 들여쓰기 등도 정확히 작성해야 한다. 작성을 마쳤으면 kubectl apply 로 실행한다.  
```bash
$ kubectl apply -f ns.yaml 
namespace/testns created
```  
간단한 네임스페이스가 생성되었다.  

'kubens' 등의 패키지를 설치하면 네임스페이스 변경을 명령어 한 줄로 실행 가능하다.  
```bash
$ kubens --help
Usage: 
                   kubens               : list the namespace 
                   kubens <NAME>        : change the active namespace 
                   kubens -c            : show the current namespace
                   kubens -             : switch to the previous namespace (This option is not work)
$ kubens testns
Context "kubernetes-admin@kubernetes" modified.
```  
default namespace 의 pod 들은 아래와 같이 확인한다. 
```bash
$ kubectl get pods -o wide -n default
NAME       READY   STATUS        RESTARTS   AGE     IP           NODE             NOMINATED NODE   READINESS GATES
myapache   1/1     Terminating   0          4h26m   10.244.2.2   w2.example.com   <none>           <none>
mynginx    1/1     Terminating   0          3h      10.244.1.2   w1.example.com   <none>           <none>
```

#### Pod  
##### Pod 메니페스트 파일 
Pod를 메니페스트 파일로 작성하여 실행하였다.  
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: apache-pod
  labels:
    app: myweb
    #labels is not essential
spec:
  containers:
  - name: myweb-container
    image: httpd:2.4
    ports:
    - containerPort: 80
```
위와 같이 apache에 대한 yaml 파일을 작성하고 아래와 같이 실행한다.  
```bash
$ kubectl apply -f apache.yaml
pod/apache-pod created
```
해당 pod에 문제가 있는지 확인하기 위해서는 kubectl describe 명령어를 사용한다.   
```bash
kubectl describe pod/apache-pod
Name:             apache-pod
Namespace:        testns
Priority:         0
Service Account:  default
Node:             <none>
Labels:           app=myweb
Annotations:      <none>
Status:           Pending
IP:               
IPs:              <none>
Containers:
  myweb-container:
    Image:        httpd:2.4
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-59wlv (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  kube-api-access-59wlv:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  79s (x17 over 81m)  default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 2 node(s) had untolerated taint {node.kubernetes.io/unreachable: }. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling..
```  
Pod를 실행하려고 하다가 계속 Pending 상태에서 넘어가지 않으며, 스케쥴링에 실패했다.  
worknode 의 가용 CPU 가 부족한 경우 해당 오류가 발생할 수 있다. 모든 pod를 삭제하고, work node로 사용하던 VM을 재기동한 뒤 원하는 pod를 다시 올려서 해결하였다.  
```bash
$ kubectl delete pods --all --grace-period=0 --force
$ kubectl create -f apache.yaml
$ kubectl get pod -o wide --show-labels
NAME         READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES   LABELS
apache-pod   1/1     Running   0          15m   10.244.2.3   w2.example.com   
$ curl 10.244.2.3
It works!
```  

##### Container Network Interface (CNI)
서로 다른 node 에 있는 pod을 연결하기 위해서는 CNI가 필요하다. 쿠버네티스에서 사용할 수 있는 CNI Network Plugin은  Flannel, Calico, Weavenet, NSX 등 여러 종류가 있다.  
CNI들은 결과적으로 같은 동작을 한다. 약간 차이가 있긴 하다. 에를 들어, calico 는 pod network 를 설정하지만, flannel은 하지 않는다는 차이가 있다.  

#### Service Object  
외부에서 쿠버네티스가 생성한 pod에 접속하려면 서비스 오브젝트가 있어야 한다. 서비스 오브젝트에 대한 메니페스트 yaml 파일을 아래와 같이 작성해 본다.  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myweb-service
spec:
  ports:
  - port: 8001
    targetPort : 80
  selector:
    app: myweb

```
서비스가 접속하기 위해서는 pod의 label이 필요하다는 사실을 새겨두자. (app에 해당 label을 입력한다.)  
그 뒤 kubectl 명령어로 create 한다.
```bash
$ kubectl create -f myweb-service.yaml
service/myweb-service created
$ kubectl apply -f myweb-service.yaml
$ kubectl get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP    29m
myweb-service   ClusterIP   10.105.77.228   <none>        8001/TCP   15m
$ curl http://10.105.77.228:8001
It works!
```
서비스가 생성되고, IP로 접속이 된다는 사실을 알 수 있다. 서비스는 패킷을 받아서 Endpoints 의 실행되는 pod들에 round robin 방식으로 데이터를 분산하여 처리하도록  한다. (멋지다)  
이 때, Cluster 안에서 직접 접속하는 것이 아니라 외부에서 접속하려면 서비스 Type을 변경할 필요가 있다.  
쿠버네티스의 경우 TYPE을 서비스 실행 중 변경할 수 있다. kubectl edit svc {service} 로 편집한다.  
```bash
$ kubectl edit service myweb-service 
service/myweb-service edited
$ kubectl get svc
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP          34m
myweb-service   NodePort    10.105.77.228   <none>        8001:30632/TCP   21m
```
NodePort 타입으로 변경되었다. NodePort 타입이면 윈도우 (외부)에서 접속할 수 있다. Node IP (클러스터의 IP)를 사용하고, Port 30632번을 사용하면 웹 페이지가 뜨는 것을 확인 가능하다.  
```bash
$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
:1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.14.50   m.example.com      m
192.168.14.51   w1.example.com     w1
192.168.14.52   w2.example.com     w2
```
이 경우 http://192.168.14.50:30632 로 접속이 가능하다. 라우팅 테이블로 바로 접속은 못 하지만 외부에 노출된 Node IP를 통해서 데이터 패킷을 보낼 수 있게 된다.  
한 가지 더, Network Load Balancer를 사용하여서 외부 접속 가능한 External-IP 를 설정할 수 있다. 이 경우는 가능하다는 것을 알아 두도록 한다.  
동일한 방식으로 서비스 객체를 통해 여러 개의 pod에 외부 웹페이지로 접속할 수 있다.  
```bash
$ kubectl get pod --show-labels
NAME         READY   STATUS    RESTARTS   AGE     LABELS
apache-pod   1/1     Running   0          78m     app=myweb
nginx-pod    1/1     Running   0          35m     app=mynginx
nginx-pod2   1/1     Running   0          3m13s   app=mynginx
nginx-pod3   1/1     Running   0          2m40s   app=mynginx
```
또, 위와 같이 같은 label을 가진 pod를 여러 대 실행한다면 round-robin 방식으로 패킷 전송이 된다. 위 상태에서 nginx-pod2, 3의 index.html 내용을 바꾼 뒤 서비스에 접속하면 변경된 내용이 번갈아 뜬다.  

#### Container 내부 접속
```bash
$ kubectl exec -it apache-pod -- /bin/bash
root@apache-pod:/usr/local/apache2#
```
오... pod의 컨테이너 내부로 접속 되었다. 멋지다. 이제 work node 에서 실행되는 어플리케이션을 중단하지 않고, master에서 바로 수정이 가능하다. 이를테면 index.html 파일을 수정할 수 있다.  

#### Multi Container Pod
간단한 multi-container pod를 작성하고 실행해보자.  
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - name: first
    image: httpd:2.4
  - name: second
    image: alpine:latest
    command: ["/bin/sleep", "3600s"]

```
apache 하나, alpine 하나다.  
```bash
$ kubectl apply -f multi-container.yaml
pod/test created
$ kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
apache-pod   1/1     Running   0          85m
test         2/2     Running   0          27s
$ kubectl describe pod test
Name:             test
Namespace:        default
Priority:         0
Service Account:  default
Node:             w1.example.com/192.168.14.51
Start Time:       Sat, 22 Apr 2023 16:58:05 +0900
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.1.6
IPs:
  IP:  10.244.1.6
Containers:
  first:
    Container ID:   containerd://96f5c57454904e824df14c0e3215a54da7f889300ab3bcb6a21399417322223c
    Image:          httpd:2.4
    Image ID:       docker.io/library/httpd@sha256:a182ef2350699f04b8f8e736747104eb273e255e818cd55b6d7aa50a1490ed0c
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 22 Apr 2023 16:58:08 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kzfth (ro)
  second:
    Container ID:  containerd://09c35c26844e2a8cfb41a39405d66e6a18f17655b163e03146e87371b772181f
    Image:         alpine:latest
    Image ID:      docker.io/library/alpine@sha256:124c7d2707904eea7431fffe91522a01e5a861a624ee31d03372cc1d138a3126
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sleep
      3600s
    State:          Running
      Started:      Sat, 22 Apr 2023 16:58:12 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-kzfth (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-kzfth:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  4m18s  default-scheduler  Successfully assigned default/test to w1.example.com
  Normal  Pulling    4m17s  kubelet            Pulling image "httpd:2.4"
  Normal  Pulled     4m15s  kubelet            Successfully pulled image "httpd:2.4" in 1.488389819s (1.488397759s including waiting)
  Normal  Created    4m15s  kubelet            Created container first
  Normal  Started    4m15s  kubelet            Started container first
  Normal  Pulling    4m15s  kubelet            Pulling image "alpine:latest"
  Normal  Pulled     4m11s  kubelet            Successfully pulled image "alpine:latest" in 4.393641963s (4.393647423s including waiting)
  Normal  Created    4m11s  kubelet            Created container second
  Normal  Started    4m11s  kubelet            Started container second
$ kubectl exec -it test -- /bin/bash
(apache)$ ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.1.6  netmask 255.255.255.0  broadcast 10.244.1.255
        inet6 fe80::3c33:33ff:fe7e:c10  prefixlen 64  scopeid 0x20<link>
        ether 3e:33:33:7e:0c:10  txqueuelen 0  (Ethernet)
        RX packets 780  bytes 8942110 (8.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 571  bytes 33172 (32.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0 
```
'READY'에 보이는 2/2가 2개 중 2개의 컨테이너의 상태를 의미한다.  생성된 'test'의 apache 컨테이너에 접속한 결과, pod 생성이 잘 되었고 첫 번째 컨테이너가 pod 와 ip가 동일하다는 사실을 확인할 수 있었다. 
두 번째 컨테이너인 alpine에도 접속 해본다.  
```bash
$ exec -it test -c second-- /bin/sh
(alpine)$ ifconfig
eth0      Link encap:Ethernet  HWaddr 3E:33:33:7E:0C:10  
          inet addr:10.244.1.6  Bcast:10.244.1.255  Mask:255.255.255.0
          inet6 addr: fe80::3c33:33ff:fe7e:c10/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:781 errors:0 dropped:0 overruns:0 frame:0
          TX packets:572 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:8942180 (8.5 MiB)  TX bytes:33242 (32.4 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
(alpine)$ curl http://10.244.1.6
It works!
```
접속이 잘 되었다. 이 때 eth0으로 접속이 되는 것은 당연하다. 그런데, 보통 본인 자신에게 loop가 되는 lo의 ip로 접속을 시도하면 무슨 일이 일어날까?  
```bash
$ curl http://127.0.0.1
It works!
```
같은 pod 안의 apache 서버 결과가 보인다. 같은 pod 안에서는 상호 접속이 가능하다는 사실을 알 수 있다.  

