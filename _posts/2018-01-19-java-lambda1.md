---
layout: post
title:  "Java8에서 람다(Lambda) 정리1"
date:   2018-01-19
categories: java
---

### 1. 람다 이전에는..

java8에서 람다사용을 정리해본다. 사실 람다를 모른다고 해서 자바8을 사용하는데 지장은 없다. 람다를 사용하지 않더라도 익명클래스를 이용해서 처리할 수 있기 때문이다.  

그런데 그렇게되면 자바의 가장 강력한 특징은 스트림을 사용하지 못하게 된다.  즉 아직 람다를 모른다면 스트림을 사용하지 못한다는 뜻이다. 그것은 더 좋은 차를 샀는데 가장 좋은 기능을 모르고 계속 차를 이용하는 것과 같다. 게다가 이미 그차의 신형(자바9)가 나온 상태이다. 따라서 스트림을 잘 알기 위해서 필수적인 항목은 람다를 먼저 다지고 시작하겠다. 

일단, 람다 이전에는 어떠한 상태인지 보자. 그럼 람다가 왜 나왔는지도 알수있다

여러분은 서점용 프로그램을 개발하고 있다. 이 서점에는 현재 9권의 책이 있다. 일단 그 책들을 배열로 만들자.

```java 
    public static void main(String[] args) {
        List<Book> bookList = Arrays.asList(
                new Book("novel1", 10000, "novel"),
                new Book("novel2", 11000, "novel"),
                new Book("novel3", 12000, "novel"),

                new Book("history1", 10000, "history"),
                new Book("history2", 11000, "history"),
                new Book("history3", 12000, "history"),

                new Book("computer1", 10000, "computer"),
                new Book("computer2", 11000, "computer"),
                new Book("computer3", 12000, "computer")
        );
    }
```

서점 주인이 요구한다. 가격이 12000부터 시작되는 책들만 출력해달라고, 아주 쉽다. 

```java 
    public static List<Book> collectBook(List<Book> bookList, int price) {
        List<Book> resultBook = new ArrayList<>();
        for (Book book : bookList) {
            if (book.getPrice() >= price) {
                resultBook.add(book);
            }
        }
        return resultBook;
    }
```

전체 책 리스트와 조건에 해당되는 가격 파라미터만 추가하면 된다. 그런데 요구사항이 또 생겼다. 가격뿐 아니라 특정 타입의 책도 같이 조건으로 해서 출력해달라고 한다. 물론 쉽다. 다형성을 이용해서 메서드명은 같고 파라미터만 추가된 메소드를 추가하면 된다.

```java 
    public static List<Book> collectBook(List<Book> bookList, int price, String type) {
        List<Book> resultBook = new ArrayList<>();
        for (Book book : bookList) {
            if (book.getType().equals(type) && book.getPrice() >= price) {
                resultBook.add(book);
            }
        }
        return resultBook;
    }
```

그런데 또 고객의 요구사항이 생겼다. book 클래스에 출판사가 들어가서 해당 출판사도 조건으로 처리하고 싶단다. 눈치 챘는가? 이런 식으로 계속 고객의 요구사항이 늘어날수록 우리는 같은 메소드를 추가하고 수정해야 한다.  

이 문제를 해결하기 위해서는 `동작 파라미터화` 라는 것을 적용해보자.

### 2. 동작 파라미터화 

동작 파라미터화는 어떤 행위를 수행하는 로직 자체를 파라미터로 전달하는 것이다. 즉 과일이라는 메소드에 사과를 깍는다, 배를 갈다라는 동작을 전달하는 것이다. 이러한 패턴을 `전략 디자인 패턴` 이라고 한다.  우리는 서점에서 가격과 책종류에 따라 결과를 다르게 받고 싶다. 그러면 가격정책과 책종류를 선택하는 것이 결국 동작(행위)를 알고리즘화 할 수 있다. 그래서 가격, 책종류에 대한 처리를 담은 각각의 알고리즘 클래스를 개발하면 된다. 하지만 이 알고리즘에는 공통적으로 해당 가격에 적합한가? 해당 책 종류에 적합한가? 라는 조건이 있기 때문에 리턴처리는 boolean로 하면 된다. 따라서 이 공통적인 부분을 인터페이스로 만들어야 한다. 

