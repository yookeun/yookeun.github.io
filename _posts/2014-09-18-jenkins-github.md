---
layout: post
title:  "Jenkins와 Github를 연동하자."
date:   2014-09-18
categories: tools
---

자동화빌드툴로 `Jenkins`를 사용하고 있고, 소스관리는 `Github`에서 관리하고 있다면 두가지를 연동해서 사용하면 무척 편리하다.

github에 push하는 순간 jenkins에서 pull하고 바로 빌드해주면서 서버에 인스톨할 수있기 때문이다. 이 작업을 하기 위해서는 jenkins가 설치된 서버에서 github로 ssh방식으로 접근하게 하는 것이 편리하다(당연히 jenkins서버에는 git이 설치되어 있어야 한다)

github에서 <[ssh등록](https://help.github.com/articles/generating-ssh-keys)>를 참고한다.

### 1. jenkins서버에서 생성한 ssh키를 github에 등록한다.

github로 로그인한후에 `settings`에서 SSH Keys -> Add SSH Key를 통해 생성된 ssh키를 등록한다.


### 2. jenkins에서 github에 관련된 플러그인을 설치한다.

관리자로 jenkins에 로그인한후에 jenkins관리로 이동한다.

![jenkins-github](/assets/images/jenkins-github.jpg)

상단의 설치가능을 선택하고 우측 Filter부분에 git를 입력하면  git과 관련된 플러그인이 나온다.

![jenkins-github2](/assets/images/jenkins-github2.jpg)

여기서 Git Plugin과 GitHub Plugin를 설치한다(4개 모두 설치해도 무방하다)
선택한후에 하단 버튼 `Downloand now and install after restart` 을 클릭해서 jenkins를 재시작해준다.

### 3. jenkins에서 해당 job에 github설정

github와 연동할 `job`으로 가서 좌측의 설정으로 들어간다. 소스코드관리에서 Git를 선택하고, Respository URL에서 해당 job의 giturl를 등록한다(git@으로 시작하는...)

![jenkins-github3](/assets/images/jenkins-github3.jpg)

조금 내려가서 `빌드 트리거`에서 다음을 선택한다.

![jenkins-github4](/assets/images/jenkins-github4.jpg)

즉, github에서 `push`된 것이 있다면 그때 `pull`해서 빌드하겠다는 뜻이다.
다음 빌드작업을 설정해준다. maven을 이용한다면 다음과 같이 세팅한다.

![jenkins-github5](/assets/images/jenkins-github5.jpg)

`Goals and options`에서 `package`작업을 한다. 보통 `clean package`를 사용한다.  

다음은 빌드후에 작업을 설정해준다. 만약 `tomcat`에 올린다면 `Context path`에는 톰켓 경로를 지정해주고,
`Manage user name`과 `Manager password`는 `tomcat`에서 설정한 정보를 입력하면 된다. tomcat URL도 마찬가지이다.

![jenkins-github6](/assets/images/jenkins-github6.jpg)

이렇게 세팅을 완료하고 저장을 하면 앞으로 github에 `push`될 때마다 jenkins에서 자동으로 `pull`하고 빌드를 한 후에 이상이 없다면 설정한 `Tomcat`으로 `Depoly`하게 된다.  


### 4. Github 설정
Jenkins와 연동하려는 Github 프로젝트로 가서 `Settings` 으로 들어간다.  
왼쪽메뉴에서 `Webhooks & Service` 에서 `Add service`를 선택하고 jenkins로 검색하면 리스트에 `Jenkins (GitHub plugin)`을 선택한다.

![jenkins-github7](/assets/images/jenkins-github7.jpg)

다음화면에서 선택한 Jenkins GitHub plugin에 대한 설정을 등록한다.  

![jenkins-github8](/assets/images/jenkins-github8.jpg)

`Install Notes`에 표시된 것처럼 설치된 jenkins경로를 `Jenkins hook url`에 넣어주고 `Add Service`를 클릭하면 된다.


`http://jenkins설치된주소/github-webhook/`


이제 개발을 즐기면 된다.
