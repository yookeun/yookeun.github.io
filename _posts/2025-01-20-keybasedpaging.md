---
layout: single
title: "OFFSET VS Key-based Paging 차이점"
date: 2025-01-20
categories: [java]
tags: [jpa]
---

배치로 페이징 처리시 보통 OFFSET 을 사용하는 경우가 많다. 

## 🚨`OFFSET` 방식의 문제점

`OFFSET`을 사용한 페이징은 데이터가 많아질수록 성능이 급격히 저하될 수 있다. 특히 `ORDER BY`가 포함된 경우 불필요한 데이터 스캔이 증가하여 비효율적이다. 따라서 리스트 기반의 페이징보다는 대량 데이터를 효율적으로 처리하는 **배치 작업**에 적합한 방법을 고려해야 한다.

### ✅ `OFFSET` 방식의 동작 원리

```sql 
SELECT * FROM TB_META_TEMP ORDER BY id ASC LIMIT 10 OFFSET 10000;
```

- `OFFSET`은 지정된 개수만큼 데이터를 건너뛴 후 가져온다.
- 데이터베이스는 **건너뛸 데이터를 먼저 읽고 버린 후** 원하는 데이터를 반환한다.
- 데이터가 많아질수록 불필요한 데이터 스캔이 증가하여 성능 저하가 발생한다.

### 📉 `OFFSET` 증가에 따른 성능 저하

| `OFFSET` 값               | 실제 조회된 행 개수 | 불필요하게 읽은 행 개수 |
| ------------------------- | ------------------- | ----------------------- |
| `OFFSET 0 LIMIT 10`       | 10                  | 0                       |
| `OFFSET 1000 LIMIT 10`    | 1010                | 1000                    |
| `OFFSET 10,000 LIMIT 10`  | 10,010              | 10,000                  |
| `OFFSET 100,000 LIMIT 10` | 100,010             | 100,000                 |

➡ 데이터가 많아질수록 불필요한 읽기 연산이 급격히 증가한다.

## 🚀 대안: `Key-based Paging (Cursor 방식)`

### ✅ Key-based Paging의 개념

- 이전 페이지의 **마지막 ID(**`**lastId**`**)를 기준**으로 데이터를 조회한다.
- `WHERE id > lastId` 조건을 사용하여 불필요한 데이터 스캔을 방지한다.
- 대량 데이터에서도 성능 저하 없이 빠른 조회가 가능하다.

### ✅ Key-based Paging 방식 (효율적)

```sql
SELECT * FROM TB_META_TEMP WHERE id > 100000 ORDER BY id ASC LIMIT 10;
```

- `OFFSET` 없이 `WHERE id > ?` 조건을 사용하여 성능 향상.
- `ORDER BY id ASC`로 정렬하여 안정적인 페이징 유지.
- 데이터 삽입 시에도 효율적인 페이징이 가능.

### ✅ Java 코드 구현 (Key-based Paging)

```java
@Override
public List<MetaTemp> selectMetaTempList(Long lastId, int limit) {
    return queryFactory
            .selectFrom(metaTemp)
            .where(lastId == null ? null : metaTemp.id.gt(lastId))
            .orderBy(metaTemp.id.asc())
            .limit(limit)
            .fetch();
}
```

 **이점:**

- `OFFSET` 없이 `WHERE id > ?` 조건을 사용하여 성능 향상.
- `ORDER BY id ASC`로 정렬하여 안정적인 페이징 유지.
- 대량 데이터에서도 성능 저하 없이 빠르게 조회 가능.

## 📌 Key-based Paging 적용 시 고려 사항

### ✅ 인덱스 적용 필수

- `id` 컬럼이 **Primary Key 또는 Index**로 설정되어 있어야 한다.
- 그렇지 않으면 `WHERE id > ?` 조건도 풀스캔이 발생할 수 있다.

### ✅ 다중 정렬 기준 적용 (예: `created_at` 기준 정렬)

```java
@Override
public List<MetaTemp> selectMetaTempList(Long lastId, LocalDateTime lastCreatedAt, int limit) {
    return queryFactory
            .selectFrom(metaTemp)
            .where(
                lastId == null || lastCreatedAt == null ? null :
                metaTemp.createdAt.gt(lastCreatedAt)
                    .or(metaTemp.createdAt.eq(lastCreatedAt).and(metaTemp.id.gt(lastId)))
            )
            .orderBy(metaTemp.createdAt.asc(), metaTemp.id.asc())
            .limit(limit)
            .fetch();
}
```

✅ `created_at`과 `id`를 함께 사용하여 중복 방지 ✅ `created_at`이 같은 경우 `id` 정렬로 순서 유지

### 🚨 Key-based Paging의 단점

- **이전 페이지로 이동이 어려움** → `OFFSET` 방식은 특정 페이지로 이동 가능하지만, Key-based Paging은 `last_id` 기반이므로 **이전 페이지로 돌아가기 어려움**.
- **데이터 삽입 시 페이지 일관성 문제** → 새로운 데이터가 삽입되면, `id > last_id` 방식이 예상과 다르게 동작할 수 있음.

## 🎯 결론

- `OFFSET` 방식은 데이터가 많아질수록 성능이 급격히 저하됨. 
- `Key-based Paging(WHERE id > last_id)`을 사용하면 성능이 대폭 개선됨. 
- 인덱스 적용 여부 및 다중 정렬 기준을 고려하여 최적화해야 함.
- 리스트의 페이징 처리보다는 배치성의 페이지를 읽어올 때 사용하면 효율적으로 사용할 수 있다. 

