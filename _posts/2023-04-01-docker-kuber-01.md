---
layout: single
title:  "[Docker] Docker 강의 정리 (1) - VM 세팅, Docker 설치"
date:   2023-04-01 10:10:00 +0900

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

4월 동안 Docker(도커), 쿠버네티스 관련 재직자 지원 강의를 총 5회 수강하게 되어 해당 내용을 정리하려 한다.  

## 2023-04-01 강의 노트  
### 00. OT
Docker (50%) / 쿠버네티스 (50%) 강의로, 아래와 같은 교재들을 활용하여 공부하기를 권장한다.  

![교재 표지](https://i.ibb.co/yQ21Hx8/2023-04-01-101012.png)

기본 교재로는 가장 좌측 교재를 사용한다. 표지의 고래가 귀엽다.  

* 강의 계획
  * 1주 ~ 3주 반 : Docker
  * 3주 반 ~ 5주 : 쿠버네티스  
* 짧은 시간 안에 쿠버네티스를 바로 마스터를 할 수는 없고, 꾸준히 현업에서 사용하는 것이 중요 
* docker / 쿠버네티스에서 공통적으로 활용하는 개념이 '컨테이너'인데, 해당 컨셉은 같은 IT, 개발 직군 안에서도 중요도가 다를 수 있음  
* 본인 업무에 맞추어서 적절하게 활용하는 게 중요  

### 01. 컨테이너의 이해
Docker에서 사용하는 컨테이너 기술을, 가상 머신과의 차이를 통해 이해한다.  

#### 가상 머신과 컨테이너
* 가상 머신과 컨테이너는, 하나의 하드웨어에서 여러 개의 애플리케이션을 마치 독립 환경에서 실행하는 것처럼 구동할 수 있게 한다는 공통점이 있음
* 하지만 가상 머신은 리소스를 많이 차지하기 때문에, 하드웨어 제한이 걸릴 수 있음
* 반면 컨테이너는 경량화가 되어 있고, 운영 체제에서 필요로 하는 것들(ex : 커널)을 설치하지 않아도 됨  
* 컨테이너의 기반 기술 : 리눅스 커널의 '네임스페이스'와 'cgroup' 

차이점을 아래 표로 정리하였다.  

|특징|가상 서버|컨테이너|
|------|------|------|
|이미지 크기 (CentOS 7.4의 경우)|최소 1.54 GB|최소 0.20GB|
|메모리 사용량|기본 640MB|기본 512MB|
|벤치 마크 성능 비교|65%(Xen HVM 가상 서버)|90%|
|OS 기동 시간|분 단위|초 단위|
|가상화|하드웨어 가상화|OS 가상화|
|가상화 소프트웨어|VMware, Xen, KVM 등|Docker 등|  

이전에도 컨테이너 기술은 있었지만, Docker 이후에 많이 쓰게 되었다.  

### 02. 실습 환경 구축  
컨테이너 기술이 리눅스를 기반으로 하다 보니, 리눅스 커널을 설치 해야 한다.  

#### Virtualbox, Vagrant 설치
Windows11 환경에서 Docker 설치 후 실습을 하기 위하여, 우선 VM으로 가상 머신을 설치한다.  
수업 자료로 제공 받은 Vagrant 이미지를 실행하여 Docker를 설치한다.  
나는 Local 환경에 Vagrant 가 없어서, 공식 주소를 참조하여 해당 소프트웨어를 설치한 뒤 실행했다. (I686은 32-bit, AMD64는 64-bit)

[vagrant 주소](https://www.vagrantup.com/)  

Vagrant는 gui 가 없으니, cmd를 열어서 명령어로 실행하면 된다.  
Vagrant에서 각 가상 환경을 'Box'라고 부르며, 필요에 의해 OS 체제를 설치하게 된다.  
Vagrantfile 예시는 아래와 같다.  

```ini
Vagrant.configure("2") do |config|
        config.vm.define "vm-name" do |cfg|
                cfg.vm.box = "centos/7"
                cfg.vm.provider "virtualbox" do |vb|
                        vb.name = "vm-name"
                        vb.cpus = 2
                        vb.memory = 2048
			vb.gui = true
                end
                cfg.vm.host_name = "serverx.example.com"
                cfg.vm.network "private_network", ip: "192.168.xx.xx"
                cfg.vm.provision "shell", path: "ssh_conf.sh", privileged: true
privileged: true
        end
end
```

Vagrant 이미지를 다운로드 받아서 VM으로 CentOS 7 환경을 실행하기 위한 cmd 전문은 아래와 같다.  

<details>
<summary>VM 생성 후 ssh 설정하는 로그 전문</summary>
<div markdown="1">

```bash
PS C:\Users\0lhnh\Desktop\WORKS\2023_Docker_Kuber> vagrant
Usage: vagrant [options] <command> [<args>]

    -h, --help                       Print this help.

Common commands:
     autocomplete    manages autocomplete installation on host
     box             manages boxes: installation, removal, etc.
     cloud           manages everything related to Vagrant Cloud
     destroy         stops and deletes all traces of the vagrant machine
     global-status   outputs status Vagrant environments for this user
     halt            stops the vagrant machine
     help            shows the help for a subcommand
     init            initializes a new Vagrant environment by creating a Vagrantfile
     login
     package         packages a running vagrant environment into a box
     plugin          manages plugins: install, uninstall, update, etc.
     port            displays information about guest port mappings
     powershell      connects to machine via powershell remoting
     provision       provisions the vagrant machine
     push            deploys code in this environment to a configured destination
     rdp             connects to machine via RDP
     reload          restarts vagrant machine, loads new Vagrantfile configuration
     resume          resume a suspended vagrant machine
     serve           start Vagrant server
     snapshot        manages snapshots: saving, restoring, etc.
     ssh             connects to machine via SSH
     ssh-config      outputs OpenSSH valid configuration to connect to the machine
     status          outputs status of the vagrant machine
     suspend         suspends the machine
     up              starts and provisions the vagrant environment
     upload          upload to machine via communicator
     validate        validates the Vagrantfile
     version         prints current and latest Vagrant version
     winrm           executes commands on a machine via WinRM
     winrm-config    outputs WinRM configuration to connect to the machine

For help on any individual command run `vagrant COMMAND -h`

Additional subcommands are available, but are either more advanced
or not commonly used. To see all subcommands, run the command
`vagrant list-commands`.
        --[no-]color                 Enable or disable color output
        --machine-readable           Enable machine readable output
    -v, --version                    Display Vagrant version
        --debug                      Enable debug output
        --timestamp                  Enable timestamps on log output
        --debug-timestamp            Enable debug output with timestamps
        --no-tty                     Enable non-interactive output
PS C:\Users\0lhnh\Desktop\WORKS\2023_Docker_Kuber> vagrant init
`Vagrantfile` already exists in this directory. Remove it before
running `vagrant init`.
PS C:\Users\0lhnh\Desktop\WORKS\2023_Docker_Kuber> vagrant up
Bringing machine 'dockerx' up with 'virtualbox' provider...
==> dockerx: Box 'centos/7' could not be found. Attempting to find and install...
    dockerx: Box Provider: virtualbox
    dockerx: Box Version: >= 0
==> dockerx: Loading metadata for box 'centos/7'
    dockerx: URL: https://vagrantcloud.com/centos/7
==> dockerx: Adding box 'centos/7' (v2004.01) for provider: virtualbox
    dockerx: Downloading: https://vagrantcloud.com/centos/boxes/7/versions/2004.01/providers/virtualbox.box
Download redirected to host: cloud.centos.org
    dockerx:
    dockerx: Calculating and comparing box checksum...
==> dockerx: Successfully added box 'centos/7' (v2004.01) for 'virtualbox'!
==> dockerx: Importing base box 'centos/7'...
==> dockerx: Matching MAC address for NAT networking...
==> dockerx: Checking if box 'centos/7' version '2004.01' is up to date...
==> dockerx: Setting the name of the VM: dockerx
==> dockerx: Clearing any previously set network interfaces...
==> dockerx: Preparing network interfaces based on configuration...
    dockerx: Adapter 1: nat
    dockerx: Adapter 2: hostonly
==> dockerx: Forwarding ports...
    dockerx: 22 (guest) => 2222 (host) (adapter 1)
==> dockerx: Running 'pre-boot' VM customizations...
==> dockerx: Booting VM...
==> dockerx: Waiting for machine to boot. This may take a few minutes...
    dockerx: SSH address: 127.0.0.1:2222
    dockerx: SSH username: vagrant
    dockerx: SSH auth method: private key
    dockerx: Warning: Connection reset. Retrying...
    dockerx: Warning: Connection aborted. Retrying...
    dockerx: Warning: Connection reset. Retrying...
    dockerx: Warning: Connection aborted. Retrying...
    dockerx: Warning: Remote connection disconnect. Retrying...
    dockerx: Warning: Connection reset. Retrying...
    dockerx:
    dockerx: Vagrant insecure key detected. Vagrant will automatically replace
    dockerx: this with a newly generated keypair for better security.
    dockerx:
    dockerx: Inserting generated public key within guest...
    dockerx: Removing insecure key from the guest if it's present...
    dockerx: Key inserted! Disconnecting and reconnecting using new SSH key...
==> dockerx: Machine booted and ready!
==> dockerx: Checking for guest additions in VM...
    dockerx: No guest additions were detected on the base box for this VM! Guest
    dockerx: additions are required for forwarded ports, shared folders, host only
    dockerx: networking, and more. If SSH fails on this machine, please install
    dockerx: the guest additions and repackage the box to continue.
    dockerx:
    dockerx: This is not an error message; everything may continue to work properly,
    dockerx: in which case you may ignore this message.
==> dockerx: Setting hostname...
==> dockerx: Configuring and enabling network interfaces...
==> dockerx: Rsyncing folder: /cygdrive/c/Users/0lhnh/Desktop/WORKS/2023_Docker_Kuber/ => /vagrant
==> dockerx: Running provisioner: shell...
    dockerx: Running: C:/Users/0lhnh/AppData/Local/Temp/vagrant-shell20230401-18544-1d9svca.sh
==> dockerx: Running provisioner: shell...
    dockerx: Running: C:/Users/0lhnh/AppData/Local/Temp/vagrant-shell20230401-18544-117onv3.sh
    dockerx: Loaded plugins: fastestmirror
    dockerx: Determining fastest mirrors
    dockerx:  * base: mirror.kakao.com
    dockerx:  * extras: mirror.kakao.com
    dockerx:  * updates: mirror.kakao.com
    dockerx: Resolving Dependencies
    dockerx: --> Running transaction check
    dockerx: ---> Package bridge-utils.x86_64 0:1.5-9.el7 will be installed
    dockerx: ---> Package net-tools.x86_64 0:2.0-0.25.20131004git.el7 will be installed
    dockerx: ---> Package vim-enhanced.x86_64 2:7.4.629-8.el7_9 will be installed
    dockerx: --> Processing Dependency: vim-common = 2:7.4.629-8.el7_9 for package: 2:vim-enhanced-7.4.629-8.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: 2:vim-enhanced-7.4.629-8.el7_9.x86_64
    dockerx: --> Processing Dependency: libperl.so()(64bit) for package: 2:vim-enhanced-7.4.629-8.el7_9.x86_64
    dockerx: --> Processing Dependency: libgpm.so.2()(64bit) for package: 2:vim-enhanced-7.4.629-8.el7_9.x86_64
    dockerx: --> Running transaction check
    dockerx: ---> Package gpm-libs.x86_64 0:1.20.7-6.el7 will be installed
    dockerx: ---> Package perl.x86_64 4:5.16.3-299.el7_9 will be installed
    dockerx: --> Processing Dependency: perl(Socket) >= 1.3 for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Scalar::Util) >= 1.10 for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl-macros for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(threads::shared) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(threads) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(constant) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Time::Local) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Time::HiRes) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Storable) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Socket) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Scalar::Util) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Pod::Simple::XHTML) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Pod::Simple::Search) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Getopt::Long) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Filter::Util::Call) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(File::Temp) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(File::Spec::Unix) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(File::Spec::Functions) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(File::Spec) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(File::Path) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Exporter) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Cwd) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: --> Processing Dependency: perl(Carp) for package: 4:perl-5.16.3-299.el7_9.x86_64
    dockerx: ---> Package perl-libs.x86_64 4:5.16.3-299.el7_9 will be installed
    dockerx: ---> Package vim-common.x86_64 2:7.4.629-8.el7_9 will be installed
    dockerx: --> Processing Dependency: vim-filesystem for package: 2:vim-common-7.4.629-8.el7_9.x86_64
    dockerx: --> Running transaction check
    dockerx: ---> Package perl-Carp.noarch 0:1.26-244.el7 will be installed
    dockerx: ---> Package perl-Exporter.noarch 0:5.68-3.el7 will be installed
    dockerx: ---> Package perl-File-Path.noarch 0:2.09-2.el7 will be installed
    dockerx: ---> Package perl-File-Temp.noarch 0:0.23.01-3.el7 will be installed
    dockerx: ---> Package perl-Filter.x86_64 0:1.49-3.el7 will be installed
    dockerx: ---> Package perl-Getopt-Long.noarch 0:2.40-3.el7 will be installed
    dockerx: --> Processing Dependency: perl(Pod::Usage) >= 1.14 for package: perl-Getopt-Long-2.40-3.el7.noarch
    dockerx: --> Processing Dependency: perl(Text::ParseWords) for package: perl-Getopt-Long-2.40-3.el7.noarch
    dockerx: ---> Package perl-PathTools.x86_64 0:3.40-5.el7 will be installed
    dockerx: ---> Package perl-Pod-Simple.noarch 1:3.28-4.el7 will be installed
    dockerx: --> Processing Dependency: perl(Pod::Escapes) >= 1.04 for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
    dockerx: --> Processing Dependency: perl(Encode) for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
    dockerx: ---> Package perl-Scalar-List-Utils.x86_64 0:1.27-248.el7 will be installed
    dockerx: ---> Package perl-Socket.x86_64 0:2.010-5.el7 will be installed
    dockerx: ---> Package perl-Storable.x86_64 0:2.45-3.el7 will be installed
    dockerx: ---> Package perl-Time-HiRes.x86_64 4:1.9725-3.el7 will be installed
    dockerx: ---> Package perl-Time-Local.noarch 0:1.2300-2.el7 will be installed
    dockerx: ---> Package perl-constant.noarch 0:1.27-2.el7 will be installed
    dockerx: ---> Package perl-macros.x86_64 4:5.16.3-299.el7_9 will be installed
    dockerx: ---> Package perl-threads.x86_64 0:1.87-4.el7 will be installed
    dockerx: ---> Package perl-threads-shared.x86_64 0:1.43-6.el7 will be installed
    dockerx: ---> Package vim-filesystem.x86_64 2:7.4.629-8.el7_9 will be installed
    dockerx: --> Running transaction check
    dockerx: ---> Package perl-Encode.x86_64 0:2.51-7.el7 will be installed
    dockerx: ---> Package perl-Pod-Escapes.noarch 1:1.04-299.el7_9 will be installed
    dockerx: ---> Package perl-Pod-Usage.noarch 0:1.63-3.el7 will be installed
    dockerx: --> Processing Dependency: perl(Pod::Text) >= 3.15 for package: perl-Pod-Usage-1.63-3.el7.noarch
    dockerx: --> Processing Dependency: perl-Pod-Perldoc for package: perl-Pod-Usage-1.63-3.el7.noarch
    dockerx: ---> Package perl-Text-ParseWords.noarch 0:3.29-4.el7 will be installed
    dockerx: --> Running transaction check
    dockerx: ---> Package perl-Pod-Perldoc.noarch 0:3.20-4.el7 will be installed
    dockerx: --> Processing Dependency: perl(parent) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
    dockerx: --> Processing Dependency: perl(HTTP::Tiny) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
    dockerx: ---> Package perl-podlators.noarch 0:2.5.1-3.el7 will be installed
    dockerx: --> Running transaction check
    dockerx: ---> Package perl-HTTP-Tiny.noarch 0:0.033-3.el7 will be installed
    dockerx: ---> Package perl-parent.noarch 1:0.225-244.el7 will be installed
    dockerx: --> Finished Dependency Resolution
    dockerx:
    dockerx: Dependencies Resolved
    dockerx:
    dockerx: ================================================================================
    dockerx:  Package                  Arch     Version                      Repository
    dockerx:                                                                            Size
    dockerx: ================================================================================
    dockerx: Installing:
    dockerx:  bridge-utils             x86_64   1.5-9.el7                    base       32 k
    dockerx:  net-tools                x86_64   2.0-0.25.20131004git.el7     base      306 k
    dockerx:  vim-enhanced             x86_64   2:7.4.629-8.el7_9            updates   1.1 M
    dockerx: Installing for dependencies:
    dockerx:  gpm-libs                 x86_64   1.20.7-6.el7                 base       32 k
    dockerx:  perl                     x86_64   4:5.16.3-299.el7_9           updates   8.0 M
    dockerx:  perl-Carp                noarch   1.26-244.el7                 base       19 k
    dockerx:  perl-Encode              x86_64   2.51-7.el7                   base      1.5 M
    dockerx:  perl-Exporter            noarch   5.68-3.el7                   base       28 k
    dockerx:  perl-File-Path           noarch   2.09-2.el7                   base       26 k
    dockerx:  perl-File-Temp           noarch   0.23.01-3.el7                base       56 k
    dockerx:  perl-Filter              x86_64   1.49-3.el7                   base       76 k
    dockerx:  perl-Getopt-Long         noarch   2.40-3.el7                   base       56 k
    dockerx:  perl-HTTP-Tiny           noarch   0.033-3.el7                  base       38 k
    dockerx:  perl-PathTools           x86_64   3.40-5.el7                   base       82 k
    dockerx:  perl-Pod-Escapes         noarch   1:1.04-299.el7_9             updates    52 k
    dockerx:  perl-Pod-Perldoc         noarch   3.20-4.el7                   base       87 k
    dockerx:  perl-Pod-Simple          noarch   1:3.28-4.el7                 base      216 k
    dockerx:  perl-Pod-Usage           noarch   1.63-3.el7                   base       27 k
    dockerx:  perl-Scalar-List-Utils   x86_64   1.27-248.el7                 base       36 k
    dockerx:  perl-Socket              x86_64   2.010-5.el7                  base       49 k
    dockerx:  perl-Storable            x86_64   2.45-3.el7                   base       77 k
    dockerx:  perl-Text-ParseWords     noarch   3.29-4.el7                   base       14 k
    dockerx:  perl-Time-HiRes          x86_64   4:1.9725-3.el7               base       45 k
    dockerx:  perl-Time-Local          noarch   1.2300-2.el7                 base       24 k
    dockerx:  perl-constant            noarch   1.27-2.el7                   base       19 k
    dockerx:  perl-libs                x86_64   4:5.16.3-299.el7_9           updates   690 k
    dockerx:  perl-macros              x86_64   4:5.16.3-299.el7_9           updates    44 k
    dockerx:  perl-parent              noarch   1:0.225-244.el7              base       12 k
    dockerx:  perl-podlators           noarch   2.5.1-3.el7                  base      112 k
    dockerx:  perl-threads             x86_64   1.87-4.el7                   base       49 k
    dockerx:  perl-threads-shared      x86_64   1.43-6.el7                   base       39 k
    dockerx:  vim-common               x86_64   2:7.4.629-8.el7_9            updates   5.9 M
    dockerx:  vim-filesystem           x86_64   2:7.4.629-8.el7_9            updates    11 k
    dockerx:
    dockerx: Transaction Summary
    dockerx: ================================================================================
    dockerx: Install  3 Packages (+30 Dependent packages)
    dockerx:
    dockerx: Total download size: 19 M
    dockerx: Installed size: 61 M
    dockerx: Downloading packages:
    dockerx: Public key for gpm-libs-1.20.7-6.el7.x86_64.rpm is not installed
    dockerx: warning: /var/cache/yum/x86_64/7/base/packages/gpm-libs-1.20.7-6.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
    dockerx: Public key for perl-Pod-Escapes-1.04-299.el7_9.noarch.rpm is not installed
    dockerx: --------------------------------------------------------------------------------
    dockerx: Total                                              1.6 MB/s |  19 MB  00:11
    dockerx: Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    dockerx: Importing GPG key 0xF4A80EB5:
    dockerx:  Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
    dockerx:  Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
    dockerx:  Package    : centos-release-7-8.2003.0.el7.centos.x86_64 (@anaconda)
    dockerx:  From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    dockerx: Running transaction check
    dockerx: Running transaction test
    dockerx: Transaction test succeeded
    dockerx: Running transaction
    dockerx:   Installing : 1:perl-parent-0.225-244.el7.noarch                          1/33
    dockerx:   Installing : perl-HTTP-Tiny-0.033-3.el7.noarch                           2/33
    dockerx:   Installing : perl-podlators-2.5.1-3.el7.noarch                           3/33
    dockerx:   Installing : perl-Pod-Perldoc-3.20-4.el7.noarch                          4/33
    dockerx:   Installing : 1:perl-Pod-Escapes-1.04-299.el7_9.noarch                    5/33
    dockerx:   Installing : perl-Encode-2.51-7.el7.x86_64                               6/33
    dockerx:   Installing : perl-Text-ParseWords-3.29-4.el7.noarch                      7/33
    dockerx:   Installing : perl-Pod-Usage-1.63-3.el7.noarch                            8/33
    dockerx:   Installing : 4:perl-macros-5.16.3-299.el7_9.x86_64                       9/33
    dockerx:   Installing : perl-Storable-2.45-3.el7.x86_64                            10/33
    dockerx:   Installing : perl-Exporter-5.68-3.el7.noarch                            11/33
    dockerx:   Installing : perl-constant-1.27-2.el7.noarch                            12/33
    dockerx:   Installing : perl-Socket-2.010-5.el7.x86_64                             13/33
    dockerx:   Installing : perl-Time-Local-1.2300-2.el7.noarch                        14/33
    dockerx:   Installing : perl-Carp-1.26-244.el7.noarch                              15/33
    dockerx:   Installing : perl-PathTools-3.40-5.el7.x86_64                           16/33
    dockerx:   Installing : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 17/33
    dockerx:   Installing : 1:perl-Pod-Simple-3.28-4.el7.noarch                        18/33
    dockerx:   Installing : perl-File-Temp-0.23.01-3.el7.noarch                        19/33
    dockerx:   Installing : perl-File-Path-2.09-2.el7.noarch                           20/33
    dockerx:   Installing : perl-threads-shared-1.43-6.el7.x86_64                      21/33
    dockerx:   Installing : perl-threads-1.87-4.el7.x86_64                             22/33
    dockerx:   Installing : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      23/33
    dockerx:   Installing : perl-Filter-1.49-3.el7.x86_64                              24/33
    dockerx:   Installing : 4:perl-libs-5.16.3-299.el7_9.x86_64                        25/33
    dockerx:   Installing : perl-Getopt-Long-2.40-3.el7.noarch                         26/33
    dockerx:   Installing : 4:perl-5.16.3-299.el7_9.x86_64                             27/33
    dockerx:   Installing : gpm-libs-1.20.7-6.el7.x86_64                               28/33
    dockerx:   Installing : 2:vim-filesystem-7.4.629-8.el7_9.x86_64                    29/33
    dockerx:   Installing : 2:vim-common-7.4.629-8.el7_9.x86_64                        30/33
    dockerx:   Installing : 2:vim-enhanced-7.4.629-8.el7_9.x86_64                      31/33
    dockerx:   Installing : bridge-utils-1.5-9.el7.x86_64                              32/33
    dockerx:   Installing : net-tools-2.0-0.25.20131004git.el7.x86_64                  33/33
    dockerx:   Verifying  : perl-HTTP-Tiny-0.033-3.el7.noarch                           1/33
    dockerx:   Verifying  : perl-threads-shared-1.43-6.el7.x86_64                       2/33
    dockerx:   Verifying  : perl-Storable-2.45-3.el7.x86_64                             3/33
    dockerx:   Verifying  : perl-Exporter-5.68-3.el7.noarch                             4/33
    dockerx:   Verifying  : perl-constant-1.27-2.el7.noarch                             5/33
    dockerx:   Verifying  : perl-PathTools-3.40-5.el7.x86_64                            6/33
    dockerx:   Verifying  : 4:perl-macros-5.16.3-299.el7_9.x86_64                       7/33
    dockerx:   Verifying  : 2:vim-enhanced-7.4.629-8.el7_9.x86_64                       8/33
    dockerx:   Verifying  : 1:perl-parent-0.225-244.el7.noarch                          9/33
    dockerx:   Verifying  : perl-Socket-2.010-5.el7.x86_64                             10/33
    dockerx:   Verifying  : 2:vim-filesystem-7.4.629-8.el7_9.x86_64                    11/33
    dockerx:   Verifying  : perl-File-Temp-0.23.01-3.el7.noarch                        12/33
    dockerx:   Verifying  : net-tools-2.0-0.25.20131004git.el7.x86_64                  13/33
    dockerx:   Verifying  : 1:perl-Pod-Simple-3.28-4.el7.noarch                        14/33
    dockerx:   Verifying  : perl-Time-Local-1.2300-2.el7.noarch                        15/33
    dockerx:   Verifying  : 1:perl-Pod-Escapes-1.04-299.el7_9.noarch                   16/33
    dockerx:   Verifying  : perl-Carp-1.26-244.el7.noarch                              17/33
    dockerx:   Verifying  : 2:vim-common-7.4.629-8.el7_9.x86_64                        18/33
    dockerx:   Verifying  : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 19/33
    dockerx:   Verifying  : bridge-utils-1.5-9.el7.x86_64                              20/33
    dockerx:   Verifying  : perl-Pod-Usage-1.63-3.el7.noarch                           21/33
    dockerx:   Verifying  : perl-Encode-2.51-7.el7.x86_64                              22/33
    dockerx:   Verifying  : perl-Pod-Perldoc-3.20-4.el7.noarch                         23/33
    dockerx:   Verifying  : perl-podlators-2.5.1-3.el7.noarch                          24/33
    dockerx:   Verifying  : 4:perl-5.16.3-299.el7_9.x86_64                             25/33
    dockerx:   Verifying  : perl-File-Path-2.09-2.el7.noarch                           26/33
    dockerx:   Verifying  : perl-threads-1.87-4.el7.x86_64                             27/33
    dockerx:   Verifying  : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      28/33
    dockerx:   Verifying  : gpm-libs-1.20.7-6.el7.x86_64                               29/33
    dockerx:   Verifying  : perl-Filter-1.49-3.el7.x86_64                              30/33
    dockerx:   Verifying  : perl-Getopt-Long-2.40-3.el7.noarch                         31/33
    dockerx:   Verifying  : perl-Text-ParseWords-3.29-4.el7.noarch                     32/33
    dockerx:   Verifying  : 4:perl-libs-5.16.3-299.el7_9.x86_64                        33/33
    dockerx:
    dockerx: Installed:
    dockerx:   bridge-utils.x86_64 0:1.5-9.el7
    dockerx:   net-tools.x86_64 0:2.0-0.25.20131004git.el7
    dockerx:   vim-enhanced.x86_64 2:7.4.629-8.el7_9
    dockerx:
    dockerx: Dependency Installed:
    dockerx:   gpm-libs.x86_64 0:1.20.7-6.el7
    dockerx:   perl.x86_64 4:5.16.3-299.el7_9
    dockerx:   perl-Carp.noarch 0:1.26-244.el7
    dockerx:   perl-Encode.x86_64 0:2.51-7.el7
    dockerx:   perl-Exporter.noarch 0:5.68-3.el7
    dockerx:   perl-File-Path.noarch 0:2.09-2.el7
    dockerx:   perl-File-Temp.noarch 0:0.23.01-3.el7
    dockerx:   perl-Filter.x86_64 0:1.49-3.el7
    dockerx:   perl-Getopt-Long.noarch 0:2.40-3.el7
    dockerx:   perl-HTTP-Tiny.noarch 0:0.033-3.el7
    dockerx:   perl-PathTools.x86_64 0:3.40-5.el7
    dockerx:   perl-Pod-Escapes.noarch 1:1.04-299.el7_9
    dockerx:   perl-Pod-Perldoc.noarch 0:3.20-4.el7
    dockerx:   perl-Pod-Simple.noarch 1:3.28-4.el7
    dockerx:   perl-Pod-Usage.noarch 0:1.63-3.el7
    dockerx:   perl-Scalar-List-Utils.x86_64 0:1.27-248.el7
    dockerx:   perl-Socket.x86_64 0:2.010-5.el7
    dockerx:   perl-Storable.x86_64 0:2.45-3.el7
    dockerx:   perl-Text-ParseWords.noarch 0:3.29-4.el7
    dockerx:   perl-Time-HiRes.x86_64 4:1.9725-3.el7
    dockerx:   perl-Time-Local.noarch 0:1.2300-2.el7
    dockerx:   perl-constant.noarch 0:1.27-2.el7
    dockerx:   perl-libs.x86_64 4:5.16.3-299.el7_9
    dockerx:   perl-macros.x86_64 4:5.16.3-299.el7_9
    dockerx:   perl-parent.noarch 1:0.225-244.el7
    dockerx:   perl-podlators.noarch 0:2.5.1-3.el7
    dockerx:   perl-threads.x86_64 0:1.87-4.el7
    dockerx:   perl-threads-shared.x86_64 0:1.43-6.el7
    dockerx:   vim-common.x86_64 2:7.4.629-8.el7_9
    dockerx:   vim-filesystem.x86_64 2:7.4.629-8.el7_9
    dockerx:
    dockerx: Complete!
    dockerx: Loaded plugins: fastestmirror
    dockerx: Loading mirror speeds from cached hostfile
    dockerx:  * base: mirror.kakao.com
    dockerx:  * extras: mirror.kakao.com
    dockerx:  * updates: mirror.kakao.com
    dockerx: Resolving Dependencies
    dockerx: --> Running transaction check
    dockerx: ---> Package yum-utils.noarch 0:1.1.31-53.el7 will be updated
    dockerx: ---> Package yum-utils.noarch 0:1.1.31-54.el7_8 will be an update
    dockerx: --> Finished Dependency Resolution
    dockerx:
    dockerx: Dependencies Resolved
    dockerx:
    dockerx: ================================================================================
    dockerx:  Package           Arch           Version                    Repository    Size
    dockerx: ================================================================================
    dockerx: Updating:
    dockerx:  yum-utils         noarch         1.1.31-54.el7_8            base         122 k
    dockerx:
    dockerx: Transaction Summary
    dockerx: ================================================================================
    dockerx: Upgrade  1 Package
    dockerx:
    dockerx: Total download size: 122 k
    dockerx: Downloading packages:
    dockerx: No Presto metadata available for base
    dockerx: Running transaction check
    dockerx: Running transaction test
    dockerx: Transaction test succeeded
    dockerx: Running transaction
    dockerx:   Updating   : yum-utils-1.1.31-54.el7_8.noarch                             1/2
    dockerx:   Cleanup    : yum-utils-1.1.31-53.el7.noarch                               2/2
    dockerx:   Verifying  : yum-utils-1.1.31-54.el7_8.noarch                             1/2
    dockerx:   Verifying  : yum-utils-1.1.31-53.el7.noarch                               2/2
    dockerx:
    dockerx: Updated:
    dockerx:   yum-utils.noarch 0:1.1.31-54.el7_8
    dockerx:
    dockerx: Complete!
    dockerx: Loaded plugins: fastestmirror
    dockerx: adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
    dockerx: grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
    dockerx: repo saved to /etc/yum.repos.d/docker-ce.repo
    dockerx: Loaded plugins: fastestmirror
    dockerx: Loading mirror speeds from cached hostfile
    dockerx:  * base: mirror.kakao.com
    dockerx:  * extras: mirror.kakao.com
    dockerx:  * updates: mirror.kakao.com
    dockerx: Resolving Dependencies
    dockerx: --> Running transaction check
    dockerx: ---> Package containerd.io.x86_64 0:1.6.20-3.1.el7 will be installed
    dockerx: --> Processing Dependency: container-selinux >= 2:2.74 for package: containerd.io-1.6.20-3.1.el7.x86_64
    dockerx: ---> Package docker-buildx-plugin.x86_64 0:0.10.4-1.el7 will be installed
    dockerx: ---> Package docker-ce.x86_64 3:23.0.2-1.el7 will be installed
    dockerx: --> Processing Dependency: docker-ce-rootless-extras for package: 3:docker-ce-23.0.2-1.el7.x86_64
    dockerx: --> Processing Dependency: libcgroup for package: 3:docker-ce-23.0.2-1.el7.x86_64
    dockerx: ---> Package docker-ce-cli.x86_64 1:23.0.2-1.el7 will be installed
    dockerx: --> Processing Dependency: docker-scan-plugin(x86-64) for package: 1:docker-ce-cli-23.0.2-1.el7.x86_64
    dockerx: ---> Package docker-compose-plugin.x86_64 0:2.17.2-1.el7 will be installed
    dockerx: --> Running transaction check
    dockerx: ---> Package container-selinux.noarch 2:2.119.2-1.911c772.el7_8 will be installed
    dockerx: --> Processing Dependency: policycoreutils-python for package: 2:container-selinux-2.119.2-1.911c772.el7_8.noarch
    dockerx: ---> Package docker-ce-rootless-extras.x86_64 0:23.0.2-1.el7 will be installed
    dockerx: --> Processing Dependency: fuse-overlayfs >= 0.7 for package: docker-ce-rootless-extras-23.0.2-1.el7.x86_64    dockerx: --> Processing Dependency: slirp4netns >= 0.4 for package: docker-ce-rootless-extras-23.0.2-1.el7.x86_64
    dockerx: ---> Package docker-scan-plugin.x86_64 0:0.23.0-3.el7 will be installed
    dockerx: ---> Package libcgroup.x86_64 0:0.41-21.el7 will be installed
    dockerx: --> Running transaction check
    dockerx: ---> Package fuse-overlayfs.x86_64 0:0.7.2-6.el7_8 will be installed
    dockerx: --> Processing Dependency: libfuse3.so.3(FUSE_3.2)(64bit) for package: fuse-overlayfs-0.7.2-6.el7_8.x86_64
    dockerx: --> Processing Dependency: libfuse3.so.3(FUSE_3.0)(64bit) for package: fuse-overlayfs-0.7.2-6.el7_8.x86_64
    dockerx: --> Processing Dependency: libfuse3.so.3()(64bit) for package: fuse-overlayfs-0.7.2-6.el7_8.x86_64
    dockerx: ---> Package policycoreutils-python.x86_64 0:2.5-34.el7 will be installed
    dockerx: --> Processing Dependency: setools-libs >= 3.3.8-4 for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: libsemanage-python >= 2.5-14 for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: --> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    dockerx: ---> Package slirp4netns.x86_64 0:0.4.3-4.el7_8 will be installed
    dockerx: --> Running transaction check
    dockerx: ---> Package audit-libs-python.x86_64 0:2.8.5-4.el7 will be installed
    dockerx: ---> Package checkpolicy.x86_64 0:2.5-8.el7 will be installed
    dockerx: ---> Package fuse3-libs.x86_64 0:3.6.1-4.el7 will be installed
    dockerx: ---> Package libsemanage-python.x86_64 0:2.5-14.el7 will be installed
    dockerx: ---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
    dockerx: ---> Package setools-libs.x86_64 0:3.3.8-4.el7 will be installed
    dockerx: --> Finished Dependency Resolution
    dockerx:
    dockerx: Dependencies Resolved
    dockerx:
    dockerx: ================================================================================
    dockerx:  Package                Arch   Version                   Repository        Size
    dockerx: ================================================================================
    dockerx: Installing:
    dockerx:  containerd.io          x86_64 1.6.20-3.1.el7            docker-ce-stable  34 M
    dockerx:  docker-buildx-plugin   x86_64 0.10.4-1.el7              docker-ce-stable  12 M
    dockerx:  docker-ce              x86_64 3:23.0.2-1.el7            docker-ce-stable  23 M
    dockerx:  docker-ce-cli          x86_64 1:23.0.2-1.el7            docker-ce-stable  13 M
    dockerx:  docker-compose-plugin  x86_64 2.17.2-1.el7              docker-ce-stable  12 M
    dockerx: Installing for dependencies:
    dockerx:  audit-libs-python      x86_64 2.8.5-4.el7               base              76 k
    dockerx:  checkpolicy            x86_64 2.5-8.el7                 base             295 k
    dockerx:  container-selinux      noarch 2:2.119.2-1.911c772.el7_8 extras            40 k
    dockerx:  docker-ce-rootless-extras
    dockerx:                         x86_64 23.0.2-1.el7              docker-ce-stable 8.8 M
    dockerx:  docker-scan-plugin     x86_64 0.23.0-3.el7              docker-ce-stable 3.8 M
    dockerx:  fuse-overlayfs         x86_64 0.7.2-6.el7_8             extras            54 k
    dockerx:  fuse3-libs             x86_64 3.6.1-4.el7               extras            82 k
    dockerx:  libcgroup              x86_64 0.41-21.el7               base              66 k
    dockerx:  libsemanage-python     x86_64 2.5-14.el7                base             113 k
    dockerx:  policycoreutils-python x86_64 2.5-34.el7                base             457 k
    dockerx:  python-IPy             noarch 0.75-6.el7                base              32 k
    dockerx:  setools-libs           x86_64 3.3.8-4.el7               base             620 k
    dockerx:  slirp4netns            x86_64 0.4.3-4.el7_8             extras            81 k
    dockerx:
    dockerx: Transaction Summary
    dockerx: ================================================================================
    dockerx: Install  5 Packages (+13 Dependent packages)
    dockerx:
    dockerx: Total download size: 109 M
    dockerx: Installed size: 384 M
    dockerx: Is this ok [y/d/N]: Is this ok [y/d/N]: Exiting on user command
    dockerx: Your transaction was saved, rerun it with:
    dockerx:  yum load-transaction /tmp/yum_save_tx.2023-04-01.02-25.nknwCQ.yumtx
    dockerx: Failed to start docker.service: Unit not found.
    dockerx: Failed to execute operation: No such file or directory
    dockerx: usermod: group 'docker' does not exist
The SSH command responded with a non-zero exit status. Vagrant
assumes that this means the command failed. The output for this command
should be in the log above. Please read the output to determine what
went wrong.
```
</div>
</details>  

새로 VM을 깔았으니 VM 내부 time-date 지역을 Asia/Seoul로 바꿔준다.  

```bash
sudo timedatectl set-timezone Asia/Seoul
```  

#### Docker 설치  
VM에 ssh로 접속하여 Docker를 설치한다.  

* 접속 방식으로는 putty, MobaXterm 등 편한 방식을 선택 (내 경우는 VsCode의 ssh 플러그 인을 사용하는 방식이 편해서, 해당 방법으로 VM에 접속하여 터미널 사용함)
* VM ip는 Vagrantfile에 정의되어 있으며, vm 내부에서 ip addr로 확인 가능 (eth1이 내부 접속용)
* 접속 후 ping 으로 외부 네트워크 사용 가능한지 확인 필요  

```bash
#! /bin/bash
# Install editor and utilities
$ yum -y install vim net-tools bridge-utils 

# set up the repository for docker install on centos7/8/9

$ yum install -y yum-utils 
$ yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker Engine
$ sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
$ systemctl start docker
$ systemctl enable docker

# setup enable docker permition to vagrant user
$ usermod -aG docker ${USER}
```

위의 설치 스크립트는 Docker 공식 사이트의 'CentOS'의 설치 가이드를 참고하였으며, 설치할 OS가 Ubuntu라면 keyrings 등을 통한 레포지토리 수정 등이 필요할 수 있다.  

[Docker 공식 설치 문서](https://docs.docker.com/engine/install/)

설치되는 docker 관련 주요 패키지는 아래와 같다.  

* docker-ce : docker engine
* docker-ce-cli : 명령어 패키지 (docker-e 는 enterprise 버전)

Docker 안에 Docker-compose가 포함되어 있다.  

### 03. Docker 실습
#### Docker 설치 패키지 및 실행 방식 
* 주요 개념 : docker, containerd, runc
* runc에서의 명령어가 dockerd로 전달되고, docker 서버가 올라갈 때 containerd (종속되어 있음)가 실행됨
* 컨테이너가 이미 실행되어 있으면 runc는 필요가 없으니까 내려감  
* prep -fl 명령어로 확인 가능

다음은 예시이다.    
```bash
$ prep -fl docker
3638 dockerd
```  

(이 때 'dockerd = docker 서버 = docker 엔진' 이다.)  

Docker 실행 시 권한 이슈가 발생한다면, 원인은 dockerd 실행 프로세스에 있다  
docker socekt에 접근을 해야 하는데, 이 권한이 없어서 못 하는 것 (위치 : /var/run/docker.sock)  
현재 사용 중인 계정에 docker group 권한을 주면 sudo 없이 실행 가능하다. 

다음은 예시이다.    
```bash
$ sudo usermod -aG docker $(USER)
```  

#### Docker 이미지 실행  
* Docker image 저장소  
Docker image는 기본적으로 docker hub에서 pull 하여 사용하지만, 다른 사이트 들도 사용 가능  
(예시 : [Docker Hub](https://hub.docker.com/), [AWS Gallery](https://gallery.ecr.aws/) )  
* Docker image는 레이어 구조로 되어 있음
* Docker image는 기능적으로 필요한 바이너리 파일들로 구성되어 있으며, 커널 기능은 포함하지 않음

예시로, 웹 서버인 아파치를 받아왔다.  
```bash
$ docker pull httpd:2.4
```  

그리고 실행한다..  

```bash
$ docker run httpd:2.4
2.4: Pulling from library/httpd
f1f26f570256: Pull complete 
a6b093ae1967: Pull complete 
6b400bbb27df: Pull complete 
d9833ead928a: Pull complete 
ace056404ed3: Pull complete 
Digest: sha256:f3e9eb9acace5bbc13c924293d2247a65bb59b8f062bcd98cd87ee4e18f86733
Status: Downloaded newer image for httpd:2.4
docker.io/library/httpd:2.4
```  

레이어 구조를 갖는 docker 이미지가 pull 되는 것을 확인할 수 있다.  
단, 그냥 run 하면 컨테이너가 foreground로 실행이 되며 dockerd 가 실행되는 동안 터미널을 쓸 수 없게 된다.  
Background 로 컨테이너를 실행하고 싶다면, run 시 -d 옵션 (=dettach)을 준다.  

```bash 
$ docker run -d httpd:2.4
d808b0592440312cd945e16ba848af202d9aaefb6257f03cb9c97d3accb67298
```  

표준 입출력이 뜨지 않으며 background로 실행이 되었다.  

```bash
$ docker exec ${CONTAINER} ll
```  

컨테이너에 명령어를 던져보자.  
컨테이너 안에서 실행되는 프로세스는 컨테이너 밖에서도 볼 수 있고, 컨테이너 밖에서 kill 할 수도 있다.  

Docker Apache 안에서 확인하면, 커널이 없다.  
ls 등 명령어 실행 시 커널은 컨테이너 외부 로컬 환경에서 처리한다.  
단 바이너리 파일은 컨테이너 내부에 있는 것을 참조하여, 환경 별로 서로 격리된다.  
따라서 다른 환경의 컨테이너를 하나의 로컬에서 띄워도 OS의 라이브러리를 쓰지 않기 때문에, 충돌이 나지 않고 배포에도 용이하다.  

컨테이너도 기본적으로 LAN 카드를 가지고 있어서, 만약 컨테이너 내부에서 필요한 게 있다면 설치해서 쓸 수 있다.  

```bash
$ docker -it exec ${CONTAINER} /bin/bash
```  

컨테이너의 상태 전이를 도식화 하면 아래 이미지와 같다.  

![컨테이너의 상태 전이](https://i.ibb.co/6ybbRZ8/2023-04-01-160101.png)  

#### Docker 기본 명령어

```bash
# 컨테이너 실행
$ docker run ${image}

# 컨테이너 중지
$ docker stop ${container}

# 컨테이너 삭제
$ docker rm $(docker ps -aq)
$ docker rm ${container}

# 컨테이너 실행
$ docker exec ${container}

# 컨테이너 내부 설정 확인
$ docker inspect ${container, image}
```  
#### Docker 저장 공간 (volume)
* docker run 시 -v 옵션으로 외부 디렉토리를 마운트 하면 컨테이너 내부 / 외부에서 공통으로 참조함  
   
아래 예시는 mysql 이미지를 실행했을 때, 컨테이너 내부에서 생성한 db를 보존하도록 실행 환경의 디렉토리를 사용하는 방법이다.  

```bash 
$ docker run -d --env MYSQL_ROOT_PASSWORD=pass -v /home/vagrant/docker-kuber/mydb/:/var/lib/mysql --name mydb mysql:5.7
$ docker exec -it mydb /bin/bash

#mydb container 내부
$ mysql -p
(mysql) create database mydb;
(mysql) use mydb;
(mysql) create table t1
    -> (name varchar(20),
    -> id char(10));
(mysql) insert into t1 values('hong kildong', 'kildong');
```  

컨테이너 종료하여도, 동일한 디렉토리를 mount 하여 이미지를 실행한다면 db를 동일하게 사용할 수 있다.  

#### Docker 네트워크
* Docker 포트를 22로 쓰면, 현재 ssh로 쓰고 있는 포트와 충돌이 나 오류 발생
* 8000번 이후로 설정

```bash
$ docker run -d --name myhttpd -p 8000:80 httpd:2.4
$ ip addr show eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:67:37:d4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.51.10/24 brd 192.168.51.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe67:37d4/64 scope link 
       valid_lft forever preferred_lft forever

$ sudo iptables -L -t nat -n | grep 8000
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8000 to:172.17.0.5:80
```  

192.168.51.10:8000 으로 접속하면 외부에서 해당 포트에 열린 도커 컨테이너(이 경우 아파치 서버)를 확인 가능하다.