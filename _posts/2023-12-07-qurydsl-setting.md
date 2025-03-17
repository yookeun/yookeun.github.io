---
layout: single
title: "Springboot3.x에서 QueryDSL 설정"
date: 2023-12-07
categories: [java]
tags: [java, querydsl]
---

스프링부트 3.x 버전에서 QueryDSL을 설정해 본다.

```java
    //querydsl
    implementation 'com.querydsl:querydsl-jpa:5.0.0:jakarta'
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jakarta"
    annotationProcessor "jakarta.annotation:jakarta.annotation-api"
    annotationProcessor "jakarta.persistence:jakarta.persistence-api"
```

다음 Qfile를 생성할 경로를 처리해준다.

```groovy
def generated = 'src/main/generated'

// Specify Qfile creation location
tasks.withType(JavaCompile) {
    options.getGeneratedSourceOutputDirectory().set(file(generated))
}

// Add querydsl QClass location to ava source set
sourceSets {
    main.java.srcDirs += [ generated ]
}

// Delete QClass directory during gradle clean
clean {
    delete file(generated)
}
```

마지막으로 javaConfig로 처리해주면 완료된다.

```java
@Configuration
public class JpaConfig {
    @PersistenceContext
    private EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(entityManager);
    }
}
```
