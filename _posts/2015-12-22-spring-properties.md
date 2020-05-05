---
layout: post
title:  "Spring에서 Properties 사용"
date:   2015-12-22
categories: java
---
Spring에서 Properties파일을 읽어들이는 부분을 처리할 수 있다.
먼저 application.xml에 다음과 같이 추가한다.

```xml
//beans  선언에  util의 선언되어야 한다.
<beans xmlns=
  .....
  xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
  ....  
/>
```

Properties파일의 경로를 설정한다.

```xml
<util:properties id="props" location="classpath:/properties/hello.properties" />     
```

여기서 classpath는 src/main/resources로 설정된 부분이다.
hello.properties 파일안에 아래와 같은 설정이 있다.

```bash
hello.number=12345
```

이제 실제 소스에서 사용해보자.
Spring Controller소스에서 사용하려면 다음과 같이 적는다.

```java
//Controller  
@Value("#{props['hello.number']}")
private String helloNumber;
```

@Value("#{properties의 아이디['properties파일내의 해당변수명']}")
이렇게 하면 helloNumber에 12345라는 값이 할당되어 진다.
