---
layout: post
title:  "Javascript에서 사용자정의 생성자함수"
date:   2015-03-08
categories: javascript
---

자바스크립트에서 객체를 만드는 방법중에 객체 리터럴, new Object(), 그리고 생성자함수를 통해 만드는 방법이 있다.

가장 좋은 것은 객체 리터럴이고, 쓰지 말아야 하는 것은 new Object()를 통해 객체를 생성하는 것이다. 이유는 인자로 전달되는 값에 때라 생성자 함수가 다른 내장 생성자에 객체 생성을 위임할 수 있고, 따라서 기대한 것과는 다른 객체가 리터될 수 있기이다.

일단, 사용자 정의 생성자함수를 만들어보자.

```javascript
var Person = function(name) {
  this.name = name;
  this.say = function() {
    return "Hello, my name is " + this.name;
  };
};

var p = new Person('kim');
console.log(p.say());
```

결과는 "Hell, my name is kim" 이라고 보여질 것이다.
생성자함수는 자바의 클래스와 매우 유사하다. 생성자함수내에는 this라는 변수를 참조할 수 있고, 해당 함수의 프로토타입도 상속받을 수 있다.

```javascript
Person.prototype.getName = function() {
  return this.name;
};
console.log(p.getName());   // kim 이 출력된다.
```

위와 같이 prototype타입을 통해서 속성이나 메소드를 추가할 수 있고, new 객체로 할당받은 인스턴스도 상속을 통해 추가된 속성이나 메소드를 사용할 수 있다.
객체리터럴로 만들어졌다면 prototype을 통해 만들 수 없고, 그렇기 때문에 상속이 되지 않는다.

new와 함께 생성자 함수를 호출하면 내부적으로 다음이 수행된다.

- 빈객체가 생성된다. 이 객체는 this를 사용할 수 있다.
- this로 참조되는 객체에 프로퍼티와 메소드를 추가된다.
- 만약 리턴값에 다른 객체를 명시하지 않으면 this로 참조된 객체가 리턴된다.

생성자함수내에 return문을 쓰지 않았더라도 암묵적으로 this를 반환하게 되어 있다.

```javascript
var Person = function(name) {
  this.name = name;

  //새로운 객체를 생성한고 반환한다.
  var that = {};
  that.name = "Park";
  return that;
};

var p = new Person('kim');
console.log(p.name);   //Park이 출력된다.
```

결과는 "Park"이 출력된다. 리턴되는 객체를 this가 아닌 내부에 생성된 다른 객체로 반환했기 때문이다.
생성자 함수는 만들때 첫글자를 대문자로 만드는 것이 명명규칙이다. 즉 사용자가 new()통해서 사용하라는 의미이다.
만약에 `new`를 사용하지 않는다면 생성자 내부의 this는 전역객체를 가르키게되므로 엉뚱한 결과가 나온다.


```javascript
var Person = function() {
  this.name = "kim";
};

var p = Person('kim');
console.log(p.name); //error 발생
console.log(window.name); //kim 출력된다.
```

위 소스처럼 `new`를 빼먹으면 this는 전역객체를 가르키므로 p.name에서 에러가 나고 의도치 않은 전역변수가 만들어지면서 값이 저장되는 것이다.  
이러한 잘못을 방지하기 위해서는 this에 모든 멤버를 추가하는 되신, that에 모든 멤버를 추가하고 that을 반환하거나,  
간단한 객체라면 that 지역변수없이 `객체리터럴` 을 통해 객체를 반환하는 방법이 있다.

```javascript
var Person = function(name) {
  var that = {};
  that.name = name;
  return that;
};

var Person2 = function(name) {
  return {
    name: name
  };
};

var p = Person('kim');
console.log(p.name);  //kim 가 출력
var p1 = new Person('kim');
console.log(p1.name); //kim 가 출력
var p2 = Person2('kim2');
console.log(p2.name); //kim2 가 출력
var p3 = Person2('kim2');
console.log(p3.name); //kim2 가 출력
```

위 소스처럼 `new` 사용없이 동일한 결과를 얻을 수 있다. 또는 스스로를 호출하는 생성자패턴으로 해결할수도 있다.

```javascript
var Person = function(name) {
  if(!(this instanceof Person)) {
    return new Person(name);
  }
  this.name = name;
};

var p = Person('kim');
console.log(p.name);  //kim 가 출력
var p1 = new Person('kim');
console.log(p1.name); //kim 가 출력
```

`new`없이 사용했을 경우를 체크해서 내부에서 new를 통해 생성하고 처리함으로서 해결한다.

**Javascript Patterns(인사이트, 스토만 스테파노프지음) 참고 함**
