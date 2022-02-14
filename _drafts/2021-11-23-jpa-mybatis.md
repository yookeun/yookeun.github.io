---
layout: post
title:  "Spring JPA VS Mybatis 개발비교"
date:   2021-11-23
categories: java
---

Spring JPA와 MyBatis를 이용한 개발 차이를 알아보자. Spring JPA에는 QueryDSL를 추가하여 가급적 실무에서 많이 사용하는 방법을 이용했다. 그외 Spring 설정은 간소화하였고, 게시판 개발등을 통해 비즈니스 로직 구현시 실제 차이점을 알아보도록 한다. 여기에는 두 구성에 대한 차이점만 확인하는 것이므로 JPA, Mybatis의 기술적인 설명은 하지 않겠다.

### 1. 테이블 엔티티 관계도 및 스크립트는 다음과 같다

![erd](/assets/images/jpa-mybatis1.png)


``` sql 
CREATE TABLE `user` (
                        `user_code` int(11) NOT NULL AUTO_INCREMENT,
                        `user_id` varchar(20) NOT NULL,
                        `user_name` varchar(20) NOT NULL,
                        `create_date` datetime(6) NOT NULL,
                        PRIMARY KEY (`user_code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;

insert into `user` (user_id, user_name, create_date) values ('hong', '홍길동', NOW());

