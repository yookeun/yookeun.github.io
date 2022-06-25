---
layout: post
title:  "elasticsearch 7.6 수동설치 TLS 설정"
date:   2020-02-22
categories: elasticsearch
---


> 로컬 경로 : /home/ykkim/elastic/elasticsearch-7.6.0

### 1. tmp 폴더생성

```
mkdir tmp
cd tmp
mkdir cert
```


### 2. instance.yml 생성 

``` yaml
instances:
    - name: 'node-1'
      ip: ['192.168.1.158']
```

### 3. 인증서 생성 

```
bin/elasticsearch-certutil cert ca --pem --in tmp/cert/instance.yml --out tmp/cert/certs.zip
```

### 4. 인증서 압축해제 

```
cd tmp/certs
unzip certs.zip -d ./certs
```

### 5. cert파일을 config 폴더로 복사 

```
cd config
mkdir certs
cd certs
cp ../../tmp/cert/certs/ca/ca.crt ../../tmp/cert/certs/node-1/* .
```

``` 
ls 
-rw-r--r-- 1 ykkim ykkim 1.2K  2월 14 15:47 ca.crt
-rw-r--r-- 1 ykkim ykkim 1.2K  2월 14 15:47 node-1.crt
-rw-r--r-- 1 ykkim ykkim 1.7K  2월 14 15:47 node-1.key
```

### 6. application.yml 설정 

``` yaml
node.name: node-1
network.host: 192.168.1.158
xpack.security.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.http.ssl.key: certs/node-1.key
xpack.security.http.ssl.certificate: certs/node-1.crt
xpack.security.http.ssl.certificate_authorities: certs/ca.crt
xpack.security.transport.ssl.key: certs/node-1.key
xpack.security.transport.ssl.certificate: certs/node-1.crt
xpack.security.transport.ssl.certificate_authorities: certs/ca.crt
discovery.seed_hosts: [ "192.168.1.158" ]
cluster.initial_master_nodes: [ "node-1"] 
```

### 7. 시작 
```
/bin/elasticsearch
```
### 8. 비번설정 
```
bin/elasticsearch-setup-passwords auto -u "https://192.168.1.158:9200"
```

```
Please confirm that you would like to continue [y/N]y


Changed password for user apm_system
PASSWORD apm_system = KwGA9NiowJiwle9PQHqo

Changed password for user kibana
PASSWORD kibana = cZAdtghFDIfdgU03tDzP

Changed password for user logstash_system
PASSWORD logstash_system = tKoKvBacSFPjEKdShmfY

Changed password for user beats_system
PASSWORD beats_system = 52tCYGE846D0BtvtGsXU

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = 4L3coyUwc8DRTB7y84ze

Changed password for user elastic
PASSWORD elastic = z2HGfIqeAoz1SwTRmXuV
```



