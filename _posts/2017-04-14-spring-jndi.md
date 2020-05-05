---
layout: post
title: "Spring에서 JNDI설정(hikaricp)"
date: 2017-04-14
categories: java
---

스프링에서 jndi를 통해 mysql과 연결하자. 

### 1. 해당 라이브러리를 tomcat/lib에 넣어준다.

- mysql-connector-java-5.1.34.jar
- slf4j-api-1.6.6.jar
- HikariCP-2.6.0.jar

### 2. Tomcat의 Server.xml에 추가  

`hikari`를 이용하자. 

```xml
<GlobalNamingResources>
<Resource auth="Container" description="User database that can be updated and saved" factory="org.apache.catalina.users.MemoryUserDatabaseFactory" name="UserDatabase" pathname="conf/tomcat-users.xml" type="org.apache.catalina.UserDatabase"/>

 <Resource name="jdbc/testdb" auth="Container"
		        factory="com.zaxxer.hikari.HikariJNDIFactory"
		        type="javax.sql.DataSource"	        
		        minimumIdle="5" 
		        maximumPoolSize="10"
		        connectionTimeout="300000"		        
		        driverClassName="com.mysql.jdbc.Driver"
		        jdbcUrl="jdbc:mysql://127.0.0.1:3306/test_db"	        
		        username="root"
		        password="1234"
	        />		

  </GlobalNamingResources>
```

### 3. Tomcat의 Context.xml에 추가 

```xml
<Context>
 <ResourceLink global="jdbc/testdb" name="jdbc/testdb" type="javax.sql.DataSource"/>
</Context>
```

### 4. Spring Project에서 Bean 추가  

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="configLocation" value="classpath:mybatis-config.xml" />
  <property name="mapperLocations" value="classpath:mapper/*.xml" />
  <property name="dataSource" ref="dataSource"/>
</bean>
	
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>		
	
 <bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
	<property name="jndiName">
		<value>java:comp/env/jdbc/testdb</value>
	</property>
</bean>
```

### 5. Spring Project의 web.xml에 추가 

```xml
<!-- JNDI 설정  -->
<resource-ref>
<description>test</description>
	<res-ref-name>jdbc/testdb</res-ref-name>
	<res-type>javax.sql.DataSource</res-type>
	<res-auth>Container</res-auth>
</resource-ref>
```



