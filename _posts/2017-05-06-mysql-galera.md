---
layout: post
title:  "MariaDB Galera, Maxscale 설치"
date:   2017-05-06
categories: database
---

Mariadb에서 Galera와 Maxscale를 설치해본다. 

### 1. Galera 설치

Galera를 사용하려면 최소 3대의 장비가 필요하다. 테스트는 아래 장비로 구성한다. 

| server name | ip           |
| ----------- | ------------ |
| server1     | 192.168.0.40 |
| server2     | 192.168.0.41 |
| server3     | 192.168.0.42 |

먼저 server1에서 my.cnf를 보면 [galera]부분이 보인다.초기는 모두 주석으로 되어 있다.3대의 클러스터가 서로 연결될 수 있는 계정을 만들어준다.

```
GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '1234' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON *.* TO cluster_user@'%' IDENTIFIED BY '1234' WITH GRANT OPTION;
```

**[Server1 세팅]**

```
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_address=gcomm://
binlog_format=row
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
#
# Allow server to accept connections on all interfaces.
#
bind-address=0.0.0.0
#
# Optional setting
#wsrep_slave_threads=1
#innodb_flush_log_at_trx_commit=0

#추가 
wsrep_provider_options="gcache.size=512M"
wsrep_cluster_name=test_cluster
wsrep_node_name=server1
wsrep_sst_method=rsync
```

wsrep_cluster_address에 최초서버에는 클러스트된 서버가 없으므로 기본으로 설정해야 한다. 추후에 다른 서버랑 클러스트되면 그때 수정한다. 저장을 하고 mariadb를 중지시키고 아래 옵션으로 재시작한다.

```
sudo service mysql stop
sudo service mysql start --wsrep_cluster_new
```

옵션 `—wsrep_cluster_new` 는 최초 1회만 수행하면 된다. 

**[Server2 세팅]**

server2에서 같은 설정을 하고 경로만 수정한다.

```
wsrep_cluster_address=gcomm://192.168.0.40,192.168.0.41
wsrep_node_name=server2
wsrep_sst_receive_address=192.160.0.41:4569
```

**[Server3 세팅]**

```
wsrep_cluster_address=gcomm://192.168.0.40,192.168.0.41,192.168.0.42
wsrep_node_name=server2
wsrep_sst_receive_address=192.160.0.41:4569
```

**[재시작]**

Server2, Server3각각의 서버를 재시작한다. 

```
sudo service mysql stop
sudo service mysql start (옵션필요없음)
```

세 개의 서버가 정상적으로 작성되면 각각의 my.cnf파일을 열어서 `wsrep_cluster_address`를 동일하게 수정한다. 이렇게 되면 만약 서버가 재시작될경우 작성된 아이피로 클러스트링이 이루어진다.

```
wsrep_cluster_address=gcomm://192.168.0.40,192.168.0.41,192.168.0.42
```

**[확인]**

세팅이 모두 끝났으면 server1에서 해당 쿼리를 실행해보자..

```
SHOW STATUS LIKE 'wsrep%';
```

그러면 아래와 같이 나오면 정상적으로 세팅된 것이다. 

| Variable_name            | Value                                |
| ------------------------ | ------------------------------------ |
| wsrep_cluster_conf_id    | 3                                    |
| wsrep_cluster_size       | 3                                    |
| wsrep_cluster_state_uuid | 00be20a6-28be-11e7-9400-678af9ffc349 |
| wsrep_cluster_status     | Primary                              |
| wsrep_connected          | ON                                   |



**[에러발생시]**

1) 에러

```
Apr 24 16:12:40 ubuntu1 mysqld[8844]: 2017-04-24 16:12:40 140660766763264 [ERROR] WSREP: It may not be safe to bootstrap the cluster from this node. It was not the last one to leave th
Apr 24 16:12:40 ubuntu1 mysqld[8844]: 2017-04-24 16:12:40 140660766763264 [ERROR] WSREP: wsrep::connect(gcomm://) failed: 7
Apr 24 16:12:40 ubuntu1 mysqld[8844]: 2017-04-24 16:12:40 140660766763264 [ERROR] Aborting
```

Galera관련 파일을 삭제한다.

```
sudo rm -rf galera.cache grastate.dat wsrep_sst_binlog.tar 
```

2) 에러 

