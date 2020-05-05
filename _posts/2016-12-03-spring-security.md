---
layout: post
title:  "Spring Security - MySQL 인증"
date:   2016-12-03
categories: java
---

본 문서는 스프링 시큐어리티에 관한 초급적인 부분을 설명한다.
스프링 시큐어리티를 처음 적용하는 부분에 초점을 맞추어었다. 또한 스프링부트가 아닌 스프링 레거시에 자바기반 설정으로 설명한다.  
(짧은 지식으로 개념이나 설명이 매끄럽지 못한 부분이 있으니, 그 부분은 책이나 블로그를 참조하기 바람, 여기서는 구현에만 초점을 맞춤)  

MySQL에 회원테이블을 만들고 인증과정을 거치도록 한다.

```sql
CREATE TABLE test_user
(
  id INT NOT NULL AUTO_INCREMENT,
  username VARCHAR(45) NOT NULL,
  password VARCHAR(70) NOT NULL,
  isadmin CHAR(1) NOT NULL DEFAULT 'N',
  PRIMARY KEY(id),
  UNIQUE KEY(username)
);

INSERT INTO test_user (username, password, isadmin) VALUES('admin', 'admin', 'Y');
INSERT INTO test_user (username, password, isadmin) VALUES('test', 'test', 'N');
```


Graldle 설정은 다음과 같다. 

```java
group 'yookeun'
version '1.0-SNAPSHOT'

apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'eclipse-wtp'
apply plugin: 'war'

sourceCompatibility = 1.8

ext {
    springVersion = '4.3.2.RELEASE'
    securityVersion = '4.1.3.RELEASE'
}

repositories {
    mavenCentral()
}

dependencies {

    // java
    providedCompile 'javax.servlet:javax.servlet-api:3.1.0'
    compile 'javax.servlet:jstl:1.2'

    // spring
    compile "org.springframework:spring-context:${springVersion}"
    compile "org.springframework:spring-context-support:${springVersion}"
    compile "org.springframework:spring-webmvc:${springVersion}"
    compile "org.springframework:spring-jdbc:${springVersion}"
    compile "org.springframework:spring-aop:${springVersion}"
    compile "org.springframework:spring-test:${springVersion}"


    //spring security
    compile "org.springframework.security:spring-security-web:${securityVersion}"
    compile "org.springframework.security:spring-security-config:${securityVersion}"
    compile "org.springframework.security:spring-security-taglibs:${securityVersion}"

    //mybatis
    compile 'org.mybatis:mybatis:3.3.0'
    compile 'org.mybatis:mybatis-spring:1.2.3'
    compile 'mysql:mysql-connector-java:5.1.37'

    // https://mvnrepository.com/artifact/com.zaxxer/HikariCP
    compile group: 'com.zaxxer', name: 'HikariCP', version: '2.4.6'

    // lombok
    providedCompile "org.projectlombok:lombok:1.16.6"
    testCompile group: 'junit', name: 'junit', version: '4.11'
}
```

### 1. 시큐어리티 자바설정 

스프링 시큐어리트를 사용하기 위해선 다음과 같은 클래스가 만들어져야 한다.  

- AbstractSecurityWebApplicationInitializer 를 상속한 클래스 : 시큐어리티 초기화 클래스 
- WebSecurityConfigurerAdapter를 상속한 클래스 : 시큐어리티에 대한 설정 클래스 
- AuthenticationSuccessHandler 를 구현한 클래스 : 인증성공후처리를 위한 클래스 
- UserDetailsService 를 구현한 클래스 : 실제 인증과정을 처리하는 클래스 
- HandlerMethodArgumentResolver를 구현한 클래스 : 컨트롤러에 도착전에 파리미터를 처리하는 클래스 


먼저 `AbstractSecurityWebApplicationInitializer` 를 상속한 클래스를 만들어주면 된다. 

```java
public class SecuirtyInit extends AbstractSecurityWebApplicationInitializer {
}
```

다음은 `WebSecurityConfigurerAdapter`를 역시 상속받아 구현한 클래스를 만들어준다.

```java
@ComponentScan("example")
@Configuration
@EnableWebSecurity  //웹보안 설정
@EnableGlobalMethodSecurity(prePostEnabled=true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    DataSource dataSource;

    @Autowired
    private LoginSuccessHandler loginSuccessHandler;

    @Autowired
    private UserDetailService userDetailService;

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/resources/**");
    }

    ... (중략) ...
}
```

