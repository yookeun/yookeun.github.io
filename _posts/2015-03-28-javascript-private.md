---
layout: post
title:  "Javascript에서 비공개맴버 처리"
date:   2015-03-28
categories: javascript
---

자바스크립트에서 객체와 생성자함수에는 자바와는 달리 public, private등의 별도의 문법이 없다.
그런데 클로저를 이용하면 이와 같은 private처리를 할 수 있다.

아래소스를 보면 객체와 생성자함수 모두 멤버에 접근하고 수정할 수가 있다.

```javascript
var person = {
  name : "Lee",
  getName: function() {
    return this.name;
  }
};
person.name = "kim"; //공개적인 접근이 가능하다.
console.log(person.name);
console.log(person.getName());

function Person() {
  this.name = "Lee";
  this.getName = function() {
    return this.name;
  };
}

var p = new Person();
p.name = "kim"; //공개적인 접근이 가능하다.
console.log(p.name);
console.log(p.getName());
```

그래서 이부분을 클로저를 이용해서 처리할 수가 있다. 먼저 생성자함수를 이용해보자.

```javascript
function Person() {
  //비공개맴버
  var name = "Lee";
  this.getName = function() {
    return name;
  };
}

var p = new Person();
p.name = "kim"; //name이라는 다른 프로퍼티가 추가된것이다.
console.log(p.name); //kim 출력된다.
console.log(p.getName()); //lee가 출력된다.
```

생성자함수에 내부에 var를 통해 지역변수로 만든 것이다. 그래서 외부에서는 클로저를 통해서만 접근이 가능하다.
p.name="kim"은 지역변수가 아니고 프로퍼티 'name'이 추가된 것이다. 즉 지역변수 name이 아니다.
그래서 p.getName()에서 "Lee"가 출력되는 것이다.

그렇다면 객체를 비공개맴버처리하려면 어떻게 할 것인가?
다음 소스를 보자.

```javascript
var per = (function(){
  var name = "Lee";
  return {
    getName: function(){
      return name;
    }
  };
})();
console.log(per.name); //undefined 출력된다.
console.log(per.getName()); //lee가 출력된다.
```

객체 per를 만들고 즉시실행함수를 통해 처리했다.
그런데 이렇게 매번 생성자 함수를 호출하는 방법을 사용하여 객체를 생성하면 비공개맴버가 매번 만들어지게 되어 메모리에 부담을 주고 중복이 발생한다는 점이 단점이다.
그래서 prototype 프로퍼티에 추가하면 같은 생성자로 만든 모든 인스턴스가 공유할수있게 된다.
이렇게 하기 위해서 비공개맴버를 만드는 패턴과 객체리터럴로 비공개맴버를 만드는 패턴을 조합해서 사용해야 한다.
아래 소스를 보자.

```javascript
function Person(){
  //비공개맴버
  var name = "Lee";

  //공개함수
  this.getName = function() {
    return name;
  };
}

Person.prototype = (function(){
  //비공개맴버
  var age = 30;
  //공개 프로토타입
  return {
    getAge: function() {
      return age;
    }
  };
})();

var p = new Person();
console.log(p.getName());
console.log(p.getAge());
```

prototype를 통해서 즉시 실행함수 부분을 넣어서 추가적인 getAge()를 추가했다.
이렇게 되면 getAge()라는 부분의 공통적인 공개함수가 공유되고, 비공개맴버가 var age가 매번 생성되는 낭비를 막을 수 있게 된다.

**참고서적 : Javascript patterns (인사이트)**
