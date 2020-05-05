---
layout: post
title:  "초급자한테 자바스크립트에서 클로저란 무슨 용도인가"
date:   2015-01-27
categories: javascript
---

먼저 본 문서는 MDN(Mozilla Developer Network)에서 클로저에 대한 글에 대한 나름대로 초급사용자용으로 추가 설명하였다.  

<https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures>  
따라서 먼저 위 사이트를 보고 잘 이해가 안되는 자바스크립트 초급자에 대한 나름대로 부연설명이다.

자바스크립트를 공부하다보면 `클로저(Closures)`라는 개념이 나온다. 이 개념에 대한 설명글들이 블로그와 사이트에서 잘 소개가 되어 있는데,
그렇다면 과연 언제 쓰는가에 대해 알아볼 필요가 있다. 위에 모질라사이트에서 다음 예제 소스를 보자.

```javascript
function init() {
  var name = "Mozilla";
  function displayName() {
    alert(name);
  }
  displayName();
}
init();
```
이 예제를 실행해 보면 위의 예제 init() 함수와 동일한 결과를 보이는걸 알 수 있다. (알람창에 "Mozilla" 문자열이 보일 것이다).
위 예제와 다른 점은 외부함수의 리턴 값이 내부함수 displayName() 라는 것이다. 흥미롭지 않은가?  

>이 코드가 문제없이 실행되는 것은 직관적이지 않다. 일반적으로 함수안에 정의된 지역변수는 함수가 종료되기 전까지만 존재한다.  
makeFunc() 함수가 종료될 때 이 함수 내부에 정의된 지역변수는 없어지는게 상식적이다. 이 코드가 문제없이 동작하는 걸 보면 다른 일이 일어나고 있는 것 같다!  

>이 퍼즐에 대한 해답은  myFunc 함수가 `클로저(closure)`를 갖는다는 것이다. 클로는 두 개의 것으로 이루어진 특별한 오브젝트이다.
첫 번째는 함수이고 두 번째는 그 함수가 만들어진 환경이다. 그 함수가 만들어진 환경은 함수가 만들어질 때 사용할 수 있었던 변수들로 이루어진다.
이 경우에 myFunc 는 displayName 함수와 "Mozilla" 문자열을 포함하는 클로져이다.

위 사이트에서 저렇게 설명하고 있다. 자바나 C등을 사용하지 않은 초급자한테는 별루 새로운 사실이 아니지만, 자바등을 사용한 사람에게는 낯설다.
자..이제부터 클로저를  모르는 우리가 개발한다고 생각해보자.

>개발자생각 : 'makeFunc() 함수를 만들고 리턴하면 "Mozila"를 반환시켜야지..'

그리고 아래와 같이 코딩한다.  

```javascript
function makeFunc() {
  var name = "Mozilla";  
  return name;
}
var myFunc = makeFunc();
console.log(myFunc());
```

실행을 시켜보자.우리가 원하는 것은 makeFunc()의 함수를 변수에 myFunc에 넣고 myFunc를 실행시키고자 하고자 한다.
그런데 저 결과는 에러가 표시된다.  자..혼란이 온다.  그러나 생각해보면 당연하다.

`var myFunc = makeFunc()`에서 이미 실행되었고 그 결과가 이미 myFunc에 담겨져 있기 때문에 myFunc는 함수를 담은 변수가 아니고,문자를  담은 변수이다.
만약 6번라인을 console.log(myFunc) 로 하면 "Mozilla"라는 문자열이 출력된다.
즉, makeFunc() 함수는 실행되고 결과를 리턴하고 소멸된 것이다.

따라서 선언된 변수(myFunc)에 함수를 넣고 그 변수(함수화된)를 실행함으로써 결과를 얻고자 한다면 바로 위에 처럼 클로저를 처리하면 된다.즉 함수내에 내부함수를 사용함으로써 리턴값을 내부함수로 보내는 것이다.


```javascript
function makeFunc() {
  var name = "Mozilla";
  return function() {
    return name;
  };  
}
var myFunc = makeFunc();
console.log(makeFunc());
console.log(myFunc())
```

즉, makeFunc의 리턴값을 함수자체로 처리하는 것이다.
만약 console.log(makeFunc())를 실행하면 리턴값이 아래와 같이 출력된다.

```javascript
function(){
   return name;
}
```

즉,내부함수가 리턴되는 것이다. 즉 9번라인이 수행됨으로써 myFunc는 함수가 된것이다. 따라서 원래는 `var myFunc = makeFunc()`에서
makeFunc()를 수행했으면 그 라인에서 소멸되서 `console.log(myFunc())`에서는 이미 소멸된 함수(makeFunc)내의 name을 볼수없게 되나
***이 부분을 내부함수를 사용함으로써 본 함수가 소멸되도 내부함수로 접근이 가능하도록 한 것이다. 이것이 바로 클로저이다.***

자, 모질자재단에서 제공하는 다른 예제 소스를 보자.
실용적인 클로저라는 부분이다.

css를 만들고

