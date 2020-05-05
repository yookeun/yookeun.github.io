---
layout: post
title:  "Spring security JWT 연동"
date:   2017-07-23
categories: java
---

### 1. 기존 oauth2의 문제점 

기존의 OAuth2의 단점은  api를 호출할때마다 accessToken이 유효한지 실제 oauth서버에 통해 검증하는 것이다. 이때 매번 oauth에서 해당 토큰의 만료여부등을 DB등에서 조회하고 새로 갱신시 업데이트 작업을 해주어야 한다.  이러한 작업이 빈번해지면 결국 oauth서버에 상당한 부담을 준다는 것이다.  이러한 문제점을 개선하기 위해서 토큰 자체에 만료일을 체크하는 부분을 첨부하고 아울러서 사용자에 대한 추가정보(계정)등을 저장하고 있고 이를 사용하면 oauth서버에 부담을 상당히 줄이게 된다. 

이러한 방식을 Claim 기반의 토큰이라고 하는데 이 개념을 이용해서 요청을 받은 서버나 서비스에서는 이 서비스를 호출한 사용자에 대한 추가정보는 이미 토큰안에 모두 포함되어 있어서 다른 곳에서 가져올 필요가 없다는 것이다. 

[JWT](https://jwt.io/)가 바로 Claim 기반의 토큰이이고  Json형식으로 데이터를 인코딩하여 토큰으로 이용한다.



- 참고출처) [이수홍님 블로그 - Spring boot로  만드는 oauth2 시스템7](https://brunch.co.kr/@sbcoba/7)
- 참고출처) [조대협의 블로그 - REST JWT(JSON Web Token)소개 - #1 개념 소개](http://bcho.tistory.com/999)
- 참고출처) [머루의 개발블로그 - spring security oauth2 jwt](http://wonwoo.ml/index.php/post/980)

### 2. Spring boot + Security + oauth2+ JWT를 만들어보자

Spring oauth2에서 사용하는 client_id를 관리하는 테이블을 만든다.  ClientDetailsServiceConfigurer 클래스에 세팅이 되어야 한다.  그 다음은 Security에 사용하는 별도의 회원테이블을 만든다. 그래서 허가된 사용자만 로그인이 되도록 한다. 

Sring oauth2 + Security는 결국 두번의 보안검증을 통과해야 한다. oauth2에서 실제 url에 외부 접근이 가능한지 체크하는 것이고, Security는 등록된 계정으로 내부  접근이 가능한지 체크하는 것이다. 

``` 
curl -u client1:client1pwd http://localhost:8081/restauth/oauth/token -d  "grant_type=password&username=user1&password=1234"
```

curl를 보자.  옵션에 -u 부분이 바로 Spring oauth2가 필요한 로그인 정보다.이것을 통해 해당 url로 접근이 가능한 클라이언트인지 먼저 체크하고 나서 해당 url(즉 Controller)에 파라미터를 전송하여 등록된 유저인지 체크하는 것이다. 

Spring oauth2  + Security를 통해서 JWT로 token을 생성하고 받아오기 위해서는 각각의 인증 절차를 걸쳐야 한다.  따라서 기존의 Spring security config 클래스에 Spring에서 oauth2를 구현하는 `AuthorizationServerConfigurerAdapter`를 상속받은 config클래스를 만들어야 한다. 그리고 이 클래스에 jwt 설정작업을 해주는 것이다. 

### 3. 테이블 생성 및 라이브러리 세팅 

Spring oauth2와 Spring jwt 라이브러리를 받아오자. 

```xml
<dependency>
  <groupId>org.springframework.security.oauth</groupId>
  <artifactId>spring-security-oauth2</artifactId>
</dependency>
		
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-jwt</artifactId>
</dependency>
```

이것으로 설정은 끝났다.  다음은 관련 테이블을 만들어보자. 실제 DB에 등록된 회원이어야 하고, oauth2를 이용하기 위한 클라이언트 정보도 있어야 하므로 두개의 테이블이 필요하다. 회원테이블은 각각 용도에 맞게 작성하면 되지만 oauth2를 위한 테이블은 다음과 같이 생성되어야 한다.

```sql
CREATE TABLE `oauth_client_details` (
  `client_id` varchar(256) COLLATE utf8_unicode_ci NOT NULL,
  `resource_ids` varchar(256) COLLATE utf8_unicode_ci DEFAULT NULL,
  `client_secret` varchar(256) COLLATE utf8_unicode_ci DEFAULT NULL,
  `scope` varchar(256) COLLATE utf8_unicode_ci DEFAULT NULL,
  `authorized_grant_types` varchar(256) COLLATE utf8_unicode_ci DEFAULT NULL,
  `web_server_redirect_uri` varchar(256) COLLATE utf8_unicode_ci DEFAULT NULL,
  `authorities` varchar(256) COLLATE utf8_unicode_ci DEFAULT NULL,
  `access_token_validity` int(11) DEFAULT NULL,
  `refresh_token_validity` int(11) DEFAULT NULL,
  `additional_information` varchar(4096) COLLATE utf8_unicode_ci DEFAULT NULL,
  `autoapprove` varchar(256) COLLATE utf8_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`client_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

바로 `oath_client_details`가 oauth2를 이용하는 클라이언트를 등록하는 테이블이다. 테스트를 위해 다음 값을 인서트한다. 

```sql
NSERT INTO oauth_client_details
(
	client_id, 
	client_secret,
	resource_ids, 
	scope, 
	authorized_grant_types, 
	web_server_redirect_uri, 
	authorities, 
	access_token_validity, 
	refresh_token_validity, 
	additional_information, 
	autoapprove
)
VALUES
(
	'client1',
	'client1pwd',
	null, 
	'read,write', 
	'authorization_code,password, implicit, refresh_token',
	null,
	'ROLE_YOUR_CLIENT',
	36000,
	2592000,
	null,
	null
);

```

다음은 사용자 테이블을 만들자. 용도에 맞게 알아서 만들면 된다. 

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
  `password` varchar(100) COLLATE utf8_unicode_ci NOT NULL,
  `user_type` char(1) COLLATE utf8_unicode_ci NOT NULL,
  `reg_date` datetime NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `user_name` (`user_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

insert into user(user_name, password, user_type, reg_date) values('user1','$2a$10$oXhcs6qujNbFA5yXauupSuWLQpMjVbAskvPbMvcUzurpsdIuSXs7m','2',now())
```

패스워드는 1234로 하자.  다만 암호화하기 위해서는  `BCryptPasswordEncoder` 를 이용하자.

```java
/**
 * 비밀번호암호화
 */
@Test
public void 비밀번호_암호화() {
	BCryptPasswordEncoder bcr = new BCryptPasswordEncoder();
	String result = bcr.encode("1234");  
	System.out.println("암호 === " + result);
}
```



먼저 security를 설정하기 위한 작업을 해주자. `UserDetailsService`를 상속받은 클래스를 만든다. 

```java
@Service
public class UserDetailService implements UserDetailsService {
	
	@Autowired
	private UserRepository userRepository;

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		User user = userRepository.findByUsername(username);
		if (user == null) {
			throw new UsernameNotFoundException("UsernameNotFound [" + username + "]");
		}
		LoginUser loginUser = createUser(user);	
		return loginUser;
	}
	
	private LoginUser createUser(User user) {
		LoginUser loginUser = new LoginUser(user);
		if (loginUser.getUserType().equals("1")) {
			loginUser.setRoles(Arrays.asList("ROLE_ADMIN"));
		} else {
			loginUser.setRoles(Arrays.asList("ROLE_USER"));
		}
		return loginUser;
	}

}
```

UserDetailService클래스는 실제 db에 저장된 사용자정보를 조회하는 클래스이다. 

다음은 `WebSecurityConfigurerAdapter`를 상속받은 Security 처리를 담당하는 클래스를 만들자.

```java

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled=true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	
	@Autowired
	private UserDetailService userDetailService;
	
    /**
     * BCryptPasswordEncoder: bcrypt 해시 알고리즘을 이용하여 입력받은 데이터를 암호화한다
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    
    @Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    	 auth.authenticationProvider(authenticationProvider());
	}

   /**
     * 데이터베이스 인증용 Provider
     * @return
     */
    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userDetailService);
        authenticationProvider.setPasswordEncoder(passwordEncoder()); //패스워드를 암호활 경우 사용한다
        return authenticationProvider;
    }

}
```

SecurityConfig 클래스는 위에서 만든 UserDetailService를 이용하여 Security를 설정하는 클래스이다. 

이제는 oauth2에서 jwt를 설정하는 클래스를 만들자. 해당 클래스는 `AuthorizationServerConfigurerAdapter` 클래스를 상속받아 구현해야 한다.

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationSeverConfig extends AuthorizationServerConfigurerAdapter{
	
	@Value("${resouce.id:spring-boot-application}")
	private String resourceId;
	
	@Value("${access_token.validity_period:3600}")
	int accessTokenValiditySeconds = 3600;
	
	@Autowired
	private AuthenticationManager authenticationManager;

	
    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }
	
	@Bean
	public JwtAccessTokenConverter accessTokenConverter() {
		JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
		//converter.setSigningKey("secret");
		KeyPair keyPair = new KeyStoreKeyFactory(new ClassPathResource("server.jks"), "passtwo".toCharArray())
				.getKeyPair("auth", "passone".toCharArray());
		converter.setKeyPair(keyPair);		
		return converter;
	}
	
	@Bean
	@Primary
	public DefaultTokenServices tokenService() {
		DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
		defaultTokenServices.setTokenStore(tokenStore());
		defaultTokenServices.setSupportRefreshToken(true);
		return defaultTokenServices;
    }
	
	@Override
	public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
		endpoints.tokenStore(tokenStore()).authenticationManager(authenticationManager).accessTokenConverter(accessTokenConverter());
	}
	
	@Bean
	@Primary
	public JdbcClientDetailsService JdbcClientDetailsService(DataSource dataSource) {
		return new JdbcClientDetailsService(dataSource);
	}
	
	@Autowired
	private ClientDetailsService clientDetailsService;

	@Override
	public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
		//oauth_client_details 테이블에 등록된 사용자로 조회한다.
		clients.withClientDetails(clientDetailsService);
	}
