---
layout: post
title:  "Javascript에서 this의 모든 것"
date:   2016-01-26
categories: javascript
---

자바스크립트에서 this에 대한 정리를 해본다.

### 1. 객체의 메소드 안에서의 this

객체의 메소드안에서 사용된 this는 해당 메소드를 호출한 객체에 바인딩된다.
아래 소스를 보자.

```javascript
var obj = {  
  sayName : function() {
    console.log(this.name);
  }
};

var kim = obj;
kim.name = "kim";
kim.sayName();   // "kim"

var lee = obj;
lee.name = "lee";
lee.sayName();   // "lee"
```

예제에 보다시피 obj 객체를 kim, lee객체를 통해서 호출하면 obj내의 sayName안에 this는 호출한 객체.name으로 연결된다.

> 메소드에서 사용된 this는 자신을 호출한 객체에 바인딩 된다.


### 2. 함수내의 this

함수내의 사용된 this는 전역객체(window)를 가리킨다.

```javascript
var val = "Hello";

var func = function() {
  console.log(this.val);

};
func();   // "Hello"
console.log(this.val);  // "Hello"
console.log(window.val);  //  "Hello"
console.log(this === window);  // true  
```

### 3. 생성자 함수내의 this

생성자 함수내의 this는 new로 생성자 함수를 생성한 객체에 바인딩되다.

```javascript
var Person = function(name) {
  this.name = name;
};

var kim = new Person('kim');
console.log(kim.name); //kim

var lee = new Person('lee');
console.log(lee.name); //lee
```

만약 new를 사용하지 않고 호출한다면 어떻게 될까?

```javascript
var park = Person('park');
console.log(park.name); //"TypeError: Cannot read property 'name' of undefined
```

>일반 함수 호출의 this는 window 전역 객체에 바인딩되는 반면에, 생성자 함수 호출의 경우는 this 는 새로 생성된 빈객체에 바인딩된다.

즉, park이라는 빈 객체에서 name이라는 프로퍼티를 찾게되므로 에러가 나기 때문에 위와 같은 메시지가 출력된다.

#### - 참고서적 : 인사이트 자바스크립트(한빛미디어) / 송형주, 고현준 지음 ####
