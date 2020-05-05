---
layout: post
title:  "Java application에서 Spring component scan을 사용하는 방법"
date:   2015-03-01
categories: java
---

Java 어플리케이션에서 (public static void main())에서 Spring으로 xml파일을 읽어들이고,
Component scan를 하는 방법을 설명한다.

먼저, `applicationContext.xml` 파일을 `src/main/resources` 안에 둔다고 하자. bean xmls에서 context 가 존재하여야 한다.
그리고 나서 scan 구문을 추가한다. (base-package부분에 scan할 경로를 기재한다)

```xml
<beans xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context" xmlns:jdbc="http://www.springframework.org/schema/jdbc" xmlns:lang="http://www.springframework.org/schema/lang" xmlns:oxm="http://www.springframework.org/schema/oxm" xmlns:tx="http://www.springframework.org/schema/tx" xmlns:util="http://www.springframework.org/schema/util" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://www.springframework.org/schema/beans" xsi:schemalocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.1.xsd
  http://www.springframework.org/schema/oxm http://www.springframework.org/schema/oxm/spring-oxm-4.1.xsd
  http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
  http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.1.xsd
  http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.1.xsd
  http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.1.xsd
  http://www.springframework.org/schema/lang http://www.springframework.org/schema/lang/spring-lang-4.1.xsd
  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.1.xsd">
</beans>

<context:annotation-config>
<context:component-scan base-package="book">
</context:component-scan></context:annotation-config>
```

BookMain 메인클래스가 존재하고 이 클래스에서 `applicaitonContext.xml`를 읽어들이고, BookController의 show()메소드를 호출하게 하는 것이 목적이다.
일단 BookController는 `@Controller` 어노테이션 추가하고 아래처럼 작성한다.

```java
package book.controller;

import org.springframework.stereotype.Controller;

@Controller
public class BookController {

 public void show() {
  System.out.println("I'm BookController");
 }
}
```

그리고 BookMain 클래스에서는 다음과 같이 작성한다.

```java
package book.thread;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Component;
import book.controller.BookController;

@Component
public class BookMain {

 private static final String[] SPRING_CONFIG_XML = new String[] {"applicationContext.xml"};
 public static void main(String[] args) {
  ApplicationContext ctx = new ClassPathXmlApplicationContext(SPRING_CONFIG_XML);
  BookMain main = ctx.getBean(BookMain.class);
  main.start();
 }

 @Autowired
 private  BookController bookController;
 private void start() {
  bookController.show();
 }
}
```

메인클래스에서 `@Component` 를 추가해준다.
그리고 나서 `ClassPathXmlApplicationContext` 를 이용해서 `ApplicationContext`를 읽어들인다.
이때, getBean()메소드를 통해서 현재 클래스를 빈으로 읽어들이고 (그래서 `@Component` 가 추가 됨)  

우리가 호출하려는 BookController를 `@Autowired` 한 다음에 main()메소드에서 bookController.show()를 호출하는
메소드를 처리하고자 하는 start()메소드안에 호출하도록 한다.

즉, Main 클래스가 로딩되면서 `ApplicationContext.xml` 파일을 읽어들이면서,
component scan를 한 다음에  자신을 Bean으로 처리하면서 `@Autowired` 된 bookController를 접근하는 start()메소드를 통해서
bookController.show()가 실행되는 것이다.

이 클래스를 실행하면 다음과 같이 표시된다.

> I'm BookController