```java 
public interface BookPredicate {
    boolean check (Book book);
}
```

이를 구현하는 책 가격을 조회하도록 하는 알고리즘 클래스를 만들어보자.

```java 
public class BookPricePredicate implements BookPredicate {

    private int price;
    public BookPricePredicate(int price) {
        this.price = price;
    }

    @Override
    public boolean check(Book book) {
        return book.getPrice() >= price;
    }
}
```

가격에 대한 파리미터를 받고 `check`에서 해당 가격이상이면 true를 리턴하게 된다. 다음은 책 종류를 가져오는 알고리즘 클래스를 작성하자.

```java 
public class BookTypePredicate implements BookPredicate {

    private String type;

    BookTypePredicate(String type) {
        this.type = type;
    }

    @Override
    public boolean check(Book book) {
        return book.getType().equals(type);
    }
}
```

여기서도 `check`를 통해 해당 책 종류가 맞는지 boolean으로 리턴해주고 있다.  위에서 만든 `collectBook`를 변경해보자.

```java
public static List<Book> collectBook(List<Book> bookList, BookPredicate p) {
	List<Book> result = new ArrayList<>();
    for (Book book : bookList) {
    	if (p.check(book)) {
              result.add(book);
        }
    }
	return result;
}
```

인터페이스 `BookPredicate` 를 파라미터로 받고 있다.  collectBook은 이제 더이상 추가되거나 수정할 필요가 없다. 해당 알고리즘만 구현한 클래스만 넘겨주면 된다. 

```java
BookPricePredicate bookPricePredicate = new BookPricePredicate(12000);
List<Book> book3 = collectBook(bookList, bookPricePredicate);
for (Book book : book3) {
  System.out.println(book.toString());
}

BookTypePredicate bookTypePredicate = new BookTypePredicate("novel");
List<Book> book4 = collectBook(bookList, bookTypePredicate);
for (Book book : book4) {
  System.out.println(book.toString());
}
```

실제 사용시 위와 같이 해당 알고리즘을 구현한 클래스(Predicated 인터페이스를 상속한) 파라미터로 전달하면 된다. 이와 같은 것을 `동작 파라미터화` 라고 한다. 

그런데 문제가 있다. 매번 저렇게 동작한 구현한 클래스 만들어야하느냐이다. 지금은 2개뿐이자면 만약 동작이 다양한 로직이라면 그만큼 클래스도 늘어나야 한다는 것이다. 비효율적이다. 이와 같은 문제를 해결하기 위해서 익명 클래스라는 방법이 있다.



### 3. 익명 클래스 

익명 클래스란 한마디로 이름의 없는 클래스이다. 즉 별도의 클래스 파일을 만들어서 처리하는 것이 아니다. 그 말은 자주 재사용하는 용도가 아니고 일회성 혹은 상황에 따라 로직변동이 자주 바뀌는 클래스이다.  위에서 처리한 로직에서 특정 가격 아래인 책과 특정 책에 대한 조건이 필요한 클래스가 필요하면 별도의 클래스파일을 만들필요 없이 익명클래스로 처리하면 된다. 

```java
List<Book> book5 = collectBook(bookList, new BookPredicate() {
     @Override
     public boolean check(Book book) {
     	return book.getPrice() < 11000;
     }
});
for (Book book : book5) {
     System.out.println(book.toString());
}
List<Book> book6 = collectBook(bookList, new BookPredicate() {
     @Override
     public boolean check(Book book) {
         return "history".equals(book.getType());
     }
});
for (Book book : book6) {
     System.out.println(book.toString());
}

```

