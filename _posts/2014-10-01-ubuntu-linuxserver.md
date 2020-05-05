---
layout: post
title:  "Ubuntu에서 linux-server 의존성 에러발생시"
date:   2014-10-01
categories: linux
---
Ubuntu에서 의존성 에러시 sudo apt-get -f install로 해도 실패가 나는 경우가 있다.

바로 아래 같은 경우이다.

```bash
$> sudo apt-get -f install
패키지 목록을 읽는 중입니다... 완료
의존성 트리를 만드는 중입니다
상태 정보를 읽는 중입니다... 완료
의존성을 바로잡는 중입니다... 완료
다음 패키지를 더 설치할 것입니다:
linux-server
다음 패키지를 업그레이드할 것입니다:
linux-server
1개 업그레이드, 0개 새로 설치, 0개 제거 및 0개 업그레이드 안 함.
1개를 완전히 설치하지 못했거나 지움.
0 바이트/1,734 바이트 아카이브를 받아야 합니다.
이 작업 후 0 바이트의 디스크 공간을 더 사용하게 됩니다.
계속 하시겠습니까 [Y/n]? y

dpkg: 의존성 문제로 linux-server을(를) 설정할 수 없습니다:
linux-server 패키지는 다음 패키지에 의존: linux-image-server (= 3.2.0.59.70): 하지만:
시스템에 있는 linux-image-server의 버전은 3.2.0.69.82입니다.
linux-server 패키지는 다음 패키지에 의존: linux-headers-server (= 3.2.0.59.70): 하지만:
시스템에 있는 linux-headers-server의 버전은 3.2.0.69.82입니다.
dpkg: linux-server을(를) 처리하는데 오류가 발생했습니다 (--configure):  

의존성 문제 - 설정하지 않고 남겨둠
보고서를 작성하지 않습니다. 오류 메시지에 따르면 예전의 실패 때문에 생긴 부수적인 오류입니다.
처리하는데 오류가 발생했습니다:  

linux-server
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

>해결방법 : 시스템에 있는  linux-image-server의 버전을 다시받아서 설치하면 된다.
(자신의 시스템의 linux-server 버전명에 맞게 설치한다)

```bash
wget https://launchpad.net/ubuntu/+archive/primary/+files/linux-server_3.2.0.69.82_amd64.deb
sudo dpkg -i linux-server_3.2.0.69.82_amd64.deb
```

참고
- <http://askubuntu.com/questions/252704/i-cannot-install-any-package-linux-image-server-linux-server-dependencies-erro>
