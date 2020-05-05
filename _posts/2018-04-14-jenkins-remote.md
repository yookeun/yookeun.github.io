---
layout: post
title: 젠킨스(Jenkins)에서 원격(Remote)으로 배포하기 
date: 2018-04-14
categories: tools
---

Jenkins에서 원격으로 배포하는 방법을 알아보자.  젠킨스에서 원격으로 배포하기 위해서는 해당 플러그인을 받아야 하다. 가장 많이 사용하는 플러그인은 아래와 같다 

- [Publish Over SSH Plugin](https://wiki.jenkins.io/display/JENKINS/Publish+Over+SSH+Plugin)
- [SSH2Easy Plugin](https://wiki.jenkins.io/display/JENKINS/SSH2Easy+Plugin)

우리는 여기서 `Publish Over SSH Plugin` 를 사용하도록 한다. (`SSH2Easy Plugin`은 사용시 바이너리전송에 문제가 발생한 경험이 있다)  

### 1. Publish Over SSH 설정

Jenkins관리 > 플러그인관리에서 설치가 잘되고 젠킨스를 리스타트했다면 `시스템 설정` 에 아래와 같은 부분을 찾을 수 있다.

![jekins-remote1](/assets/images/jenkins-remote1.jpg)

`SSH Servers` 부분에 각 항목을 기재하도록 하자

| 항목             | 설명                                                |
| ---------------- | --------------------------------------------------- |
| Name             | Job에서 표시될 이름을 지정하면 된다.                |
| Hostname         | IP Address                                          |
| Username         | ssh 접근 계정                                       |
| Remote Directory | 업로드될 디렉토리 (여러개가 있다면 상위만 지정하자) |

다음은 고급을 클릭해서 계정이나 포트정보를 입력하도록 한다. 

![jenkins-remote2](/assets/images/jenkins-remote2.jpg)

패스워드와 포트를 입력하고 나서 하단에 `Test Configuration`를 클릭해서 `Success`가 표시되는지 확인하면 된다. 

### 2. 사용

이제 해당 Item `구성`으로 가보자. `빌드환경` 부분을 아래와 같이 설정하자. 

![jenkins-remote3](/assets/images/jenkins-remote3.jpg) 

`Send files or execute commands over SSH after the build runs` 를 체크해서 빌드가 완료되면 실행하도록 한다. `Name` 부분에서 우리가 아까 등록한 이름을 선택하면 된다. 

| Transfers 항목   | 설명                                                         |
| ---------------- | ------------------------------------------------------------ |
| Sources files    | jar 혹은 war가 빌드된 위치를 적는다.<br />ex) test-server/builds/libs/test-server.jar |
| Remove prefix    | 파일 앞부분에 경로부분을 적는다 => test-server/builds/libs/  |
| Remote directory | 업로드될 경로이다. <br/>주의할 것은 위에 `시스템 설정`에서 Remote Directory 내의 디렉토리를 적어야 한다.<br> 최종 경로가 /usr/local/test-server 라면 => 이미 위에서 `/usr`를 적었기에 여기에는<br>`local/test-server` 만 적어야 한다. |
| Exec command     | 실행할 명령어를 적으면 된다. ex) test-server.sh restart      |

마지막으로 윗 부분에 `고급` 클릭해서 설정정보를 마무리하자. 

![jenkins-remote4](/assets/images/jenkins-remote4.jpg)

특히 `Verbose output in console` 부분을 체크하면 빌드할 때 상세 내역이 표시되므로 유용하다. 

이렇게 설정이 완료되면 빌드가 정상적으로 되면 해당 원격지로 배포가 이루어진다. 