CREATE TABLE `board` (
                         `board_no` int(11) NOT NULL AUTO_INCREMENT,
                         `title` varchar(100) NOT NULL,
                         `content` varchar(500) NOT NULL,
                         `user_code` int(11) DEFAULT NULL,
                         `create_date` datetime(6) NOT NULL,
                         PRIMARY KEY (`board_no`),
                         KEY `FK_user_code` (`user_code`),
                         CONSTRAINT `FK_user_code` FOREIGN KEY (`user_code`) REFERENCES `user` (`user_code`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3;
```

user테이블을 생성한 이유는 조인등을 사용하기 위함이다. 


### 2. application.yml 설정
jpa와 mybatis를 동시에 사용하기 위해 아래와 같이 설정한다. 데이터베이스는 동일하게 MariaDB를 사용했다.

```yaml 
server:
  port: 8080

spring:
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: org.mariadb.jdbc.Driver
    url: jdbc:mariadb://localhost:3306/test
    username: root
    password: 1234
    hikari:
      max-lifetime: 30000
  jpa:
    database-platform: org.hibernate.dialect.MariaDB103Dialect
    show-sql: true
    generate-ddl: false
    hibernate:
      ddl-auto: none

    properties:
      hibernate.format_sql: true

mybatis:
  mapper-locations: classpath:mapper/*.xml
  config-location: classpath:mybatis-config.xml

```


### 3. gradle 설정 

spring, mybatis, jpa, querydsl, mapstruct dependencies 설정 

Mybatis 라이브러리 
```yaml 
implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.1.4'
```


JPA, QueryDSL, Mapstruct 라이브러리.
```yaml
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'com.querydsl:querydsl-apt'
implementation 'com.querydsl:querydsl-jpa'
implementation 'org.mapstruct:mapstruct:1.4.2.Final'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
testAnnotationProcessor "org.mapstruct:mapstruct-processor:1.4.2.Final"
```

### 4. mybatis 개발

엔티티 Board 모델 클래스를 만든다. 

```java
@Getter
@Setter
public class Board {
    private int boardNo;
    private String title;
    private String content;
    private int userCode;
    private LocalDateTime createDate;
    private String userId;
    private String userName;
}
```
엔티티 User 모델 클래스를 만든다. 

```java
@Getter
@Setter
@Alias("user")
public class User {
    private int userCode;
    private String userId;
    private String userName;
    private LocalDateTime createDate;
}

```

SQL xml 개발한다. 게시판의 CRUD 처리 부분이다. 

```xml 
<mapper namespace="com.example.test.mybatis.repository.BoardMyBatisMapper">

    <insert id="insertBoard" parameterType="board">
        INSERT INTO board (title, content, user_code, create_date)
        VALUES(#{title}, #{content}, #{userCode}, NOW())
    </insert>

    <select id="selectBoard" resultType="board">
        SELECT
               b.board_no,
               b.content,
               b.title,
               b.create_date,
               b.user_code,
               u.user_id,
               u.user_name
        FROM board b INNER JOIN user u on b.user_code = u.user_code
        ORDER BY b.board_no DESC
    </select>

    <update id="updateBoard" parameterType="board">
        UPDATE board
        <set>
            <if test='title != null and title !=""'>
                title = #{title},
            </if>
            <if test='content != null and content !=""'>
                content = #{content},
            </if>
        </set>
        WHERE board_no = #{boardNo}
    </update>

    <delete id="deleteBoard" parameterType="board">
        DELETE FROM board WHERE board_no = #{boardNo}
    </delete>

</mapper>
```

다음은 repository 인테페이스 부분이다.

```java 
@Mapper
public interface BoardMyBatisMapper {
    int insertBoard(Board board);
    List<Board> selectBoard();
    int updateBoard(Board board);
    int deleteBoard(Board board);
}
```

service 클래스를 만든다.

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MyBatisService {
    private final BoardMyBatisMapper boardMapper;

    @Transactional
    public int insertBoard(Board board) {
        return boardMapper.insertBoard(board);
    }

    public List<Board> selectBoard() {
        return boardMapper.selectBoard();
    }

    @Transactional
    public int updateBoard(Board board) {
        return boardMapper.updateBoard(board);
    }

    @Transactional
    public int deleteBoard(Board board) {
        return boardMapper.deleteBoard(board);
    }

}
```

이제 control 부분을 만들면 끝난다.

```java

@RestController
@RequestMapping("/api/mybatis")
@RequiredArgsConstructor
public class BoardMybatisController {

    private final MyBatisService myBatisService;

    @GetMapping("/board")
    @ResponseBody
    public ResultJson selectBoard() {
        ResultJson resultJson = new ResultJson();
        resultJson.setItems(myBatisService.selectBoard());
        resultJson.setSuccess(true);
        return resultJson;
    }

    @PostMapping("/board")
    @ResponseBody
    public ResultJson inserBoard(Board board) {
        ResultJson resultJson = new ResultJson();
        boolean result = false;
        if (myBatisService.insertBoard(board) > 0) {
            result = true;
        }
        resultJson.setSuccess(result);
        return resultJson;
    }

    @PutMapping("/board")
    @ResponseBody
    public ResultJson updateBoard(Board board) {
        ResultJson resultJson = new ResultJson();
        boolean result = false;
        if (myBatisService.updateBoard(board) > 0) {
            result = true;
        }
        resultJson.setSuccess(result);
        return resultJson;
    }

    @DeleteMapping("/board")
    @ResponseBody
    public ResultJson deleteBoard(Board board) {
        ResultJson resultJson = new ResultJson();
        boolean result = false;
        if (myBatisService.deleteBoard(board) > 0) {
            result = true;
        }
        resultJson.setSuccess(result);
        resultJson.setSuccess(true);
        return resultJson;
    }
}
```


### 5. jpa 개발

Board 엔티티 객체부터 만든다.

```java
@Getter
@Setter
@Entity
@DynamicInsert
@DynamicUpdate
@Table(name = "board")
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "board_no")
    private Integer boardNo;

    @Column(name = "title", nullable = false, length = 100)
    private String title;

    @Column(name = "content", nullable = false, length = 500)
    private String content;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_code")
    private User user = new User();

    @Column(name = "create_date", nullable = false)
    public LocalDateTime createDate;
}
```

User 엔티티 객체를 만든다.

```java
@Getter
@Setter
@DynamicUpdate
@DynamicInsert
@Entity
@Table(name = "user")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_code")
    private Integer userCode;

    @Column(name = "user_id", nullable = false, length = 20, unique = true)
    private String userId;

    @Column(name = "user_name", nullable = false, length = 20)
    private String userName;

    @Column(name = "create_date", nullable = false)
    private LocalDateTime createDate;
}
```

> 여기서 JPA는 entity 객체를 바로 controller 부분에 사용하는 것은 권장하지 않는다. 그 이유는 테이블과 매핑된 객체이기 때문이다. 그래서 controller단에서 파라미터등으로 받고 처리하는 별도의 클래스가 필요하고 편의상 dto 로 정의한다.

여기서는 게시판(board) 작업만 처리하기 때문에 boardDto만 만든다.

```java

@Getter
@Setter
@NoArgsConstructor
public class BoardDto {
    private int boardNo;
    private String title;
    private String content;
    private int userCode;
    public LocalDateTime createDate = LocalDateTime.now();
    private String userId;
    private String userName;

    @QueryProjection
    public BoardDto(int boardNo, String title, String content, LocalDateTime createDate, int userCode, String userId, String userName) {
        this.boardNo = boardNo;
        this.title = title;
        this.content = content;
        this.createDate = createDate;
        this.userCode = userCode;
        this.userId = userId;
        this.userName = userName;
    }
}
```

`@QueryProjection` 어노테이션이 선언된 생성자부분은 Querydsl에서 해당 생성자를 사용하기 위해 만든 것이다. 보통 Querydsl에서 조회 결과등을 entity객체로 리턴하기 보다는 위에 dto를 통해 처리한다.


respository 설정을 한다. jpa는 mybatis처럼 직접 쿼리를 작성하는 부분이 없고 대신 각각 처리하는 클래스가 필요하다. 

```java
public interface BoardJpaRepository extends JpaRepository<Board, Integer>, BoardJpaRepositoryCustom {
    List<Board> findAll();
    Board findBoardByBoardNo(Integer boardNo);
}
```

`findAll`를 통해 전체 리스트를 가져오고 `findBoardByBoardNo`를 통해서 특정 게시물을 가져온다. 위 부분에 대한 쿼리는 만들 필요가 없다.
하지만 board와 user를 조인하는 쿼리가 필요한데, 이때 JPQL를 쓰거나 Querydsl를 사용한다. 보통 프로젝트에서는 Querydsl를 많이 사용하기에 Querydsl로 처리한다.


Querydsl를 처리하기 위해선 `QuerydslRepositorySupport`를 상속받아 설정해주는 작업클래스가 별도로 있는게 편의상 좋다.

```java 
public abstract class QuerydslRepositorySupportExtended extends QuerydslRepositorySupport {

    private JPAQueryFactory jpaQueryFactory;

    /**
     * Creates a new {@link QuerydslRepositorySupportExtended} instance for the given domain type.
     *
     * @param domainClass must not be {@literal null}.
     */
    @SuppressWarnings("SpringJavaInjectionPointsAutowiringInspection")
    public QuerydslRepositorySupportExtended(Class<?> domainClass) {
        super(domainClass);
    }

    @PostConstruct
    public void validate() {
        super.validate();
        this.jpaQueryFactory = new JPAQueryFactory(getEntityManager());
    }

    protected JPAQueryFactory jpaQueryFactory() {
        return jpaQueryFactory;
    }

    @Nullable
    protected <T> AbstractJPAQuery<T, JPAQuery<T>> query() {
        return getQuerydsl().createQuery();
    }

    @Nullable
    public AbstractJPAQuery<Object, JPAQuery<Object>> query(EntityPath<?>... paths) {
        return getQuerydsl().createQuery(paths);
    }
}
```

물론 QuerydslRepositorySupport를 extend해서 Querydsl를 처리하는 클래스에서 구현해도 된다. 

실제 board, user를 조인하는 querydls를 클래스를 만든다. 

```java

@Repository
public class BoardJpaRepositoryImpl extends QuerydslRepositorySupportExtended implements BoardJpaRepositoryCustom {

    private final QBoard qBoard = QBoard.board;
    private final QUser qUser = QUser.user;

    public BoardJpaRepositoryImpl() {
        super(Board.class);
    }

    //QueryResults : 조회한 리스트 + 전체 개수를 포함한 QueryResults 반환. count 쿼리가 추가로 실행된다.
    @Override
    public QueryResults<BoardDto> selectBoardQueryDsl() {
        return jpaQueryFactory().select(new QBoardDto(
                    qBoard.boardNo,
                    qBoard.title,
                    qBoard.content,
                    qBoard.createDate,
                    qUser.userCode,
                    qUser.userId,
                    qUser.userName
                ))
                .from(qBoard)
                .innerJoin(qUser)
                .on(qBoard.user.userCode.eq(qUser.userCode))
                .orderBy(qBoard.createDate.desc())
                .fetchResults();
    }
}
```
`selectBoardQueryDsl` 메소드는 리턴값이 `QueryResults`이고 이것은 실제 select ~ from 쿼리가 수행되면서 select count(*) ~ 쿼리도 함께 수행되는 장점이 있다.
따라서 페이징 처리시 별도 쿼리를 작성할 필요가 없다. 
`BoardJpaRepositoryImpl`를 만들었으면 실제 `BoardJpaRepository`에서 호출할 수 있도록 별도의  커스텀 인터페이스가 필요하다. 

```java 
public interface BoardJpaRepositoryCustom {
    QueryResults<BoardDto> selectBoardQueryDsl();
}
```

`BoardJpaRepositoryImpl`에서 위에 인터페이스를 상속받았으니, 이제 jpa메소드랑 querydsl 메소드를 해당 인테페이스에서 호출 할 수 있게 된다. 


이제 service 부분을 구현하자.

```java
@RequiredArgsConstructor
@Service
@Transactional(readOnly = true)
public class JpaService {
    private final BoardJpaRepository boardJpaRepository;
    private final BoardMapper boardMapper;

    public QueryResults<BoardDto> selectBoardQueryDsl() { return boardJpaRepository.selectBoardQueryDsl(); }

    @Transactional
    public Board saveBoard(BoardDto boardDto) {
        Board board = boardMapper.toEntity(boardDto);
        board.setTitle(boardDto.getTitle());
        board.setContent(boardDto.getContent());
        board.setCreateDate(LocalDateTime.now());
        return boardJpaRepository.save(board);
    }

    @Transactional
    public Board deleteBoard(BoardDto boardDto) {
        Board board = boardJpaRepository.findBoardByBoardNo(boardDto.getBoardNo());
        if (board == null) return null;
        boardJpaRepository.delete(board);
        return board;
    }
}
```

mybatis와 비교하면 update, insert가 하나의 save에서 처리된다. jpa에서는 PK 파리미터가 있다면 update 없다면 insert로 처리한다. 
`saveBoard` 메소드를 보면 `Board board = boardMapper.toEntity(boardDto);` 이부분이 보이는데 이것이 바로 dto 객체를 entity로 변환해주는 소스이다.
위 부분을 빼고 실제로 entity로 구현해도 되지만 controller에서는 dto로 파라미터를 처리하고 entity로 변환해주는 것을 권장한다. 
이런 작업은 수동으로 해도 되지만 자동으로 구현해주는 것이 바로 mapstruct이다. 

mapstruct외 modelmapper도 있는데 아래 블로그를 참조하면 이해하기가 쉬울 것이다.

[ModelMapper와 MapStruct에 대해서 학습하기](https://mangchhe.github.io/spring/2021/01/25/ModelMapperAndMapStruct/)


따라서 여기서는 boardMapper 인터페이스를 별도로 구현한다.

```java
@Mapper(componentModel = "spring", unmappedTargetPolicy = ReportingPolicy.IGNORE)
public interface BoardMapper {
    @Mapping(target = "user.userCode", source = "userCode")
    Board toEntity(BoardDto boardDto);
    BoardDto toDto(Board board);
}
```
`@Mapping(target = "user.userCode", source = "userCode")` 부분을 주의할 필요가 있다. board엔티티를 보면 

```java
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_code")
    private User user = new User();
```
user_code 필드를 다대일로 구현했다. 그러므로 dto > entity로 변환시 이부분에 대한 참조정보가 필요하다. 그래서 위와 같이 처리해야 한다.


이제 마지막으로 control 부분을 구현하자. mybatis와 다른 부분이 거의 없다.

```java
@RestController
@RequestMapping("/api/jpa")
@RequiredArgsConstructor
public class BoardJpaController {

    private final JpaService jpaService;

    @GetMapping("/board")
    @ResponseBody
    public ResultJson selectBoard() {
        ResultJson resultJson = new ResultJson();
        QueryResults<BoardDto> queryResults = jpaService.selectBoardQueryDsl();
        resultJson.setItems(queryResults.getResults());
        resultJson.setTotal(queryResults.getTotal());
        resultJson.setSuccess(true);
        return resultJson;
    }

    @PostMapping("/board")
    @ResponseBody
    public ResultJson inserBoard(BoardDto boardDto) {
        ResultJson resultJson = new ResultJson();
        boolean result = false;
        if (jpaService.saveBoard(boardDto) != null) {
            result = true;
        }
        resultJson.setSuccess(result);
        return resultJson;
    }

    @PutMapping("/board")
    @ResponseBody
    public ResultJson updateBoard(BoardDto boardDto) {
        ResultJson resultJson = new ResultJson();
        boolean result = false;
        if (jpaService.saveBoard(boardDto) != null) {
            result = true;
        }
        resultJson.setSuccess(result);
        return resultJson;
    }

    @DeleteMapping("/board")
    @ResponseBody
    public ResultJson deleteBoard(BoardDto boardDto) {
        ResultJson resultJson = new ResultJson();
        boolean result = false;
        if (jpaService.deleteBoard(boardDto) != null) {
            result = true;
        }
        resultJson.setSuccess(result);
        return resultJson;
    }
}
```


### 6. 단순비교
그럼 mybatis와 jpa(querydsl, mapstruct추가)로 개발할때 어떤 것이 더 생산성이 좋을까? 단순 무식하게 일단 클래스 갯수와 소스 라인수(일반적인 주석포함)부터 따져보자.


**설정파일** 

|구분|설정파일|라인수|
|----|-----|--------|
|MyBatis|Mybastis-config.xml|26|
|MyBatis|DatabaseConfig|11|
|MyBatis|Application.yml에 추가|3|
|MyBatis|build.gradle에 추가|1|
|JPA|QuerydslRepositorySupport.java|45|
|JPA|Application.yml에 추가|9|
|JPA|build.gradle에 추가|25|

단순히 봐도 jpa쪽이 설정작업이 좀 많다. 그런데 어차피 설정은 한번만 하면 되기때문에 큰 의미는 없다. 실제 개발소스를 보자.

**MyBatis 파일 및 소스**

|구분|XML|Model|Repository|Service|Control|라인수|
|----|-----|----------|-------|-------|------|----|
|MyBatis|BoardMyBatisMapper.xml|||||41|
|MyBatis||Board||||20|
|MyBatis||User||||17|
|MyBatis|||BoardMyBatisMapper|||15|
|MyBatis||||MyBatisService||37|
|MyBatis|||||BoardMybatisController|62|

총 7개의 파일을 작성했고, 총 192라인을 작성햇다.

**JPA 파일 및 소스**

|구분|XML|Model|Repository|Service|Control|라인수|
|----|-----|----------|-------|-------|------|----|
|JPA|||||||
|JPA||Board||||35|
|JPA||BoardDto||||32|
|JPA||User||||31|
|JPA|||BoardJpaRepository|||10|
|JPA|||BoardJpaRepositoryCustom|||8|
|JPA|||BoardJpaRepositoryImpl|||39|
|JPA||||JpaService||39|
|JPA|||||BoardJpaController|62|

총 8개의 파일을 작성했고, 총 256라인을 작성했다.
단순무식한 방법으로 비교하면 JPA가 좀 더 작성부분이 많았다(물론 필자가 JPA고수가 아니라 이런 결과가 나올수 있으니 맹신은 하지 않길 바란다)

### 7. 결론
오해가 없기 바란다. 단순히 라인수와 파일수로 생산성을 운운하는 것은 그리 좋은 판단이 아니다. JPA와 MyBatis는 SQL에 의존적인개발(즉 관계형데이터베이스중심)이냐 객체지향적모델링의 개발이냐의 관점의 차이이다. 

관계형 데이터베이스 모델은 객체모델과는 다른 목표와 구조를 가지고 있다. 따라서 객체를 테이블구조에 맞게 매핑하는 작업을 하기 때문에 객체지향보다는 테이블지향 구현을 하게 된다. 이 과정에서 에너지가 상당히 많이 소요된다. 즉 SQL매퍼 작업에 더 많은 시간이 소요되는 것이다. 

JPA는 ORM기술이다. ORM은 객체와 테이블을 매핑해주는 기술을 제공함으로 개발자가 단순 매퍼가 아닌 객체지향적인 개발에 집중하게 한다. 이것이 가장 큰 장점이다. 객체지향적인 개발에 집중할 수 있다는 것은 더 좋은 성능과 더 나은 생산성과 더 좋은 유지보수를 만들 수 있다는 것이다. 
그러므로 단순히 비교해서는 안된다. 필자가 위에 처럼 단순 비교는 하나의 참고용이다. JPA로 개발한다고 해서 수백줄의 코드를 몆줄이면 해결된다는 것이 아니다. 아울러 테이블구조에 대한 지식도 불필요하다는 의미도 아니다. 

다만, ORM은 점점 발전하고 있고 많은 곳에서 채택되고 있다. 그러므로 아직 접하지 않았다면 접할 기회를 잡거나 적극 활용하기 바란다. 

[spring-jpa-mybatis](https://github.com/yookeun/spring-jpa-mybatis)