```

핵심은 JwtTokenStore의 빈을 생성하고 accessTokenConverter 메소드를 이용하여 KeyPair를 통해 암호화한다. 실제 jwt 에서 AccessToken은 암호화해야 하기 때문에 그 암호화/복호화하는 키를  accessTokenConverter에서 처리해주는 것이다. 그 다음에 이를 JwtTokenStore로 저장하고 이를  DefaultTokenServices로 등록한다. 

물론 암호화부분은  간단히 converter.setSigningKey("비번")으로 설정할 수 있으나 그러면 실제 운영에는 그렇게 적용할 수 없으므로 우리는 여기서 KeyPair를 만들고 사용하도록 하자.keytool를 통해서 jks파일을 먼저 생성하자.  

***이때 생성된 파일명.cer, 파일명.jks은 프로젝트안에 가장 상위에 위치시키자.***

keytool에 대한 설명은 다음 블로그를 참조하자. http://m.blog.naver.com/wndrlf2003/220649843082

### 4. AccessToken을 받아오자.

이제 스프링을 실행하자. 정상적으로 실행이 되었으면 curl를 쳐보자.

```
curl -u client1:client1pwd http://localhost:8081/restauth/oauth/token -d  "grant_type=password&username=user1&password=1234"
```

그러면 다음과 같이 토큰값이 나오면 성공한 것이다. 

```
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDA4MzkyNTksInVzZXJfbmFtZSI6InVzZXIxIiwiYXV0aG9yaXRpZXMiOlsiUk9MRV9VU0VSIl0sImp0aSI6IjBmNTMzZWYzLTgwMDYtNGY3ZS04NzIwLTI1NThlZGRiZGMwMSIsImNsaWVudF9pZCI6ImNsaWVudDEiLCJzY29wZSI6WyJyZWFkIiwid3JpdGUiXX0.oOLJ_cw67CYss46B2gsTpztw6dW2FGPyJpE1pxPmGgzj9jMap3xH_EAo9AFzIA8fPQfZgDjG9uOuL2_IvjAslV9SQsKjy-H_V4zWzGGRJ5Dnv-8XeI22rXaHZRLp4lECg7FUqzjxC9dr1VY2WlXO-x2LNuIv312Mq16EP2hzSPV7BCUDma6Kk_nfbUsiHUHVvRdRP25O0ve0zGc0lUI0TxUKAk1bQfKx7tgp6BYUAYGBN7j066ODRJ9WOzv8Qacs61LjUxJTTfACWhuUAnlehbzUhKKVrxPTUmejUo-d7JPUxlhntMzlxiNChDOkVU_4aCdmuMBN54BvS43ph1zyYg","token_type":"bearer","expires_in":35999,"scope":"read write","jti":"0}
```

### 5. API(Resource) 서버에서 사용할 때.

jwt용 oauth2 서버가 구성되었다. 그렇다면 api서버를 호출할때 위에서 받은 accessToken을  던져줘야 한다. 그러면 api서버에서는 약속된 복호화키를 이용해서 키를 복호화하여 유효한 토큰인지 확인하는 작업이 필요하다. 이를 위해서 api서버에서도 jwt 관련 라이브러리가 필요하다. 위에 maven을 그대로 사용하면 된다. 그런 다음에 `ResourceServerConfigurerAdapter`를 상속받은 Config클래스를 만들어야 한다. 

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

	@Value("${resource.id:spring-boot-application}")
	private String resourceId;
	
	@Value("${security.oauth2.resource.jwt.key-value}")
	private String publicKey;

    @Bean
    public TokenStore tokenStore() {
        return new JwtTokenStore(accessTokenConverter());
    }
	
	@Bean
	public JwtAccessTokenConverter accessTokenConverter() {		
		JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
		//converter.setSigningKey("secret");
		converter.setVerifierKey(publicKey);
		return converter;
	}
	
	@Bean
	@Primary
	public DefaultTokenServices tokenService() {
		DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
		defaultTokenServices.setTokenStore(tokenStore());
		defaultTokenServices.setSupportRefreshToken(true);
		return defaultTokenServices;
	}

	@Override
	public void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests()
        .antMatchers("/api/**").hasRole("USER");
	}

	@Override
	public void configure(ResourceServerSecurityConfigurer resources) throws Exception {		
		resources.resourceId(resourceId);
	}
```