어노테이션중에 `@EnableWebSecurity`, `@EnableGlobalMethodSecurity(prePostEnabled=true)` 가 시큐어리티 관련 어노테이션이다.
`LoginSuccessHandler`는 우리가 `AuthenticationSuccessHandler`를 클래스를 상속받아 구현할 클래스이다. 
로그인 성공후 작업을 처리하는 클래스이다.  
`UserDetailService` 클래스도 `UserDetailsService`를 상속받아 구현처리해야 하는 클래스이다. 
위에 클래스에서 `WebSecurityConfigurerAdapter`에서 정의된 메소드를 오바라이드 해야 한다.

```java
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/resources/**");
    }

    @Override
    protected void configure(HttpSecurity http) throws  Exception {
        http.authorizeRequests().anyRequest().authenticated().and().formLogin().loginPage("/login")
                .loginProcessingUrl("/login").failureUrl("/login?error=true").successHandler(loginSuccessHandler)
                .usernameParameter("username").passwordParameter("password").permitAll().and().logout().deleteCookies("remove")
                .invalidateHttpSession(true).logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
                .logoutSuccessUrl("/").and().csrf().disable();
    }

```

두개의 `configure` 메소드를 오버라이딩하였다.
첫번째에서 매개변수가 `WebSecurity`를 받은 configure 메소드에서 시큐어리티를 적용하지 않을 경로를 지정한다. 
당연히 리소스 관련된(css, js, image) 파일들 일것이다.

두번째 매개변수가 `HttpSecurity`을 받은 configure 메소드를 보자. 
http의 모든 요청에 인증절차를 받는 것이고, 로그인 페이지를 별도로 설정할수있다. 그리고 로그인 처리를 담당하는 경로와 실패를 처리하는 경로까지 정의되고 있고 로그인 성공시 처리되는 클래스 `loginSuccessHandler`로 설정되어 있다. 
로그아웃을 클릭하면 쿠키까지 제거하는 기능도 들어가져 있다. `username`,`password`는 login페이지에서 넘어오는 파라미터를 말한다. 
그리고 마지막 로그아웃을 하면 보이는 경로는 초기경로(첫페이지)등으로 표시했다. 여기서는 csrf는 사용안함으로 처리했다.  

아래 메소드도 추가되어야 한다. 

```java
    /**
     * BCryptPasswordEncoder: bcrypt 해시 알고리즘을 이용하여 입력받은 데이터를 암호화한다
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
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
       // authenticationProvider.setPasswordEncoder(passwordEncoder()); //패스워드를 암호활 경우 사용한다
        return authenticationProvider;
    }
```

우리는 데이터베이스에 저장된 회원으로 인증을 처리할 예정이니 `configureGlobal` 메소드에서 그 부분을 처리한다. 
만약 패스워드를 암호화할 예정이면 `authenticationProvider.setPasswordEncoder(passwordEncoder())` 을 사용하면 된다.

전체 소스를 보자.

```java
package example.spring.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

import javax.sql.DataSource;

/**
 * Created by yookeun on 2016. 9. 17..
 */
@ComponentScan("example")
@Configuration
@EnableWebSecurity  //웹보안 설정
@EnableGlobalMethodSecurity(prePostEnabled=true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    DataSource dataSource;

    @Autowired
    private LoginSuccessHandler loginSuccessHandler;

    @Autowired
    private UserDetailService userDetailService;

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/resources/**");
    }

    @Override
    protected void configure(HttpSecurity http) throws  Exception {
        http.authorizeRequests().anyRequest().authenticated().and().formLogin().loginPage("/login")
                .loginProcessingUrl("/login").failureUrl("/login?error=true").successHandler(loginSuccessHandler)
                .usernameParameter("username").passwordParameter("password").permitAll().and().logout().deleteCookies("remove")
                .invalidateHttpSession(true).logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
                .logoutSuccessUrl("/").and().csrf().disable();
    }

    /**
     * BCryptPasswordEncoder: bcrypt 해시 알고리즘을 이용하여 입력받은 데이터를 암호화한다
     * @return
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
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
       // authenticationProvider.setPasswordEncoder(passwordEncoder()); //패스워드를 암호활 경우 사용한다
        return authenticationProvider;
    }
}

```

