---
layout: post
title:  "Spring에서 스케줄러 사용하기"
date:   2015-01-16
categories: java
---

별도의 배치 프로그램을 만들지 않고 스프링에서 특정 시간에 반복으로 처리할 수 있는 스케줄링 기능이 있다.  


<http://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html>


위 사이트에 가보면 스케줄러에 대해 자세히 설명을 하였다(그렇지만 항상 원서는 너무 길고 복잡하다. 그냥 간단히 쓰고 싶다구!!!)  

자! 이제부터 초 간단 스케줄러를 만드는 방법을 설명한다. 더 이상의 자세한 설명은 생략한다.  
바로 `테스크(Task)`를 이용하는 방법이다.  
여기서는 애노테이션을 이용한 스케줄링을 사용하겠다 그게 제일 간단하다.


### 1. ApplicationContext.xml에 먼저 task를 beans에 정의하자.

`ApplicationContext.xml`에 beans xmlns에 추가한다

```xml
xmlns:tx="http://www.springframework.org/schema/tx"
xsi:schemaLocation="http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-3.2.xsd"
```

어노테이션를 추가한다.

```xml
<task:annotation-driven/>
```

### 2. Controller에 스케줄링 메소드를 등록한다.

```java
/**
 * 스케줄에 따라서 일정기간에 시작한다.
 * cron 스케줄 : 매일 자정에 (00:00) 에 한번 체크한다.
 */
@Scheduled(cron="0 0 0 * * ? ")
public void doSchedule() {
   logger.info("Spring Schedule Start!");
}
```

매우 간단하다. 스케줄로 실행될 메소드에 `@Scheduler`로 해주고
`cron`설정으로 값을 세팅해무면 된다. 예제에서는 매일 자정에 한번 실행이 된다.
(cron설정방법은  <http://en.wikipedia.org/wiki/Cron> 에서...)  
만약 30초마다 체크할 일이 있다면 `@Scheduled(fixedRate=30000)` 로 세팅해주면 된다.

이렇게 하면 간단히 스케줄링 메소드를 사용할 수 있다.
