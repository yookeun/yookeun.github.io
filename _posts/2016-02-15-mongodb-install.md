---
layout: post
title:  "Ubuntu에서 Mongodb 설치하기"
date:   2016-02-15
categories: database
---

Ubuntu 14.04 에서 Mongodb를 설치해보자.  
Mongodb 홈페이지에서 tgz로 압축된것을 받고 자신의 홈디렉토리에 압축을 푼다.

프로필을 수정하자.

```bash
sudo vi /etc/profile

export MONGODB_HOME = /home/userhome/mongodb
export PATH=$PATH:$MONGODB_HOME/bin:

source /etc/profile
```

`mongod` 를 실행하면 아마 에러가 나올 것이다. 즉 기본 /data/db가 없다고 표시된다.
데이타베이스를 보관할 디렉토리를 만들어야 한다.
Mongodb에서는 기본 디렉토리를 /data/db로 사용하고 있다.
따라서 기본을 그냥 쓰려면

```bash
sudo mkdir -p /data/db
```

로 하면 되지만, 이렇게 되면 루트에 저장되므로 우리는 홈디렉토리에 저장하려고 한다.

```bash
mkdir mongodb-data
```

그리고 `--dbpath` 옵션을 주어 실행한다.

```bash
mongod --dbpath mongodb-data
```

그런데 실행시키면 중간에 `WARNING` 표시가 난다.

```bash
2016-02-15T11:21:52.362+0900 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2016-02-15T11:21:52.362+0900 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
```

Ubuntu는 아래와 같이 파일을 만들고 내용을 복사한다.

```bash
sudo gedit /etc/init.d/disable-transparent-hugepages
```

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          disable-transparent-hugepages
# Required-Start:    $local_fs
# Required-Stop:
# X-Start-Before:    mongod mongodb-mms-automation-agent
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Disable Linux transparent huge pages
# Description:       Disable Linux transparent huge pages, to improve
#                    database performance.
### END INIT INFO

case $1 in
  start)
    if [ -d /sys/kernel/mm/transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/transparent_hugepage
    elif [ -d /sys/kernel/mm/redhat_transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/redhat_transparent_hugepage
    else
      return 0
    fi

    echo 'never' > ${thp_path}/enabled
    echo 'never' > ${thp_path}/defrag

    unset thp_path
    ;;
esac
```

실행파일을 만들고 부팅시 설정파일에 적용하고 재부팅하면 된다.

```bash
sudo chmod 755 /etc/init.d/disable-transparent-hugepages

sudo update-rc.d disable-transparent-hugepages defaults
```


참고 : <https://docs.mongodb.org/manual/tutorial/transparent-huge-pages/#transparent-huge-pages-thp-settings>
