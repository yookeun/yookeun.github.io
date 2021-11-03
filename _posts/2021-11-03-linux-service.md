---
layout: post
title:  "Linux에 쉘스크립트(sh) 서비스등록"
date:   2021-11-03
categories: linux
---
스프링 부트등을 실행가능한 jar로 만들고 서비스를 등록할 경우가 생긴다. 이때 jar를 실행한 스크립트를 만들고 사용하는 경우가 있는데 해당 스크립트를 서비스에 등록하도록 한다.

### 1. 실행 스크립트 작성(start.sh)
test.jar를 실행시키는 스크립트 
```bash 
PID_FILE_NAME=test.pid
JAVA_OPT="-Xms1024m -Xmx2048m"
JAVA_OPT="$JAVA_OPT -DLog4jContextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector"
SERVICE_NAME=test.jar
PROFILE=dev

echo "Starting $SERVICE_NAME...."
sleep 2
if [ ! -f $PID_FILE_NAME ]; then
    java -jar $JAVA_OPT -Dspring.profiles.active=$PROFILE $SERVICE_NAME >> /dev/null & echo $! > $PID_FILE_NAME
    echo "$SERVICE_NAME started..."
else
    echo "$SERVICE_NAME is already running..."
fi   
```

### 2. 중지 스크립트 작성(stop.sh)
test.jar를 중지시키는 스크립트 

```bash 
#!/bin/sh

PID_FILE_NAME=kte-api.pid
JAVA_OPT="-Xms1024m -Xmx2048m"
SERVICE_NAME=kte-api.jar
PROFILE=prod

if [ -f $PID_FILE_NAME ]; then
    PID=$(cat $PID_FILE_NAME);
    echo "$SERVICE_NAME stoping..."
    sleep 2 
    kill -9 $PID
    echo "$SERVICE_NAME stoped..." 
    rm $PID_FILE_NAME
else
    echo "$SERVICE_NAME is not running..."
fi
```

### 3. 서비스 작성 
서비스를 작성한다. 
```bash
sudo vi /etc/systemd/system/test.service
```
```bash
[Unit]
Description=TEST 

[Service]
Type=forking
User=ec2-user
Group=ec2-user
ExecStart=/home/ec2-user/test/start.sh
ExecStop=/home/ec2-user/test/stop.sh
WorkingDirectory=/home/ec2-user/test

[Install]
WantedBy=multi-user.target
```
만약 jar파일을 직접 실행한다면 type=simple로 처리하는데 위 스크립트등으로 pid를 처리한다면 forking으로 해주어야 정상가동된다. 

### 4. 서비스 등록

```bash
sudo chmod +x /etc/systemd/system/kte-admin.service
sudo systemctl daemon-reload
sudo systemctl enable test.service
sudo systemctl start test.service
```

서비스상태를 확인하고 싶으면 
```bash
sudo systemctl status test.service
```

종료를 하고 싶다면 
```bash
sudo systemctl stop test.service
```

이렇게 하면  리눅스등이 재부팅될때 자동 실행하게 처리한다. 