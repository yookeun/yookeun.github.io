---
layout: post
title:  "Spring에서 JSON과 연동방법"
date:   2014-11-21
categories: java
---

Spring에서 JSON과 REST API로 연결하는 방법은 두 가지가 있다.

### 1. MappingJackson2HttpMessageConverter 을 이용하여 세팅하는 방법

먼저 `pom.xml`에서 관련된 라이브러리를 다운받는다.

```xml
<!--  JSON 방식으로 통신 ( MappingJackson2HttpMessageConverter) -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.0.4</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.0.4</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.0.4</version>
</dependency>
```    

그리고 `servlet.xml`에 다음을 추가한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- Spring 과 json과의  연동 설정 -->   
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
    <list>    
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="supportedMediaTypes">
                <list>
                    <value>text/html;charset=UTF-8</value>
                    <value>application/json;charset=UTF-8</value>
                </list>
            </property>
        </bean>            
    </list>
    </property>                        
</bean>    
```

JSON으로 넘겨주는 JS파일은 아래와 같이 처리하면 된다.

```javascript
// JavaScript Code
var param = {
    "name": "홍길동",
    "age": 22    
};

$.ajax({
    type: 'POST',
    dataType: 'JSON',
    data: param,
    url:  'test/testing/select',
    error: function() {
    	//에러처리
    },                
    success: function(returnJSON) {
    	//성공처리
	}    
}):
```

### 2. MappingJackson2HttpMessageConverter을 이용하지 않고 JSON으로 연동하는 방법

`pom.xml`과 `server.xml`에 설정할 필요가 없이
JS파일에만 처리해주면 된다.

```javascript
// JavaScript Code
var param = {
    "name": "홍길동",
    "age": 22    
};

$.ajax({
    type: 'POST',
    dataType: 'JSON',
    data:  JSON.stringify(search),
    contentType:"application/json; charset=UTF-8",
    url:  'test/testing/select',
    error: function() {
        //에러처리
    },                
    success: function(returnJSON) {
    	//성공처리
	}    
}):
```

data부분을 JSON.stringfy로 변환해주고 `contentType`에 `application/json;charset=UTF-8`만 추가해주면 간단하게 처리된다.
