---
layout: single
title: "Spring에서 @Transactional 사용시 주의점"
date: 2022-07-31
categories: [java]
tags: [spring, transaction]
---

Spring에서 @Transactional를 사용시 주의점이 있다. 바로 하위 메소드에 @Transactional를 거는 것이다.

Controller에서 Service를 호출해서 사용한다.

```java
    @PostMapping("/items")
    public List<ItemDto> saveAll(@Valid @RequestBody List<ItemDto> itemDtos) {
        return txTestService.saveAll(itemDtos);
    }

```

보통 Service에서 바로 @Transactinal를 saveAll등 메소드에 걸지만, 하위메소드로 별도로 분리해서 사용할 경우가 있다. 처리할 로직이 많아서 분리할 경우인데 이때 최초 진입점(Controller 에서 바로 호출하는 메소드)에 @Transactional을 안 걸고 하위에 걸었다면 실제 @Transactional이 작동되지 않는다. 바로 아래의 경우이다.

```java

    public List<ItemDto> saveAll(List<ItemDto> itemDtos) {
        return subSaveAll(itemDtos);
    }

    @Transactional
    public List<ItemDto> subSaveAll(List<ItemDto> itemDtos) {
        List<Item> items = itemDtos.stream().map(ItemDto::toEntity).collect(Collectors.toList());
        items.forEach(item -> itemRepository.save(item));
        return items.stream().map(ItemDto::of).collect(Collectors.toList());

```

최초 진입 메소드인 saveAll() -> subSaveAll() 를 호출하면서 트랜젠셕을 하위 메소드에 건 상태이다. 이럴 경우 트랜젝션이 적용되지 않는다.

그런데 만약 jpa에서 제공하는 saveAll 메소드를 사용하면 트랜젝션이 적용된다(심지어 하위메소드에 @Transactional를 붙이지 않음)

```java
    public List<ItemDto> subSaveAll2(List<ItemDto> itemDtos) {
        List<Item> items = itemDtos.stream().map(ItemDto::toEntity).collect(Collectors.toList());
        return itemRepository.saveAll(items)
                .stream()
                .map(ItemDto::of)
                .collect(Collectors.toList());
    }
```

saveAll 뿐 아니라, save, delete등 JPA에서 제공하는 메소드는 기본적으로 트랜젝션이 적용되어 있다. saveAll메소드는 엔티티 리스트를 일괄 처리하므로 begin trans -> save(1), save(2), save(3)... -> commit 등이 되는 것과 같다. 위에 subSaveAll 에서 안되는 이유는 각각 save가 begin trasn -> save(1) -> commit, begin trans -> save(2) -> commit 등으로 각각 별도의 트랜젠션이 되므로 전체 트랜젝션이 적용되지 않는 것이다.

트랜젠션은 엔티티 필드수정에서도 중요하다. 보통 jpa에서 엔티티의 필드를 수정함으로 업데이트등을 진행할 수 있다.

예를 들어 Item 엔티티에 아래와 같이 메소드를 통해 필드값을 직접 업데이트가 가능하다.

```java

    public void update(ItemDto itemDto) {
        this.itemName = itemDto.getItemName();
        this.itemType = itemDto.getItemType();
    }
```

그런데 아래와 같이 하위메소드에 transactional을 걸어보자.

Controller

```java
    @PutMapping("/items/{id}")
    public ItemDto update(@PathVariable Long id, @Valid @RequestBody ItemDto itemDto) {
        itemDto.setItemId(id);
        return txTestService.update(itemDto);
    }
```

Service

```java

    public ItemDto update(ItemDto itemDto) {
        return subUpdate(itemDto);
    }

    @Transactional
    public ItemDto subUpdate(ItemDto itemDto) {
        Item item = itemRepository
                .findById(itemDto.getItemId())
                .orElseThrow(() -> new IllegalArgumentException("유효하지 않은 itemID"));
        item.update(itemDto);
        return ItemDto.of(item);
    }
```

Controller에서 update -> Service의 update > subUpdate로 처리해서 해당 id로 엔티티를 가져오고 업데이트를 진행해보면 실제 DB에서 update 가 전혀 진행되지 않는다. 바로 하위메소드에 Transactional을 걸었기 때문이다.

따라서 제대로 동작하기 위해서는 Controller에서 Service로 바로 진입하는 메소드에 @Transactional를 걸어야 된다.

```java
    @Transactional  --> 반드시 진입메소드에 적용해야 한다.
    public ItemDto update(ItemDto itemDto) {
        return subUpdate(itemDto);
    }

    public ItemDto subUpdate(ItemDto itemDto) {
        Item item = itemRepository
                .findById(itemDto.getItemId())
                .orElseThrow(() -> new IllegalArgumentException("유효하지 않은 itemID"));
        item.update(itemDto);
        return ItemDto.of(item);
    }
```

한가지 더 테스트 해보자.

Item엔티티에는 deleteYn이라는 필드가 있다. 우리는 이 필드를 통해서 삭제 여부를 적용할 것이다. 즉 Controller에서 delete가 오면 이 필드가 업데이트 되는 것이다.

Controller

```java
    @DeleteMapping("/items")
    public ItemDto delete(@RequestBody ItemDto itemDto) {
        return txTestService.deleteEntity(itemDto);
    }

```

```java
    @Transactional
    public ItemDto deleteEntity(ItemDto itemDto) {
        Item item = itemDto.toEntity();
        item.delete();
        return ItemDto.of(item);
    }
```

ItemDto를 toEntity를 통해 Entity 타입으로 변환하고 거기에 item.delete() 메소드를 수행한다.

Item 엔티티

```java

    public void delete() {
        this.deleteYn = "Y";
    }
```

메소드에 @Transactional을 걸었으므로 업데이트 될 것이라 생각할 수 있지만 전혀 업데이트가 진행되지 않는다. 실제로 영속성 컨텍스트에 Item 엔티티가 없기 때문에 DB에 반영되지 않는다.

방법은 두 가지이다. itemRepository.save(item); 를 추가하는 방법과 jpa에 find메소드를 이용해서 Item을 불러와서 영속성 컨텍스트에 담는 것이다.

```java
    @Transactional
    public ItemDto deleteEntity(ItemDto itemDto) {
        Item item = itemDto.toEntity();
        item.delete();
        itemRepository.save(item)
        return ItemDto.of(item);
    }
```

```java
    @Transactional
    public ItemDto deleteEntity(ItemDto itemDto) {
        Item item = itemRepository.findById(itemDto.getItemId()).get();
        item.delete();
        return ItemDto.of(item);
    }
```

이렇게 되면 잘 수행된다. Spring에서 @Transactional 처리 부분과 Jpa에서 entity 처리 부분은 컴파일 상 에러가 나지 않는 부분이므로 잘 이해하고 개발을 진행해야 한다.
