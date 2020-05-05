---
layout: post
title:  "자바스크립트의 프로토타입(Prototype)은 무엇인가"
date:   2015-02-08
categories: javascript
---

자바스크립트에서 중요한 개념중에 하나가 바로 프로토타입이다. 프로토타입은 무엇인가?  

`더글락스의 크락포드의 자바스크립트의 핵심가이드(한빛미디어)`에서 프로토타입을 설명을 부분이다.

>모든 객체는  속성을 상속하는 프로토타입 객체에 연결되어 있습니다.  
>객체 리터럴로 생성되는 모든 객체는 자바스크립트의 표준 객체인 Object의 속성인 prototype(Object.prototype)객체에 연결됩니다.

자바스크립트에서는 class가 존재하지 않는다. 그래서 자바스크립트에서는 객체의 상속을 위해서 프로토타입이라는 것이 존재하게 된다.
kimParent라는 객체를 만들고 kimChild라는 객체를 만들었다. 우리는 kimChild에서 kimParent의 속성을 상속받으면서 사용하고 싶다. 그래서 아래처럼 코딩을 했다.

```javascript
var kimParent = {
  name: "kim parent",
  age : 30
};
var kimChild = kimParent;
console.log("kimChild = " + kimChild.name);
kimChild.name = "kim child";
console.log("kimChild = " + kimChild.name);
console.log("kimParent = " + kimParent.name);
```

그런데 우리가 의도한 결과와는 다르게 출력된다.  

```javascript
console.log("kimChild = " + kimChild.name); // kim parent가 출력된다.  
console.log("kimChild = " + kimChild.name); // kim child가 출력된다
//여기까지는 좋다. 그런데  
console.log("kimParent = " + kimParent.name); //kim child가 출력된다.
```

즉, 상속이 개념이 아니고 객체의 참조가 전달된 것이다. 같은 객체인 것이다. 자바스크립트에서 객체는 복사가 되지 않게 참조가 된다.
이럴때 바로 객체의 속성을 상속하는 프로토타입 객체를 사용하는 것이다.

Object.create에 대한 함수를 구현했다. 생성자 함수를 만들고 그 함수에 prototype 속성만들고 그 생성자 함수를 리턴한다.

```javascript
if (typeof Object.create !== 'function') {
  Object.create = function(o) {
    var F = function(){};
    F.prototype = o;
    return new F();
  };
}

var kimChild = Object.create(kimParent);
kimChild.book = "javascript";
console.log("kimChild.book = " + kimChild.book);
console.log("kimParent.book = " + kimParent.book);
kimParent.wine = "Red wine";
console.log("kimChild.wine = " + kimChild.wine);
console.log("kimParent.wine = " + kimParent.wine);
```

Object.create메소드에서 kimParent객체를 kimChild의 프로토타입객체로 등록한다.
그렇게 되면 kimChild에서는 kimParent의 속성을 그대로 쓰면서 별도로 kimParent속성과 다르게 구현할 수 있다.
즉, kimParent.name과 kimChild.name은 별개로 관리되어지는 것이다 만약에 kimChild에 새로운 속성을 만들어도 kimParent에는 아무 상관이 없다.
그러나 kimParent에 새로운 속성을 추가하면 kimChild에도 그 속성을 사용할 수 있다.

위에 소스 하단에 보면 kimChild에 book속성을 추가했다. 따라서 kimChild.book에는 "javascript"가 출력되지만, kimParent.book에는 "undefined"가 출력된다.
그러나 kimParent에 wine속성을 추가하면 kimParent, kimChild 모두 wine속성에
"Red wine"이라고 표시된다.

이렇게 객체를 프로토타입으로 처리함으로써 자바와 같은 상속으로 사용할 수 있는 것이다.

**참고서적 : 더글라스 크락포드의 자바스크립트 핵심가이드 (한빛미디어)**