```
Apr 24 16:40:15 ubuntu1 mysqld[10977]: 2017-04-24 16:40:15 139623460747520 [ERROR] WSREP: failed to open gcomm backend connection: 110: failed to reach primary view: 110 (Connection ti
Apr 24 16:40:15 ubuntu1 mysqld[10977]:          at gcomm/src/pc.cpp:connect():158
Apr 24 16:40:15 ubuntu1 mysqld[10977]: 2017-04-24 16:40:15 139623460747520 [ERROR] WSREP: gcs/src/gcs_core.cpp:gcs_core_open():208: Failed to open backend connection: -110 (Connection 
Apr 24 16:40:15 ubuntu1 mysqld[10977]: 2017-04-24 16:40:15 139623460747520 [ERROR] WSREP: gcs/src/gcs.cpp:gcs_open():1380: Failed to open channel 'test_cluster' at 'gcomm://192.168.0.4
Apr 24 16:40:15 ubuntu1 mysqld[10977]: 2017-04-24 16:40:15 139623460747520 [ERROR] WSREP: gcs connect failed: Connection timed out
Apr 24 16:40:15 ubuntu1 mysqld[10977]: 2017-04-24 16:40:15 139623460747520 [ERROR] WSREP: wsrep::connect(gcomm://192.168.0.40,192.168.0.41,192.168.0.42) failed: 7
Apr 24 16:40:15 ubuntu1 mysqld[10977]: 2017-04-24 16:40:15 139623460747520 [ERROR] Aborting
```

해결방법

```
sudo galera_new_cluster
```

### 2. Galera의 사용시 유의점 

Galera설치후에 auto_increment가 증가하는 문제가 있다.
즉, 같은  인서트문을 3번 수행하면  auto_increment가 5번부터 시작일때  5,6,7으로 되지 않고 아래처럼 증가한다. 

| id   | val  |
| ---- | ---- |
| 5    | val1 |
| 8    | val2 |
| 13   | val3 |
| 18   | val4 |

한마디로 auto_increment가 +1이 아닌 노드 갯수만큼 증가한다. 물론 이와 같은 것을 방지하기 위해서는 아래 옵션을 줘서 해결할 수 있다. 

```
[galera]
wsrep_auto_increment_control=off
```

문제는 이와 같이만 설정하면 데드락 문제가 발생한다. 관련되서는 아래서 블로그를 참조하자.

[MariaDB / Galera Cluster / 데드락이 생긴다!?](https://trazy.gitbooks.io/dbms/content/mariadb-galera-deadlock.html)

### 3. Maxscale 설치 

문제를 해결하기 위해서 우리는 [Maxscale](https://mariadb.com/products/mariadb-maxscale) 를 사용하도록 한다. 설치는 쉽다. 홈페이지에서 다운받고 패키지를 설치하면 된다. 우분투에서는 다음과 같이 설치하도록 하자. 

```
sudo dpkg -i maxscale-2.0.5-1.ubuntu.xenial.x86_64.deb
```

그럼 다음 각각의 maxscale에서 각각의 mariadb를 접속하기 위해서 계정을 만들어주자. 

```
GRANT ALL PRIVILEGES ON *.* TO maxscale@'%' IDENTIFIED BY '1234' WITH GRANT OPTION;
```

/etc/maxscale.cnf 에서 파일을 수정하자

```
[maxscale]
#threads=1
threads=4

# Server definitions
#
# Set the address of the server to the network
# address of a MySQL server.
#

[server1]
type=server
address=192.168.0.40
port=3306
protocol=MySQLBackend

[server2]
type=server
address=192.168.0.41
port=3306
protocol=MySQLBackend

[server3]
type=server
address=192.168.0.42
port=3306
protocol=MySQLBackend

# Monitor for the servers
#
# This will keep MaxScale aware of the state of the servers.

# MySQL Monitor documentation:
# https://github.com/mariadb-corporation/MaxScale/blob/master/Documentation/Monitors/MySQL-Monitor.md

#[MySQL Monitor]
#type=monitor
#module=mysqlmon
#servers=server1
#user=myuser
#passwd=mypwd
#monitor_interval=10000

[Splitter Service]
type=service
router=readwritesplit
servers=server1,server2,server3
user=maxscale
passwd=1234


[Galera Monitor]
type=monitor
module=galeramon
servers=server1,server2,server3
#use_priority=true
user=maxscale
passwd=1234


# Service definitions
#
# Service Definition for a read-only service and
# a read/write splitting service.
#

[Splitter Listener]
type=listener
service=Splitter Service
protocol=MySQLClient
port=4040
socket=/tmp/ClusterMaster

[MaxAdmin Listener]
type=listener
service=MaxAdmin Service
protocol=maxscaled
socket=default


[CLI]
type=service
router=cli

[CLI Listener]
type=listener
service=CLI
protocol=maxscaled
address=localhost
port=6603

```

설정이 되면 `service maxscale start` 를 실행시키면 된다.  

 