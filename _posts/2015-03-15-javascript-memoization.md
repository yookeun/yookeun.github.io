---
layout: post
title:  "자바스크립트에서 메모이제이션(Memoization) 패턴"
date:   2015-03-15
categories: javascript
---

자바스크립트에서 메모이제이션패턴에 대해 알아보자.
메모이제이션 패턴 : 함수내에 불필요한 작업을 피하기 위해 이전에 연산된 결과를 저장하고 사용하는 패턴

이 글은 `더글라스 크락포드의 자바스크립트 핵심가이드` 에서 메모제이션패턴의 소스와 내용을 그대로 참조했다.
피보나치 수열을 구현한 함수를 만든다고 하자.

```javascript
var count1 = 0;
var fibonacci = function(n) {
  count1++;
  return n < 2 ? n: fibonacci(n - 1) + fibonacci(n - 2);
};

for (var i = 0; i <= 10; i += 1) {
  console.log(i + " = " + fibonacci(i));
}
console.log("count1 = " + count1);
```

위 함수를 수행하면 무려 fibonacci함수가 453번이 실해된다(count1를 확인해보라)
이렇게 되면 성능에 막대한 영향을 준다. 따라서 위에 함수를 메모제이셔패턴으로 아래처럼 변경해보자.

```javascript
var fibonacci2 = function(){
  var memo = [0, 1];
  var count2 = 0;
  var fib = function(n) {
    count2++;
    var result = memo[n];
    if (typeof result !== 'number') {
      result = fib(n-1) + fib(n-2);
      memo[n] = result;
    }
    return result;
  };
  return fib;
}();

for (var i = 0; i <= 10; i += 1) {
  console.log(i + " = " + fibonacci2(i));
}
console.log("count2 = " + count2)
```

이렇게 변경되면 29번만 함수가 실행된다(count2를 확인해보자)
전자의 경우에는 1~10번까지의 모든 각각의 수를 비교하기 위해서 fibonacci 함수가 호출되었으나.
후자인 경우에는 핵심은 memo라는 배열을 만들고 그 배열을 `클로저` 를 통해 접근한다.
로직을 처리하는 `클로저` 가 반복되서(피보니치수열을 찾을때까지) 수행됨으로 더 빠르게 처리할 수 있는 것이다.