```css
body {
  font-family: Helvetica, Arial, sans-serif;
  font-size: 12px;
}
h1 {
  font-size: 1.5em;
}
h2 {
  font-size: 1.2em;
}
```

html내에 아래 소스를 넣는다.

```html
<a id="size-12">12</a>
<a id="size-14">14</a>
<a d="size-16">16</a>
```

그리고 자바스크립트를 작성한다.

```javascript
function makeSizer(size) {
  return function() {
    document.body.style.fontSize = size + 'px';
  };
}
var size12 = makeSizer(12);
var size14 = makeSizer(14);
var size16 = makeSizer(16);
document.getElementById('size-12').onclick = size12;
document.getElementById('size-14').onclick = size14;
document.getElementById('size-16').onclick = size16;
```
소스를 보면 'size-12'를 클릭하면  폰트 사이즈가 12가 되고 14를 클릭하면 14, 16을 클릭하면 16으로 된다. 이것이 실용적인 클로저라고 한다. 맞다. 진짜 좋은 예제이다.
그런데 우리가 클로저를 모르는 상태에서 저 소스를 짠다고 생각해보자.

>개발자생각 : 'a태그를 클릭해서 각각 해당 사이즈의 폰트를 변경하는 것을 만들어야지..그 함수명은 makeSizer라고 하고 사이즈값을  파라미터로 받고  그 사이즈로 변경하게 만들어지 ㅋㅋ'

그리고 아래처럼 소스를 작성했다.

```javascript
function makeSizer(size) {
   document.body.style.fontSize = size + 'px';  
}
document.getElementById('size-12').onclick = makeSizer(12);
document.getElementById('size-14').onclick = makeSizer(14);
document.getElementById('size-16').onclick = makeSizer(16);
```

위 소스에서 뭐가 이상한가?  `size-12`를 클릭하면 `makeSizer(12)` 실행되고
`size-14`를 클릭하면 `makeSizer(14)` 실행되고 `size-16`를 클릭하면 `makeSizer(16)` 실행되게 하려는 아주 잘 만든 소스이다. 그렇지 않은가?  

정답은 아니다! 위 소스를 실행하면 모든 폰트가 벌써 16으로 변경되고 만다. 심지어 12,14을 클릭해도 폰트는 16인체로 나온다. 대체 뭐가 문제인가!!!  

위에 소스를 아래과 같이 고쳐보자.

```javascript
function makeSizer(size) {
    alert(size);
    document.body.style.fontSize = size + 'px';  
}
document.getElementById('size-12').onclick = makeSizer(12);
document.getElementById('size-14').onclick = makeSizer(14);
document.getElementById('size-16').onclick = makeSizer(16);
```

`alert(size)`를 추가했다. 자 시작해보자.
어떤가? 바로 12,14,16이라는 alert박스가 실행됨을 알수 있다.클릭할때 함수가 실행되는 것이 아니고 이미 함수가 실행된것이다!!

즉, `document.getElementById('size-12').onclick = makeSizer(12);`에서 이미 함수가 실행되고 결국
`document.getElementById('size-16').onclick = makeSizer(16);` 라인까지 실행되어서
html내의 모든 폰트가 12->14>16으로 변환되면서 마지막 16으로 고정되고 만 것이다.

자바스크립트에서 클릭이나  키보드 누르기등은 이벤트 기반으로 만들어진다. 어떤 특정한 이벤트에 반응하는 코드를 만드는데 이런 코드를 콜백함수라고 한다.즉 이벤트가 행동 후 실행되는함수이다.

따라서 위에서는
`document.getElementById('size-16').onclick = makeSizer(16)`로 작성할때 onclick에 함수 자체가 리턴되어야 콜백으로 등록되면서 클릭시 해당함수가 호출되는데
함수가 리턴되지않고 값이 리턴되면서 콜백으로 처리되는 것이 아니라 함수가 벌써 실행되고 리턴을 처리한 것이 되고만다.이렇게 해서 뜻하지 않은 결과를 초래하게 되고 만 것이다.

따라서 아래와 같이 내부함수를  리턴하게 처리하면 클릭이벤트에서 사용시 makeSizer를 선언해도 바로 내부 함수는 실행되지 않고 클릭시 실행되고 된다.

```javascript
function makeSizer(size) {
	return function() {
		document.body.style.fontSize = size + 'px';
	};
}
```
실제 그런지 소스를 통해 확인할 수 있다.


```javascript
function makeSizer(size) {
  console.log(size);
  return function() {
    document.body.style.fontSize = size + 'px';
    console.log(size);
  };
}
document.getElementById('size-12').onclick = makeSizer(12);
document.getElementById('size-14').onclick = makeSizer(14);
document.getElementById('size-16').onclick = makeSizer(16);
```

위 소스를 실행하면, 12,14,16이 console에 출력된다 당연히 소스상에 makeSizer(사이즈값)를 입력했기 때문이다. 그러나 내부함수는 처리되지 않고 있다.
이 함수가 바로 콜백함수로 처리된 것이다. 그래서 클릭시 비로서 내부함수가 실행되면서 해당폰트로 변경하게 된다.
콘솔에서 해당 사이즈가 클릭시 출력됨을 알 수 있다.

