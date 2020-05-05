---
layout: post
title:  "익명클래스 사용방법"
date:   2017-01-24
categories: java
---

자바에서 익명클래스 (혹은 익명객체)를 사용하는 방법을 알아보자. 
보통의 경우, 우리는 부모클래스를 상속받아 처리하려면 부모클래스를 상속받는 클래스를 별도로 만들어 처리한다.

```java
public class Person {
	 void  whoAmI() {
		System.out.println("나는 Person 이다.");
	}
}
```

위와 같은 부모클래스를 사용하려면 상속받는 자식클래스를 만들어서 사용하게 된다. 

```java
public class Child extends Person {
	@Override
	void whoAmI() {
		System.out.println("I'm child");
	}
}
```

그런데 만약 Person를 상속받아 처리해야 하는 클래스가 또 필요한 경우, 매번 Child2, Child3… 등등을 만드는 것은 낭비고 불필요한 클래스만 많아진다. 상속받은 클래스가 재사용되면 모를까, 그냥 한번 쓰고 버려진다면 굳이 클래스 파일을 만들 필요는 없다. 이럴 경우에 바로 익명 클래스를 사용하면 된다. 

실행클래스에서 바로 사용한다고 하면 다음과 같이 하면 된다. 

``` java
public static void main(String[] args) {
		Person p = new Person() {
			String name = "Kim";
			@Override
			void whoAmI() {
				System.out.println("나는 " + name + " 이다.");
			}
		};
		p.whoAmI();  // 나는 Kim 이다. 
	}
```

익명클래스의 형식은 다음과 같다

```java
부모클래스 인스턴스 = new 부모클래스() {

};
```

부모클래스의 인스턴스를 생성하면서 `중괄호{}`를 넣고 그 안에 처리구문을 넣어주면 된다. 이때 부모클래스의 메소드를 오버라이드 해주면 된다. 그러면 해당 인스턴스는 익명클래스의 인스턴스가 되어 오바라이드된 메소드를 실행하게 된다. 이렇게 하면 일회성의 자식클래스를 매번 파일로 만들 필요없이 바로 사용하고 버리면 된다. 

하지만 주의 사항이 있다. 익명클래스에 생성된 메소드나 필드는 익명클래스 밖에선 접근할 수가 없다. 

```java
Person p = new Person() {
	String name = "Kim";
	public void callMe() {
		System.out.println("Call callMe method");
	}
	@Override
	void whoAmI() {
		System.out.println("나는 " + name + " 이다.");
	}
};
//p.callMe(); //실행되지 않는다. 에러.
p.whoAmI();
```

`callMe()`라는 메소드를 public으로 만들었지만 익명 클래스 밖에선 `p.callMe()`로 접근되지 않는다. 생각해보면 당연한다. 익명클래스를 선언할 때 **부모클래스 인스턴스 = new 부모클래스(){};** 로 만들지 않았는가? 부모클래스의 인스턴스이므로 당연히 부모클래스의 필드나 메소드만 접근이 가능하다.  

또한 익명클래스는 매개변수로도 전달이 가능하다. 아래와 같이 실행클래스에 메소드를 추가했다.

```java
public static void walk2(Person p) {
	p.whoAmI();
}
```

그러면 `walk2` 메소드에 `Person` 클래스를 매개변수로 넘겨주게 되어 있다. 이때 해당 클래스 파일을 별도 만들필요없이 익명클래스를 바로 만들어서 넘겨주면 된다. 

```java
walk2(new Person(){
	String name = "Lee";
	@Override
	void whoAmI() {
		System.out.println("나는 " + name + " 이다.");
	}
});
```

인터페이스도 익명구현클래스로 만들면 편하게 사용할 수 있다. 

```java
public interface Remocon {
	public void on();
	public void off();
}
```

Remocon인터페이스를 익명클래스로 구현한 클래스를 만들자. 형식은 기존과 같다.

```java
public class Machine {
	Remocon tv = new Remocon() {
		@Override
		public void on() {
			System.out.println("TV을 켭니다");
		}
		@Override
		public void off() {
			System.out.println("TV을 끕니다.");
		}
	};
}
```

이제 실행하는 클래스에서 `클래스명.필드명.메소드`로 구현해주면 된다.

```java
Machine machine = new Machine();
machine.tv.on();   //TV를 켭니다.
machine.tv.off();  //TV를 끕니다. 
```

물론 익명클래스를 메소드나 익명클래스를 매개변수로 받는 메소드로 만들수도 있다. 두가지를 다 구현해보자. 

```java
public void radioWork() {
	Remocon radio = new Remocon() {
		@Override
		public void on() {
			System.out.println("Radio을 켭니다");
		}
		@Override
		public void off() {
			System.out.println("Radio을 끕니다.");
		}
	};	
	radio.on();
	radio.off();
}
	
public void machineWork(Remocon remocon) {
	remocon.on();
	remocon.off();
}
```

radioWork 메소드는 라디오를 켜는 로직이 있고, machineWork는 넘겨준 쪽에서 직접 정하도록 인터페이스만 받는 것으로 처리했다.  실행하는 부분에서 아래처럼 처리하면 된다.

```java
machine.radioWork();
machine.machineWork(new Remocon(){
	@Override
	public void on() {
		System.out.println("컴퓨터를 켭니다.");
	}
	@Override
	public void off() {
		System.out.println("컴퓨터를 켭니다.");
	}
});
```

자. 이렇게 익명클래스를 처리하면 아주 간단히 클래스를 만들고 바로 처리할 수 있는 장점이 있다. 