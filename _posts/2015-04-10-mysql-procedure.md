---
layout: post
title:  "MySQL OR MariaDB에서 프로시저(Procedure)를 만들어보자."
date:   2015-04-10
categories: database
---

개발하다보면 프로시저를 만들때가 있다.

아래의 같은 구조의 테이블이 존재한다고 가정하자.

```sql
CREATE TABLE BOOKS
(
	bookID CHAR(5) NOT NULL,
	bookName VARCHAR(20) NOT NULL,
	bookOriginPrice DOUBLE NOT NULL,
	bookType VARCHAR(10) NOT NULL,
	PRIMARY KEY(bookID)
);

CREATE TABLE BOOKS_SELL
(
	bookID CHAR(5) NOT NULL,
	bookSellPrice DOUBLE NOT NULL,
	bookType VARCHAR(10) NOT NULL,
	PRIMARY KEY(bookID)
);
```

### 1. Procedure에서 transaction 처리
BOOK테이블에는 초기책에 대한 정보가 입력되고 그리고 BOOK_SELL에 판매될 책의 정보가 입력된다.
프로시저를 이용해서 BOOKS, BOOKS_SELL 테이블에 입력하도록 한다.

```sql
/* DELIMITER는 프로시저 앞,뒤의 위치하여 안에 있는 부분은  한번에 실행될 수 있게 하는 역할을 한다. */
DELIMITER $$
CREATE PROCEDURE INSERT_BOOK
(IN _BOOKID CHAR(5), IN _BOOKNAME VARCHAR(20), _PRICE DOUBLE, _BOOKTYPE VARCHAR(10), OUT RESULT INT)
/*
@DESCRIPTION
	BOOKS 테이블에  인서트하고 BOOKS_SELL에 추가된 금액으로 인서트한다.
@PARAM
	_BOOKID: 고유키
	_BOOKNAME : 제목
	_PRICE: 원가
	_BOOKTYPE : 책종류
@RETURN
	RESULT : 실패(-1), 성공 (0)
*/

BEGIN
	/* 가격을 변경할 변수를 선언한다. */
	DECLARE _SELLPRICE DOUBLE;

	/* 만약 SQL에러라면 ROLLBACK 처리한다. */
	DECLARE exit handler for SQLEXCEPTION
	  BEGIN
		ROLLBACK;        
		SET RESULT = -1;  
	END;

	/* 트랜젝션 시작 */
	START TRANSACTION;
		/* BOOK에 인서트 */
		INSERT INTO BOOKS(bookID, bookName, bookOriginPrice, bookType)
		VALUE(_BOOKID, _BOOKNAME, _PRICE, _BOOKTYPE);		

		/* 책종류에 맞게 가격조정 */
		IF _BOOKTYPE = 'novel' THEN
			SET _SELLPRICE = _PRICE + _PRICE * (10/100);
		ELSEIF _BOOKTYPE = 'art' THEN
			SET _SELLPRICE = _PRICE + _PRICE * (15/100);
		ELSE
			SET _SELLPRICE = _PRICE + _PRICE * (20/100);
		END IF;

		/* 조정된 값을 BOOKS_SELL에 저장한다. */
		INSERT INTO BOOKS_SELL(bookID, bookSellPrice, bookType)
		VALUE(_BOOKID, _SELLPRICE, _BOOKTYPE);

	/* 커밋 */
	COMMIT;
	SET RESULT = 0;
END$$
DELIMITER ;
```

작성된 프로시저를 실행해보자.

```sql
CALL INSERT_BOOK('00001','AAA',10000,'novel',@RESULT);
SELECT @RESULT;

CALL INSERT_BOOK('00002','AAB',15000,'art',@RESULT);
SELECT @RESULT;

CALL INSERT_BOOK('00003','AAC',20000,'novel',@RESULT);
SELECT @RESULT;
```

`@RESULT` 는 결과값이 리턴된다. (성공시:0, 실패시 -1)
각 테이블을 조회해보면 정상적으로 레코드가 보일 것이다.

### 2. Procedure에서 cursor 처리

그런데 요구 사항이 발생되어 BOOKS테이블의 원가(bookOriginPrice)가 소설(novel)은 1000원이 올랐고, art는 1500원, 그외는 2000원이 올랐다고 한다.
그렇다면 각각 테이블을 읽어들여 BOOKS테이블의 bookOriginPrice를 업데이트하고,
BOOKS_SELL테이블의 bookSellPrice 값도 변경해주어야 한다. 이것을 프로시저를 만들어보자.