다음은 로그인 성공후를 처리하는 클래스를 만들어보자 

```java
@Component
public class LoginSuccessHandler implements AuthenticationSuccessHandler {

    /**
     * 로그인 성공후에 해당 URL로 이동한다
     * @param request
     * @param response
     * @param authentication
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        response.setStatus(HttpServletResponse.SC_OK);
        response.sendRedirect("/");
    }
}
```

로그인 성공후에 응답메시지와 이동경로(보통은 index)로 이동하도록 한다. 간단한 클래스이다. 

그리고 데이터베이스에서 등록된 사용자로 로그인되도록 설정하는 클래스를 만든다. 

```java
@Service
public class UserDetailService implements UserDetailsService {

    @Autowired
    private UserDao dao;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        UserDto userDto = dao.select(username);
        if (userDto == null) {
            throw new UsernameNotFoundException("UsernameNotFound [" + username + "]");
        }
        LoginUser user = createUser(userDto);
        return user;
    }

    private LoginUser createUser(UserDto userDto) {
        LoginUser loginUser = new LoginUser(userDto);
        if (loginUser.getIsAdmin().equals("Y")) {
            loginUser.setRoles(Arrays.asList("ROLE_ADMIN"));
        } else {
            loginUser.setRoles(Arrays.asList("ROLE_USER"));
        }
        return loginUser;
    }
}
```

`UserDetailsService`를 상속받아 구현해야 하고 `loadUserByUsername` 메소드를 오바라이딩 해야 한다.
dao에서 해당 파라미터로 조회하는 메소드가 있어야 하고, 실패처리부분과 성공시 처리되는 부분을 작성해주면 된다.
`LoginUser`라는 사용자가 만든 클래스에서 등록한 사람이 관리자인지 일반인이지 구분하여 처리할 수 있다.

`LoginUser`는 아래와 같다. 

```java
package example.web.user;

import lombok.Data;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

/**
 * Created by yookeun on 2016. 11. 9..
 */
@Data
public class LoginUser implements UserDetails {

    private static final long serialVersionUID = -1286609632181552601L;

    private Integer id;
    private String username;
    private String password;
    private List<String> roles;
    private String isAdmin;


    public LoginUser(){
    }

    public LoginUser(UserDto userDto) {
        this.id = userDto.getId();
        this.username = userDto.getUsername();
        this.password = userDto.getPassword();
        this.isAdmin = userDto.getIsadmin();

    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        List<GrantedAuthority> authorities = new ArrayList<>();
        for (String role : roles) {
            authorities.add(new SimpleGrantedAuthority(role));
        }
        return authorities;
    }
}

```
LoginUser 클래스가 스프링에 사용자인증 클래스로 사용되기 위해서는 반드시 `UserDetails`을 Implement해고 관련 메소드를 구현해야 한다.
바로 `isAccountNonExpired`(게정이 만료되었는지), `isAccountNonLocked`(계정이 잠겨있는지), `isCredentialsNonExpired`(자격이 만료되었느지), `isEnabled`, `getAuthorities` 이다.
이 중 `getAuthorities`는 사용자에게 부여된 권한을 명시하는 메소드이다. 


