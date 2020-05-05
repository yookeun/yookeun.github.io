---
layout: post
title: "Spring Boot에서 배포환경 나누기"
date: 2017-04-08
categories: java
---

스프링부트에서 war로 만들고 각 서버에 톰켓으로 배포할 때 설정하는 방법을 알아보자. 
몇가지 방법이 있지만, 가장 간단하고 일반적인 방법으로 설정을 해보자. 스프링부트에서는 application.yml를 통해 아래와 같이 profile를 설정하는 방법이 있다.  배포환경을 나누는 이유는 DB설정과 로그파일경로 때문이다.  

### 1. application.yml에 profiles 구분 

```xml
# === 중략 ==== 
--- #구분     
spring:
  profiles:
  active: local  
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.jdbc.Driver   
    url: jdbc:mysql://localhost:3306/test_db?useUnicode=true&characterEncoding=utf8
    username: root
    password: 1234
    hikari:
      maximum-pool-size: 10
      max-lifetime: 30     

logging:
  file: D:/logs/jpa-example.log
  pattern:
    file: D:/logs/jpa-example.%d{yyyy-MM-dd}.log  
    
--- #구분

spring:
  profiles: dev
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.jdbc.Driver   
    url: jdbc:mysql://192.168.0.1:3306/test?useUnicode=true&characterEncoding=utf8
    username: root
    password: 1234
    hikari:
      maximum-pool-size: 10
      max-lifetime: 30      
  
logging:
 file: D:/logs_dev/jpa-example.log
 pattern:
   file: D:/logs_dev/jpa-example.%d{yyyy-MM-dd}.log  
```

소스를 보면 yml에서 구분자 `- - -` 를 통해서 profile를 구분하고 있다. 이때 spring.profile.active 부분이 있는데 active부분은 한 곳만 지정해야 한다. 일정의 디폴트개념으로 스프링부트에서 실행옵션이 없을 경우 active로 설정된 profile를 먼저 읽게 된다. 

### 2. log 경로 설정 

spring boot에서는 기본적으로 logback를 사용한다. 우리는 여기서 서버별로 분리를 할 예정이니 `logback-spring.xml`를 만들고 application.yml를 읽어들여 처리하는 방법을 이용한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.classic.PatternLayout">
    <pattern>%d{HH:mm:ss.SSS} [%thread] %-4level [%logger.%method:%line]- %msg%n</pattern>
    </layout>
  </appender>  
    <appender name="LOGFILE" class="ch.qos.logback.core.rolling.RollingFileAppender">		
		<file>${LOG_FILE}</file>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
		<fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.log</fileNamePattern>
		<!-- 30일 지난 파일은 삭제한다.  -->
		 <maxHistory>30</maxHistory>
		</rollingPolicy>		
		<encoder>
		<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-4level [%class{0}.%method:%line] - %msg %n</pattern>
		</encoder> 			
	</appender>

    <!-- 	RULES for logging DEBUG < INFO < WARN < ERROR < FATAL.-->
    <logger name="JPA-EXAMPLE" additivity="false">
        <level value="DEBUG"/>
        <appender-ref ref="LOGFILE"/>
        <appender-ref ref="CONSOLE"/>
    </logger>    

  	<root>
		<level value="INFO" />
        <appender-ref ref="LOGFILE"/>
		<appender-ref ref="CONSOLE" />
	</root>

</configuration>
```

`${LOG_FILE}` 설정을 통해 배포 및 실행할 때 해당 profiles 정보로 세팅된다. 

### 3. pom.xml 설정 

spring boot에서 war로 만들고 톰켓에서 배포하려면 추가적인 설정이 필요하다. 

```xml
<packaging>war</packaging>
```

packaging 부분을 war로 변경한다. 다음은 톰켓 배포용 라이브러리를 추가한다. 

```xml
<!-- war 배포용 추가  -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>	
```

### 4. SpringBootServletInitializer 구현 클래스 작성  

스프링부트에서는 embeded로 톰켓이 내장되어 있어서 `@SpringBootApplication`를 구현한 클래스만 있으면 되지만, 우리는 톰켓으로 배포하고 실행되어야 하므로 `SpringBootServletInitializer` 를 상속받은 추가적인 클래스가 필요하다. 

```java
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

public class ServletInitializer extends SpringBootServletInitializer {
	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {		
		return application.sources(SpringJpaExampleApplication.class);
	}
	
}
```

보통 eclipse등에서 부트를 실행할때  jvm옵션으로 `-Dspring.profiles.active={프로파일명}` 등을 주어서 실행하면 각각의 설정을 읽어들이게 되는 것이다. logging도 마찬가지로 분류해서 넣어주면 된다. 즉 아래처럼 실행 및 패키징을 만들면 된다. 

```bash
mvn spring-boot:run -Dspring.profiles.active-dev
mvn package -Dspring.profiles.active=dev
```

### 5. Tomcat에서 설정 

이제 각각 서버의 톰켓으로 배포해야 한다. 스프링레거시에서는 메이븐을 이용하면 각각의 resource파일을 분류해서 패키징할 수 있다. 하지만 스프링부트에서는 application.yml에 설정파일을 하나로 처리했다면 실행옵션을 주어서 톰켓에서 deploy되어야 한다. 따라서 톰켓에서 `-Dspring.profiles.active=dev`등을 처리할 수 있도록 해야 한다. 

톰켓의 실행경로 `{tomcat_dir}/bin` 안에 `setenv.sh`를 만든다. 윈도우라면 `setenv.bat` 를 만든다. 그리고 OS에 맞게 다음을 기재한다. 

```bash
(리눅스) export JAVA_OPTS="$JAVA_OPTS -Dspring.profiles.active=dev"
(윈도우) set JAVA_OPTS=%JAVA_OPTS% -Dspring.profiles.active=dev
```

이것으로 설정은 완료되었다. 이제 각각 서버의 톰켓으로 배포하면 끝이다. 