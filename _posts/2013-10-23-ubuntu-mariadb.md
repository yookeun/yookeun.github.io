---
layout: post
title:  "Ubuntu에서 mariadb 설치하기"
date:   2013-10-23
categories: linux
---

mariaDB 파일 설치
mariaDB는 홈페이지에서 각 OS에 맞게 다운로드 하는 설명이 자세히 나와 있다.
여기서는 파일을 다운로드 받아서 설치하는 방법을 설명한다.
mariaDB를 버전에 맞게 다운받는다.

루트가 아닌 홈디렉토리에 압축을 해제한다.

```bash
tar -zxvpf /path-to/mariadb-VERSION-OS.tar.gz
ln -s mariadb-VERSION-OS mysql
```

mysql의 data파일을 디폴트로 사용하지 말고 별도로 만들어서 관리한다.
(만약 기존의 mysql data를 사용한다면 그것으로 그대로 연결하면 된다)
이렇게 되면 나중에 버전업하거나 재설치할때 용이하다.

```bash
mkdir mysqldata
```

넘겨받은 my.cnf에 datadir, basedir를 올바르게 기재한후에  `/etc/my.cnf` 로 만든다.
넘겨받은 mysqld에서 datadir, basedir, my.cnf 위치를 확인한후에  `/etc/init.d`로 복사한다.

```bash
sudo su -->root로 로긴
cp my.cnf /etc
cp mysqld /etc/init.d
chmod 755 mysqld
vi /etc/my.cnf ==> 내용확인할 것
vi /etc/init.d/mysqld => 내용확일할 것

basedir = /home/경로/mysql
datadir = /home/경로/mysqldata
```

mysql계정과 그룹생성

```bash
groupadd mysql
sudo useradd -g mysql mysql
```


설치된 mysql로 이동

```bash
cd mysql
chown -R root .  (소유주를 root로 )
chgrp -R mysql . (그룹을 mysql로 )
cd ..
cd mysqldata
chown -R mysql .
chgrp -R mysql .

cd mysql
./scripts/mysql_install_db --user=mysql
ls -l ../mysqldata/ --> 제대로 설치되어있는지 확인
/etc/init.d/mysqld start -> mysql시작..아무런 메시지가 나오지 말아야 한다.
ps -ef | grep mysql  -> mysql 가동 확인
./bin/mysql -uroot  -> 접속
==> 잘 적용되었다면 빠져나오자.
mysql> quit
```

서비스에 등록하자 ubuntu에는 `chkconfig`가 없다. 대신 `update-rc.d`를 이용한다.

```bash
cd /etc/init.d
update-rc.d mysqld default
update-rc.d -f mysqld defaults  (만약 제거한다면 update-rc.d -f mysqld remove)
reboot (리부팅해서 최종확인한다).
```
