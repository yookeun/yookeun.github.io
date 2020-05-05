---
layout: post
title: "카프카(Kafka)를 설치(Install)해보자"
date: 2018-07-01
categories: kafka
---

### 1 . 환경

카프카를 설치하는 환경은 Virtualbox를 통해 Centos7에 설치를 하겠다. 기본적으로 클러스트를 구성하기 위해서 3대로 설치가  진행된다. 카프카를 설치할 경우 주키퍼(Zookeeper)도 설치해야 하는데 보통 주키퍼3대, 카프카3대로 별도로 구성하는데 여기서는 그냥 테스트용도이므로 가상머신 3대에 주키퍼랑 카프카를 같이 설치 진행한다. 

버츄얼박스에 Centos7 설치는 여기서는 생략한다. 기본적으로 Centos7에 OpenJDK8 혹은 Oracle JDK8 등 `Java8` 버전이 설치되어 있어야 한다. 이것도 역시 생략한다. 

### 2. 설치전 작업 (root)

먼저 자바등이 모두 설치되었다면 각 서버의 호스트명을 지정하자. 필수조건은 아닌데. 설정해두면 좀 편하다.

```bash 
hostnamectl set-hostname kafka1(호스트명) # 변경후 재로그인 
```

여기서는 각각의 서버에 kafka1, kafka2, kafka3 으로 호스트명을 입력했다.  그리고 hosts 파일을 열어 수정하자

```bash
vi /etc/hosts
```

```bash
0.0.0.0 kafka1  
192.168.57.3 kafka2
192.168.58.3 kafka3
```

자기 자신은 `0.0.0.0` 으로 호스트명과 세팅하고 다른 서버는 각각 아이피를 등록하여 세팅한다. 

다음은 주키퍼와 카프카를 위한 방화벽설정을 해주자. 사용하는 포트를 추가하면 된다. 

```bash 
## 주키퍼 포트 
firewall-cmd --permanent --zone=public --add-port=2181/tcp
firewall-cmd --permanent --zone=public --add-port=2888/tcp
firewall-cmd --permanent --zone=public --add-port=3888/tcp

## 카프카 포트
firewall-cmd --permanent --zone=public --add-port=9092/tcp

## 방화벽 재시작 
firewall-cmd --reload
```

이제 설치전 작업은 끝났다. 본격적으로 설치를 진행하자. 주키퍼와 카프카를 설치할 때는 root계정으로 해도 되지만, 여기서는 사용자계정(ykkim)을 별도로 만들어 실행했다. 

### 3. 주기퍼(Zookeeper) 설치