JwtAccessTokenConverter에서 공개키를 이용해서 accessToken를 복호화해야 한다. 이를 위해서는 미리 oauth에서 생성한 키를 application.yml에 등록해서 복호화할 때 사용하면 된다. 

```yml
security:
  user:
    name: user
    password: test
  oauth2:
    resource:
      jwt:
        key-value: 
            -----BEGIN PUBLIC KEY-----
            MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAr5ui+5AA8v1LivSYNGdZ
            0O76SBgktU3CjG+BcUb0mJ1fl34+So75bRI3/+n8AJP//Xp3OS62niQyRB5LEdCr
            7ox5RAQlLHFkkJHQysi2/236Br8ZDiM1AfT4Hz5AQ9D4Pk5H/n6stKuS2ZffHAWA
            W7/qK7PC6z4ncSlhVYr/zXNj7HVx9YFP7gZx6faaar6UvV96vX8W4xcITxvLpaCB
            nF9TeOUxLHYI3tmnxmE8gRQXpdCH7X3IB3f4QfVCMIIrSD8d3si5w5VArlEMRQ9I
            TEOA0Iq8VJKGjlu+haycTgIPJD3yGhU7Zb3EQfBZvQGC31beogycEf9imnvHDW4N
            vQIDAQAB
            -----END PUBLIC KEY-----
```

