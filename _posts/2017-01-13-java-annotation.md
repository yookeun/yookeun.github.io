---
layout: post
title:  "Java annotaion 기본설명"
date:   2017-01-13
categories: java
---

스프링등을 사용하면 어노테이션을 참 많이 사용한다. @Controller, @Component .. 등등
어노테이션에 대해 간단히 알아보고 커스텀 어노테이션을 아주 간단히 만들어보도록 하자.
**용어 및 설명은 `이것이 자바다 (신용권의 Java programming) 정복, 한빛미디어` 이라는 책을 100% 참고했고 인용함을 밝힌다**(강력추천)


### 1. 기본 용어 설명 

어노테이션(Annotation) 이란? **일종의 메타데이터(Metadata)**이다.
메타데이터는 소스코드에 대한 정보를 담고 있다. 

- 컴파일러에게 코드 문법 에러를 체크하도록 정보를 제공
- 소프트웨어 개발 툴이 빌드나 배치 시 코드를 자동으로 생성할 수 있도록 정보를 제공
- 실행시 (런타임시) 특정 기능을 실행하도록 정보를 제공

어노테이션을 만들때  용도를 분명하게 해야 한다. 소스상에서만 유지해야 할지, 컴파일된 클래스에도 유지해야 할지, 런타임 시에도 유지해야 할지를 지정해야 한다. 

이때 `리플렉션(Reflection)`이라는 용어가 등장한다.

> 런타임 시에 클래스의 메타 정보를 얻는 기능을 말한다. 예를 들어 클래스가 가지고 있는 필드가 무엇인지, 어떤 생성자를 가지고 있는지, 어떤 메소드를 가지고 있는지, 적용된 어노테이션이 무엇인지를 알아내는 것이 리플렉션이다.



### 2. 어노테이션 만들기

간단하게 용어정리를 해봤으니 한번 만들어 보자.

```java
public @interface GreetingAnnotation {   
}
```

`@interface` 가 바로 어노테이션 클래스를 말한다. 어노테이션에는 엘리먼트(element)를 가질 수 있다. 엘리먼트는 타입과 이름으로 구성된 어노테이션의 멤버라고 할 수 있다.   

```java
public @interface GreetingAnnotation {
    String value() default "ko";
    String greeting() default "안녕하세요.";
}
```

value() 디폴트 엘리먼트 이고, greeting()이라는 멤버는 추가된 엘리먼트이다.  메소드처럼 `()` 를 써줘야 한다. 
우리가 만들 GreetingAnnotation은 관련 메소드를 호출시 해당 나라의 언어의 인사말을 표현해주는 기능을 추가한다. default는 한국어로 설정해두었다. 이제 어노테이션을 적용할 대상을 정해보자 

어노테이션을 적용할 대상은 `java.lang.annotation.ElementType` 열거 상수로 정의되어 있다.

| ElementType 열거상수 | 적용대상             |
| ---------------- | ---------------- |
| TYPE             | 클래스, 인터페이스, 열거타입 |
| ANNOTAION_TYPE   | 어노테이션            |
| FIELD            | 필드               |
| CONSTRUCTOR      | 생성자              |
| METHOD           | 메소드              |
| LOCAL_VARIABLE   | 로컬 변수            |
| PACKAGE          | 패키지              |

우리는 여기서 `METHOD`를 사용해본다. `@Target`를 이용하여 적용할 대상을 지정한다. 여러개를 지정할 수 있다. 

```java
@Target({ElementType.METHOD})
public @interface GreetingAnnotation {
    String value() default "ko";
    String greeting() default "안녕하세요.";
}
```

다음은 어노테이션 유지정책을 구현해보자. 소스상에 유지할 것인지, 컴파일된 클래스까지 유지할 것인지, 런타임시에도 유지할 것인지 정하는 것이다. 어노테이션 유지정책은 `java.lang.annotation.RetentionPolicy` 의 열거상수로 되어 있다.

| RetentionPolicy 열거상수 | 설명                                       |
| -------------------- | ---------------------------------------- |
| SOURCE               | 소스상에서만 어노테이션 정보를 유지한다. 소스 코드를 분석할 때만 의미가 있으며 바이트 코드 파일에는 정보가 남지 않는다. |
| CLASS                | 바이트 코드 파일까지 어노테이션 정보를 유지한다. 하지만 리플렉션을 이용해서 어노테이션 정보를 얻을 수는 없다. |
| RUNTIME              | 바이트 코드 파일까지 어노테이션 정보를 유지하면서 리플렉션을 이용해서 런타임시에 어노테이션 정보를 얻을 수 있다. |

대부분 커스텀 어노테이션을 만드는 경우는 대부분 런타임 시점이기에 아래처럼 적용한다.

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface GreetingAnnotation {
    String value() default "ko";
    String greeting() default "안녕하세요.";
}
```

자. 이제 어노테이션 만들었으니 적용하는 클래스를 만들자.

```java
public class GreetingService {
    @GreetingAnnotation()
    public void sayHelloKorean() {
        System.out.println("홍길동");
    }

    @GreetingAnnotation(value = "en", greeting = "Hello. ")
    public void sayHelloEnglish() {
        System.out.println("홍길동");
    }
}
```

`sayHelloKorean`은 기본 어노테이션 엘리먼트를 그대로 사용한다.  `sayHelloEnglish`는 엘리먼트에 각각의 값을 주었다.
이제 실행메소드를 만들어보자 

런타임시 어노테이션의 정보를 사용하기 위해선  `java.lang.reflect` 패키지의 Field, Constructor, Method 타입의 배열을 가져와야 한다. 

| 리턴타입          | 메소드명(매개 변수)          | 설명                        |
| ------------- | -------------------- | ------------------------- |
| Field[]       | getFields()          | 필드정보를 Field배열로 리턴         |
| Constructor[] | getConstructors()    | 생성자 정보를 Constructor배열로 리턴 |
| Methods[]     | getDeclaredMethods() | 메소드 정보를 Method배열로 리턴      |

최종 실행 소스는 아래와 같다. 

```java
public class Main {
    public static void main(String[] args) {
        getAnnotation();
    }
    public static void getAnnotation() {
        //GreetingService 클래스에 있는 모든 메소드들을 가져온다
        Method[] methos = GreetingService.class.getDeclaredMethods();
        for (Method method : methos) {
            //가져온 메소드에 어노테이션이 적용되어 있다면
            if(method.isAnnotationPresent(GreetingAnnotation.class)) {
                //GreetingAnnotation 객체를 얻는다.
                GreetingAnnotation greetingAnnotation = method.getAnnotation(GreetingAnnotation.class);
                System.out.print(greetingAnnotation.greeting());
                try {
                    //GreetingService의 메소드를 호출한다.
                    method.invoke(new GreetingService());
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                } catch (InvocationTargetException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

실행하면 아래와 같이 결과가 출력된다. 상세한 설명은 주석으로 대신한다. 

```bash
안녕하세요.홍길동
Hello. 홍길동
```

전체소스는 아래에서 확인할 수 있다.

- [annotation-test](https://github.com/yookeun/annotation-test)
