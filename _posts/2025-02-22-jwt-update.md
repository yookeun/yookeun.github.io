---
layout: single
title: "JWT(0.11.5 → 0.12.6) 업데이트"
date: 2025-02-22
categories: [java]
tags: [jwt]
---

기존의 JWT 라이브러리 버전을 0.11.5 → 0.12.6 로 업그레이드 했더니 기존에 소스중에 deprecated 된 것이 있어 수정한다.

```groovy
// jwt
implementation 'io.jsonwebtoken:jjwt-api:0.12.6'
runtimeOnly  'io.jsonwebtoken:jjwt-impl:0.12.6'
runtimeOnly  'io.jsonwebtoken:jjwt-jackson:0.12.6'
```

**기존 0.11.5 버전 소스에  setClaims, setIssuedAt, setExpiration 은 모두  deprecated 됨**

```java
public String createToken(Map<String, Object> claims) {
    String secretKeyEncodeBase64 = Encoders.BASE64.encode(jwtSecretKey.getBytes());
    byte[] keyBytes = Decoders.BASE64.decode(secretKeyEncodeBase64);
    Key key = Keys.hmacShaKeyFor(keyBytes);
    return Jwts.builder()
            .signWith(key)
            .setClaims(claims)
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * (accessTokenExpiredMin)))
            .compact();
}
```

**0.12.6 소스에서 (claim, issusedAt, expiration 으로 변경)**

```java
public String createToken(Map<String, Object> claims) {
    String secretKeyEncodeBase64 = Encoders.BASE64.encode(jwtSecretKey.getBytes());
    byte[] keyBytes = Decoders.BASE64.decode(secretKeyEncodeBase64);
    Key key = Keys.hmacShaKeyFor(keyBytes);
    return Jwts.builder()
            .signWith(key)
            .claims(claims)
            .issuedAt(new Date(System.currentTimeMillis()))
            .expiration(new Date(System.currentTimeMillis() + 1000 * 60 * (accessTokenExpiredMin))
            .compact();
}
```

**기존 0.11.5 버전 소스 (SignatureAlgorithm, parserBuilder, setSigningKey, parseClaimsJws, getBody) 처리 부분** 

```java 
public JwtResult extractAllClaims(String token) {
    if (StringUtils.isNullOrEmpty(token)) return null;
    SignatureAlgorithm sa = SignatureAlgorithm.HS256;
    SecretKeySpec secretKeySpec = new SecretKeySpec(jwtSecretKey.getBytes(), sa.getJcaName());
    Claims claims;
    JwtResult jwtResult = new JwtResult();
    try {
        claims = Jwts.parserBuilder().setSigningKey(secretKeySpec).build()
                .parseClaimsJws(token).getBody();
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

**0.12.6 소스 (parser, verifyWith, parseSignedClaims, getPayload) 로 수정** 

```java 
public JwtResult extractAllClaims(String token) {
    if (StringUtils.isNullOrEmpty(token)) return null;
    byte[] keyBytes = jwtSecretKey.getBytes();
    SecretKeySpec secretKeySpec = new SecretKeySpec(keyBytes, "HmacSHA256");
    Claims claims;
    JwtResult jwtResult = new JwtResult();
    try {
        claims = Jwts.parser().verifyWith(secretKeySpec).build()
                .parseSignedClaims(token).getPayload();
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

