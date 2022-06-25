---
layout: post
title:  "Docker로 PostgreSQL설치하고 연결하기"
date:   2021-12-25
categories: docker
---

이미지 받고 설치

``` bash
docker pull postgres 
docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD=1234 -d postgres

```
설치 및 가동확인 
```bash
docker ps 
```

PosgreSQL 구동확인 
```bash 
docker logs postgres
```

컨테이너 접속 
```bash 
docker exec -it postgres /bin/bash
```

Postgres 접속 후 데이터베이스 생성
```bash
psql -U postgres
CREATE DATABASE testDB;
\q (접속종료)
exit (컨텐이너접속종료)
```

최종 DB툴로 접속하면 완료된다.

추가로 MySQL설치 방법도 거의 동일하다. 

MySQL 설치 
```bash 
docker run --name mysql -e MYSQL_ROOT_PASSWORD=1234 -d -p 3306:3306 mysql:latest
```

MySQL 접속
```bash 
docker exec -it mysql bash
```