[Zookeeper](http://apache.tt.co.kr/zookeeper/stable/) 를 다운로드 하고 나서 압축을 풀면 된다. 적당한 곳에 풀자.

```bash
tar zxf zookeeper-3.4.12.tar.gz 
ln -s zookeeper-3.4.12 zookeeper
```

각각의 서버에 주기퍼 노드를 구분하기 위한 id가 필요하다 (루트 계정일 경우에는 mkdir -p /data)

```bash 
mkdir data

#(1: kafka1, 2: kafka2, 3: kafka3)
echo 1 > data/myid  
```

각각 서버의 id는 아래와 같이 구성한다. 
- kafka1 -> myid: 1
- kafka2 -> myid: 2
- kafka3 -> myid: 3

설정파일을 만들자. `zookeeper/conf` 안에 `zoo_sample.cfg`가 있으니 `zoo.cfg`로 복사해서 사용하자.

```bash 
cd zookeeper/conf
cp zoo_sample.cfg zoo.cfg
vi zoo.cfg
```

아래 처럼 주석을 풀고 추가작성을 하도록 한다. 3개의 서버에 모두 같은 설정이다. 

```bash 
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
#dataDir=/tmp/zookeeper
dataDir=/home/ykkim/data

# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=kafka1:2888:3888
server.2=kafka2:2888:3888
server.3=kafka3:2888:3888
```

이제 실행시켜보자.

```bash 
./zookeeper/bin/zkServer.sh start
```

정상적이라면 아래처럼 메시지가 출력된다.

 ```bash
ZooKeeper JMX enabled by default
Using config: /home/ykkim/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
 ```

세개 모두 정상적으로 가동하면 `zookeeper/zookeeper.out` 로그에  표시되는지 확인해본다. 

서비스등록을 위해 종료하자. 서비스를 등록하여 시스템 재시작시 자동으로 가동토록 해주자. 

```bash 
./zookeeper/bin/zkServer.sh stop
```

 `zookeeper-server.service`라는 스크립트를 만들고 서비스에 등록하자(root작업)

```bash 
 vi /etc/systemd/system/zookeeper.service
```

```bash 
[Unit]
Description=zookeeper
After=network.target

[Service]
Type=forking
User=ykkim
Group=ykkim
SyslogIdentifier=zookeeper
WorkingDirectory=/home/ykkim/zookeeper
Restart=always
RestartSec=0s
ExecStart=/home/ykkim/zookeeper/bin/zkServer.sh start
ExecStop=/home/ykkim/zookeeper/bin/zkServer.sh stop

[Install]
WantedBy=multi-user.target
```

저장하고 나서 서비스를 재시작하고 주키퍼를 실행하고 정상적으로 실행되면 시스템부팅시 자동실행 설정을 지정하자.

```bash 
# 서비스 데몬 재시작
systemctl daemon-reload
# 주키퍼 실행 (종료는 stop, 재시작은 restart)
systemctl start zookeeper-server.service 
# 실행상태 확인
systemctl status zookeeper-server.service
# 시스템 부팅할때 자동실행 설정
systemctl enable zookeeper-server.service
```

다음은 카프카를 설치해보자.

### 4. 카프카(Kafka) 설치

[Kafka](https://kafka.apache.org/downloads) 를 다운로드 받는다. 적당한 위치에 압축을 풀면 된다. 카프카도 별도서버에 설치하면 위에 /data/myid 파일을 만들고 그안에 숫자를 넣으면 되는데 여기선 주키퍼랑 같이 쓰니까 그걸 그대로 사용한다. 다만 data1, data2라는 디렉토리를 만들 것이고 이것은 카프카에 메시지가 저장되는 장소이다. 하나로 만들수 있지만 분산 저장을 테스트하기 위해서 두개의 디렉토리를 만들었다. 운영에서는 별도의 파티션으로 구분해서 만드는 것이 좋겠다. 여기서는 그냥 사용자 홈디렉토리에 만들겠다.

```bash 
mkdir data1, data2
```

이제 카프카 설정파일(`/kafka/config/server.properties`)을 수정하자. 

```bash 
vi kafka/config/server.properties
```

아래 내용을 찾아서 수정해주자. 

```bash

############################# Server Basics #############################
#서버 data/myid에 값으로 각각 세팅
broker.id=1 

############################# Logs Basics #############################
## 카프카 메시지저장분산디렉토리 
log.dirs=/home/ykkim/data1,/home/ykkim/data2

############################# Zookeeper #############################
## 주기퍼 연결설정
## 서버1호스트명:서버1포트,서버2호스트명:서버2포트,서버3호스트명:서버2포트/주기퍼노드명
zookeeper.connect=kafka1:2181,kafka2:2181,kafka3:2181/ykkim-kafka

```
`zookeeper.connect` 에서 마지막 `/ykkim-kafka`는 주기퍼노드명이다. 작성하지 않으면 주기퍼 루트노드에 저장된다. 그렇게 되면 관리하기가 어려우므로 이렇게 별도로 노드명을 기재해준다. 이제 실행해보자. 

```bash 
./kafka/bin/kafka-server.start.sh /home/ykkim/kafka/config/server.properties
```

정상적으로 뜨면 꽤 많은 메시지가 나오면서 그중에 서버아이디가 표시된 메시지가 보일 것이다. 

```bash 
....중략....
[KafkaServer id=1] started (kafka.server.KafkaServer)
```

서비스 등록을 위해 종료하자.

```bash 
/home/ykkim/kafka/bin/kafka-server-stop.sh
```

마지막으로 `kafka-server.service`라는 스크립트를 만들고 서비스에 등록하자.

```bash 
vi /etc/systemd/system/kafka.service
```

```bash
[Unit]
Description=kafka
After=network.target

[Service]
Type=simple
User=ykkim
Group=ykkim
SyslogIdentifier=kafka
WorkingDirectory=/home/ykkim/kafka
Restart=always
RestartSec=0s
ExecStart=/home/ykkim/kafka/bin/kafka-server-start.sh /home/ykkim/kafka/config/server.properties
ExecStop=/home/ykkim/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```

저장하고 나서 서비스를 재시작하고 카프카를 실행하고 정상적으로 실행되면 시스템부팅시 자동실행 설정을 지정하자.

```bash 
# 서비스 데몬 재시작
systemctl daemon-reload 
# 카프카 실행
systemctl start kafka.service 
# 실행 상태 확인
systemctl status kafka.service
# 시스템 부팅할때 자동실행 설정
systemctl enable kafka.service
```

이것으로 설치가 완료되었다. 테스트를 진행해보자. 

### 5. 테스트

먼저 주기퍼와 카프카가 정상적으로 포트가 오픈되어 있는지 확인한다

```bash 
### 주키퍼 포트확인
netstat -ntlp | grep 2181

### 카프카 포트확인
netstat -ntlp | grep 9092
```

포트가 정상적으로 `LISTEN` 상태이면 주기퍼 지노드를 이용해서 카프크 정보를 확인해보겠다.

```bash 
./zookeeper/bin/zkCli.sh
```

주키퍼로 접속한 다음 `ls /` 를 치면 기본 노드와 우리가 추가한 노드가 보일 것이다.

```bash 
[zookeeper, ykkim-kafka]
```

카프카 클러스터 노드들이 잘 연결되었는지 확인한다.

```bash 
ls /ykkim-kafka/brokers/ids

##출력결과 
[1, 2, 3]
```

 잘되었으면 `quit` 로 빠져나온다. 다음은 카프카에 토픽을 생성하도록 하자. `ykkim-topic` 이라는 토픽을 생성한다. 토픽은 카프카에서 `kafka-topics.sh` 를 이용해서 생성한다. 

```bash 
./kafka/bin/kafka-topics.sh --zookeeper kafka1:2181,kafka2:2181,kafka3:2181/ykkim-kafka --replication-factor 1 --partitions 1 --topic ykkim-topic --create
```

그런 다음  카프카 프로듀서(kafka-console-producer.sh)를 통해서 접속한다. 

```bash 
./kafka/bin/kafka-console-producer.sh --broker-list kafka1:9092,kafka2:9092,kafka3:9092 --topic ykkim-topic
```

엔터를 치고 기다려면 > 프롬포트가 깜빡인다. 그러면 메시지를 입력하자. 

```bash 
## 입력
> Hello Kafka1
> Hello Kafka2
```

그리고 나서 카프카 컨슈머(kafka-console-consumer.sh)에서 해당 토픽에 메시지를 읽어보도록 하자. 

```bash 
./kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092 --topic ykkim-topic --from-beginning
```

엔터를 치면 프로듀서에 입력된 값이 컨슈머에서 읽어오는 것을 확인할 수 있다. 다른 클러스트에서도 확인이 가능하다. 

```bash 
## 출력 
Hello Kafka1
Hello Kafka2
```

이것으로 설치와 테스트가 모두 완료 되었다. 

[참고서적]

> 카프카, 데이터 플랫폼의 최강자
>
> 고승범 저/공용준 공저 ( 출판사 : 책만 ) 

