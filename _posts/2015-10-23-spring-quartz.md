---
layout: post
title:  "Spring과 Quartz 연동"
date:   2015-10-23
categories: java
---

Spring과 Quartz를 연동해서 개발해본다.
Spring 3.2 와 Quartz 1.8을 연동한다. (Quartz 2.x 버전은 Spring 3.2 버전과 잘 안되므로 1.8로 한다)

### 1. pom.xml 에 추가

pom.xml에서 quartz와 연동하기 위해서 이클립스에서 생성된 spring 모듈이외에 다음의 spring 모듈이 필요하고, quartz를 추가한다.

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context-support</artifactId>
  <version>${org.springframework-version}</version>
</dependency>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-tx</artifactId>
  <version>${org.springframework-version}</version>
</dependency>

<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz</artifactId>
  <version>1.8.6</version>
</dependency>

```

### 2. QuartzJobBean클래스를 상속받는 클래스 작성

우리는 quartz에서 Job클래스를 만들고 그 클래스에서 Spring Controller 클래스에 접근하여 해당 메소드를 수행하도록 한다.  

Controller 클래스를 다음과 같이 작성한다.

```java
@Controller
public class TestController {
	@Autowired
	private TestService testService;

	public void showMe(String msg) {
		testService.showMe(msg);
	}
}
```

다음은 Job 클래스를 만들어보자.
Quartz에서 Job를 생성하려면 `QuartzJobBean`를 상속해야 한다.
그런데 job1.class, job2.class 등 여러개의 Job 클래스를 만드는 경우가 많으므로 공통의 추상클래스를 만들어주고 Job 클래스에서 상속받아 처리하기 하자.



```java
public abstract class MyAbstractJob  extends QuartzJobBean {

	private ApplicationContext ctx;
	private TestController testController;

	@Override
	protected void executeInternal(JobExecutionContext context) throws JobExecutionException {		
		ctx = (ApplicationContext)context.getJobDetail().getJobDataMap().get("applicationContext");		
		jobInit(context);
	}

	private void jobInit(JobExecutionContext context) {
		testController = (TestController) ctx.getBean("testController");
		executeJob(testController);
	}

	protected abstract void executeJob(TestController testController);
}
```



`MyAbstractJob`라는 추상클래스를 만들었다. `TestController`는 스프링 Controller이다.  quartz에서 만든 Job클래스에서 스프링 controller 클래스로 접근하여 작업을 하고자 한다.
즉, applicationContext.xml에 bean 구현된 'testController'를 통해 접근이 가능하게 한다. 모든 Job클래스에서 같은 부분을 사용하니까 추상클래스에서 만들어주고
Job 클래스는 `executeJob` 만 각각 용도에 맞게 처리하면 된다.

```java
public class MyJob1 extends MyAbstractJob {
	@Override
	protected void executeJob(TestController testController) {
		testController.showMe("MyJob1 called ");			
	}
}
```

```java
public class MyJob2 extends MyAbstractJob {
	@Override
	protected void executeJob(TestController testController) {
		testController.showMe("MyJob2 called ");			
	}
}
```


### 3. xml 설정
스프링에서 사용하는 applicationContext.xml 만든다. 그리고 Job 클래스에 접근할 controller를 등록해준다.

```xml
<bean id="testController" class="test.control.TestController"/>
```

다음 해당 Job 클래스를 등록해준다.

```xml
<bean id="myjob1" class="org.springframework.scheduling.quartz.JobDetailBean">
  <property name="jobClass" value="test.main.MyJob1" />
  <property name="applicationContextJobDataKey" value="applicationContext" />
</bean>

<bean id="myjob2" class="org.springframework.scheduling.quartz.JobDetailBean">
  <property name="jobClass" value="test.main.MyJob2" />
  <property name="applicationContextJobDataKey" value="applicationContext" />
</bean>		
```

그 다음에는 해당 Job를 Cron 방식의 스케줄을 등록해주면 된다.

```xml
<bean id="cronTrigger1" class="org.springframework.scheduling.quartz.CronTriggerBean">
  <property name="jobDetail" ref="myjob1" />		
  <!-- 매 10초마다 실행 -->
  <property name="cronExpression" value="*/10 * * * * ?"/>
</bean>

<bean id="cronTrigger2" class="org.springframework.scheduling.quartz.CronTriggerBean">
  <property name="jobDetail" ref="myjob2" />
  <!-- 매 15초마다 실행 -->
  <property name="cronExpression" value="*/15 * * * * ?" />
</bean>

<bean id="scheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
  <property name="triggers">
    <list>
      <ref bean="cronTrigger1" />
      <ref bean="cronTrigger2" />
    </list>
  </property>
</bean>
```

### 4. 실행

Main 클래스에서 반드시 ApplicationContext.xml를 읽어들여야 한다.

```java
public class QuartzMain {
	public static void main(String[] args) {		
		// 설정 xml를 읽어 들인다.
		ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
		System.out.println("QuartzMain Start!");  
	}
}
```
실행하면 각각의 Job 클래스과 Cron 설정에 맞게 실행됨을 볼 수 있다.


### 5. 추가사항 : Spring 4.1 과 Quartz 2.2 를 사용할 경우

xml에서 Quartz 관련 설정만 변경해주면 된다.

`JobDetailBean -> JobDetailFactoryBean`  

`CronTriggerBean -> CronTriggerFactoryBean`


샘플소스 : <https://github.com/yookeun/spring-quartz-test>
