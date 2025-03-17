---
layout: single
title: "JWT Refresh Token 사용하기"
date: 2023-12-30
categories: [java]
tags: [java, jwt]
---

Springboot에서 JWT를 이용해서 토큰을 발급받는데 refresh token도 발급받아서 사용해보자.

기본 절차는 다음과 같다.

1. 로그인해서 인증이 허가되면 accessToken, refreashToken, userId를 리턴받는다.
2. accessToken의 만료기간은 30분으로 하고 refreshToken의 만료기간은 7일로 한다.
3. accessToken의 만료되면 클라이언트는 이전에 발급받은 refreshToken으로 accessToken을 갱신받는다.
4. refreshToken이 만료되었다면 클라이언트는 다시 로그인 과정을 수행한다.

Refresh Token과 Access Token은 모두 redis로 관리하여 보다 빠른 성능으로 처리하게 한다.

### Redis Setting

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

Spring에서 제공하는 redis 라이브러리를 사용한다.

yml 에 설정을 한다.

```yaml
spring:
    data:
        redis:
            host: localhost
            port: 16694
```

그리고 Redis를 사용할 Configuration를 작성한다.

```java
@Configuration
@EnableRedisRepositories
@RequiredArgsConstructor
public class RedisConfig {

    private final RedisProperties redisProperties;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration configuration = new RedisStandaloneConfiguration();
        configuration.setHostName(redisProperties.getHost());
        configuration.setPort(redisProperties.getPort());
        configuration.setPassword(redisProperties.getPassword());
        return new LettuceConnectionFactory(configuration);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setDefaultSerializer(new StringRedisSerializer());
        return redisTemplate;
    }

}
```

다음은 accessToken과 refreshToken를 생성해주는 메소드를 만들어서 유틸이나 핸들러등으로 작성한다.

```java
private String createToken(Map<String, Object> claims) {
    String secretKeyEncodeBase64 = Encoders.BASE64.encode(jwtSecretKey.getBytes());
    byte[] keyBytes = Decoders.BASE64.decode(secretKeyEncodeBase64);
    Key key = Keys.hmacShaKeyFor(keyBytes);

    return Jwts.builder()
            .signWith(key)
            .setClaims(claims)
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 30))  //30분
            .compact();
}

private String createRefreshToken(Map<String, Object> claims) {
    String secretKeyEncodeBase64 = Encoders.BASE64.encode(jwtSecretKey.getBytes());
    byte[] keyBytes = Decoders.BASE64.decode(secretKeyEncodeBase64);
    Key key = Keys.hmacShaKeyFor(keyBytes);

    return Jwts.builder()
            .signWith(key)
            .setClaims(claims)
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 24 * 7)) //7일
            .compact();
}
```

추가로 jwt 토큰을 파싱 처리해주는 메소드도 추가한다. 각각 토큰성공, 토큰만료, 토큰 Validation를 체크하도록 한다.

```java
    public JwtResult extractAllClaims(String token) {
        if (StringUtils.isEmpty(token)) return null;
        SignatureAlgorithm sa = SignatureAlgorithm.HS256;
        SecretKeySpec secretKeySpec = new SecretKeySpec(jwtSecretKey.getBytes(), sa.getJcaName());
        Claims claims;
        JwtResult jwtResult = new JwtResult();
        try {
            claims = Jwts.parserBuilder().setSigningKey(secretKeySpec).build()
                    .parseClaimsJws(token).getBody();            ;
            jwtResult.setJwtResultType(JwtResultType.TOKEN_SUCCESS);
            jwtResult.setClaims(claims);

        } catch (ExpiredJwtException e) {
            jwtResult.setJwtResultType(JwtResultType.TOKEN_EXPIRED);
            jwtResult.setClaims(null);

        } catch (JwtException e) {
            jwtResult.setJwtResultType(JwtResultType.TOKEN_INVALID);
            jwtResult.setClaims(null);
        }
        return jwtResult;
    }
```

토큰처리 결과에 대한 Enum를 만들면 편하게 사용할 수 있다.

```java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class JwtResult {

    private JwtResultType jwtResultType;
    private Claims claims;


    public enum JwtResultType {
        TOKEN_EXPIRED,
        TOKEN_INVALID,
        TOKEN_SUCCESS,
        UNUSUAL_REQUEST
    }
}
```

이렇게 설정해서 로그인 성공시에는 아래와 같이 리턴되도록 처리한다.