자. 이것이 바로 실용적인 클로저이다.

# 다음은 클로저를 이용해서 private 함수 흉내내기 라는 부분을 보자.  

```javascript
var makeCounter = function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  };  
};
var Counter1 = makeCounter();
var Counter2 = makeCounter();
console.log(Counter1.value());
Counter1.increment();
Counter1.increment();
Counter2.increment();
console.log(Counter1.value());
console.log(Counter2.value());
```

샘플소스에서 alert부분을 console로만 변경했다.
클로저가 없다면 저 소스는 실행될 수 없다. 자바스크립트에서 함수내에 선언된 변수와 함수는
함수가 호출된 다음에는 모두 소멸된다. 즉 접근 할 수가 없는것이다.하지만 클로저가 있음으로해서 우리는 자바의클래스처럼 자바스크립트를 사용할 수가 있다.

익명함수를 만들고 makeCounter라는 변수에 할당했다.
이미 `var Counter1 = makeCounter();` 에서 이미 makeCounter 함수는 실행된 것이다.
그러나 return 타입에 세개의 메소드이다.

>자바스크립트에서 함수와 객체는 같은 기능을 수행하지만 엄연히 다르다
메서드는 어떤 객체의 속성으로 지정된 함수를 말한다.  함수는 그 자체의 어떤 기능을 정의하지만,  메소드는 해당 객체를 통해서 기능을 수행하게 되는 것이다.


따 라서 자바스크립트에서도 함수를 만들고 마치 자바의 클래스처럼 사용하게 할 수 있는 것이 클로저이다. 클로저를 통해서 privateCounter변수, 내부함수인 changeBy를 통해 접근이 가능해진다. 즉, 위에 소스를 아래의 자바처럼 구현하는 것과 같다.

```java
class MakeCounter {
    private int privateCounter = 0;    
    private void  changeBy(int val) {
        privateCounter += val;
    }    
    public void  increment() {
        changeBy(1);
    }
    public void  decrement() {
        changeBy(-1);
    }
    public int  getValue() {
        return privateCounter;
    }
}
public class MakeCounterMain {
    public static void main(String[] args) {
        MakeCounter makeCount = new MakeCounter();
        MakeCounter makeCount2 = new MakeCounter();    
        System.out.println(makeCount.getValue());   //0
        makeCount.increment();
        makeCount.increment();
        System.out.println(makeCount.getValue());   //2
        makeCount.decrement();
        System.out.println(makeCount.getValue());  //1        
        System.out.println(makeCount2.getValue());        //0         
    }
}
```

즉, 클로저를 통해서 public메소드를 만들고 그것을 통해 객체의 내부변수나 함수에 접근할 수 있게 되는 것이다.

#자주하는 실수 : 반복문 안에서 클로저 만들기#
제공되는 샘플소스를 보자

```html
E-mail: <input id="email" name="email" type="text" />
Name: <input id="name" name="name" type="text" />
Age: <input id="age" name="age" type="text" />
```
```javascript
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}
function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];
  for (var i = 0; i &lt; helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
}
setupHelp();
```

위 소스를 실행하려는 개발자는 다음과 같이 생각하고 코딩을 한 것이다.

>'각 입력란의 포커스를 체크하여 각각 해당되는 메시지를 출력해야지..'

그러나 수행해보면 모든 유효성메시지가 같은 메시지가 출력된다.

>'Your age (you must be over 16)'


이것도 역시 당연한 결과이다. for구문으로 돌때 이미 `showHelp()`함수가 실행된 결과가 리턴되고 결국 최종적으로 마지막 메시지만 모두 적용된 것이다.
`document.getElementById(item.id).onfocus()` 는이벤트이다. 즉 콜백함수를 받아야 하는 것인다.
showHelp의 리턴값은 함수자체가 아니고 이미 수행된 결과를 처리하게 된다.

결국 순차적으로 `'Your e-mail address' -> 'Your full name' -> 'Your age(you must be over 16)`이 help 엘리먼트에 할당되면서 우리는 마지막엔
`Your age...`만 보게 되는 것이다.
따라서 `showHelp`함수에 클로저를 적용해서 리턴값이 함수자체로 만들면 된다.
아래처럼 해결되면 된다. (MDN사이트처럼 별도의 함수를 만들어서 처리해도 된다.)

```javascript
function showHelp(help) {  
  return function(){
      document.getElementById('help').innerHTML = help;  
  };
}
function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];
  for (var i = 0; i &lt; helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = showHelp(item.help);
  }
}
setupHelp()
```

이렇게 되면 `document.getElementById(item.id).onfocus()`에는 `showHelp`의 내부함수가 리턴되면서 그것이 콜백함수로 등록이 되면서 포커스 이벤트가 발생시
등록된 콜백함수가 호출됨으로써 비로서 원하는 결과를 얻게 된다.
