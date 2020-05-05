---
layout: post
title:  "Javascript에서 Prototype은 무엇인가"
date:   2016-01-27
categories: javascript
---

자바스크립트에서 어렵고 이해가 가지 않는 부분이 바로 Prototype이다.
특히 자바개발자에게는 더더욱 헷갈리다. 본문에서는 자바개발자의 마인도로 이해하도록 하겠다.
자바스크립트에서는 자바와 같은 클래스가 존재하지 않는다. 클래스와 없다면 객체지향개발, 즉 상속등이 불가능하다는 의미다.
그런데 자바스크립트에서는  Prototype를 통해서 각 객체와 상속관계를 맺을 수가 있다. 즉 객체지향적인 개발이 가능하다는 뜻이다.

일단 Prototype은 함수만 가질수 있는 속성 즉 프로퍼티이다.

> 모든 함수는 prototype 프로퍼티를 가지고 있고 아울러서 내부 프로퍼티(__prop__)도 가지고 있다.

생성자 함수를 만들어 보고 크롬브라우저-개발자도구의 콘솔에서 확인해보자.

```javascript
function Person(name) {
  this.name = name;
}

var kim = new Person('kim');
console.dir(Person);
console.dir(kim);
```

Person이라는 생성자 객체를 생성하고 kim이라는 객체에 할당했다.
Person의 객체와 kim 객체의 구조를 보자.

![prototype1](/assets/images/prototype1.jpg)

먼저 Person객체를 보자.
Person객체의 구조는 function이라고 표시된다.
이 중 두 가지 속성을 보면 바로 `prototype` 과 `__proto__`이다.

![prototype2](/assets/images/prototype2.jpg)

prototype 프로퍼티 안에 보면 두개의 프로퍼티인  `consturctor`, `__proto__` 보인다.  
그 바깥쪽에는 `__proto__`도 존재한다.
`consturctor`는 function Person(name)으로 표시되어 있다.  그리고 `__proto__`은 object로 쓰여있다.  

즉, Person.prototype의 속성은 자기 자신을 가리키는 것과 같다.  prototype이 원형이라는 뜻이니, 결국 자기자신의 원형을 가르키는 속성이 `prototype`이다.  

`__proto__`속성은 Object를 가르키고 있다. 모든 자바스크립트 함수의 최상위 객체는 Object이다.
그러므로 이 프로퍼티(속성)은 부모객체를 가르킨다는 것을 알 수 있다.  정확히는 부모객체의 prototype을 가르키는 것이다.

바깥쪽의 `__proto__`를 보면 `function()`을 가르킨다. 즉 이 객체는 생성자함수이기 때문에 바로 함수를 상속받는다는 의미이다.
우리가 만든 모든 함수는 함수를 상위객체로 가지고 있는 것이다.

그러면 'consturctor' 는 무엇인가?

>자바스크립트에서는 함수를 생성할 때 함수 자신과 연결된 프로토타입 객체를 동시에 생성하며 서로 prototype과 consturctor로 연결된다.
함수(prototype) <------> 프로토타입객체(consturctor)


다음은 kim객체의 구조를 보자.

![prototype3](/assets/images/prototype3.jpg)

kim객체는 Person객체가 할당되므로 name속성만 kim으로 변경되고 Person객체의 구조로 나타난다.
여기서는 `name`, `__proto__` 프로퍼티만 보여진다. `__proto__`를 보면 Person.prototype를 가리키는 것을 알 수 있다.
그렇기 때문에 kim 객체는 Person의 속성을 모두 사용할 수 있다.

![prototype4](/assets/images/prototype4.jpg)

즉, 생성자 함수로 할당된 변수(객체)는 내부의 `__proto__` 프로퍼티가 해당 생성자 함수의 prototype 프로퍼티를 가리키므로 그 생성자 함수내의 모든 속성과 메소드등을 사용할수 있는 것이다.

아래 그림을 통해 정리를 하자.

![prototype5](/assets/images/prototype5.jpg)

> 함수(Person)를 만들면 그 함수는 자신의 프로토타입 객체(Person.prototype)를 만들고 함수내의 `prototype`과 프로토타입 객체내의 `consturctor`와 연결된다.  함수를 참조하는 객체(kim)는 함수의 프로토타입 객체와 `__prop__`이라는 자신의 내부 프로퍼티와 연결이 되어 해당 함수(Person)의 속성과 메소드에 접근이 가능하다.

즉, 객체(or 함수)등은 자신내부의 `__prop__`의 프로퍼티를 통해서 상위객체의 접근이 가능하다는 것이다. 이것이 바로 프로토타입체이닝이다.
다음 소스를 보자

```javascript
function Person(name) {
  this.name = name;
  this.getName = function() {
    return this.name;
  };
}

var kim = new Person("kim");
console.log(kim.getName());     //kim 출력
console.log(kim.hasOwnProperty('getName')); //true 출력
console.dir(kim);
```

kim.hasOwnProperty를 사용했다. 분명히 kim객체는 hasOwnProperty를 정의한적이 없다. 마찬가지로 상위객체인 Person에서도 정의한적이 없다.
그렇다면 hasOwnProperty를 어떻게 사용할 수 있단 말인가?   
아래 그림을 보자.

![prototype6](/assets/images/prototype6.jpg)

감이 오지 않는가? 변수 kim은 Person를 가리키고 객체변수이다.
Person.__prop__는 Person를 가르킨다(정확히는 Person.prototype)
그안에 __prop__는 Object를 가르킨다(Object.prototype) Object안에 프로퍼티를 보면 바로 `hasOwnProperty`가 존재하기 때문에
kim에서 사용이 가능한 것이다.

이렇게 내부 `__prop__` 프로퍼티로 자신의 부모객체(부모객체.prototype)와 연결이 가능한것이 `프로토타입 체이닝` 이라고 한다.  

이런식으로 부모객체에 연결되다가 종점은 `Object.prototype` 에서 끝이 난다.
Object보다 상위객체는 없기 때문이다.

#### - 참고서적 : 인사이트 자바스크립트(한빛미디어) / 송형주, 고현준 지음 ####
