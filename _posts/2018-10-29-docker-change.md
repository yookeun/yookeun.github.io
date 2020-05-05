---
layout: post
title:  "Docker에서 /var/lib 변경"
date:   2018-10-29
categories: docker
---

우분투나 데비안계열에서 도커 설치시 컨테이너 저장경로가 디폴트로 `/var/lib` 안에 설정된다. 여기 파티션크기가 적다면 용량문제에 접하게 되거나 root안에 있다면 os재설치시 문제가 생긴다. 

따라서 docker 설치후에 위 저장경로를 변경하는 것을 설명한다.
docker를 apt를 통해 설치했을 경우로 가정한다. 도커설정은 변경은 2가지로 처리할 수 있다.

### 1. 도커 실행 서비스에서 설정변경 

`/lib/systemd/system/docker.service` 파일을 열고 아래 내용을 수정한다.

```
#ExecStart=/usr/bin/dockerd -H fd://
ExecStart=/usr/bin/dockerd -g /home/ykkim/docker -H fd://
```

기존에 ExecStart 부분에 docker 데이터가 저장되는 위치를 지정한다. 
> -g /도커데이터 경로

수정이 되면 도커를 중지시킨다. 

```
sudo systemctl stop docker
sudo systemctl daemon-reload
```

도커 데이터가 저장될 경로를 먼저 만들어주어야 한다.
```
mkdir /home/ykkim/docker
```

그런 다음 기존에 docker 경로를 위 경로로 통째로 복제해준다.
```
sudo rsync -aqxP /var/lib/docker /home/ykkim/docker
```

이제 도커를 실행한다
```
sudo systemctl start docker
```

도커를 명령어를 실행하고 (sudo docker ps) docker를 찾아보면 -g 옵션 다음에 우리가 지정한 경로로 나오는 것을 확인할 수 있다. 

```
ykkim@ykkim-ubuntu:~$ ps -ef | grep docker
root     17887     1  1 10:22 ?        00:00:00 /usr/bin/dockerd -g /home/ykkim/docker -H fd://
```

### 2. 도커 설정파일을 통해서 변경 

이번에는 도커에서 설정파일을 통해 변경하는 법을 알아보자 

`deamon.json` 파일을 만들어준다. 이 파일은 기본적으로 존재하지 않는다. 따라서 해당 경로 별도로 만들어주어야 한다. 
```
vi /etc/docker/daemon.json
```
변경될 경로를 넣어준다. 이렇게 되면 도커가 실행할때 저 파일을 읽어들이게 된다. 
```
{
    "graph": "/home/ykkim/docker"
}
```
저장을 하고 도커를 재시작한다. 
```
sudo systemctl restart docker
```
컨테이너를 실행하고 도커를 확인(`ps -ef | grep docker` )하면 아래처럼 변경된 경로로 나오는 것을 확인할 수 있다.


```
root     21800 19983  0 10:52 ?        00:00:00 docker-containerd-shim -namespace moby -workdir /home/ykkim/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/8adc00f3bbdd4fd44110f42a7b25e8598019963a83b6b8d3d29266c426701db8 -address /var/run/docker/containerd/docker-containerd.sock -containerd-binary /usr/bin/docker-containerd -runtime-root /var/run/docker/runtime-runc
```

이렇게 해서 디렉토리 변경을 처리할 수 있다. 