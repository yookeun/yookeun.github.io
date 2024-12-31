---
layout: post
title:  "JAVA객체에서 XML 생성"
date:   2024-12-10
categories: java
---

### 라이브러리 추가 

```groovy
implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml'
```

### 해당 객체에 어노테이션 적용 

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

### 객체를 XML 문자열로 만들어 주는 메소드 

```java
    public <T> String toXmlString(T obj) throws Exception {
        XmlMapper xmlMapper = new XmlMapper();        
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        xmlMapper.registerModule(javaTimeModule);
        xmlMapper.configure(ToXmlGenerator.Feature.WRITE_XML_DECLARATION, true);
        return xmlMapper.writeValueAsString(obj);
    }
```

### XML 생성 테스트 

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