바로 jwt.key-value에 oauth서버에 생성한 공개키를 넣어주면 된다.  이것으로 api서버작업은 완료가 되었다.  이제 해당 api를 호출할때마다 토큰을 날려주면 api서버에는 oauth서버에 전달없이 토큰의 유효성을 체크하여 사용할 수 있다. 

```bash
curl -H "authorization: bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE0OTMyMDgzMzUsInVzZXJfbmFtZSI6InVzZXIxIiwiYXV0aG9yaXRpZXMiOlsiUk9MRV9VU0VSIl0sImp0aSI6ImM2OTZlODRhLTNjZGYtNGFjNC04YzJkLWNlODAzNjMxODUwYSIsImNsaWVudF9pZCI6ImNsaWVudDEiLCJzY29wZSI6WyJyZWFkIiwid3JpdGUiXX0.EeQ0cCQF5hPEpuY2c95Bw7zdGP0tBCt3BBBwlh83YEIvexI82oUoHUxxDHRrsVCSWD5vTfahqY_FsUyhI5yoFkXUpUMarXDmIWz-5Ww_XPyzAoLIqmcjw-WCR9aJDn36w9ylz7-vXXmPnCXpxUpETBmgqKmJBF-6Yi-THi4OPs7kP0eZ_pyUB5pZSIO6pnH0E2kbqZfjEgoQ84AZJD2oXCjFXFO48HaMOLVU8nDTI1VnwM_5f5BfsA_UzJH_ktD8lUpARBvVkRSQVRU5Ek4p8FPZHNhPfxJlilDwaN4BSjpIfCQg0NMbzqQ4s7twlRr2Yi3gGCykMJlkzTz2LPNwtQ" http://localhost:8080/api/user
```

전체소스는 아래에서 확인할 수 있다. 

- [springboot-jwt-exampe](https://github.com/yookeun/springboot-jwt-example)



