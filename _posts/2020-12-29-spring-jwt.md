---
layout: post
title:  "Springboot에서 JWT 간단 사용하기"
date:   2020-12-29
categories: java
---

기존에 [Spring security JWT 연동](https://yookeun.github.io/java/2017/07/23/spring-jwt/)에 기술했던 내용에서는 spring aouth2를 활용해서 jwt연동을 수행했다. 하지만 aouth2 설정, private key, public key 생성등 복잡한 과정이 많았다. 단순하게 REST API서버에서 해당 암호키를 가지고 있고, 등록된 회원인증을 거쳐서 토큰을 발행하면 좀더 심플하게 처리가 가능하다.  기존의 Security 설정은 거의 동일하다. 

### 1. 회원 테이블 

```sql 
# user테이블 생성
CREATE table `users`
(
    `user_key` bigint NOT NULL AUTO_INCREMENT COMMENT '고유키',
    `user_id` varchar(50) NOT NULL COMMENT '계정',
    `password` varchar(100) NOT NULL COMMENT '패스워',
    `user_name` varchar(50) NOT NULL COMMENT '이름',
    `role` varchar(20) NOT NULL COMMENT '권한',
    `reg_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '등록일' ,
    PRIMARY KEY(`user_key`),
    UNIQUE KEY(`user_id`)
);

insert into users(user_id, password, user_name, role)
values('hong', '{bcrypt}$2a$10$5ueMHBZpCGZ9oesru.MQluiHxOLuMzAcmqHqrfier3ILUCxhiXNBm', '홍길동', 'ROLE_USER');
```

### 2. build.grade의 dependencies 정보 

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.apache.commons:commons-lang3:3.11'
    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'mysql:mysql-connector-java'
    annotationProcessor 'org.springframework.boot:spring-boot-configuration-processor'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
    implementation 'io.jsonwebtoken:jjwt-api:0.11.2'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.2',
            // Uncomment the next line if you want to use RSASSA-PSS (PS256, PS384, PS512) algorithms:
            //'org.bouncycastle:bcprov-jdk15on:1.60',
            'io.jsonwebtoken:jjwt-jackson:0.11.2' // or 'io.jsonwebtoken:jjwt-gson:0.11.2' for gson
}

```

`io.jsonwebtoken:jjwt-api` 라이브러리를 설정한다. 

### 3. 구현하려는 REST API 

| url                 | Param            | 설명                                                         |
| ------------------- | ---------------- | ------------------------------------------------------------ |
| GET /api/v1/hello   |                  | 토큰을 가지고 있으면 해당 API를 접근할 수 있는 유저만 호출이 가능하다. <br />호출이 성공되면 "Hello, {유저명}" response된다. |
| POST /auth/v1/login | userId, password | userId, password로 조회해서 등록된 회원이면 JWT 토큰을 리턴한다. |

ApiController에서 /api/v1/hello부분을 아래와 같이 단순하게 처리했다.

```java 
@RestController
@RequestMapping("/api/v1")
public class ApiController {

    @GetMapping("/hello")
    @ResponseBody
    public ResultJson hello(HttpServletRequest request) {
        ResultJson resultJson = new ResultJson();
        resultJson.setCode(ResultCode.SUCCESS.getCode());
        resultJson.setMsg("Hello, " + request.getSession().getAttribute("userId").toString());
        return resultJson;
    }
 }
```

이때 `/api/v1/hello/ 를 호출할때는 header부분에 jwt토큰을 파싱해서 정상적이면 위에 로직이 수행되면서 파싱할때 세션에 담은 userId를 리턴해주는 것이다. 이렇게 하기 위해서는 먼저 /api/~~를 호출할때 jwt필터를 처리해줄 필터클래스가 필요한데 그전에 jwt 토큰을 생성하고 파싱하는 유틸클래스를 만들어보자 

### 4. JwtUtil 생성 

아래소스는 아래 깃랩 소스를 참고한 것임을 밝힌다.

https://github.com/koushikkothagal/spring-security-jwt/blob/master/src/main/java/io/javabrains/springsecurityjwt/util/JwtUtil.java

```java 
package com.example.springjwt.util;

