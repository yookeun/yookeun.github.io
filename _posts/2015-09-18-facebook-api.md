---
layout: post
title:  "Javascript에서 Facebook API 사용"
date:   2015-09-19
categories: javascript
---

페이스북 API를 사용하려면 먼저 Facebook Developer로 등록해야 하고 사용할 App을 등록해야 한다.
여기서 App를 등록하는 것은 생략한다. app를 등록하면 appId를 받는다. 그것을 가지고 작업을 해야 한다.

보통 index.html등에 웹사이트를 시작하므로 거기에 다음과 같이 넣어준다.

### 기본 세팅

```javascript
<script>
  window.fbAsyncInit = function() {
    FB.init({
      appId      : '{appID}',
      xfbml      : true,
      version    : 'v2.4'
    });
  };

  (function(d, s, id){
     var js, fjs = d.getElementsByTagName(s)[0];
     if (d.getElementById(id)) {return;}
     js = d.createElement(s); js.id = id;
     js.src = "//connect.facebook.net/en_US/sdk.js";
     fjs.parentNode.insertBefore(js, fjs);
   }(document, 'script', 'facebook-jssdk'));
</script>
```

`appId`에 등록한 앱의 아이디를 입력한다.

### 로그인체크 기능

```javascript
FB.getLoginStatus(function(response) {
    statusChangeCallback(response);
});
```

`response`에는 아래의 값이 JSON 형태로 리턴된다.

```json
{
    status: 'connected',
    authResponse: {
        accessToken: '...',
        expiresIn:'...',
        signedRequest:'...',
        userID:'...'
    }
}
```

`status`에서는 다음과 같은 값이 넘겨진다.

statas 값|설명
------------|---
connected|현재 페이스북에 로그인되어 있고 앱도 등록되어 있다.
not_authorized|페이스북에는 로그인되어 있으나, 앱은 등록되어 있지 않다.
unknown|페이스북에 로그인되어 있지 않다. 그래서 앱도 등록되어 있는지 알수없다.


`authResponse`에서는 다음과 같은 값이 넘겨진다.  

authResponse 값|설명
------------|---
accessToken|앱에 접근할 수 있는 토큰값
expiresIn| UNIX시간으로 토큰이 만기되어 다시 만드는 시간
signedRequest|앱을 사용한 사용자의 서명값이 담긴 파라미터 정보
userID|앱을 사용하는 페이스북의 아이디 (계정이 아님)


따라서 `response.status`에 따라서 로그인처리상태를 확인한후에 `response.authResponse.*`의 값을 통해 내부작업을 진행할수 있다.

### 로그인 기능

```javascript
 FB.login(function(response) {
   // handle the response
 }, {scope: 'public_profile,email'});
```

`FB.login`를 통해 실제로 페이스북로그인을 처리한다. 이때 `scope`를 통해서 앱의 `permissions`등을 설정할 수 있다.
보통 아래와 같은 `permission'를 사용한다.
자세한 설명은 페이스북 퍼미션부분을 참고한다.  
참고사이트 : <https://developers.facebook.com/docs/facebook-login/permissions#optimizing>


### Graph API를 통해 페이스북에서 현재 사용자의 프로필 가져오기

```javascript
FB.api('/me?fields=id,name,picture.width(100).height(100).as(picture_small)', function(response) {
	//handling
});
```

현재 페이스북에서 로그인된 사용자의 이름, 프로필사진(작은거)를 가져온다.  
여기서 `/me`는 현재 로그인된 페이스북 사용자를 의미한다.

response|설명
------------|---
response.name| 페이스북 사용자 이름
response.picture_small.data.url| 페이스북 사용자의 작은 사진 URL   

참고사이트 : <https://developers.facebook.com/docs/graph-api/using-graph-api/v2.4>

### Graph API를 통해 페이스북에서 현재 사용자의 페이지 모두 가져오기

```javascript
FB.api('/{user-id}/accounts', function(response) {						  
   //response handling...
});
```

`{user-id}` 은 페이스북 로그인체크하여 나온 값은 response.authResponse.userID 값을 의미한다.  
response.data에 사용자가 등록된 정보가 담겨져 있다.

```json
{
  "data": [
    {
      "category": "Product/service",
      "name": "Sample Page",
      "access_token": "{access-token}",
      "id": "1234567890",
      "perms": [
        "ADMINISTER",
        "EDIT_PROFILE",
        "CREATE_CONTENT",
        "MODERATE_CONTENT",
        "CREATE_ADS",
        "BASIC_ADMIN"
      ]
    },
}
```

참고사이트 : <https://developers.facebook.com/docs/pages/access-tokens>
