---
layout: single
title: "JPA에서 JoinTable 사용"
date: 2022-06-30
categories: [java]
tags: [spring, jpa]
---

JPA에서 Join테이블을 사용해본다.

ERD구조는 아래와 같다.

![erd](/assets/images/demo2.jpg)

회원과 아이템 테이블 사이에 my_item이라는 조인테이블을 생성한다. 회원별로 아이템을 등록할 수 있지만, 회원에게 중복된 아이템 등록은 허용하지 않는다. Member 엔티티에 아래와 같이 조인테이블을 설정해준다. 조인테이블은 별도의 엔티티가 필요하지 않다.

```java
@OneToMany
@JoinTable(name = "my_item",
          joinColumns = @JoinColumn(name = "member_id"),
          inverseJoinColumns = @JoinColumn(name = "item_id")
)
@Builder.Default
private List<Item> items = new ArrayList<>();

```

`joinsColumns`에 member와 조인을 할 member_id를 지정하고 `inverseJoinColumns`에는 Item.item_id를 지정하면 되다.

@oneToMany관계이기 때문에 List로 사용한다.

추가적으로 편의메소드를 두개 만든다.

```java

public void addItems(Item item) {
	if (!items.contains(item)) {
	    items.add(item);
    }
}

public List<ItemDto> getItemDtos() {
	return items.stream().map(item -> ItemDto.of(item)).collect(Collectors.toList());
}

```

`addItem()` 메소드는 items에 넣기 위한 메소드이다. 회원마다 동일한 아이템을 중복하지 않기 위해서 contains로 체크하도록 한다.

`getItemDtos()`는 Member를 dto로 변환할때 연관된 Item엔티티를 ItemDto로 처리해서 불러오기 위한 메소드이다.

MemberDto에 itemDtos를 처리하는 로직을 추가한다.

```java
@Builder.Default
private List<ItemDto> itemDtos = new ArrayList<>()

public static MemberDto of(Member member) {
return MemberDto.builder()
	.memberId(member.getMemberId())
	.userId(member.getUserId())
	.userName(member.getUserName())
	.itemDtos(member.getItemDtos())
	.build();
}
```

이제 아이템 등록을 위한 Service를 추가하도록 한다.

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class ItemService {
    private final ItemRepository itemRepository;

    @Transactional
    public List<ItemDto> saveAll(List<ItemDto> itemDtos) {
        List<Item> items = itemDtos.stream().map(itemDto ->itemDto.toEntity()).collect(Collectors.toList());
        return itemRepository.saveAll(items)
                .stream()
                .map(item -> ItemDto.of(item))
                .collect(Collectors.toList());
    }
}
```

jpa에서 제공하는 `saveAll()`메소드를 이용해서 아이템을 일괄 등록하도록 하자. item 일괄등록 json은 아래와 같다.

```json
[
    {
        "itemName": "자바책",
        "itemType": "BOOK"
    },
    {
        "itemName": "진공청소기",
        "itemType": "ELECTRONIC"
    },
    {
        "itemName": "소고기",
        "itemType": "FOOD"
    },
    {
        "itemName": "점퍼",
        "itemType": "DRESS"
    }
]
```

이제 해당 회원에게 아이템을 저장하도록 하자.

```java

@Transactional
public MemberDto addItem(Long memberId, Long[] itemIds) {
	Member member = memberRepository
		.findById(memberId)
		.orElseThrow(() -> new IllegalArgumentException("유효하지 않은 ID"));

	for (Long itemId : itemIds) {
		Item item = itemRepository.findById(itemId)
			.orElseThrow(() -> new IllegalArgumentException("유효하지 않은 Item ID"));
		member.addItems(item);
	}
	return MemberDto.of(member);
}
```

Service에서 addItem은 먼저 memberId를 파라미터로 받고 Item에서 검색된 itemId 값들을 Long[] 받아서 처리한다.

memberId의 유효성을 체크하고 itemIds의 유효성도 체크한다. 그리고 item 엔티티를 가져와서 Member엔티티의 `addItem` 메소드를 이용해서 저장한다. 결과값은 아래와 같다.

```java
{
    "memberId": 2,
    "userId": "park",
    "userName": "박지성",
    "itemDtos": [
        {
            "itemId": 1,
            "itemName": "자바책",
            "itemType": "BOOK"
        },
        {
            "itemId": 2,
            "itemName": "진공청소기",
            "itemType": "ELECTRONIC"
        }
    ]
}
```

실제DB에서 my_item을 select하면 member_id, item_id가 각각 잘 저장되어 있음을 확인할 수 있다.

이렇게 JPA에서 @JoinTable을 이용하면 간단하고 쉽게 처리할 수 있다. 다만 만약 조인테이블이 각각 연관된 테이블의 PK등을 담는 것을 넘어선 어떤 상태값이나, 날짜등의 추가적인 컬럼이 필요하다면 별도의 entity를 만들어서 구현하는 것이 좋다.

[jpa-demo](https://github.com/yookeun/jpa-demo)
