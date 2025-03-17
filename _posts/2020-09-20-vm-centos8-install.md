---
layout: single
title: "Vritual Box에서 CentOS minimal Install"
date: 2020-09-20
categories: [linux]
tags: [linux]
---

Centos에서 [CentOS-8.2.2004-x86_64-minimal.iso](http://mirror.kakao.com/centos/8.2.2004/isos/x86_64/CentOS-8.2.2004-x86_64-minimal.iso) 를 다운받는다.

미니멀버전을 설치하는 이유는 vm에서 좀더 가볍게 사용하기 위함이다. 굳이 UI(X-Window)를 사용하지 않고 터미널작업만 한다면 미니멀 버전이 적격이다.

### 1. virtaul box 환경설정

Virtaul box를 실행하고 환경설정에서 NatNetwokr를 추가해주자. NatNetwork를 추가하는 이유는 추후에 여러개의 vm를 추가할 경우 vm간의 통신을 하기 위함이다.

![centos8m1](/assets/images/centos8m1.png)

호스트 네트워크 관리자에 네트워크 어댑터가 등록되어 있는지 확인하고 없다면 만들기를 통해 생성해준다. 이 네트워크 어댑터는 vm이 추가될때마다 사용되는 어댑터이다. 아래 주소 `192.168.56.1` 를 갈무리하자.

![centos8m2](/assets/images/centos8m2.png)

다음은 환경설정에서 가상머신 호스트 키 조합을 기존의 Right Control를 적당한 것으로 변경해주자. 이렇게 하지 않으면 가상머신의 내의 키보드와 호스트간의 키보드 전환시 Right Control 없는 키보드가 있어 가상호스트에 키보드가 갇히게 된다. 만약 설정을 잊어서 갇힌 상태라면 `Ctrl + Alt + Del `키로 나오면 된다.

![centos8m3](/assets/images/centos8m3.png)

기본 설정은 끝났다. 이제 위에서 받은 centos를 설치해보다.

### 2. virtau box에서 centos 설치

새로만들기를 클릭하고 가상 머신 만들기 에서 이름과 저장될 위치를 적절하게 기재한다.

![centos8m4](/assets/images/centos8m4.png)

메모리는 적당히 로컬사양에 맞게 기재하고 별다른 설정없이 다음을 계속 클릭하자. 마지막 만들기까지 클릭하면 아래처럼 구성된다.

![centos8m5](/assets/images/centos8m5.png)

시작버튼을 클릭하면 시동디스크를 선택해야 한다. 다운로드 받은 centos iso파일을 선택해주자.

![centos8m6](/assets/images/centos8m6.png)

그리고 나서 시작을 누른다.

![centos8m7](/assets/images/centos8m7.png)

키보드 방향키로 Install CentOS Linux 8을 선택하고 엔터를 치자. 설정이 진행되면서 아래와 같은 화면이 나온다. 이때 설치언어를 선택하면 호스트키 정보가 나온다. 1에서 우리가 설정흔 Right Shift가 호스트키이다. vm에서 호스트로 키보드를 전환하려면 Right Shift 를 눌러야 한다. `이 메시지를 다시 표시하지 않기`를 체크하고 잡기를 클릭하여 한국어를 설정하자.

![centos8m8](/assets/images/centos8m8.png)

설치 요약 화면에서 설치목적지에 빨간표시가 나오니 클릭하고 그냥 변경없이 완료를 클릭하자.

![centos8m9](/assets/images/centos8m9.png)

아래처럼 설치시작 버튼이 활성화되면 진행을 하도록 한다.

![centos8m10](/assets/images/centos8m10.png)

Root암호와 유저생성을 권장하면서 설치가 진행된다. 필요에 맞게 설정해주면 된다.

![centos8m11](/assets/images/centos8m11.png)

설치가 완료되면 재부팅 버튼이 표시되는데 그대로 재부팅되면 다시 인스톨 화면이 나오기 때문에 아래와 같이 디스크를 빼주는 작업을 하고

vm을 닫고 다시 시작하면 로그인으로 이동하게 된다.

![centos8m12](/assets/images/centos8m12.png)

### 3. CentOS 8에서 설정작업

인터넷 연결이 되기 위해 네트워크 설정을 해야한다.

```bash
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

ONBOOT=no를 yes로 변경한다. reboot해준다.

yum update가 잘 되는지 확인하고 업데이트를 해준다. openssh-server는 이미 설치되어 있다. 확인하기 위해서 아래 명령어를 치면된다.

```bash
systemctl status sshd
```

![centos8m13](/assets/images/centos8m13.png)

방화벽이 가동중이니 ssh포트를 추가해주자.

```bash
firewall-cmd --permanent --zone=public --add-port=22/tcp
firewall-cmd --reload
```

가상호스트의 아이피를 조회해본다.

```bash
hostname -I
```

10.0.2.15가 표시되었다(각 호스트마다 다르다)

### 4. 포트포워딩

이 주소로 포트포워딩을 하도록 하자.

파일->환경설정->네트워크->NatNetwork에서 네트워크정보 -> 포트포워딩

![centos8m14](/assets/images/centos8m14.png)

위에서 호스트 네트워크 관리자에 네트워크 어댑터로 설정된 아이피로 포트포워딩을 해주자.

![centos8m15](/assets/images/centos8m15.png)

가상호스트에서 설정을 클릭한다.

![centos8m18](/assets/images/centos8m18.png)

네트워크에서 어댑터1, 어댑터2를 각각 아래와 같이 설정한다.

![centos8m18](/assets/images/centos8m19.png)

![centos8m18](/assets/images/centos8m20.png)

가상호스트를 실행한다.

이제 Putty를 통해 설정해주면 정상적으로 접속됨을 확인할 수 있다.

![centos8m16](/assets/images/centos8m16.png)

필수사항은 아니지만 hostname을 설정하면 여러개의 vm사용시 putty를 사용할 경우 편리하다.

```bash
hostnamectl set-hostname centos8_1
```

최종 완료된 모습이다.

![centos8m17](/assets/images/centos8m17.png)
