---
layout: post
title:  "AngularJS 공부를 위한 세팅준비"
date:   2015-01-18
categories: javascript
---

Javascript framework인 AngularJS를 공부하기 위해 세팅부터 하자.  


일단 그냥 html만들고, js만들어서 브라우저를 실행시켜도 되지만, 웹서버에서 구동하면서 공부하면 더 많은 장점을 얻을 수 있다.
Tomcat등을 이용해서 활용할 수도 있지만, node.js의 npm를 이용해서 설치하도록 한다.

맨먼저 node.js를 설치해야 한다. 우분투(14.04)를 기준으로 설명한다(맥이나, 윈도우는 node.js사이트에서 참조하면 된다)

### 1. 다음과 같이 최신의 node.js설치한다.

```bash
sudo apt-get install python-software-properties
sudo apt-add-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```

### 2. 이제 grunt를 설치하자.

[Grunt](http://gruntjs.com/)는 Command Line에서 실행되는 Javascript build tool이다. 여기서 자세한 설명은 생략한다.

```bash
sudo npm install -g grunt-cli
```

### 3. 다음은 Scaffold의 도구인 Yeoman 를 설치한다.

[Yeoman](http://yeoman.io/)은 웹사이트를 만드는 기본적은 뼈대를 생성해주는 도구이다.

```bash
sudo npm install -g yo
```

### 4. 다음은 [bower]를 설치하자.

[Bower](http://bower.io/)는 javascript 패키지 매니저이다. jquery, bootstrap, angualrjs등의 패키지를 쉽게 관리하게 해준다.

```bash
sudo npm install -g bower
```


### 5. Yeoman을 이용해서 angular 용 웹사이트를 만들려면, angualr generator를 설치한다.

```bash
sudo npm install -g generator-angular
```
이제 모든 프로그램이 설치완료가 되었다.  
화면UI는 bootstrap를 사용하고 angualrJS용 웹사이트의 뼈대를 구축하자.

```bash
mkdir angularStudy(프로젝트명)
cd angularStudy
yo angular
```

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/yeoman1.jpg" style="width:100%">
</div>  
Sass설치를 묻는다. 취향대로 선택하자.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/yeoman2.jpg" style="width:100%">
</div>
Bootstrap를 사용할지 묻는다. 당연히 Y다.


<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/yeoman3.jpg" style="width:100%">
</div>
설치할 angularJS 라이브러리를 묻는다. 그냥 기본으로 가자. 엔터를 친다.
그렇게 되면 설치가 진행 될 것이다.  
만약 설치시 콘솔에 아래와 같이 나오면 npm 권한문제이므로

```bash
npm ERR! Error: EACCES, mkdir '/usr/local/lib/node_modules/yo'
```

권한을 부여하고 생성한 프로젝트를 삭제후 yo angular를 다시 실행한다.

```bash
$> sudo chonw -R 사용자아이디 ~/.npm
```

### 6. 웹사이트구동

Grunt를 구동하자.

```bash
grunt server
````
정상적으로 작동되면 아래와 같은 초기화면이 보여진다.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/yeoman4.jpg" style="width:100%">
</div>

### 7. yeoman으로 구성된 웹디렉토리이다.

`app`폴더안에 해당 소스들이 구성되어 있고, bower를 이용해서 설치했으므로 `bower_component` 폴더안에 angular등이 설치되어 있다.
단, yeoman으로 설치되면 angular는 최신버전 (1.3버전대)가 자동 설치된다.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/yeoman5.jpg" style="width:100%">
</div>

자! 이제 공부를 하면서 app안에 소스를 만들고 수정하면 grunt덕분에 웹사이트에서 자동 갱신되어 편리하게 개발을 진행할 수가 있다.
