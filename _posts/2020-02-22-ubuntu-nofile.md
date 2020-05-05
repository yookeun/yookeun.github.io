---
layout: post
title:  "Ubuntu에서 file descriptors & vm.max_map_count 설정"
date:   2020-02-22
categories: linux
---
우분투 Desktop에서 단순히 `/etc/security/limits.conf` 만 수정해선 안된다.
아래 순서대로 진행해야 한다. 

### 1. sudo vi /etc/systemd/system.conf
: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
> DefaultLimitNOFILE=65536



### 2. sudo vi /etc/security/limits.conf
```
*       soft    nofile  65536
*       hard    nofile  65536
*       soft    nproc   65536
*       hard    nproc   65536
*       soft    memlock unlimited
*       hard    memlock unlimited
```

### 3. sudo vi /etc/sysctl.conf
: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
> vm.max_map_count=262144