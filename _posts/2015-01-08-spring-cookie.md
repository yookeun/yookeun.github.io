---
layout: post
title:  "JQuery와 Spring간의 Cookie 연동"
date:   2015-01-08
categories: java
---

JQuery에서 쿠키를 저장하고 Spring Controller에서 저장된 쿠키를 체크하는 기능이 필요할 때가 있다.  

일단, JQuery에서 쿠키를 처리하는 플러그인을 받는다.
[jquery-cookie](https://github.com/carhartl/jquery-cookie) 에서 다운받거나


`bower`를 쓴다면 아래 처럼 실행한다.

```bash
bower install jquery.cookie
```

JS파일에서 어떤 응답을 처리한 다음에 쿠키를 저장한다면 다음과 같이 하면 된다.

```javascript
$.ajax({
    type : "POST",
    dataType : "JSON",
    data: JSON.stringify(param),
    contentType : "application/json; charset=UTF-8",
    url: '/test/testurl'
    error : function() {
        alert('error');
    },
    success : function(result) {
        $.cookie('testCookie', '12345');       
    }
});
```

`testCookie`라는 값에 "12345" 라는 문자열을 저장했다. 기타 옵션은 `jquery-cookie` github 사이트에서 확인하면 된다.  

다음은 Spring Controller에서 cookie를 체크하는 기능이다.

```java
@RequestMapping(value = "/check/cookie", method = RequestMethod.GET)
protected String cookieCheck(@CookieValue(value="testCookie", required=false, defaultValue="") String testCookie) {            
    System.out.println("testCookie=="+testCookie);
    return "/test.jsp";
}
```

`@CookieValue` 어노테이션을 사용한다.
지정한 쿠키값이 없을 경우에 예외가 발생하지 않도록  required를 false로 조정할 수 있다.
또한 defaultValue로 쿠키값이 없을 경우 초기값을 지정할 수 있다.
이렇게 하면 간단히 JQuery하고 Spring간의 Cookie를 연동할 수 있게 된다.
