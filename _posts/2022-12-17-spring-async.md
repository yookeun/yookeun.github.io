---
layout: single
title: "Spring에서 @Async 사용"
date: 2022-12-17
categories: [java]
tags: [spring, async]
---

Spring에서는 비동기 처리를 @Async 어노테이션을 통해 간단히 처리할 수 있다.

별도로 설정하지 않고 application 실행 클래스나 configuration 클래스에 `@EnableAsync` 만 추가해주면 바로 사용이 가능하다.

```java
@Configuration
@EnableAsync
public class AsyncConfig  {
}
```

그러나 이렇게 사용하게 되면 spring에서 제공하는 기본 `SimpleAsyncTaskExecutor` 를 사용하게 되고 이것은 요청이 오는 대로 스레드를 생성한다. 즉 스레드 관리가 되지 않는다는 것이다. 그래서 보통은 아래처럼 처리한다.

```java
@Configuration
@EnableAsync
public class AsyncConfig extends AsyncConfigurerSupport {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(50); //기본 스레드 수
        executor.setMaxPoolSize(100); //최대 스레드 수
        executor.setQueueCapacity(200); // Queue 사이즈
        executor.setThreadNamePrefix("ASYNC-TEST");
        executor.initialize();
        return executor;
    }
}

```

`AsyncConfigurerSupport` 를 상속받거나 혹은 Excutor를 @Bean으로 생성해서 사용하기도 한다.

-   setCorePoolSize : 기본 대기 스레드 수
-   setMaxPoolSize: 동시 동작하는 최대 스레드 수
-   setQueueCapacity: core 사이즈보다 많은 요청이 올 경우 증가되는 큐 사이즈

이렇게 되면 비동기 스레드를 적절히 상황에 맞게 관리할 수 있다.

실제 구현을 해보자

```java
@Service
@Transactional(readOnly = true)
@Slf4j
public class TestService {

    public void callNormalAndAsync1() {
        normalMethod1();
        asyncMethod1();
    }

    public void normalMethod1() {
        log.info("[NORMAL] normalMethod1 Called!");
    }

    @Async
    public void asyncMethod1() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        log.info("[ASYNC] asyncMethod1");
    }
}
```

Serivce에서 우리는 일반 메소드(normalMethod1), 비동기 메소드(asyncMethod1)로 나누고 이 둘을 순서대로 호출하기 위해서 `callNormalAndAsync1`를 만들고 이것을 Controller로 호출하도록 해보자.

```java
    @GetMapping("/test/async1")
    public void async1() {
        log.info("========= async1 start ==========");
        testService.callNormalAndAsync1();
        log.info("========= async1 end ==========");
    }
```

아래 수행 결과이다.

```bash
2022-12-17 17:19:54.400  INFO 23224 --- [nio-8080-exec-6] c.e.s.controller.TestController          : ========= async1 start ==========
2022-12-17 17:19:54.401  INFO 23224 --- [nio-8080-exec-6] c.e.springasynctest.service.TestService  : [NORMAL] normalMethod1 Called!
2022-12-17 17:19:59.408  INFO 23224 --- [nio-8080-exec-6] c.e.springasynctest.service.TestService  : [ASYNC] asyncMethod1
2022-12-17 17:19:59.408  INFO 23224 --- [nio-8080-exec-6] c.e.s.controller.TestController          : ========= async1 end ==========
```

문제가 발생했다. normalMethod1 호출이 되고 바로 async1 end 로 되어야 하는데 asyncMethod1가 호출이 되었다. asyncMethod1는 5초 후에 실행되는데 결국 비동기기 처리되지 않은 것이다.

실제로 asyncMethod1를 호출한 스레드를 보면 우리가 AsyncConfig에 정한 스레드가 아닌 스프링 실행 스레드로 처리됨을 확인할 수 있다.

그렇다면 Controller에서 아래처럼 직접 순서를 정하면 어떻게 될까?

```java
    @GetMapping("/test/async2")
    public void async2() {
        log.info("========= async2 start ==========");
        testService.normalMethod1();
        testService.asyncMethod1();
        log.info("========= async2 end ==========");
    }

```

수행 결과

```java
2022-12-17 17:30:19.089  INFO 19132 --- [nio-8080-exec-2] c.e.s.controller.TestController          : ========= async2 start ==========
2022-12-17 17:30:19.106  INFO 19132 --- [nio-8080-exec-2] c.e.springasynctest.service.TestService  : [NORMAL] normalMethod1 Called!
2022-12-17 17:30:19.109  INFO 19132 --- [nio-8080-exec-2] c.e.s.controller.TestController          : ========= async2 end ==========
2022-12-17 17:30:24.125  INFO 19132 --- [    ASYNC-TEST1] c.e.springasynctest.service.TestService  : [ASYNC] asyncMethod1
```

이제 정상이다. normalMethod1가 수행되고 종료되면서 5초후에 asyncMethod1가 수행되었다. 실제 스레드명( ASYNC-TEST1])을 보면 알 수 있다.

> @Async 메소드는 내부 메소드로 사용하면 안된다. 당연히 private로 사용할 수도 없다.

그래서 Controller에서 직접 호출하거나 Async 메소드를 별도로 구현한 Service로 분리하는 것이다.

```java
@Service
@Slf4j
public class AsyncSerivce {
    @Async
    public void asyncMethod2() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        log.info("[ASYNC] asyncMethod2");
    }
}
```

TestService

```java
 private final AsyncSerivce asyncSerivce;
... 중략 ...

    public void callNormalAndAsync2() {
        normalMethod1();
        asyncSerivce.asyncMethod2();
    }
```

수행 결과

```bash
2022-12-17 17:38:31.379  INFO 7892 --- [nio-8080-exec-5] c.e.s.controller.TestController          : ========= async3 start ==========
2022-12-17 17:38:31.379  INFO 7892 --- [nio-8080-exec-5] c.e.springasynctest.service.TestService  : [NORMAL] normalMethod1 Called!
2022-12-17 17:38:31.381  INFO 7892 --- [nio-8080-exec-5] c.e.s.controller.TestController          : ========= async3 end ==========
2022-12-17 17:38:36.397  INFO 7892 --- [    ASYNC-TEST1] c.e.s.service.AsyncSerivce               : [ASYNC] asyncMethod2
```

정상적으로 비동기로 처리되었다. 가급적 비동기 메소드는 별도의 Service로 분리해서 사용하는 것이 위와 같은 실수를 줄이는 좋은 방법이라 할 수 있다.

[spring-async-test](https://github.com/yookeun/spring-async-test)
