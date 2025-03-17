---
layout: single
title: "JPA에서 DTO 내의 컬렉션 처리"
date: 2024-12-01
categories: [java]
tags: [spring, jpa]
---

JPA에서 DTO 내의 컬렉션 처리를 설명하는 두 가지 방법과 각각의 장단점에 대해 알아본다

### 방법 1: DTO + 컬렉션 분리 조회 후 매핑

**주요 흐름**

-   부모 `OrderDto`와 자식 `OrderItemDto`를 각각 조회합니다.
-   부모 DTO의 ID를 기준으로 자식 DTO를 IN 절을 사용해 조회합니다.
-   조회한 자식 데이터를 `Map`으로 변환하여 부모 DTO와 매핑합니다.

**구현 과정**

-   `OrderDto` 리스트를 조회

```java
List<OrderDto> result = findOrders(); // order 조회 처리
```

-   부모 DTO의 ID 목록 생성

```java
List<Long> orderIds = result.stream()
    .map(OrderDto::getOrderId)
    .collect(Collectors.toList());
```

-   자식 DTO 조회 (IN 절 사용)

```java
List<OrderItemDto> orderItemDtos = em.createQuery(
    "SELECT new com.example.dto.OrderItemDto(...) " +
    "FROM OrderItem oi " +
    "WHERE oi.orderId IN :orderIds", OrderItemDto.class)
    .setParameter("orderIds", orderIds)
    .getResultList();
```

자식 데이터를 `Map`으로 변환:

```java
Map<Long, List<OrderItemDto>> orderItemMap = orderItemDtos.stream()
    .collect(Collectors.groupingBy(OrderItemDto::getOrderId));
```

부모 DTO와 자식 DTO 매핑

```java
result.forEach(order -> order.setOrderItems(orderItemMap.get(order.getOrderId())));
```

**장점**

-   **효율성**: 부모-자식을 개별적으로 쿼리하므로, 페이징 처리가 가능함
-   **유연성**: 필요한 데이터만 조회하여 처리할 수 있다.

**단점**

-   **추가 로직 필요**: 부모와 자식을 수동으로 매핑해야 하므로 코드가 다소 복잡함
-   **다중 쿼리 발생**: 부모와 자식 데이터를 각각 조회하므로 쿼리 호출이 많아질 수 있다.

### 방법 2: Flat DTO 사용

**주요 흐름**

-   부모와 자식 데이터를 한 번의 쿼리로 조회하고, 이를 Flat 형태의 DTO로 매핑.
-   Flat DTO에서 부모와 자식을 분리하여 최종적인 DTO로 재구성.

**구현과정 **

-   Flat DTO 정의

```java
public class OrderFlatDto {
    private Long orderId;
    private String orderName;
    private String orderItemName;
    private int orderItemPrice;
}
```

-   쿼리로 Flat DTO 조회

```java
List<OrderFlatDto> flatResult = em.createQuery(
    "SELECT new com.example.dto.OrderFlatDto(o.orderId, o.name, i.name, i.price) " +
    "FROM Order o " +
    "JOIN o.orderItems i", OrderFlatDto.class)
    .getResultList();

```

-   Flat 데이터를 그룹핑하여 최종 DTO로 변환

```java
Map<Long, List<OrderFlatDto>> flatMap = flatResult.stream()
    .collect(Collectors.groupingBy(OrderFlatDto::getOrderId));

List<OrderDto> result = flatMap.entrySet().stream()
    .map(entry -> {
        Long orderId = entry.getKey();
        List<OrderFlatDto> flatItems = entry.getValue();

        OrderDto order = new OrderDto(orderId, flatItems.get(0).getOrderName());
        List<OrderItemDto> items = flatItems.stream()
            .map(flat -> new OrderItemDto(flat.getOrderItemName(), flat.getOrderItemPrice()))
            .collect(Collectors.toList());
        order.setOrderItems(items);
        return order;
    })
    .collect(Collectors.toList());

```

**장점**

-   **단일 쿼리**: 부모와 자식 데이터를 한 번에 조회하여 데이터베이스 호출이 적다.
-   **간결한 쿼리**: 단일 쿼리로 필요한 데이터를 모두 가져올 수 있다.

**단점**

-   **페이징 불가**: 부모-자식 관계의 데이터가 조인되면서 결과가 폭발적으로 증가하므로 페이징이 어렵다
-   **복잡한 변환 작업**: Flat 데이터를 부모-자식 구조로 변환하는 추가 로직이 필요하다.

### **결론**

-   **방법 1**은 **페이징**이나 **대규모 데이터 처리**가 필요한 경우 적합하다. 비록 코드가 다소 복잡하지만, 데이터 효율성과 유연성 면에서 유리함.
-   **방법 2**는 작은 데이터셋을 다룰 때 적합. 쿼리 호출 횟수를 최소화하고 데이터베이스 작업을 간소화할 수 있지만, 변환 과정이 추가되어 관리가 어렵다

실제 프로젝트에서는 데이터 규모와 성능 요구 사항을 고려하여 적합한 방식을 선택하는 것이 중요하다.
