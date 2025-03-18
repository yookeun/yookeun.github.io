---
layout: single
title: "@QueryProjection VS Projections.constructor 차이점"
date: 2025-01-05
categories: [java]
tags: [jpa, querydsl]
---

`Projections.constructor`와 `@QueryProjection`은 서로 다른 상황에서 사용되며, 각각의 장단점이 있다. 그러나 두 기능은 서로 독립적으로 작동할 수 있기 때문에, `Projections.constructor`를 사용할 때 꼭 `@QueryProjection`을 사용할 필요는 없다.

### ✅ Projections.constructor

`Projections.constructor`는 QueryDSL에서 제공하는 기능으로, 직접적으로 DTO 객체를 생성할 때 사용된다. 이 방법은 DTO 클래스의 생성자를 명시적으로 호출하여 결과를 매핑한다. `Projections.constructor`를 사용할 때는 DTO의 생성자가 public으로 선언되어 있어야 하며, QueryDSL 컴파일 시에는 이 생성자의 존재를 검사하지 않는다.

```java 
QUser user = QUser.user;

List<UserDto> users = queryFactory
    .select(Projections.constructor(UserDto.class,
            user.name,
            user.email,
            user.registrationDate))
    .from(user)
    .fetch();
```

### ✅ @QueryProjection

`@QueryProjection`은 QueryDSL의 애노테이션이며, 이를 사용하면 QueryDSL의 코드 생성 단계에서 DTO의 생성자에 대한 참조를 포함하게 된다. 이 방법을 사용하면 타입 안전성이 더 강화되고, 개발 도구에서 자동 완성 기능 등의 이점을 누릴 수 있다. 그러나 이를 위해서는 QueryDSL의 APT(Annotation Processing Tool)가 DTO 클래스를 처리하여 Q-타입 클래스를 생성해야 한다.

```java 
// DTO 클래스 내부
@QueryProjection
public UserDto(String name, String email, LocalDate registrationDate) {
    this.name = name;
    this.email = email;
    this.registrationDate = registrationDate;
}

// QueryDSL 사용
QUser user = QUser.user;

List<UserDto> users = queryFactory
    .select(new QUserDto(
            user.name,
            user.email,
            user.registrationDate))
    .from(user)
    .fetch();
```

### ✅ 사용 차이점

QueryDSL에서 `new`를 사용하여 직접 DTO를 생성할 때, 타입이 올바르게 매핑되지 않으면 오류가 발생할 수 있다. 이는 QueryDSL에서 `new`를 사용하여 직접 DTO를 생성하려고 시도할 때 발생하는 일반적인 문제다. 이 문제를 해결하려면 `Projections.constructor` 또는 `Projections.bean`을 사용하여 DTO를 생성하는 것이 좋다.

#### ❌ 사례 1: 잘못된 DTO 생성

다음과 같은 코드에서 오류가 발생할 수 있다.

```java 
QServiceIo serviceIo = QServiceIo.serviceIo;

List<ServiceIoExternalDto> results = queryFactory
    .select(new QServiceIoExternalDto(
            serviceIo.id,
            serviceIo.name,
            serviceIo.status
    ))
    .from(serviceIo)
    .fetch();
```

이 코드에서는 `new QServiceIoExternalDto(...)`를 사용하고 있지만, QueryDSL에서 자동 생성된 Q-타입 DTO가 올바르게 생성되지 않았거나, 타입이 일치하지 않는 경우 `ClassCastException`이 발생할 수 있다.

#### ✅ 해결 방법: `Projections.constructor` 사용

아래처럼 `Projections.constructor`를 사용하면 더 안전하게 DTO를 생성할 수 있다.

```java 
QServiceIo serviceIo = QServiceIo.serviceIo;

List<ServiceIoExternalDto> results = queryFactory
    .select(Projections.constructor(ServiceIoExternalDto.class,
            serviceIo.id,
            serviceIo.name,
            serviceIo.status
    ))
    .from(serviceIo)
    .fetch();
```

### 📌수정된 QueryDSL 쿼리

다음은 `Projections.constructor`와 `Projections.bean`을 사용하여 `ServiceIoDto`와 `ServiceIoExternalDto`를 올바르게 생성하도록 한 코드다.

```java 
import com.querydsl.core.types.Projections;
import com.querydsl.jpa.impl.JPAQueryFactory;
import org.springframework.stereotype.Repository;
import javax.persistence.EntityManager;
import java.util.List;

@Repository
public class BookRepository {

    private final JPAQueryFactory queryFactory;

    public BookRepository(EntityManager entityManager) {
        this.queryFactory = new JPAQueryFactory(entityManager);
    }

    public List<BookDto> findAllBooks() {
        QBook book = QBook.book;
        QAuthor author = QAuthor.author;

        return queryFactory
                .select(Projections.constructor(BookDto.class,
                        book.bookId,
                        book.title,
                        author.id, author.name,
                        book.publishDate
                ))
                .from(book)
                .leftJoin(author).on(author.id.eq(book.authorId))
                .where(book.isAvailable.eq(true))
                .orderBy(book.publishDate.desc())
                .limit(10)
                .offset(1)
                .fetch();
    }
}
```

### 🎯결론

`@QueryProjection`을 사용하면 생성 시점에 타입 체크가 가능하여 오류를 미리 잡을 수 있지만, DTO가 QueryDSL에 종속되며, 프로젝트 설정이 좀 더 복잡해진다. 반면, `Projections.constructor`는 좀 더 유연하고 DTO가 순수하게 유지되지만, 타입 안전성이 상대적으로 약하다.

어떤 방법을 사용할지는 프로젝트의 요구 사항과 개발 팀의 선호에 따라 결정할 수 있다. `Projections.constructor`를 사용한다면 `@QueryProjection` 없이도 충분히 작동하지만, 타입 안전성과 개발 편의성을 고려하여 `@QueryProjection`을 함께 사용하는 것도 좋은 선택이 될 수 있다.

`Projections.constructor`를 사용하는 것이 더 좋은 경우도 있다.  특히, DTO 클래스가 내부 클래스로 정의되어 있거나, HQL 쿼리에서 직접 사용할 때 경로 문제가 발생하는 경우에 유용하다. `Projections.constructor`를 사용하면 QueryDSL을 통해 컴파일 타임에 타입 검사를 수행할 수 있어 더 안전하다.