다음은 `HandlerMethodArgumentResolver`를 구현한 클래스를 만들어야 한다.

 > HandlerMethodArgumentResolver :
 스프링 사용 시, 컨트롤러(Controller)에 들어오는 파라미터(Parameter)를 수정하거나 공통적으로 추가를 해주어야 하는 경우가 있다.
 예를 들어, 로그인을 한 사용자의 사용자 아이디나 닉네임등을 추가하는것을 생각해보자.
 보통 그런 정보는 세션(Session)에 담아놓고 사용하는데, DB에 그러한 정보를 입력할 때에는
 결국 세션에서 값을 꺼내와서 파라미터로 추가를 해야한다.
 그런 경우가 뭐 하나나 두번 정도 있다면 몰라도, 여러번 사용되는 값을 그렇게 일일히 세션에서 가져오는건 상당히 번거로운 일이다.
 HandlerMethodArgumentResolver 는 사용자 요청이 Controller에 도달하기 전에 그 요청의 파라미터들을 수정할 수 있도록 해준다.

 >[addio3305.tistory.com/75](http://addio3305.tistory.com/75)에서 발췌함

`HandlerMethodArgumentResolver` 인터페이스를 구현한 클래스를 만들자. 

```java
public class ExampleHandlerArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return LoginUser.class.isAssignableFrom(parameter.getParameterType());
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication != null && authentication.getPrincipal() instanceof LoginUser) {
            return authentication.getPrincipal();
        }
        return null;
    }
}
```

`supportsParameter`메소드는 로그인페이지에서 넘긴 파라미터가 `LoginUser` 클래스의 멤버변수등과 적합한지 여부를 판단하고,
`resolveArgument`메소드는 파라미터 정보를 토대로 실제 인증된 객체를 리턴하게 된다. 

이제 위에서 만든 `ExampleHandlerArgumentResolver`를 `WebMvcConfigurerAdapter`를 상속받아 구현 클래스에 추가해주어야 한다.


```java
@Configuration
@EnableWebMvc
@ComponentScan("example")
public class WebConfig extends @Configuration
@EnableWebMvc
@ComponentScan("example")
public class WebConfig extends WebMvcConfigurerAdapter {
    ... 중략....
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(new ExampleHandlerArgumentResolver());
    }
}
```

이것을 스프링 시큐어리트의 모든 설정은 완료가 되었다.

### 2. 화면 적용 

로그인 페이지를 만들어본다. 파라미터명는 LoginUser클래스에 적용된 멤버와 동일해야 한다(username, password)

```html
<div class="container">
    <form class="form-signin" method="POST" action="/login">
        <h2 class="form-signin-heading">Please sign in</h2>
        <label for="" class="sr-only">ID</label>
        <input type="text" name="username" class="form-control" placeholder="ID" required autofocus>
        <label for=""  class="sr-only">Password</label>
        <input type="password" name="password" class="form-control" placeholder="Password" required>
        <div class="checkbox">
            <label>
                <input type="checkbox" value="remember-me"> Remember me
            </label>
        </div>
        <!--input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"-->
        <button class="btn btn-lg btn-primary btn-block" type="submit">Sign in</button>
    </form>

</div> <!-- /container -->
```

다음은 로그인처리를 하는 컨트롤로를 보자. 

```java
@Controller
public class UserController {

    @Autowired
    private UserService service;


    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public ModelAndView login() {
        return new ModelAndView("login");
    }

    @RequestMapping(value = "/", method = RequestMethod.GET)
    public ModelAndView index(LoginUser loginUser) {
        ModelAndView model = new ModelAndView("index");
        model.addObject("user", loginUser);
        return model;
    }


}
```

매우 간단하다. 로그인페이지를 보여주는 매핑메소드와 로그인 성공되었을 경우 처리해주는 인덱스(루트경로)만 해주면 끝이다.
실제로는 `index`메소드에 도달하기 전에 스프링 시큐어리티에서 먼저 처리한다음 그 다음 과정을 진행하게 된다. 
만약 등록된 사용자가 아니라면 `http://localhost:8080/login?error=true` 라는 URL로 반복처리되어 로그인창만 계속 보이게 된다. 
물론 별도의 에러페이지로 전환할수도 있다. 

로그인이 성공되면 jsp에서 관리자, 일반사용자로 구분하여 처리할수 있게 된다. 

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags" %>
<html>
<head>
    <title>Title</title>
    <!-- Latest compiled and minified CSS -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css"
          integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
    <!-- Latest compiled and minified JavaScript -->
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"
            integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa"
            crossorigin="anonymous"></script>
    <link rel="stylesheet" href="/resources/css/signin.css">
</head>
<body>
<div class="container">
    <h1>Login Success!!!</h1>
    <h2>${user.username} 님 환영합니다. </h2>
    <p></p>
    <h3><a href="/home">home</a></h3>
    <sec:authorize access="hasRole('ROLE_ADMIN')">
        <h3><a href="javascript:alert('관리자군요.')";>관리자만 보임</a></h3>
    </sec:authorize>
    <h3><a href="/logout">logout</a></h3>
</div>
</body>
</html>
```

taglib로 시큐어리티 태그 `sec`를 사용해야 한다. `<sec:authorize access="hasRole('ROLE_ADMIN')">` 로 관리자 유무를 처리할 수 있다.
관리자인지 처리 여부는 위에 `UserDetailService` 클래스에서 이미 처리했다.

위에 전체 소스는 아래에 경로에서 확인할 수 있다.  

- [spring-security-example](https://github.com/yookeun/spring-security-example)





