---
layout: post
title: "Centos7 minimal 설치 후 작업"
date: 2018-06-24
categories: linux
---

Centos7 minimal 버전을 설치하고 기본적으로 꼭 필요한 추가 작업을 진행해보자. 여기서는 보통 virtualbox를 통해 테스트용으로 선택했을 것이다. 따라서 이 문서에서는 이를 배경으로 설명하겠다. 일단 설치과정은 너무 쉽고 또 쉽게 찾을 수 있으므로 여기선 생략하겠다.

### 1. 네트워크 설정

네트워크설정 목적에 따르다. 가상머신을 여러대 설치하고 가상머신끼리 통신하고 호스트에서 가상머신별로 접속이 가능하게 하려면 Nat네트워크 + 호스트전용네트워크 조합으로 가면 된다. 여기서는 그렇게 가정하고 진행하겠다. 

대메뉴의 `Virtualbox -> 설정 -> 네트워크` 메뉴로 들어가서 추가해주자. 추가이미지 버턴을 누르면 알아서 디폴트로 생성해준다.

![centosmini1](/assets/images/centosmini1.jpg)

다음은 호스트전용네트워크를 추가해주다. 이것은 가상머신 갯수만큼 생성해주면 된다.  대메뉴의 `파일 -> 호스트네트워크관리자`메뉴로 들어가서 추가하면 된다. 

![centosmini2](/assets/images/centosmini2.jpg)

DHCP서버는 사용함으로 설정하자. 

이제 centos7 minimal이 설치된 가상머신을 선택하고 `설정 -> 네트워크`으로 들어가자.

먼저 어댑터1에 Nat 네트워크를 설정하고 위에서 설정한 네트워크명을 지정하자. 

![centosmini3](/assets/images/centosmini3.jpg)

그리고 어댑터2에 호스트전용네트워크를 선택하고 등록된 호스트전용네트워크를 지정해주자. 

![centosmini4](/assets/images/centosmini4.jpg)

이제 접속해서 ip를 확인해보자(ip a, ip address)

![centosmini5](/assets/images/centosmini5.jpg)

### 2. yum 업데이트 및 openssh-sever 설치

Root로 접속하고 `yum upate` 를 쳐보자. 만약 아래와 같은 에러가 나온다면

![centosmini6](/assets/images/centosmini6.jpg)

nameserver 설정이 안되서 그런것이다. 직접 `vi /etc/resolv.conf ` 로 설정하는 방법(nameserver 168.126.63.1)도 있지만 간단하게 `dhclient` 명령어를 치면 알아서 위에 설정을 해결해준다. 

그럼 다음 아래 설정정보를 열고 맨 하단에 `ONBOOT=yes` 로 변경해야 리부팅될때 다시 적용된다.  

```bash 
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

다시 yum update를 치면 잘 되는 것을 확인할 수 있다. `upgrade`도 해주고 이제 `ssh server` 를 설치해보자 

```
yum install -y openssh-server
```

그리고 호스트네임도 변경해보자

```
hostnamectl set-hostname centos7_1
```

로그아웃하고 다시 로그인하면 아래처럼 호스트명이 변경되어 있다. 이렇게 하면 로컬콘솔에서 작업시 로컬과 헷갈리지 않게 된다.

```
[root@centos_7]#
```

이제 로컬에서 ssh로 접속(`ssh root@192.168.56.3`)하면 된다.

### 3. 방화벽 포트 열기

centos7에 현재 방화벽 상태를 확인해보자. 기본적으로 우리가 설치할 프로그램의 포트를 열어주는 작업을 해주어야 한다. 

``` 
firewall-cmd --state
```

`running`이라고 표시가 될 것이다. 만약 8080포트를 허용하려면 아래와 같이 입력한다

```
firewall-cmd --permanent --zone=public --add-port=8080/tcp
```

다시 제거할 경우 `--add` 부분을 `--remove` 로 하면 된다. 추가나 변경이 이루어지면 방화벽을 재시작해야 한다.

```
firewall-cmd --reload
```

이제 용도에 맞게 추가적인 작업을 진행하면 된다. 