```json
{
    "userId": "admin",
    "name": "ADMIN",
    "password": "$2a$10$rJThj2spTwIY0/v1bkYSNOgjTug5zatsuXZ.RmrWbSIHJUiYBlaOW",
    "accessToken": "eyJhbGciOiJIUzI1NiJ9.eyJhdXRob3JpdGllcyI6IkFETUlOLE9SREVSLElURU0iLCJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjk3MTg0NTY4LCJleHAiOjE2OTcxODYzNjh9.y958F4_oPtvPZyqlsgVC8D9RfLWzTTQAQ6a8Hk4_vog",
    "refreshToken": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNjk3MTg0NTY4LCJleHAiOjE2OTc3ODkzNjh9.8VqAmsMG8euHeyNVHRU0oFqvsc2iB7YktsnW6DRaLTU"
}
```

클라이언트에게 위와 같은 리턴메시지를 전달하기 전에 위 정보를 redis에 저장해서 관리하도록 한다. Redis에서는 key, value방식으로 저장되기 때문에 `key = userId, value = {accessToken, refreshToken}` 식으로 저장하도록 한다.

**\*ex) key = “kim01”, value = {“accessToken: “12345A”, refreshToken: “45678B”}\***

위 과정은 다음의 프로세스를 거친다.

1. 로그인 성공하면 refreshToken, accessToken, userId로 redis에 저장한다. 이때 클라이언트가 로그인할 때마다 refreshToken과 accessToken은 새로 발급되어 userId 키값으로 저장된다.
2. 만약 refreshToken를 유출되었더라도 사용자가 다시 로그인하면 refreshToken이 새로 생성되기 때문에 유출된 refreshToken은 사용할 수 없다.

사용자 토큰 처리 객체

```java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserTokenInfo {

    private String userId;
    private String accessToken;
    private String refreshToken;

}


```

Redis에 저장한다. 이때 redis에 저장되는 key값도 만료값을 지정한다(redisTemplate.expire) 여기선 7일로 설정했다. 이렇게 하면 클라이언트가 로그인을 하지 않더라도 7일이 지나면 redis에서 해당키(userId)가 삭제되기 때문에 다시 로그인해야 하다.

```java
private void saveRedis(UserTokenInfo userTokenInfo) throws JsonProcessingException {
    ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
    ObjectMapper objectMapper = new ObjectMapper();
    valueOperations.set(userTokenInfo.getUserId(), objectMapper.writeValueAsString(userTokenInfo));
    redisTemplate.expire(userTokenInfo.getUserId(), 7, TimeUnit.DAYS);
}
```

로그인을 새로 하면 기존 토큰을 갱신한다.

```java
public void updateRedis(UserTokenInfo userTokenInfo) throws JsonProcessingException {
    ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
    ObjectMapper objectMapper = new ObjectMapper();
    valueOperations.set(userTokenInfo.getUserId(), objectMapper.writeValueAsString(userTokenInfo));
}
```

토큰이 문제가 있을 경우 redis에서 해당 키에 해당되는 토큰을 모두 삭제한다. 이렇게 되면 클라이언트는 다시 로그인 절차를 수행해야 한다.

```java
public void deleteRedis(String key) {
    ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
    valueOperations.getAndDelete(key);
}
```

userId를 통해서 redis에 저장된 accessToken, refreshToken를 가져올 수 있다.

```java
private Optional<UserTokenInfo> findByRefreshToken(String userId)
        throws JsonProcessingException {
    ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
    String json = (String) valueOperations.get(userId);

    if (StringUtils.isNullOrEmpty(json)) {
        return Optional.empty();
    }
    ObjectMapper objectMapper = new ObjectMapper();
    return Optional.of(objectMapper.readValue(json, UserTokenInfo.class));

}
```

### 만료된 토큰 갱신 요청

토큰 갱신 요청 DTO

```java
Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class RefreshTokenDto {

    @NotBlank(message = "required")
    private String userId;

    @NotBlank(message = "required")
    private String accessToken;

    @NotBlank(message = "required")
    private String refreshToken;
}
```

토큰 갱신 요청을 처리하는 앤드포인트

```java
@PostMapping("/token/reissue")
public ResponseEntity<Map<String, String>> refreshToken(@Valid @RequestBody RefreshTokenDto refreshTokenDto) {
    Map<String, String> result = new HashMap<>();
    result.put("accessToken", memberService.getRenewAccessToken(refreshTokenDto));
    return ResponseEntity.ok(result);
}
```

정상적인 절차를 통해서 토큰갱신 요청이 되면 accessToken를 갱신해서 발급해준다. 이때 정상적인 절차는 아래와 같은 단계를 밟는다.