```sql
/* DELIMITER는 프로시저 앞,뒤의 위치하여 안에 있는 부분은  한번에 실행될 수 있게 하는 역할을 한다. */
DELIMITER $$
CREATE PROCEDURE UPDATE_BOOK
(IN _NOVEL_ADD_PRICE DOUBLE, IN _ART_ADD_PRICE DOUBLE, IN _ETC_ADD_PRICE DOUBLE, OUT RESULT INT)
/*
@DESCRIPTION
	BOOKS 테이블에  원가를 올리고,  BOOKS_SELL에 추가된 금액으로 인서트한다.
@PARAM
	_NOVEL_ADD_PRICE : 소설책원가 추가금액
	_ART_ADD_PRICE : 예술책원가  추가금액
	_ETC_ADD_PRICE : 기타원가 추가금액

@RETURN
	RESULT : 실패(-1), 성공 (0)
*/

BEGIN
	/* 종료 구분 변수 */
	DECLARE _done INT DEFAULT FALSE;
	/* 처리된 건수 */
	DECLARE _row_count INT DEFAULT 0;

	/* BOOKS테이블의  각각 컬럼값을 담을 변수 */
	DECLARE _bookID CHAR(5);
	DECLARE _bookOriginPrice DOUBLE;
	DECLARE _bookType VARCHAR(10);

	/* BOOKS_SELL에 bookSellPrice을 담을 변수 */
	DECLARE _bookSellPrice DOUBLE;

	/* 새로운 원가 */
	DECLARE _NEW_ORIGIN_PRICE DOUBLE;
	/* 새로운 판매가 */
	DECLARE _NEW_SELL_PRICE DOUBLE;
	/* BOOKS테이블을 읽어오는 커서를 만든다. */
	DECLARE CURSOR_BOOK CURSOR FOR SELECT bookID, bookOriginPrice, bookType FROM BOOKS;

	/* 커서 종료조건 : 더이상 없다면 종료 */
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET _done = TRUE;

	OPEN CURSOR_BOOK;
	REPEAT
		/* bookID => _bookID, bookOriginPrice => _bookOriginPrice, bookType = _bookType 로 할당한다. */
		FETCH CURSOR_BOOK INTO _bookID, _bookOriginPrice, _bookType;
		IF NOT _done THEN
			SELECT bookSellPrice INTO _bookSellPrice FROM BOOKS_SELL WHERE bookID = _bookID;
			IF _bookType = 'novel' THEN
				/* 원래 책원가에 추가 */
				SET _NEW_ORIGIN_PRICE = _bookOriginPrice + _NOVEL_ADD_PRICE;
				/* 판매책값 변경된 원가에 다시 마진 적용 */
				SET _NEW_SELL_PRICE = _NEW_ORIGIN_PRICE + _NEW_ORIGIN_PRICE * (10/100);
			ELSEIF _bookType = 'art' THEN
				/* 원래 책원가에 추가 */
				SET _NEW_ORIGIN_PRICE = _bookOriginPrice + _ART_ADD_PRICE;
				/* 판매책값 변경된 원가에 다시 마진 적용 */
				SET _NEW_SELL_PRICE = _NEW_ORIGIN_PRICE + _NEW_ORIGIN_PRICE * (15/100);
			ELSE
				/* 원래 책원가에 추가 */
				SET _NEW_ORIGIN_PRICE = _bookOriginPrice + _ETC_ADD_PRICE;
				SET _NEW_SELL_PRICE = _NEW_ORIGIN_PRICE + _NEW_ORIGIN_PRICE * (20/100);

			END IF;
			UPDATE BOOKS SET bookOriginPrice = _NEW_ORIGIN_PRICE WHERE bookID = _bookID;
			UPDATE BOOKS_SELL SET bookSellPrice = _NEW_SELL_PRICE WHERE bookID = _bookID;
			SET _row_count = _row_count + 1;
			SET _done = FALSE;
		END IF;
	UNTIL _done END REPEAT;

	/* 커서를 닫아준다. */
	CLOSE CURSOR_BOOK;
	SET RESULT = _row_count;	 
END$$
DELIMITER ;
```

각각 테이블을 조회하면 증가된 값을 확인할 수 있다.
이처럼 프로시저내에 select처리를 `cursor`와 `repeat`로 처리 할 수 있다.  

커서사용법의 주의사항은 아래 블로그를 참조하는 것이 좋다.

<http://bizadmin.tistory.com/entry/MySQL-Fetch-Cursor-%EB%AC%B8-%EC%82%AC%EC%9A%A9%EB%B0%A9%EB%B2%95>