import com.example.springjwt.entity.Users;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.ExpiredJwtException;
import io.jsonwebtoken.JwtException;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.io.Encoders;
import io.jsonwebtoken.security.Keys;
import io.jsonwebtoken.security.SignatureException;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtUtil {
    @Value("${jwt.secret-key}")
    private String secretKey;


    private String createToken(Map<String, Object> claims) {
        String secretKeyEncodeBase64 = Encoders.BASE64.encode(secretKey.getBytes());
        byte[] keyBytes = Decoders.BASE64.decode(secretKeyEncodeBase64);
        Key key = Keys.hmacShaKeyFor(keyBytes);

        return Jwts.builder()
                .signWith(key)
                .setClaims(claims)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 24))
                .compact();

    }

    private Claims extractAllClaims(String token) {
        if (StringUtils.isEmpty(token)) return null;
        String secretKeyEncodeBase64 = Encoders.BASE64.encode(secretKey.getBytes());
        Claims claims = null;
        try {
            claims = Jwts.parserBuilder().setSigningKey(secretKeyEncodeBase64).build().parseClaimsJws(token).getBody();
        } catch (JwtException e) {
            claims = null;
        }
        return claims;
    }



    public String extractUsername(String token) {
        final Claims claims = extractAllClaims(token);
        if (claims == null) return null;
        else return claims.get("username",String.class);
    }

    public String generateToken(Users users) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", users.getUserId());
        return createToken(claims);
    }
}

```

`${jwt.secret-key}` 를 application.yml에 세팅한다. 비밀키로 이건 서버에만 있으니까 그냥 UUID으로 고정값으로 처리했다. 외부에 유출되지 않도록 해야 한다. 

```yml 
jwt.secret-key: c88d74ba-1554-48a4-b549-b926f5d77c9e
```

` .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 24))` 로 토큰 만료시간을 24시간으로 지정했다. createToken에서는 주어진 userId를 Jwt토큰의 subject에 담도록 처리했다.  이제 해당 유틸을 처리하는 필터클래스를 만들어보자.

파싱시에 `JwtException` 처리를 통해서 토큰이상여부(만료일 포함)를 체크하도록 한다. 

### 5. JwtRequestFilter 생성 

JwtRequestFilter는  url를 호출할때 필터링을 처리해주는 클래스이다. 해당 클래스는 `OncePerRequestFilter`를 상속받아 처리한다. 

> OncePerRequstFilter는 같은 요청에 대해서 단 한번만 처리가 수행되는 것을 보장하는 기반 클래스이다.

```java 
package com.example.springjwt.filter;

import com.example.springjwt.config.UserDetailService;
import com.example.springjwt.util.JwtUtil;
import com.example.springjwt.util.ResultCode;
import com.example.springjwt.util.ResultJson;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;

@Component
@RequiredArgsConstructor
public class JwtRequestFilter extends OncePerRequestFilter {
    private final JwtUtil jwtUtil;
    private final UserDetailService userDetailService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {

        String path = request.getRequestURI();

        //아래 경로는 이 필터가 적용되지 않는다.
        if (path.startsWith("/auth")) {
            filterChain.doFilter(request, response);
            return;
        }

        final String authorizationHeader = request.getHeader("Authorization");
        String username = null;
        String token = null;
        HttpSession session = request.getSession();

        //Header에서 Bearer 부분 이하로 붙은 token을 파싱한다.
        if (authorizationHeader != null && authorizationHeader.startsWith("Bearer ")) {
            token = authorizationHeader.substring(7);
        }
        username = jwtUtil.extractUsername(token);
        if (username == null) {
            exceptionCall(response, "invalidToken");
            return;
        }
        UserDetails userDetails = userDetailService.loadUserByUsername(username);
        if (SecurityContextHolder.getContext().getAuthentication() == null) {
            UsernamePasswordAuthenticationToken usernamePasswordAuthenticationToken
                    = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
            usernamePasswordAuthenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            SecurityContextHolder.getContext().setAuthentication(usernamePasswordAuthenticationToken);
            session.setAttribute("userId", username);
        }

        filterChain.doFilter(request, response);
    }

