---
layout: single
title: "JAVA객체에서 XML 생성"
date: 2024-12-10
categories: [java]
tags: [java, xml]
---

### 라이브러리 추가

`build.gradle` 파일에 아래와 같이 의존성을 추가한다.

```groovy
implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml'
```

### **객체 정의 및 어노테이션 적용**

XML 매핑을 위해 객체에 Jackson 어노테이션을 추가했다.

-   **`@JacksonXmlRootElement`**: XML 루트 엘리먼트를 지정한다.
-   **`@JacksonXmlProperty`**: XML 속성 또는 엘리먼트의 이름을 정의한다.
-   **`@JacksonXmlElementWrapper`**: 컬렉션 데이터를 감싸는 부모 엘리먼트를 정의한다.
-   **`@JsonFormat`**: 날짜 포맷 지정에 사용된다.

`Person` 클래스 예시

```java

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@JacksonXmlRootElement(localName = "PERSON")
public class Person {

    @JacksonXmlProperty(isAttribute = true, localName = "ID")
    private Long id;

    @JacksonXmlProperty(isAttribute = true, localName = "NAME")
    private String name;

    @JacksonXmlProperty(localName = "ADDRESS")
    private Address address;

    @JacksonXmlProperty(isAttribute = true, localName = "AGE")
    private Integer age;

    @JacksonXmlProperty(isAttribute = true, localName = "CREATEDATE")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createDate;

    @JacksonXmlElementWrapper(localName = "GRADEDS")
    @JacksonXmlProperty(localName = "GRADE")
    private List<Grade> grades;

    @Getter
    @Setter
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Address {
        @JacksonXmlProperty(isAttribute = true, localName = "ZIPCODE")
        private String zipCode;
        @JacksonXmlProperty(isAttribute = true, localName = "ADDR")
        private String addr;
    }

    @Getter
    @Setter
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Grade {
        @JacksonXmlProperty(isAttribute = true, localName = "SUBJECT")
        private String subject;
        @JacksonXmlProperty(isAttribute = true, localName = "SCORE")
        private Integer score;
    }
}
```

### **객체를 XML로 변환하는 메소드**

`XmlMapper`를 활용해 객체를 XML로 변환한다. `JavaTimeModule`은 `LocalDateTime`과 같은 Java 8 날짜/시간 API를 처리하기 위해 추가된다.

메소드 구현

```java
    public <T> String toXmlString(T obj) throws Exception {
        XmlMapper xmlMapper = new XmlMapper();
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        xmlMapper.registerModule(javaTimeModule);
        xmlMapper.configure(ToXmlGenerator.Feature.WRITE_XML_DECLARATION, true);
        return xmlMapper.writeValueAsString(obj);
    }
```

**`WRITE_XML_DECLARATION`**: XML 헤더 선언(`<?xml version="1.0" encoding="UTF-8"?>`)을 포함시킨다.

### **XML 생성 테스트**

`JUnit`을 활용해 객체를 XML로 변환하는 메소드를 테스트했다.

테스트 코드

```java

    @DisplayName("자바 객체를 XML로 변환하는 메소드 ")
    @Test
    void testToXml() throws Exception {
        //given
        Person person = Person.builder()
                .id(10L)
                .name("홍길동")
                .build();

        Address address = Address.builder()
                .zipCode("123456")
                .addr("서울특별시")
                .build();

        Grade grade1 = Grade.builder()
                .subject("ENG")
                .score(100)
                .build();

        Grade grade2 = Grade.builder()
                .subject("MATH")
                .score(200)
                .build();

        person.setAddress(address);
        person.setGrades(List.of(grade1, grade2));

        //when
        String xmlSample = interfaceIoSenderHandler.toXmlString(person);

        //then
        System.out.println(xmlSample);

    }
```

### **실행 결과 예시**

테스트 실행 시 출력될 XML 문자열의 예시는 다음과 같다:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<PERSON ID="10" NAME="홍길동">
    <ADDRESS ZIPCODE="123456" ADDR="서울특별시"/>
    <GRADEDS>
        <GRADE SUBJECT="ENG" SCORE="100"/>
        <GRADE SUBJECT="MATH" SCORE="200"/>
    </GRADEDS>
</PERSON>

```

### **마무리**

이 코드 구현은 Java 객체와 XML 간 변환 작업을 효율적으로 처리할 수 있게 한다. 추가적으로 필요하다면, XML 출력 형식을 커스터마이징하거나, 다양한 객체 구조를 지원하도록 확장 가능하다.
