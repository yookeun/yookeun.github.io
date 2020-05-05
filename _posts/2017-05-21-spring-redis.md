---
layout: post
title: "Spring에서 Redis 설정"
date: 2017-05-21
categories: java
---

Spring(boot)에서 Redis를 설정하는 것은 무척 간단하다. 레디스를 이용해서 간단하게 페이지 방문자수를 업데이트해주는 것을 만들어보자.  

먼저 메이븐에서 먼저 관련 라이브러리를 추가해주자.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

`application.yml` 에 레디스 url, port등을 입력하자

```yaml
server:
  port: 8080

spring:
  profiles:
    active:
    - local
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp    
  redis:
    host: 127.0.0.1
    port: 6379
```

설정은 이것으로 끝났다. 이제 스프링에서 레디스를 읽어올 Java Config를 만든다.

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;


@Configuration
public class RedisConfig {
	
	private @Value("${spring.redis.host}") String redisHost;
	private @Value("${spring.redis.port}") int redisPort;

	@Bean
	public JedisConnectionFactory connectionFactory() {				
		JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory();
		jedisConnectionFactory.setHostName(redisHost);
		jedisConnectionFactory.setPort(redisPort);
		jedisConnectionFactory.setUsePool(true);		
		return jedisConnectionFactory;
	}
	
	@Bean
	public RedisTemplate<String, Object> redisTemplate() {
		RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
		redisTemplate.setKeySerializer(new StringRedisSerializer());
		redisTemplate.setValueSerializer(new StringRedisSerializer());
		redisTemplate.setConnectionFactory(connectionFactory());		
		return redisTemplate;
	}
}
```

`jedisConnectionFactory` 를 통해서 Redis 컨넥션을 관리해준다. 

`RedisTemplate` 를 이용해서 실제 레디스를 스프링에서 사용하는데 중요한 것은 `setKeySerializer()`, `setValueSerializer()`  메소드들이다. 이 메소드를 빠트리면 실제 스프링에서 조회할 때는 값이 정상으로 보이지만  `redis-cli`로 보면 key값에 \xac\xed\x00\x05t\x00\x0 이런 값들이 붙는다. ([관련 내용](http://stackoverflow.com/questions/31608394/get-set-value-from-redis-using-redistemplate))

이제 Redis를 사용하는 Service를 만들어보자 

```java
import javax.annotation.Resource;

import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Service;

@Service
public class RedisService {
	
	@Resource(name = "redisTemplate")
	private ValueOperations<String, String> valusOps;	
	
	public Long getVisitCount() {
		Long count = 0L;
		try {
			valusOps.increment("spring:redis:visitcount", 1);		
			count = Long.valueOf(valusOps.get("spring:redis:visitcount"));
		} catch (Exception e) {
			System.out.println(e.toString());
		}
		return count;
	}
}
```

`RedisTemplate`에서 필요한 타입을 가져오면 된다. 레디스에서 숫자 증가를 처리할 수 있는 `incr`를 사용하기 위해서 `ValueOperations`를 이용한다.  이렇게 각각 레디스에 대응되는 타입이나 메소드를 [RestTemplet 레퍼런스](http://docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/core/RedisTemplate.html)에서 쓰고 그에 관련된 메소드를 만들면 편하게 사용할 수 있다. 



이제 `Controller`에서 가져다 쓰면된다. 

```java
@Controller
public class HomeController {

	@Autowired
	private RedisService redisService;
	
	@RequestMapping(value = "/", method = RequestMethod.GET)
	public ModelAndView index() {
		ModelAndView model = new ModelAndView("index");
		model.addObject("count", redisService.getVisitCount());
		return model;
	}
}
```

이제 페이지에 접근할때마다 방문자수가 증가한다. 