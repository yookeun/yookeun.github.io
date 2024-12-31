---
layout: post
title:  "JPA에서 DTO 내의 컬렉션 처리"
date:   2024-12-01
categories: java
---

### 방법1

```java
List<OrderDto> result = findOrders(); ///order 조회 처리

List<Long>orderIds = result.stream().map(o -> o.getOrderId()).collect(Collectors.toList()))
```

쿼리 수행시 자식 부분은 in 절로 처리 된다 

```sql

select m from Order m join orderItem i where o.orderId = i.order_id
where i.orderId in :orderIds
```

자식 부분 처리 

```java
Map<Long, List<OrderItemDto>> orderItemMap = orderItem.stream()
.collect(Collectors.groupingBy(orderItemDto -> orderItemDto.getOrderId)));
```

부모쪽에 넣어준다.

```java
result.foreach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));
```



### 방법2

```java
public class OrderFlatDto {
	private int orderId
	private String orderName;
	
	-- Item을 아래 그대로 flat로 넣어준다. 
	private int orderItemName;
	private int orderItemPrice;
}

```

쿼리라 실행되면 데이터가 뻥튀기 된다(item 갯수 만큼 수행)

```sql
select new OrderFlatDto(....)
from Order o 
join o.member m on o.memberId = m.meberId
join o.orderItem i on o.orderId = i.orderId

```

#### 단점

1. 페이징이 안된다. 
2. OrderFlatDto -> OrderDto로 변환하는 복잡한 처리를 해야 한다.

### 결론 

방법1로 처리하는 것이 비교적 간단하다. 
