---
layout: single
title: "Springboot Library 만들고 사용하기"
date: 2023-02-28
categories: [java]
tags: [spring]
---

스프링부트로 라이브러리를 만들어 보자. 그래서 다른 스프링부트에서 해당 라이브러리를 임포트해서 사용하도록 한다.

먼저 라이브러리 build.gradle에 아래와 같이 추가한다

```groovy

bootJar {
    enabled = false
}

jar {
    enabled = true
    archiveClassifier = '' //-plain 이름 제거
}
```

라이브러리는 직접 실행하는 하는 것이 아닌 jar형태이기 때문에 위와 같이 처리하고 archiveClassifier 는 jar로 만들때 이름에 -plain이라는 이름이 자동으로 붙는다. 그냥 두어도 상관없지만, 제거하고 싶을 경우에 사용하는 옵션이라고 보면 된다.

라이브러리는 간단히 Service에서 테스트용 메소드를 만든다.

```java

@Service
@Slf4j
public class TestLibService {
    public String testLibCall() {
        return "TestLibService.testLibCall()";
    }
}

```

빌드를 하면 jar파일이 만들어진다.

이제 실제 라이브러리를 사용하는 곳에 build.gradle에 implementation에 추가하도록 하자.

```groovy
  implementation files('C:\\workspace\\spring-test-lib\\build\\libs\\spring-test-lib-0.0.1-SNAPSHOT.jar')
```

실제로는 작성된 라이브러리는 넥서스등을 통해서 내부 레파지토리에 별도로 관리를 할 것이다. 지금은 테스트를 위해서는 직접 로컬에서 호출하는 것으로 처리한다. 참고로 윈도우일 경우 C는 반드시 대문자로 처리해야 인식한다.

임포트가 되면 아래처럼 보이게 된다.

![lib1](/assets/images/lib1.png)

실제 실제 라이브러리를 사용해보자.

```java
@Slf4j
@RestController
@RequiredArgsConstructor
public class TestMainController {

    private final TestLibService testLibService;

    @GetMapping("/test")
    public String test() {
        return testLibService.testLibCall();
    }
}
```

Controller에서 TestLibService를 이용해서 해당 메소드를 호출하는 것이다. 컴파일 오류도 없이 잘 구현된 것으로 보인다. 가동시켜보자

```java

***************************
APPLICATION FAILED TO START
***************************

Description:

Parameter 0 of constructor in com.example.springtest.TestMainController required a bean of type 'com.example.springtestlib.TestLibService' that could not be found.


Action:

Consider defining a bean of type 'com.example.springtestlib.TestLibService' in your configuration.

```

하지만 실제 가동하면 우리가 추가한 TestLibService를 찾을 수 없다고 나온다.

사실 당연한 결과이다 우리가 만든 라이브러리를 스프링컨테이너가 해당 빈을 등록하지 못했기 때문이다.

방법은 해당 라이브러리 경로를 추가해주면 된다.

```java
@ComponentScan(basePackages = {"com.example"})
```

이렇게 되면 문제가 해결되지만, 고민을 해보자. 매번 라이브러리를 만들때마다 사용하는 곳에서 @ComponentScan이나 @Import 등으로 처리해서 추가해야 하나? 그냥 라이브러리만 implementation 해서 사용하고 싶은데, 실제 우리가 Spring 관련 라이브러리를 사용할때 우리는 implementation 만 구현하고 바로 사용했다.

우리가 @SpringBootApplication 이라는 어노테이션을 사용해서 구동하는데 그 소스에는 아래와 같이 구성되어 있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

@EnbleAutoConfiguration 이라는 어노테이션이 보인다. 이 어노테이션은 spring.factories 안에 설정된 파일을 읽어들여 스프링 Bean으로 등록해주는 역할을 한다.

먼저 라이브러리 소스에 Configuration을 만든다.

```java
package com.example.springtestlib;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("com.example.springtestlib")
public class TestLibConfig {
}

```

해당 Configuration파일에 @ComponentScan을 해당 라이브러리 패키지 경로로 지정한다.

다음에 resources에 META-INF 라는 디렉토리를 만들고 spring.factories 라는 파일을 만들고 거기에 위에 설정파일을 적어준다.

```groovy
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.example.springtestlib.TestLibConfig
```

**[중요!]**

spring.factories가 deprecated 되었다고 한다. Spring 3.0 부터는 지원이 안된다고 한다. 따라서 3.0 이후에는 아래처럼 처리해야 한다.

아래 폴더에 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일 생성

> src/main/resources/META-INF/srping/org.springframework.boot.autoconfigure.AutoConfiguration.imports

그 파일안에 자동설정할 클래스를 작성한다.

> com.example.springtestlib.TestLibConfig

다시 빌드하고 사용하는 곳에서 빌드 리프레쉬하여 사용해보자. 이때 @ComponentScan(basePackages = {"com.example"}) 이부분은 이제 필요없다. 그냥 Implementaion만 구현되어 있으면 된다.

```java
@SpringBootApplication
public class SpringTestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringTestApplication.class, args);
    }
}
```

이제 실제 구현해보면 잘 수행이 된다. 즉 해당 서비스가 가동될때 라이브러리를 읽는데 이때 라이브러리에 spring.factories가 있다면 해당 파일을 읽어들여 스프링 컨테이너에 빈등록을 하게 된다. 이때 추가한 TestLibConfig가 설정빈으로 등록되면서 그안에 @ComponentScan으로 인해 우리가 추가한 라이브러리의 Service가 빈으로 등록되게 된다.

![lib2](/assets/images/lib2.png)

이렇게 처리하면 라이브러리를 사용하는 곳에서는 전혀 소스를 수정하거나 변경할 필요가 없고 그냥 implmenation만 처리하고 바로 사용하면 된다.
