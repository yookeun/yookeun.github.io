---
layout: post
title:  "twitter4j를 통한 twitter 로그인 연동"
date:   2015-10-08
categories: java
---

트위터와 연동하는 자바모듈로 유명한 <[twitter4j](http://twitter4j.org/ko/index.html)>가 있다.
스프링과 메이븐을 사용한다면 다음과 같이 pom.xml에 추가한다.

```xml
<dependency>
  <groupId>org.twitter4j</groupId>
  <artifactId>twitter4j-core</artifactId>
  <version>4.0.4</version>
</dependency>
```

>로컬에서 테스트할때 트위터 앱을 등록할 경우 Callback URL를 `http://localhost`가 아닌 `http://127.0.0.1`로 해야 한다.

### 1.트위터 로그인 연동

```java
Twitter twitter = new TwitterFactory().getInstance();
//twitter로 접근한다.
twitter.setOAuthConsumer(consumerKey, consumerSecret);

//성공시 requestToken에 해당정보를 담겨져온다.
RequestToken requestToken =  twitter.getOAuthRequestToken();

//requestToken 을 반드시 세션에 담아주어야 한다.
request.getSession().setAttribute("requestToken", requestToken);
requestToken.getAuthorizationURL()  //접속할 url값이 넘어온다.
requestToken.getToken() //token값을 가져온다.
requestToken.getTokenSecret() //token Secret값을 가져온다.

```
`consumerKey`, `consumerSecret` 는 앱등록시 정보를 입력하면 된다.
중요한 것은 아래다. 아래부분이 이전 버전과 다른 점이다.
>request.getSession().setAttribute("requestToken", requestToken)  

여기에 세션을 담아야 사용자정보를 가져올 수 있다.

사용자의 token,tokenSecret를 별도로 잘 저장한다. AuthorizationURL를 통해서 사용자가 트위터에 접속하는 페이지로 연결처리해준다.
`requestToken.getAuthorizationURL`에는 사용자가 접속하는 트위터 url이 리턴된다.

```html
https://api.twitter.com/oauth/authorize?oauth_token=TEir9QAAAAAAWeDHAAABUEX12e
```


### 2. 로그인된 사용자의 정보 가져오기 ###

위에서 트위터접속페이지에서 정상적으로 로그인하면 트위터에 등록된 callback url로 아래와 같이 리턴된다.

```html
http://127.0.0.1:8080/twitter-response.jsp?oauth_token=qwd1AAAWeDHAAABUEXVgyE&oauth_verifier=Ae12e12Xsu5VIWhs0wq2
```

twitter-response.jsp는 리턴값을 처리해주기 위한 개발한 페이지이다. 파라미터로 넘어온 `oauth_token`과 `oauth_verifier`를 잘 저장한다.
`oauth_verifier`를 이용해서 사용자 정보를 가져올 수 있다.


```java
Twitter twitter = new TwitterFactory().getInstance();			
twitter.setOAuthConsumer(consumerKey, consumerSecret); //저장된 consumerKey, consumerSecret
AccessToken accessToken = null;		

String oauth_verifier = callbackURL에서 전달받은 oauth_verifier

//트위터 로그인 연동시 담은 requestToken 의 세션값을 가져온다.
RequestToken requestToken = (RequestToken )request.getSession().getAttribute("requestToken");			
accessToken = twitter.getOAuthAccessToken(requestToken, oauth_verifier);			
twitter.setOAuthAccessToken(accessToken);

//해당 트위터 사용자의 이름과 아이디를 가져온다.
System.out.println(accessToken.getUserId());    //트위터의 사용자 아이디
System.out.println(accessToken.getScreenName()); //트워터에 표시되는 사용자명 			

```
