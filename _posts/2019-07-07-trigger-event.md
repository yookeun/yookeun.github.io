---
layout: post
title:  "Mysql[Mariadb]에서 Trigger and Event 사용"
date:   2019-07-07
categories: database
---

### 1. Trigger 사용법 

트리거는 특정 테이블에 레코드가 입력(수정)될 경우 연관된 작업을 처리하는 방법이다. 예를 들어서 책을 관리하는 테이블에서 책이 입력될때마다 판매테이블에 책가격에 마진을 포함해서 넣는 과정이 필요하다고 가정하자.

``` sql 
CREATE TABLE `book` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(100) NOT NULL,
  `price` int(11) NOT NULL,
  `reg_date` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE `sale` (
  `book_id` int(11) NOT NULL AUTO_INCREMENT,
  `sale_price` int(11) NOT NULL,
  `reg_date` datetime NOT NULL,
  PRIMARY KEY (`book_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

우리가 하려는 작업은 book테이블에 책이 등록되면 자동적으로 계산해서 판매테이블(sale)에 입력되는 것이다. 

이제 트리거를 만들어보자 

```sql 
DROP TRIGGER IF EXISTS calc_input_sale;
DELIMITER $$
CREATE TRIGGER calc_input_sale
AFTER INSERT on book
FOR EACH ROW 
BEGIN
	DECLARE _book_price int;  -- 원래 책
	DECLARE _sale_price int;  -- 책값에 마진을 포함한 
	DECLARE _book_id int;	  -- 고유키 
	
	
	SET _book_id = new.id;
	SET _book_price = new.price;

	SET _sale_price = _book_price + 1000;

    REPLACE INTO sale(book_id, sale_price, reg_date) VALUES(_book_id, _sale_price, now());
END $$  
DELIMITER ;

```

방금입력된 값을 가져오기 위해선 `new`. 컬럼명을 사용하면 된다. 실제 book 테이블에 입력되면 sale테이블에 잘 입력되었음을 확인할 수 있다. 

### 2. Event 사용 

프로시저등을 사용할때 주기적으로 반복해서 사용할 경우 Event를 사용하면 된다.  이벤트를 사용하려면 my.cnf에 다음과 같이 설정되어있어만 만약에 DB가 재시작될 경우에도 사용할 수 있다. 

``` 
[mysqld]
event_scheduler=ON
```

##### 5분마다 실행되는 이벤트 

``` sql 
DROP EVENT IF EXISTS `event_name`;
DELIMITER $$
CREATE EVENT `event_name`
ON SCHEDULE EVERY 5 MINUTE
ON COMPLETION NOT PRESERVE
ENABLE
DO 
begin
call `procedure_name`;
end $$
DELIMITER ;
```

이벤트 등록시점시간으로 5분마다 실행된다. 매시 정각 5분마다 할 경우 아래를 참조해야 한다. 

##### 매일 새벽 2시에 한번만 실행할 경우 

``` sql 
DROP EVENT IF EXISTS `event_name`;
DELIMITER $$
CREATE EVENT `oneday_call`
ON SCHEDULE EVERY 1 DAY 
-- 매일 새벽 2시에 실행 
STARTS (TIMESTAMP(CURRENT_DATE) + INTERVAL 1 DAY + INTERVAL 2 HOUR)
ON COMPLETION NOT PRESERVE
ENABLE
DO 
begin
call `procedure_name`;
end $$
DELIMITER ;
```

`TIMESTAMP` 과 `INTERVAL`  를 이용해서 특정시각에 정확히 실행할 수 있다. 

