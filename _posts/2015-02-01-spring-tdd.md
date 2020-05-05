---
layout: post
title:  "Spring에서 TDD를 이용하여 DB를 테스트해보자."
date:   2015-02-01
categories: java
---

Spring에서 TDD를 적용해본다.  

여기서는 db가 구축되어 있고, db조회, 입력, 처리부분을 TDD로 테스트할 경우를 예를 들겠다.  
(원래는 db도 DBUnit로 처리되어 있다면 아래처럼 할 필요는 없다. )

Spring에서 UI를 개발하기 전에 DB가 구축되어 있고, mybatis로 쿼리가 등록되어 있고, dao, service등의 클래스가 이미 만들어져 있어서
Controller에서  DB입력, 조회, 테스트만 하고 싶을 경우가 있다. 실제로 DB에 잘 입력이 되는지...

먼저 `src/src/java`에 TDD로 작성할 클래스를 선택하고 오른쪽 클릭 New>JUnit Test Case를 선택하면 아래의 창이 나온다.

<div style="text-align:center;margin-bottom: 30px;"><img src="/assets/images/tdd.jpg" style="width:100%"></div>  

`Source folder` 부분을 `Browse`버튼을 이용해서 `src/test/java`로 변경하고 `Finish`누르면
`src/test/java`안에 패키지가 생성되면서 해당클래스가 만들어진다.

```java
package com.yk.yboard.control;

import static org.junit.Assert.*;
import org.junit.Test;

public class YboardControllerTest2 {
    @Test
    public void test() {
        fail("Not yet implemented");
    }
}
```
pom.xml에 Spring UNIT 테스트 모듈을 dependency에 등록해야 한다.

```xml
<!--  Spring JUNIT 테스트  -->
<dependency>
 <groupid>org.springframework</groupid>
 <artifactid>spring-test</artifactid>
 <version>${org.springframework-version}</version>
</dependency>
```

version에 spring버전에 해당되는 값을 세팅한다. (3.2.8) 이렇게 하게 프로젝트명을 오른쪽클릭 후 `maven>Update Project..`를 클릭하면
Maven Dependencies에 `spring-test-.3.2.8.RELEASE.jar`가 받아진다.  
YboardControllerTest 클래스에 다음과 같이 애노테이션을 추가한다.

>@RunWith(SpringJUnit4ClassRunner.class) : JUnit Test 클래스를 실행하기 위한 러너(Runner)를 명시적으로 지정한다.


또한 ApplicationContext.xml파일을 별도로 Test용도로 만들어서 가져오자.

왜냐하면 mybatis-config.xml에 설정된 xml파일경로가 웹경로이기때문에 TDD에서는 톰켓가동없이 테스트가 목적이므로 별도로 분리해주어야 하기 때문이다.

# 톰켓용 ApplicationContext.xml에 configLocation 변경

```xml
<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
  <property name="dataSource" ref="dataSource">
  <property name="configLocation" value="/WEB-INF/sql/mysql/mybatis-config.xml">            
</property></property></bean>
```

# TDD용 ApplicationContext.xml configLocation 변경

```xml
<!-- 이부분은 원본과 다른 부분이다 -->
<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
   <property name="dataSource" ref="dataSource">
   <property name="configLocation" value="file:src/main/webapp/WEB-INF/sql/mysql/mybatis-config.xml">
</property></property></bean>  
```

실제 파일경로가 표시되어 있다. 이부분만 빼고 나머지는 ApplicationContext.xml과 동일하다. 클래스에 다음을 추가한다.

```xml
@ContextConfiguration(locations = { "/applicationContext_test.xml" })
```

다음은 트랜젝션을 걸어서 처리한다.  

```xml
@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = false)
@Transactional
```

defaultRollBack=true는 무조건 롤백이 된다. 데이터가 DB에 인서트된것을 확인하고자 하면 false로 해야 한다.

아래는 최종 클래스 소스이다.

```java
@RunWith(SpringJUnit4ClassRunner.class)
// applicationContext_test.xml는 applicationContext.xml의 복사본이다.
// src/test/resource에 넣는다.
// 복사본을 만든 이유는 sqlSessionFactory에 mybatis-config.xml의 경로설정이 tomcat가동할때가 틀리기
// 때문이다.

@ContextConfiguration(locations = { "/applicationContext_test.xml" })
// defaultRollBack=true는 무조건 롤백이 된다. 데이터가 DB에 인서트된것을 확인하고자 하면 false로 해야 한다.
@TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = false)
@Transactional
public class YboardControllerTest {
    @Autowired
    private YboardService yboardService;
    @Ignore
    @Test
    public void testViewYboard() {
        Map<string object=""> map = new HashMap<string object="">();
        map.put("boardID", 1);
        Yboard yboard = yboardService.viewYboard(map);
        assertEquals(yboard.getBoardID(), 1);
    }
}
```
이렇게 하면 TDD로 실제 DB 처리등을 테스트해볼수있다.

이것은 코드커버리지 100% TDD가 아니다. 따라서 DB입력, 조회등을 간단한 테스트용도로만 사용하고, Jenkins등에 maven 자동 테스트등으로 사용 해서는 안된다(입력/수정/삭제메소드가 있을 경우 실제 수행되기 때문이다) 따라서 다음과 같이 TDD 스킵이 되어야 한다(skipTests -> true)

```xml
<!-- TDD 스킵! -->
<plugin>
  <groupid>org.apache.maven.plugins</groupid>
  <artifactid>maven-surefire-plugin</artifactid>
  <version>2.15</version>
  <configuration>
    <skiptests>true</skiptests>
  </configuration>
</plugin>
```
아니면 사용하고나서 해당메소드에 `@Ignore`를 붙여도 된다.
