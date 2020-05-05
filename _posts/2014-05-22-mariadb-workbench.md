---
layout: post
title:  "ubuntu에서 mariaDB와 mysql-workbench 설치"
date:   2014-05-22
categories: database
---

우분투에서 mariadb를 수동으로 설치하는 방법과 동시에 apt-get으로 설치하는 방법이 있다.

그런데 수동설치(즉,압축해제)를사용했을경우 설치후에 실행이 잘 되지만, mysql-workbench를 설치하면 workbench가설치될때 같이설치되는 mysql관련모듈때문에 mariadb가 실행이 되지 않는다. (can't start mysql (usr/bin/mysql_safe) 등의 메시지표시됨)
그래서 mariadb를 apt-get등으로 시스템설치를 하고 나서 workbench를 설치해야 한다.

mariaDB 홈페이지에 아래 경로로 가서 자신의 os에 맞는 것을 설치한다.
<https://downloads.mariadb.org/mariadb/repositories/#mirror=kaist&distro=Ubuntu&distro_release=trusty>

![workbench](/assets/images/workbench.jpg)

설치를 진행하면 mariaDB root 패스워드 입력을 묻는 창이 나오고 입력하면 설치가 진행되면서 실행이 된다.
그런데 mysql 데이터 파일이 모두 루트파티션에 설치되므로 추후에 OS재설치문제가 생기면 문제가 생긴다.
따라서 mysql 데이터 파일은 다른 파티션에 설치하는 것이 좋다.
일단, 가동된 mariadb를 내린다.

```bash
sudo service mysql stop
```

그리고 나서 mariadb의 data폴더를 다른 파티션에 옮긴다.mariadb 데이타파일은 `/var/lib/mysql`에 있다.
이것을 자신의 홈디렉토리 즉, 다른 파티션에 옮기자. 여기서는 /home/abc라는 곳에 mysql이라는 곳으로 옮기겠다.

```bash
sudo cp -R -p /var/lib/mysql /home/abc/mysqldata
```

그러면 /home/abc/mysqldata이라는 디렉토리로 카피가 된다. 이제 `my.cnf`에서 datadir를 변경해주어야 한다.

`sudo vi /etc/mysql/my.cnf` 를 열어서 datadir 부분을 수정한다.

```bash
[mysqld]
#datadir = /var/lib/mysql   --> #주석으로 막는다
datadir = /home/abc/mysqldata // 신규로 수정
```

그리고 나서 sudo service mysql start 로 해서 정상으로 실행되는지 확인한다.
`mysql-workbench`는 아래의 경로에서 다운받는다

<http://dev.mysql.com/downloads/tools/workbench/>
