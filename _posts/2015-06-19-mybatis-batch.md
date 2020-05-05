---
layout: post
title:  "MyBatis에서 Batch처리(SqlSession과 foreach)"
date:   2015-06-19
categories: java
---

Spring과 MyBatis연동시 배치를 처리할 경우가 있다. 한꺼번에 인서트나 업데이트가 필요한 경우있다.
이 때 SqlSession을 반복적으로 처리하는 방법과 xml에서 foreach를 처리하는 방법이 있다.

먼저 DB는 Mariadb(MySql)을 기준으로 설명한다.
test_book_origin 테이블이 있다. 이 테이블에는 originPrice 라는 컬럼이 있고, 각각의 값들이 있다.
originPrice 값을 일괄적으로 0으로 변경해주는 배치를 만들어본다. 물론 쿼리에서 한번에 할 수 있으나, 여기선 배치를 해보기위해 테스트한다.

### 1.SqlSession에서 배치처리

applicationContext.xml에 다음을 추가한다.

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
	<constructor-arg index="0" ref="sqlSessionFactory" />  	
	<constructor-arg index="1" value="BATCH" />
</bean>
```

그리고 mybatis용 SQL처리 xml에는 아래와 같이 일반적으로 작성한다.

```xml
<update id="updateBatch1" parameterType="TestBook">
	UPDATE test_book SET originPrice = 0  WHERE bookID = #{bookID}
</update>
```
그리고 나서 DAO 클래스에서는 다음과 같이 처리한다.

```java
@Override
public void updateBatch1(List<TestBook> list) {
	for (TestBook testBook : list) {
		sqlSession.update("TestBook.updateBatch1", testBook);
	}
}
```

TestBook은 test_book_origin의 엔티티 클래스이다.  
Controller클래스에서 `executeUpdateBatch`를 만들었는데 `test_book_origin` 테이블의 레코드를 읽여들여서
`originPrice`를 0으로 업데이트하는 메소드이다. 여기서는 `batchType=1`(SqlSession방식)로 실행된다.

```java
/**
 * 배치업데이트
 * @param batchType
 */
public void executeUpdateBatch(String batchType) {
	List<TestBook> list = null;
	Map<String, Object> map = null;
	int start = 0;
	final int LIMIT = 500;
	Map<String, Object> paramMap = null;

	try {
		while (true) {
			map = new HashMap<String, Object>();
			map.put("start", start);
			map.put("limit", LIMIT);
			list = bookSerive.selectListTestBook(map);
			start += list.size();

			//더이상 가져올 것이 없다면 빠져나간다.
			if (list == null || list.size() == 0) {
				System.out.println("All finished");
				break;
			}
			//sqlSession 배치라면
			if (batchType.equals("1")) {
				bookSerive.updateBatch1(list);
			}
			//xml의 foreach  배치라면
			else if (batchType.equals("2")) {
				paramMap = new HashMap<String, Object>();
				paramMap.put("list", list);
				bookSerive.updateBatch2(paramMap);
			}				
		} //end while 							

	} catch (Exception e) {
		e.printStackTrace();
	} finally {
		if (list != null) list.clear();
	}		
}
```

Main클래스에서 호출하는 `executeUpdateBatch` 메소드이다.

```java
/**
 * 배치처리 시작
 * @param batchType
 */
private void executeUpdateBatch(String batchType) {
	long startTime = System.currentTimeMillis();
	bookController.executeUpdateBatch(batchType);
	long endTime = System.currentTimeMillis();
	long resutTime = endTime - startTime;
	String batchTypeDesc = "";
	if (batchType.equals("1")) {
		batchTypeDesc = "[sqlSession batch]";
	} else {
		batchTypeDesc = "[foreach batch]";
	}
	System.out.println(batchTypeDesc + " 소요시간  : " + resutTime/1000 + "(ms)");
}
```

실제로 위 메소드를 실행한 소요시간이다. (10만건 업데이트)  

```bash
[sqlSession batch] 소요시간  : 68(ms)
```

이것을 이번에는  xml에서 foreach 처리로 바꾸어보자.

### 2. XML에서 foreach구문으로 배치처리

sql구문처리하는 xml소스를 보자.updateBatch2 메소드가 추가된다.

```xml
<update id="updateBatch2" parameterType="map">
	UPDATE test_book SET originPrice = 0 WHERE bookID IN
	<foreach item="testBook" index="index" collection="list" open="(" separator="," close=")">
		#{testBook.bookID}
	</foreach>
</update>
```
bookID를 IN으로 처리하고 그 부분을 foreach로 처리한다. collection은 list라는 맴변수이고 거기에는 TestBook의 클래스들이 리스트로 담겨져있다.
그러므로 item은 testBook으로 하면 `#{testBook.bookID}`로 접근해야 한다.
DAO클래스를 보자.

```java
@Override
public void updateBatch2(Map<String, Object> map) {
	sqlSession.update("TestBook.updateBatch2", map);
}
```
DAO클래스는 간단하게 작성되었다.(Serive 클래스도 같다) 중요한 것은 실제로 호출하는 Controller클래스이다.

```java
List<TestBook> list = bookSerive.selectListTestBook(map);
...중략...
Map<String, Object>paramMap = new HashMap<String, Object>();
paramMap.put("list", list);
bookSerive.updateBatch2(paramMap);
```
즉, Map파라미터에 list라는 값을 넣고 실제로 collection값을 넣어야 한다. 그래서 foreach구문에 collection값에 map에 할당된 변수를 넣어주고,
item에 변수를 지정하여 그 값으로 반복처리하는 것이다.

이렇게 해서 실행시켜보면 아래과 같은 속도가 나왔다. (10만건 업데이트)

```bash
All finished
[foreach batch] 소요시간  : 7(ms)
```

__무려 9배의 속도가 단축되었다.__ (어찌보면 당연한다 한번에 쿼리로 일괄처리하는 것이 더 효율적인 것이다)

인서트부분도 같은 10만건에 데이트를 다른 테이블에 인서트하는 테스트를 해보았는데 역시 같은 수준의 속도가 나왔다.

> SqlSession 배치 : 75(ms) -> XML에서 Foreach 처리 : 12(ms)

MySQL, MariaDB에서는 대량 부분을 다음과 같이 쿼리로 적용할 수 있다.
즉 VALUES에 해당되는 부분만 반복해서 넣어주면 된다.

```sql
INSERT INTO TEST(id, name) VALUES
(1,'KIM1'),
(2,'KIM2'),
(3,'KIM3')
```

따라서 VALUES 부분만 foreach로 처리해주면 된다.

```xml
<insert id="insertBatch2" parameterType="map">
	INSERT INTO test_book_backup(bookID, bookName, originPrice, registDate)
	VALUES
	<foreach item="testBook" index="index" collection="list" open="" separator="," close="">
		(#{testBook.bookID}, #{testBook.bookName}, #{testBook.originPrice}, NOW())
	</foreach>		
</insert>
```

`()`부분이 반복되므로 open, close는 비워주고, separator만 ","로 처리하면 된다.  
가급적 배치처리에는 Mybatis의  foreach를 사용하는 것이 훨씬 효율적이다.

위 예제전체소스는 <https://github.com/yookeun/mybatis-batch-test> 에서 확인할 수 있다.
