---
layout: post
title:  "Maven에서 서버별 패키징하기"
date:   2015-07-20
categories: java
---


Maven으로 개발시 환경별로 패키징해야 할 필요가 있다.  
즉, DB가 로컬, 데모, 리얼서버별로 각각 다를 경우 DB접속정보는 각각 다르다. 이때 각각 다른 접속정보를 가지고 패키징이 되어야 한다.
또한 로그도 경로가 다르기때문에 각각 패키징해야 한다.

### 1. 각각의 패키징 폴더를 만들어라.

로컬, 데모, 리얼서버(실제운영서버)별로 패키징폴더를 만들자.  
src/main안에 resoucres-local, resources-demo, resources-real 이렇게 3개의 폴더를 만들고 폴더안에 각각
log4j.xml, properties/jdbc.properties 를 만든다. 아래 그림을 참조하자.

<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/packaging1.jpg" style="width:100%">
</div>  

그리고 properties/jdbc.properties에 jdbc.url에 로컬, 데모, 서버별로 각각 기록한다.
log4j.xml도 마찬가지이다.

```bash
########################################################################
# JDBC 설정정보
########################################################################
jdbc.driverType=mysql
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test?characterEncoding=UTF-8&autoReconnect=true
jdbc.username=testuser
jdbc.password=111222333
```

### 2. Java Build Path 수정

`Add Folder...` 클릭한다.
<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/packaging2.jpg" style="width:100%">
</div>

`resources-local`를 선택하고 `OK`를 클릭한다.
<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/packaging3.jpg" style="width:100%">
</div>

`resources-local`를 `resources` 바로 아래에 위치시킨다.
<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/packaging4.jpg" style="width:100%">
</div>


최종적으로 완료된 모습이다.
<div style="text-align:center;margin-bottom: 30px;">
<img src="/assets/images/packaging5.jpg" style="width:100%">
</div>


### 3. pom.xml에 설정추가

`resources` 부분을 추가한다.

```xml
<resources>
	<resource>
		<directory>src/main/resources</directory>
	</resource>
	<resource>
		<directory>src/main/resources-${env}</directory>
	</resource>
	<resource>
		<directory>${project.build.sourceDirectory}</directory>
		<includes>
			<include>**/*.xml</include>
		</includes>
	</resource>
</resources>
```

`plugins` 안에 배포용 구분 plugin을 넣는다.

```xml
<plugin>
<!-- 메이븐 배포용 구분처리 플러그인  -->
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-resources-plugin</artifactId>
	<version>2.5</version>
	<configuration>
	     <encoding>UTF-8</encoding>
	</configuration>
 </plugin>   
```


`profiles`를 추가한다.

```xml
<!-- 배포용 설정파일 구분  -->
<profiles>
	<!-- Local server -->
	<profile>
		<id>local</id>
		<activation>
		<activeByDefault>true</activeByDefault>
		</activation>
		<properties>
			<env>local</env>
		</properties>
	</profile>
	<!-- Development server -->
	<profile>
		<id>demo</id>
		<properties>
			<env>demo</env>
		</properties>
	</profile>
	<!-- real server -->
	<profile>
		<id>real</id>
		<properties>
			<env>real</env>
		</properties>
	</profile>
</profiles>
```

### 4. application.xml 변경

기존에 application.xml등에 직접 jdbc설정하는 부분을 propeties를 읽어들이는 부분으로 변경한다.


```xml
<bean id="propertyPlaceholderConfigurer"
       class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
       <property name="locations">
           <value>classpath:properties/jdbc.properties</value>
       </property>
</bean>

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"  destroy-method="close">
	<property name="driverClassName" value="${jdbc.driverClassName}" />		 
	<property name="url" value="${jdbc.url}" />
	<property name="username" value="${jdbc.username}" />
	<property name="password" value="${jdbc.password}" />
	<property name="validationQuery" value="select 1"/>  <!-- 오라클은 select 1 from dual -->
	<property name="validationQueryTimeout" value="3600"/>
</bean>		
```

### 5. 배포

배포할 때는 `pom.xml`의 `<profiles>`의 정의된 `<id>`를 붙혀주면 된다.   
로컬은 디폴트이므로 생략이 가능하다.

```bash
mvn install -p demo  //데모서버
mvn install -p real  //실제서버
```

이렇게하면 각각 서버별의 설정파일에 맞게 패키징이 된다.
