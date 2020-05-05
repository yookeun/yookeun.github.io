---
layout: post
title:  "Mac에서 homebrew를 통한 mariadb 설치"
date:   2014-11-16
categories: mac
---

맥에서 homebrew를 이용해서 mariadb를 설치해본다.

환경:OX Yosemite  
먼저 <[homebrew](http://brew.sh/)>를 설치해야 한다. homebrew는 우분투의 apt-get 같은 패키지인스톨 프로그램이라고 생각하면 된다.  

위 사이트에서 설치를 진행한다. 설치가 완료된 다음 터미널에서 brew 를 치면 명령어의 설명이 나오면 정상으로 설치된 것이다.

먼저 brew를 업데이트해준다. (최신의 정보로 업데이트된다)

```bash
brew update
```

다음 maridb를 검색해보자 (꼭 할 필요는 없다)

```bash
brew search mariadb
```

검색하면 mariadb가 검색된다. brew에서는 기본적으로 최신버전을 설치하게 된다. 2014.11월 기준으로 mariadb의 최신버전은 10.0.14이다.

```bash
brew info mariadb (역시 꼭 할필요는 없다)

mariadb: stable 10.0.14 (bottled)
```

만약 brew install mariadb로 인스톨하면 위와 같이 stable버전이 설치된다.
자. 이제 설치를 해보자.

```bash
brew install mariadb
```

```bash
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/mariadb-10.0.14_1.yosemite.bottle.tar.gz
... 중략 ...
==> Summary

/usr/local/Cellar/mariadb/10.0.14_1: 524 files, 125M
```

그러면 위와 같이 설치가 진행이 된다. 이제 세팅을 해준다.

```bash
unset TM PDIR
```
설치된 경로로 이동하자. 10.0.14_1버전이 설치되어 있다. (탭키로 치면 경로나옴)

```bash
cd /usr/local/Cellar/mariadb/10.0.14_1/
```

mariadb의 DB를 설치한다.

```bash
mysql_install_db
```

그러면 아래와 같이 설치가 진행이 된다.


```bash
Installing MariaDB/MySQL system tables in '/usr/local/var/mysql' ...
16 18:09:08 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
user created by default.  
You can start the MariaDB daemon with:

..중략...

Support MariaDB development by buying support/new features from
SkySQL Ab. You can contact us about this at sales@skysql.com.
Alternatively consider joining our community based development effort:
http://mariadb.com/kb/en/contributing-to-the-mariadb-project/
```


자. 이제 mariadb를 가동시켜본다.

```bash
mysql.server start
...
Starting MySQL

. SUCCESS!
```

SUCCESS가 나오면 성공이다.

- 참고 사이트 : <https://mariadb.com/blog/installing-mariadb-10010-mac-os-x-homebrew>
