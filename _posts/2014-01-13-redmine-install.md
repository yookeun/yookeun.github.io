---
layout: post
title:  "Redmine을 ubunt14.04에 설치하기"
date:   2014-01-13
categories: linux
---

우분투에 [레드마인(redmine)](http://www.redmine.org/)을 설치해보자.

설치환경은 아래와 같다.

>OS  : Ubuntu 14.04 Server 64bit
DB  : '5.5.33a-MariaDB'

레드마인은 아피치+MYSQL로 연동되어진다. 그러므로 두개가 설치되어야 한다.
(mariaDB도 가능하다)

<http://www.redmine.org/projects/redmine/wiki/HowTo_Install_Redmine_on_Ubuntu_step_by_step>(1)
를 보면 공식적으로 레드마인 설치단계를 자세히 보여준다.

그런데 위에 문서는 두가지 문제가 있다.

첫째는 기존의 mysql or mariaDB를 사용할때에는 위 문서대로 하면 안된다.
(만약 mysql도 새로 설치해야 한다면 위문서대로 가면 된다. 그러나 libapache2-mod-passenger는 해결해야 한다)

둘째는 libapache2-mod-passenger이것을 설치하면 문제가 발생한다.   
<http://openarisu.tistory.com/231> (2) 에 둘째 문제에 대해 잘 설명하고 있다.

그래서 (1), (2)문서를 참고와 토대로 설치작업을 진행한다. (우분투이고 해당서버에 이미 mysql or mariadb가 설치된 경우)

### 1. 먼저 mysql 설정부터 변경하자.

기존의 `mysql.sock` 의 설정값을 redmine에서 읽을 수 있도록 해야 한다.
그렇지 않으면 redmine을 설치할때 mysql에 접속이 되지 않아서 관련된 테이블 설치를 하지 못하고 실패하게 된다.
설치된 서버의 my.cnf를 열자

만약 거기서
`socket = /tmp/mysql.sock` 로 세팅되어 있다면 레드마인에서 못 읽는다.   
그 이유는 레드마인을 설치할때 `/var/run/mysqld/mysqld.sock` 를 통해서 mysql에 접근하기 때문이다.

따라서 다음과 같이 임시로 링크를 만들어준다.

```bash
ln -s /tmp/mysql.sock /var/run/mysqld/mysqld.sock
```

(만약 mysqld폴더가 없다면 만들어주어라)
이 링크는 재시작하면 사라진다. 그 이유는 모르겠다. 그래서 일단 이렇게 레드마인이 성공적으로 설치되면

```bash
sudo vi /etc/redmine/default/database.yml  을 열고
socket : /tmp/mysql.sock 을 추가하면 된다.
```

### 2. 이제 레드마인을 설치하자.

```bash
sudo apt-get install redmine redmine-mysql
```

위 명령이 실행되면 아래의 처럼 prompt화면이 출력된다.

 ┌──────────────────────────┤ Configuring redmine ├──────────────────────────┐  
 │                                                                           │  
 │ The redmine/instances/default package must have a database installed and  │  
 │ configured before it can be used.  This can be optionally handled with    │  
 │ dbconfig-common.                                                          │  
 │                                                                           │  
 │ If you are an advanced database administrator and know that you want to   │  
 │ perform this configuration manually, or if your database has already      │  
 │ been installed and configured, you should refuse this option.  Details    │  
 │ on what needs to be done should most likely be provided in                │  
 │ /usr/share/doc/redmine/instances/default.                                 │  
 │                                                                           │  
 │ Otherwise, you should probably choose this option.                        │  
 │                                                                           │  
 │ Configure database for redmine/instances/default with dbconfig-common?    │  
 │                                                                           │  
 │                    <Yes>                       <No>                       │  
 │                                                                           │  
 └───────────────────────────────────────────────────────────────────────────┘  


`<Yes>`를 선택하면 설정이 시작된다. Mysql(or mariaDB)루트계정과 레드마인이 사용할 패스워드를 지정해주자.
이부분은 [HowTo_Install_Redmine_on_Ubuntu_step_by_step](http://www.redmine.org/projects/redmine/wiki/) 를 참조하라.
설치가 완료되면 mysql or mariaDB에 redmine_default 라는 DB가 생성되어 있을 것이다.

다음과정을 진행하자.

### 3. Passenger 설치 (<http://openarisu.tistory.com/231> 참고함)

기존의 `libapache2-mod-passenger` 가 있을지 모르니 미리 삭제

```bash
sudo apt-get remove libapache2-mod-passenger
//Passenger와 bundler를 설치하자.
sudo gem install passenger
sudo gem install bundler
```

실행하면 시간이 좀 걸린다. 인내심을 갖고 기다리자.
끝났으면 이제 아파치와 연동하는 모듈을 설치하자.

```bash
$sudo apt-get install libcurl4-openssl-dev libssl-dev zlib1g-dev apache2-prefork-dev libapr1-dev libaprutil1-dev
$sudo passenger-install-apache2-module
```

자 이렇게 하면 갑자기 prompt화면으로 변경이 된다. 일단 엔터를 친다.  계속 엔터를 친다.
그러면 아래와 같은 화면이 보이게 된다.

```bash
Please edit your Apache configuration file, and add these lines:

LoadModule passenger_module /var/lib/gems/1.9.1/gems/passenger-5.0.13/buildout/apache2/mod_passenger.so
PassengerRoot /var/lib/gems/1.9.1/gems/passenger-5.0.13
PassengerDefaultRuby /usr/bin/ruby1.9.1

After you restart Apache, you are ready to deploy any number of web
applications on Apache, with a minimum amount of configuration!
Press ENTER to continue.
```

위에 블록된 부분을 Copy해서 메모장등에 저장하자(버전명에 주의할 것!)

이제 아파치 설정으로 넘어가자.

### 4. 아파치 설정


```bash
sudo vi /etc/apache2/mods-available/passenger.load
```

기존의 LoadModule를 지우거나, 주석으로 막고 아래 내용을 추가하자.

```bash
LoadModule passenger_module /var/lib/gems/1.9.1/gems/passenger-5.0.13/buildout/apache2/mod_passenger.so
```

다음 설정파일을 열고 아래 내용을 추가하자.

```bash
sudo vi /etc/apache2/mods-available/passenger.conf
```

기존꺼는 주석으로 막거나 지워라.

```bash
<IfModule mod_passenger.c>
PassengerRoot /var/lib/gems/1.9.1/gems/passenger-5.0.13
PassengerDefaultRuby /usr/bin/ruby1.9.1
PassengerDefaultUser www-data
</IfModule>
```


레드마인을 연결하기 위한 심볼링 링크를 만들자.

```bash
sudo ln -s /usr/share/redmine/public /var/www/html/redmine
```

다음 설정파일을 열고 아래 내용을 추가하자. (다른 `<VirtualHost>` 안에)

```bash
sudo vi /etc/apache2/sites-available/000-default.conf
```

```bash
<Directory /var/www/html/redmine>
    RailsBaseURI /redmine
    PassengerResolveSymlinksInDocumentRoot on
</Directory>
```

### 5. 아파치 가동

자 이제 passenger와 아파치를 가동하자.

```bash
sudo a2enmod passenger
sudo service apache2 restart
```

정상적으로 가동되면 `ok...`라는 것이 보일 것이다.

### 6. 브라우저 확인

`http://아파치설치아이피/redmine`을 치면 레드마인 초기화면이 보일 것이다.


이제 레드마인을 즐기면 된다.

---
---

### 7. 아래와 같은 장애발생시 조치사항

장애1)

>ERROR: 'rake/rdoctask' is obsolete and no longer supported. Use 'rdoc/task' (available in RDoc 2.4.2+) instead.

해결방법)>

```bash
sudo gem install rdoc
```


장애2)

>no such file to load -- fastercsvError when running rake db:migrate, check database configuration.

해결방법)

```bash
sudo gem install fastercsv

//설치후에는
sudo vi /usr/share/redmine/Gemfile

// 열고 맨 마지막 라인에 아래같이 등록한다.
gem 'fastercsv'
```

장애3)

>Gemfile.lock 퍼미션 에러가 난 경우 (웹페이지에서)

해결방법)

```bash
sudo touch /usr/share/redmine/Gemfile.lock
sudo chown www-data:www-data /usr/share/redmine/Gemfile.lock
```
