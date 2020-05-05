---
layout: post
title:  "Spring에서 Device별(desktop, mobile, tablet)로 접근 구분하는 방법"
date:   2014-09-26
categories: java
---

`Spring`에서 간단하게 접속하는 `Device(desktop, mobile, tablet)`를 구분하는 방법이 있다.

### 1. 먼저 spring mobile 라이브러리를 받는다. `pom.xml`에 다음을 추가한다.

```xml
<dependency>
 <groupId>org.springframework.mobile</groupId>
 <artifactId>spring-mobile-device</artifactId>
 <version>1.1.0.RELEASE</version>
</dependency>
```

### 2. `action-servlet.xml`에서 아래 내용을 추가한다.

```xml
<mvc:interceptors>
  <beans:bean class="org.springframework.mobile.device.DeviceResolverHandlerInterceptor" />
</mvc:interceptors>
```

### 3. `web.xml`에서 필터링부분을  추가한다.

```xml
<filter>
  <filter-name>deviceResolverRequestFilter</filter-name>
  <filter-class>org.springframework.mobile.device.DeviceResolverRequestFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>deviceResolverRequestFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```

#4. Controller에서 Device 구분

```java
import org.springframework.mobile.device.Device;
import org.springframework.mobile.device.DeviceUtils;
@RequestMapping("/hello")
    public @ResponseBody String detectDevice(HttpServletRequest request) {        
        Device device = DeviceUtils.getCurrentDevice(request);        
        if (device == null) {
            return "device is null";
        }
        String deviceType = "unknown";
        if (device.isNormal()) {
            deviceType = "nomal";
        } else if (device.isMobile()) {
            deviceType = "mobile";
        } else if (device.isTablet()) {
            deviceType = "tablet";
        }
        return "Hello " + deviceType + " browser!";
    }
```
