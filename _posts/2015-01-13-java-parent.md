---
layout: post
title:  "[JAVA] 자바에서 부모클래스에서 자식클래스 메소드 사용하기"
date:   2015-01-13
categories: java
---

부모클래스에서 자식클래스의 메소드를 사용할 때가 있다. 자칫 혼돈되기 쉬운 부분이다.  

다음의 부모 클래스가 있다고 한다..

```java
public class Parent {
    public void parentMethod() {
        System.out.println("나는 부모다");
    }
}
```

그리고 자식 클래스에서 부모클래스를 상속받는다고 하자.

```java
public class Child extends Parent {
    public void childMethod() {
        System.out.println("나는 자식이다.");
    }
}
```

클라이언트에서 부모클래스의 인스턴스를 생성하면서 아래처럼 사용하면

```java
Parent parent = new Child();
parent.parentMethod();
```

"나는 부모다" 라는 메시지가 출력된다.  그리고 parent는 당연히 부모 클래스의 메소드(parentMethod)만 접근할 수 있다.
그런데 만약 parent에서 자식 메소드(childMethod)를 호출하고 싶다면 어떻게 해야 할까?  

간단한 것인데 의외로 모를 수 있다.

### 상위클래스를 하위클래스로 처리하기 위해 캐스트가 필요하다.

바로 캐스트로 처리하면 "나는 자식이다" 라는 메시지가 출력된다.

```java
((Child) parent).childMethod()
```