1. 만약 userId에 해당되는 키값이 redis에 없다면 `401 Unauthorized` 로 처리한다.
2. 만약 전달받은 refreshToken이 userId에 저장된 refreshToken가 다르다면 유출된 토큰으로 간주해서 `401 Unauthorized`로 처리한다.
3. 만약 전달받은 refreshToken이 만료되었거나, 유효하지 않다면 `401 Unauthorized`로 처리한다.
4. 만약 갱신전의 전달받은 accessToken이 redis에 저장된 토큰과 다르다면 비정상 요청으로 `401 Unauthorized`로 처리한다.
5. 만약 accessToken이 아직 만료되지 않았는데 갱신 요청을 한 것이라면 비정상 요청으로 간주하여 `401 Unauthorized` 로 처리한다.

클라이언트에서는 `401 Unauthorized`로 응답받는 것은 다시 로그인 절차를 수행해야 한다.

위에 절차를 체크하는 메소드이다.

```java
private UserTokenInfo checkToken(RefreshTokenDto refreshTokenDto) {
    JwtResult jwtResult ;

    Optional<UserTokenInfo> optionalUserTokenInfo;
    try {
        optionalUserTokenInfo = redisTokenHandler.findSavedAccessToken(
                refreshTokenDto.getUserId());
    } catch (JsonProcessingException e) {
        log.error(e.toString());
        throw new RuntimeException(e);
    }

    //1. 만약 userId에 해당되는 키값이 redis에 없다면 `401 Unauthorized`로 처리한다.
    if (optionalUserTokenInfo.isEmpty()) {
        log.warn("Not found Redis key = {}", refreshTokenDto.getUserId());
        throw new AuthException(JwtResultType.TOKEN_EXPIRED.name());
    }

    UserTokenInfo userTokenInfo = optionalUserTokenInfo.get();

    //2.만약 전달받은 refreshToken이 userId에 저장된 refreshToken가 다르다면 유츨된 토큰으로 간주해서 `401 Unauthorized`로 처리한다. .
    if (!refreshTokenDto.getRefreshToken().equals(userTokenInfo.getRefreshToken())) {
        log.warn("requested refreshToken != saved refreshToken");
        throw new AuthException(JwtResultType.UNUSUAL_REQUEST.name());
    }

    //3.만약 전달받은 refreshToken이 만료되었거나, 유효하지 않다면 `401 Unauthorized`로 처리한다.
    jwtResult = jwtTokenHandler.extractAllClaims(userTokenInfo.getRefreshToken());
    if (jwtResult.getJwtResultType() != JwtResultType.TOKEN_SUCCESS) {
        redisTokenHandler.deleteRedis(userTokenInfo.getUserId());
        log.warn("refreshToken invalid or expired: {}", jwtResult.getJwtResultType().name());
        throw new AuthException(jwtResult.getJwtResultType().name());
    }

    //4.만약 갱신전의 전달받은 accessToken이 redis에 저장된 토큰과 다르다면 `401 Unauthorized`로 처리한다..
    if (!refreshTokenDto.getAccessToken().equals(userTokenInfo.getAccessToken())) {
        redisTokenHandler.deleteRedis(userTokenInfo.getUserId());
        log.warn("requested accessToken != saved accessToken");
        throw new AuthException(JwtResultType.UNUSUAL_REQUEST.name());
    }

    //5.만약 accessToken이 아직 만료되지 않았는데 갱신 요청을 한 것이라면 `401 Unauthorized` 로 처리한다. .
    jwtResult = jwtTokenHandler.extractAllClaims(userTokenInfo.getAccessToken());
    if (jwtResult.getJwtResultType() == JwtResultType.TOKEN_SUCCESS) {
        redisTokenHandler.deleteRedis(userTokenInfo.getUserId());
        log.warn("accessToken has not expired yet : {}", jwtResult.getJwtResultType().name());
        throw new AuthException(JwtResultType.UNUSUAL_REQUEST.name());
    }

    return userTokenInfo;
}
```

Service에 처리하는 메소드는 다음과 같다

```java
public String getRenewAccessToken(RefreshTokenDto refreshTokenDto) {

    UserTokenInfo userTokenInfo = checkToken(refreshTokenDto);

    //Create a new access token.
    Member member = memberRepository.findByUserId(userTokenInfo.getUserId())
            .orElseThrow(() -> new AuthException(JwtResultType.UNUSUAL_REQUEST.name()));
    String renewAccessToken =  jwtTokenHandler.generateToken(member);
    userTokenInfo.setAccessToken(renewAccessToken);

    //Update to redis
    try {
        redisTokenHandler.updateRedis(userTokenInfo);
    } catch (JsonProcessingException e) {
        log.error(e.toString());
        throw new RuntimeException(e);
    }
    return renewAccessToken;
}
```

checkToken() 메소드를 통해 정상적인 갱신요청이면 토큰을 새로 생성하고 그 값을 redis에 업데이트 해주고 리턴해준다.