이렇게 하면 얼마든지 조건에 따란 익명 클래스를 만들고 파라미터로 전달할 수 있다. 단 이때 해당 익명 클래스는 인터페이스를 구현하도록 해야 한다. 우리는 collectBook에 이미 인터페이스를 파라미터로 받기로 했기 때문이다. 

하지만 이런 익명클래스도 단점이 있으니 바로 코드의 장황함이다. 파라미터로 전달하는 익명클래스가 너무 길다.  그래서 이러한 것을 해결하기 위해서 드디어 람다(Lambda) 표현식이 등장하는 것이다.

### 4. 람다

위에 익명클래스에서 정의한 메소드가 어떻게 바뀌는지 보자.

```java
/*
List<Book> book5 = collectBook(bookList, new BookPredicate() {                    
    @Override                                                                     
    public boolean check(Book book) {                                             
        return book.getPrice() < 11000;                                           
    }                                                                             
});                                                                               
*/                                                                                
List<Book> book5 = collectBook(bookList, (Book book) -> book.getPrice() < 11000); 
/*
List<Book> book6 = collectBook(bookList, new BookPredicate() {                             
    @Override                                                                              
    public boolean check(Book book) {                                                      
        return "history".equals(book.getType());                                           
    }                                                                                      
});                                                                                        
*/                                                                                         
List<Book> book6 = collectBook(bookList, (Book book) -> "history".equals(book.getType())); 
for (Book book : book6) {                                                                  
    System.out.println(book.toString());                                                   
}                                                                                          
```

총 6줄의 메소드가 간단히 1줄로 요약되었다. 코드가독성도 좋다. 이렇듯 람다표현식은 메소드로 전달할 수 있는 익명함수를 단순화시킨 것이다. 

##### - 람다표현식은  크게 3부분으로 나눈다.

- 파라미터 : (Book book)
- 화살표 : -> 
- 바디 (구현부분) : book.getPrice() < 11000

##### - 람다 표현식 

- () - {} 
- () -> "Seoul"
- () -> {return "Seoul"}
- (Interger a) -> {return "Seoul"+a;}

`return` 사용하면 반드시 중괄호를 사용해야 한다. 

**람다를 사용할 경우에는 반드시 함수형 인터페이스를 통해서 사용할 수 있다.**

함수형 인터페이스는 정확히 하나의 추상 메소드만 지닌 인터페이스를 말한다. 자바 API에서는 Comparator, Runnable 등등 함수형 인터페이스가 존재하는데 바로 이런 함수형 인터페이스만이 람다를 사용할 수 있다. 위에서 샘플로 정의한 `BookPredicate`  인터페이스도 역시 한 개의 메소드밖에 없으므로 함수형 인터페이스이다. 

```java
public interface BookPredicate {
    boolean check (Book book);  //한 개의 메소드만 존재!!!
}
```

그런데 만약 다른 누군가가 함수형 인터페이스에 다른 메소드를 추가하면 어떻게 되는가? 그렇게 되면 람다표현식을 사용한 코드에서는 컴파일에러가 발생하게 된다. 

```java 
public interface BookPredicate {
    boolean check (Book book);
    int get(); // 누군가가 메소드를 추가해버렸다!!! 람다를 쓰는 모든 코드에서 에러가 발생한다.
}
```

이와 같은 오류를 방지하기 위해서 특별한 어노테이션인 `@FunctionalInterface` 을 부여한다. 이렇게 하면 함수형 인터페이스라고 선언하게 되고 만약 메소드를 추가하면 바로 컴파일 에러가 발생한다.

```java
@FunctionalInterface
public interface BookPredicate {
    boolean check (Book book);
}
```

따라서 `@FunctionalInterface` 라는 어노테이션을 추가하는 것이 바람직하다.  

1편은 여기서 끝내고 2편에서는 람다에 대해 좀더 깊이 들어가보도록 하겠다.

[참고서적]

- Java8 in Action (출판사 : Manning,  한빛미디어)