---
layout: post
title:  "[Spring4.1]log4j를 logback으로 변경하기"
date:   2015-11-10
categories: java
---
그동안 Spring에서 오랫동안 사용해온 log4j를 새로운 logback으로 변경해보자.  
Maven을 기반으로 설명한다. 일단 기존의 등록된 log관련은 모두 지워준다.  

`Spring 4.1과 mybatis 3.3.0, mybatis-spring-1.2.3` 으로 구축한다
(Spring 3.2.8 + mybatis 3.2.0 + mybatis-spring.1.1.0 에서는 구현되지 못했다)

Spring3에서 Spring4로 넘어가면 spring-tx, spring-jdbc가 따로 분리되기 때문에 별도로 dependency에 추가해주어야 한다.
그리고 아울러서 mybatis부분도 버전을 업그레이드한다.  

logback과 상관없지만, MappingJackson2HttpMessageConverter 2.0 버전을  사용한다면 그 부분도 업그레이드해준다.

### 1. pom.xml에 Spring, Mybatis 버전업그레이드

```xml
<properties>
  <java-version>1.7</java-version>
  <org.springframework-version>4.1.8.RELEASE</org.springframework-version>
  <org.aspectj-version>1.6.10</org.aspectj-version>
  <org.slf4j-version>1.7.7</org.slf4j-version>
</properties>
...
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-tx</artifactId>
  <version>${org.springframework-version}</version>
</dependency>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
  <version>${org.springframework-version}</version>
</dependency>

<!-- myBatis -->
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.3.0</version>
</dependency>

<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>1.2.3</version>
</dependency>


<!-- JSON ( MappingJackson2HttpMessageConverter) -->
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-core</artifactId>
  <version>2.6.3</version>
</dependency>

<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-annotations</artifactId>
  <version>2.6.3</version>
</dependency>

<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.6.3</version>
</dependency>

<dependency>
  <groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-asl</artifactId>
  <version>1.9.13</version>
</dependency>
```

### 2. logback 세팅

기존에 아래에 작성된 부분을 모두 삭제한다.

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>${org.slf4j-version}</version>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>jcl-over-slf4j</artifactId>
  <version>${org.slf4j-version}</version>
  <scope>runtime</scope>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>${org.slf4j-version}</version>
  <scope>runtime</scope>
</dependency>

<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>1.2.16</version>
</dependency>
```
그래서 위 부분을 아래 부분으로 대체한다.

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
    <version>1.7.7</version>
    <scope>runtime</scope>
</dependency>

<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-classic</artifactId>
	<version>1.1.3</version>
</dependency>
```
Spring에서 기본적으로 Commons Logging를 사용하므로 다음과 같이 commons-logging를 제외해준다.

```xml
<!-- Spring -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>${org.springframework-version}</version>
  <exclusions>
    <!-- Exclude Commons Logging in favor of SLF4j -->
    <exclusion>
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

### 3. logback.xml 생성

기존의 `src/main/resources/log4j.xml`이 있는 자리에 `logback.xml`를 만든다.
log4j.xml은 작업완료후 삭제하면 된다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
    <pattern>%d{HH:mm:ss.SSS} [%thread] %-4level [%logger.%method:%line]- %msg%n</pattern>
    </layout>
  </appender>  

    <appender name="LOGFILE" class="ch.qos.logback.core.rolling.RollingFileAppender">		
		<file>/home/mydir/logs/my-web.log</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
		<fileNamePattern>my-web.%d{yyyy-MM-dd}.log</fileNamePattern>
		<!-- 30일 지난 파일은 삭제한다.  -->
		 <maxHistory>30</maxHistory>
		</rollingPolicy>		
		<encoder>
		<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-4level [%logger.%method:%line] - %msg %n</pattern>
		</encoder> 			
	</appender>

    <!-- 	RULES for logging DEBUG < INFO < WARN < ERROR < FATAL.-->
    <logger name="myweb" additivity="false">
        <level value="INFO"/>
        <appender-ref ref="LOGFILE"/>
        <appender-ref ref="CONSOLE"/>
    </logger>    

  	<root>
		<level value="INFO" />
		<appender-ref ref="CONSOLE" />
	</root>

</configuration>
```
주의) additivity="false" 속성을 빠트리면 같은 로그가 두번씩 찍히는 문제가 발생한다.

### 4. java

import 부분만  org.slf4j로 바꾸어주면 된다. logback은 slf4j를 이용한다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

....
protected static Logger logger = LoggerFactory.getLogger("myweb");
```

### 5. 참고사이트 

logback에 대한 유용한 블로그는 아래와 같다.

- [logback을 사용해 보자](http://knot.tistory.com/92)
- [How to setup SLF4J and LOGBack in a web app - fast](https://goo.gl/YyB6Wx)
- [How to log in Spring with SLF4J and Logback](http://www.codingpedia.org/ama/how-to-log-in-spring-with-slf4j-and-logback/)
