---
layout: single
title: "Hibernate.initialize()란"
date: 2025-02-10
categories: [java]
tags: [jpa]
---

Hibernate.initialize()는 Hibernate에서 지연 로딩(Lazy Loading) 된 엔터티나 컬렉션을 명시적으로 초기화하는 메서드다. 지연 로딩 전략을 사용할 때, 관련 엔터티나 컬렉션은 데이터베이스에서 즉시 로드되지 않고, 실제로 접근할 때 데이터베이스 쿼리가 실행된다. 하지만 Hibernate 세션이 닫혀 있으면 LazyInitializationException이 발생할 수 있다. 이를 방지하기 위해 Hibernate.initialize()를 사용하여 트랜잭션 내에서 명시적으로 데이터를 로드할 수 있다.

## 💡 주요 기능 및 사용 사례

### 1. 🔄 지연 로딩된 엔터티 또는 컬렉션 강제 초기화

-   `Hibernate.initialize()`를 사용하면 세션이 닫힌 후에도 데이터를 안전하게 사용할 수 있다.
-   관계 매핑 예시 (`@OneToMany`, `@ManyToOne`, `@OneToOne` 등)에서 `LAZY`로 설정된 필드를 트랜잭션이 종료되기 전에 초기화한다.

**예제 코드:**

```java
@Transactional
    AuthRole authRole = authRoleRepository.findById(roleId).orElseThrow(() -> new EntityNotFoundException());
    // 지연 로딩된 컬렉션 강제 초기화
    Hibernate.initialize(authRole.getAuthRolePages());
    return authRole;
}
```

### 2. 🚫 LazyInitializationException 방지

-   Hibernate 세션이 닫힌 후 지연 로딩된 엔터티에 접근하려고 하면 발생하는 예외다.
-   `Hibernate.initialize()`를 사용해 세션이 닫히기 전에 초기화하여 예외를 방지할 수 있다.

### 3. 🔑 Hibernate 세션이 열려 있는 동안 호출 필요

-   `Hibernate.initialize()`는 **트랜잭션 내에서 호출되어야 한다**. 세션이 열려 있는 동안 호출해야만 지연 로딩된 데이터가 안전하게 초기화된다.
-   세션이 닫힌 후 호출할 경우 `LazyInitializationException`이 발생할 수 있다.

### 4. 📦 Hibernate 프록시 초기화

-   Hibernate는 지연 로딩된 엔터티를 프록시 객체로 대체하여 필요할 때 데이터베이스에서 로드하도록 설계되었다.
-   `Hibernate.initialize()`를 사용하여 프록시 객체를 실제 데이터로 변환할 수 있다.

## 📚 사용 예제

### 지연 로딩된 컬렉션 초기화

```java
@Transactional
public AuthRole findByIdWithPages(Long roleId) {
    AuthRole authRole = authRoleRepository.findById(roleId).orElseThrow(() -> new EntityNotFoundException());
    Hibernate.initialize(authRole.getAuthRolePages());  // 컬렉션 강제 초기화
    return authRole;
}
```

## ✍️ 요약

-   `Hibernate.initialize()`는 **지연 로딩된 엔터티나 컬렉션을 강제로 초기화**하는 메서드다.
-   트랜잭션이 종료되기 전에 호출하여 `LazyInitializationException`을 방지할 수 있다.
-   반드시 세션이 열려 있는 동안에만 호출해야 정상적으로 작동한다.
-   주로 `Lazy Loading`이 적용된 관계에서 사용되며, 서버 이중화 환경이나 캐시 적용 상황에서 유용하다.