    private HttpServletResponse exceptionCall(HttpServletResponse response, String errorType) throws IOException {
        ResultJson resultJson = new ResultJson();
        if (errorType.equals("invalidToken")) {
            resultJson.setCode(ResultCode.INVALID_TOKEN.getCode());
            resultJson.setMsg(ResultCode.INVALID_TOKEN.getMsg());
        }

        ObjectMapper objectMapper = new ObjectMapper();
        response.getWriter().write(objectMapper.writeValueAsString(resultJson));
        response.setCharacterEncoding("utf-8");
        response.setContentType("application/json");
        return response;
    }
}
```

해당 필터의 내용을 보면 `/auth/**` 에서는 토큰 처리를 하지 않도록 한다. 당연히 로그인을 하지 않은 상태이므로 토큰이 없기 때문이다. 그래서 `/api/**` 에서만 토큰필터를 처리한다. Bearer로 시작된 토큰을 읽어서 username, 토큰정상여부(만료일등)등을 확인하는 검증을 거치고 비정상적인 토큰이라면 `invalid token`으로 리턴해주고 정상이면 세션값 userId에 username을 세팅한다. 

### 6. SecurityConfig 설정 

WebSecurityConfigurerAdapter를 상속받은 SecurityConfig를 설정하고 HttpSecurity부분을 아래와 같이 설정한다. 

```java 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                .antMatchers("/auth/**").permitAll()
                .antMatchers("/api/**").hasRole("USER")
                .and()
                .addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
    }
```

`/auth/**`는 누구나 접근이 가능하고 `/api/**` 는 해당유저가 `ROLE_USER`라는 권한이 있어야 접근이 가능하다. 그리고 위에서 만든 jwtRequestFilter를 등록했다. 

### 7. Controller 설정 

계정/패스워드를 검증해서 통과하면 토큰을 생성해주는 `AuthController`이다.

```java 
@RestController
@RequestMapping("/auth/v1")
@RequiredArgsConstructor
public class AuthController {
    private final UserRepository userRepository;
    private final JwtUtil jwtUtil;

    @PostMapping("/login")
    @ResponseBody
    public ResultJson login(@RequestParam(name = "userId", required = true) String userId,
                            @RequestParam(name = "password", required = true) String password) {

        ResultJson resultJson = new ResultJson();
        Users users = userRepository.findUserByUserId(userId);
        String token = "";
        PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
        if (users != null && passwordEncoder.matches(password, users.getPassword())) {
            token = jwtUtil.generateToken(users);
        } else {
            resultJson.setCode(ResultCode.LOGIN_FAIL.getCode());
            resultJson.setMsg(ResultCode.LOGIN_FAIL.getMsg());
            return resultJson;
        }
        resultJson.setCode(ResultCode.SUCCESS.getCode());
        resultJson.setMsg(ResultCode.SUCCESS.getMsg());
        resultJson.setToken(token);
        return resultJson;
    }
}
```

userId, password를 검증하고 통과하면 jwt토큰을 발행하고 리턴한다. 

다음은 토큰값과 접근권한이 있어야만 접근할 수 있는 `ApiController`이다. 

```java 
@RestController
@RequestMapping("/api/v1")
public class ApiController {

    @GetMapping("/hello")
    @ResponseBody
    public ResultJson hello(HttpServletRequest request) {
        ResultJson resultJson = new ResultJson();
        resultJson.setCode(ResultCode.SUCCESS.getCode());
        resultJson.setMsg("Hello, " + request.getSession().getAttribute("userId").toString());
        return resultJson;
    }
 }
```



### 8. 결과화면

POST /auth/v1/login에 유효한 계정과 패스워드로 전송하면 아래와 같이 JWT로 생성된 토큰이 만들어진다. 해당 토큰에는 username라는 json필드에 userId값을 세팅한 상태이다.

![jwt1](/assets/images/jwt1.png)

위에서 받은 토큰값을 이용해서 GET /api/v1/hello를 접근해본다.

![jwt2](/assets/images/jwt2.png)

만약 토큰이 없거나 올바르지 않거나 혹은 만료일이 지났거나 잘못된 토큰이라면 아래와 같이 에러로 표시된다.

![jwt4](/assets/images/jwt3.png)

해당 URL에 대한 ROLE(권한)이 다르다면 아래와 같이 에러 표시된다.

 ![jwt4](/assets/images/jwt4.png)

해당 전체 소스는 아래에서 확인할 수 있다. 

[https://github.com/yookeun/spring-jwt](https://github.com/yookeun/spring-jwt)