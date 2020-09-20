---
layout: post
title:  "AWS IoT Greengrass와 라즈베리파이 설정"
date:   2020-09-08
categories: aws
---

AWS IoT Greengrass에서 Raspberry Pi를  연결하는 세팅을 해본다.

사전에 라즈베리에 라즈베리안 os와 ssh기능이 활성화 되어야 하고 인터넷연결이 가능한 상태이어야 한다.

### 1. 유저/그룹생성 

``` bash 
sudo adduser --system ggc_user
sudo addgroup --system ggc_group
```

### 2.  hardlink 및 softlink(symlink) 보호를 활성화

``` bash 
cd /etc/sysctl.d
ls
sudo vi 98-rpi.conf
```

98-rpi.conf 마지막 라인에 아래 라인 추가 

``` bash 
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
```

저장후 라즈베리파이 재부팅한다. 

``` bash 
sudo reboot
```

### 3.  Lambda 함수에 대한 메모리 제한 설정

``` bash
cd /boot/
sudo vi cmdline.txt
```

새줄이 아니라 기존줄의 맨끝에 한칸 띄우고 다음을 추가한다.

``` bash 
cgroup_enable=memory cgroup_memory=1
```

다시 리부팅한다.

### 4. 필요한 설정이 모두 되어 있는지 체크 

``` bash 
cd /home/pi/Downloads
mkdir greengrass-dependency-checker-GGCv1.10.x
cd greengrass-dependency-checker-GGCv1.10.x
wget https://github.com/aws-samples/aws-greengrass-samples/raw/master/greengrass-dependency-checker-GGCv1.10.x.zip
unzip greengrass-dependency-checker-GGCv1.10.x.zip
cd greengrass-dependency-checker-GGCv1.10.x
sudo modprobe configs
sudo ./check_ggc_dependencies | more
```

선택사항에 대한 Missing optional dependencies: 에러는 무시해도 된다.



### 5. Greengrass 그룹작업 

그룹생성한다.

![aws1](/assets/images/aws-iot-hello1.png)

기본 생성 사용을 사용한다.

![aws2](/assets/images/aws-iot-hello2.png)

그룹명을 지정해주자.

![aws3](/assets/images/aws-iot-hello3.png)

그룹 함수에 대한 코어가 만들어진다. 다음을 클릭하자.

![aws4](/assets/images/aws-iot-hello4.png)

그룹 및 코어 생성 클릭.

![aws5](/assets/images/aws-iot-hello5.png)

보안 리소스를 다운받아야 한다. 나중에 받을 수 없으니 지금 받아야 한다.

그런 다음에 루트 CA선택은 라즈베리파이에서 나중에 받도록 하겠다.

![aws6](/assets/images/aws-iot-hello6.png)

### 6. AWS IoT Greengrass 코어 소프트웨어 설치

위에서 받은 보안리소스 파일 (xxx-setup.tar.gz)와 root.ca.pem 파일을  라즈베리파이 /home/pi/Downloads에 업로드한다.

라즈베리파이 콘솔로 접속하여 아래 경로로 이동하자. /greengrass로 압축해제 한다. 

``` bash 
cd /home/pi/Downloads
```

``` bash 
sudo mkdir -p /greengrass
sudo tar -xzvf xxx-setup.tar.gz -C /greengrass
```

루트 CA인증서를 다운로드 받는다

``` bash 
cd /greengrass/certs/
sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
```

리포지토리를 추가한다.

``` bash 
wget -O aws-iot-greengrass-keyring.deb https://d1onfpft10uf5o.cloudfront.net/greengrass-apt/downloads/aws-iot-greengrass-keyring.deb
sudo dpkg -i aws-iot-greengrass-keyring.deb
echo "deb https://dnw9lb6lzp2d8.cloudfront.net stable main" | sudo tee /etc/apt/sources.list.d/greengrass.list
```

코어스프트웨어를 설치한다

``` bash 
sudo apt update
sudo apt install aws-iot-greengrass-core
```

실행하고 확인한다.

``` bash 
sudo systemctl start greengrass.service
sudo systemctl status greengrass.service
```

정상 여부 확인하고 /greengrass에 가보면 config, gcc라는 디렉토리가 생성됨을 확인할 수 있다.  모든 설정이 끝났다. 

