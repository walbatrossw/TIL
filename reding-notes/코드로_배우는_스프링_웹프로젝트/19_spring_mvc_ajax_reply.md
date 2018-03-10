
본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.

그리고 본 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정 파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](http://doublesprogramming.tistory.com/171?category=667155) 포스팅을 참고해주시면 감사하겠습니다.

포스팅하고 있는 현재 프로젝트의 예제가 혹시 필요하신 분은 깃주소(https://github.com/walbatrossw/spring-mvc-ex)를 통해 얻으실 수 있습니다.

---

#### Spring MVC 기본 개념 및 테스트 정리 포스팅 링크
|순서|포스팅 제목|
|:---:|---|
|1|[IntelliJ에서 Spring MVC Project 생성하기](http://doublesprogramming.tistory.com/171)|
|2|[Spring MVC - MySQL 연결테스트](http://doublesprogramming.tistory.com/172)|
|3|[Spring MVC - MyBatis 설정 및 테스트](http://doublesprogramming.tistory.com/173)|
|4|[Spring MVC 구조](http://doublesprogramming.tistory.com/174)|
|5|[Spring MVC - Controller 작성 연습, WAS없이 Controller 테스트 해보기](http://doublesprogramming.tistory.com/175)|
|6|[Spring MVC + MyBatis](http://doublesprogramming.tistory.com/176)|
|7|[Spring - RestController, ResponseEntity, AJAX](http://doublesprogramming.tistory.com/204)|

#### Spring-MVC 게시판 예제  이전 포스팅 링크
|순서|포스팅 제목|
|:---:|---|
|1|[Intellij를 이용한 Spring-MVC 프로젝트 생성 및 세팅](http://doublesprogramming.tistory.com/177)|
|2|[Bootstrap 템플릿 세팅 (AdminLTE)](http://doublesprogramming.tistory.com/178)|
|3|[기본적인 CRUD 구현 : Persistence, Business 계층](http://doublesprogramming.tistory.com/195)|
|4|[기본적인 CRUD 구현 : Control, Presentation 계층](http://doublesprogramming.tistory.com/196)|
|5|[예외처리](http://doublesprogramming.tistory.com/197)|
|6|[페이징처리 : Persistence, Business 계층](http://doublesprogramming.tistory.com/198)|
|7|[페이징처리 : Control, Presentation 계층](http://doublesprogramming.tistory.com/199)|
|8|[페이징처리 개선, 목록페이지 정보 유지](http://doublesprogramming.tistory.com/200)|
|9|[프로젝트 구조 변경 및 수정사항](http://doublesprogramming.tistory.com/201)|
|10|[검색처리, 동적 SQL](http://doublesprogramming.tistory.com/202)|
---

# Spring MVC - 댓글처리

## 1. 댓글처리를 위한 준비

#### # 전달방식과 처리방식 결정
Rest방식을 이용할 것이라면 해당 컨트롤러를 먼저 작성하고, 그에 맞는 URI를 결정하는 것이 첫단계이다. 댓글처리는 아래와 같은 방식으로 호출하도록 한다.

|URI|전송방식|설명|
|---|---|---|
|`/replies/all/123`|GET|123번게시글의 모든댓글 리스트|
|`/replies/+데이터`|POST|새로운데이터추가|
|`/replies/200 + 데이터`|PUT/PATCH|200 댓글수정|
|`/replies/200`|DELETE|200 댓글삭제|

#### # 댓글을 위한 테이블 생성
하나의 게시글은 여러개의 댓글을 가지는 1:N관계이므로 게시글 테이블과 댓글 테이블의 구조는 아래와 같은 형태가 된다.
![relation](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/20180308_213705.png?raw=true)

아래와 같이 댓글 테이블을 생성하고, 댓글 참조키를 설정해준다.
```sql
-- 댓글 테이블 생성
CREATE TABLE tbl_reply (
  reply_no INT NOT NULL AUTO_INCREMENT,
  article_no INT NOT NULL DEFAULT 0,
  reply_text VARCHAR(1000) NOT NULL,
  reply_writer VARCHAR(50) NOT NULL,
  reg_date TIMESTAMP NOT NULL DEFAULT NOW(),
  update_date TIMESTAMP NOT NULL DEFAULT NOW(),
  PRIMARY KEY (reply_no)
);

-- 댓글 참조키 설정
ALTER TABLE tbl_reply ADD CONSTRAINT FK_ARTICLE
FOREIGN KEY (article_no) REFERENCES tbl_article (article_no);
```

#### # 댓글을 위한 도메인 객체 설계
`src/main/java/기본패키지/reply/domain`패키지를 생성하고 아래와 같이 클래스를 작성해준다.
```java
public class ReplyVO {

    private Integer replyNo;
    private Integer ArticleNo;
    private String replyText;
    private String replyWriter;
    private Date regDate;
    private Date updateDate;

    // getter, setter, toString() 생략...
}
```

#### # 댓글 영속 계층 구현
`/src/java/main/기본패키지/reply/Persistence`패키지를 생성하고, `ReplyDAO`인터페이스에 추상메서드를 정의해준다. 그리고 `ReplyDAO`인테피이스를 구현한 `ReplyDAOImpl`클래스를 만들고, 메서드 오버라이드하여 구현을 해준다. 마지막으로 `replyMapper.xml`에서 댓글 목록, 입력, 수정, 삭제 SQL을 작성해준다.

**`ReplyDAO`**
```java
public interface ReplyDAO {

    List<ReplyVO> list(Integer articleNo) throws Exception;

    void create(ReplyVO replyVO) throws Exception;

    void update(ReplyVO replyVO) throws Exception;

    void delete(Integer replyNo) throws Exception;

}
```
**`ReplyDAOImpl`**
```java
@Repository
public class ReplyDAOImpl implements ReplyDAO {

    private static String NAMESPACE = "com.doubles.mvcboard.mappers.reply.ReplyMapper";

    private final SqlSession sqlSession;

    @Inject
    public ReplyDAOImpl(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public List<ReplyVO> list(Integer articleNo) throws Exception {
        return sqlSession.selectList(NAMESPACE + ".list", articleNo);
    }

    @Override
    public void create(ReplyVO replyVO) throws Exception {
        sqlSession.insert(NAMESPACE + ".create", replyVO);
    }

    @Override
    public void update(ReplyVO replyVO) throws Exception {
        sqlSession.update(NAMESPACE + ".update", replyVO);
    }

    @Override
    public void delete(Integer replyNo) throws Exception {
        sqlSession.delete(NAMESPACE + ".delete", replyNo);
    }
}
```

**`replyMapper.xml`**
```sql
<mapper namespace="com.doubles.mvcboard.mappers.reply.ReplyMapper">

    <select id="list" resultMap="ReplyResultMap">
        SELECT
          reply_no
          , article_no
          , reply_text
          , reply_writer
          , reg_date
          , update_date
        FROM tbl_reply
        WHERE article_no = #{articleNo}
        ORDER BY reply_no
    </select>

    <insert id="create">
        INSERT INTO tbl_reply (
            article_no
            , reply_text
            , reply_writer
        ) VALUES (
            #{articleNo}
            , #{replyText}
            , #{replyWriter}
        )
    </insert>

    <update id="update">
        UPDATE tbl_reply
        SET
            reply_text = #{replyText}
            , update_date = NOW()
        WHERE reply_no = #{replyNo}
    </update>

    <delete id="delete">
        DELETE FROM tbl_reply
        WHERE reply_no = #{replyNo}
    </delete>

    <resultMap id="ReplyResultMap" type="ReplyVO">
        <id property="replyNo" column="reply_no"/>
        <result property="ArticleNo" column="article_no"/>
        <result property="replyText" column="reply_text"/>
        <result property="replyWriter" column="reply_writer"/>
        <result property="regDate" column="reg_date"/>
        <result property="updateDate" column="update_date"/>
    </resultMap>

    <resultMap id="ArticleResultMap" type="ArticleVO">
        <id property="articleNo" column="article_no"/>
        <result property="title" column="title"/>
        <result property="content" column="content"/>
        <result property="writer" column="writer"/>
        <result property="regDate" column="regdate"/>
        <result property="viewCnt" column="viewcnt"/>
    </resultMap>

</mapper>
```

**`mybatis-config.xml`**
`replyMapper.xml`에서 `resultMap`이나 `resultType`을 짧게 쓰기 위해 `ReplyVO`를 `alias` 설정해준다.
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <typeAliases>
        <typeAlias type="com.doubles.mvcboard.article.domain.ArticleVO" alias="ArticleVO"/>
        <typeAlias type="com.doubles.mvcboard.reply.domain.ReplyVO" alias="ReplyVO"/>
    </typeAliases>

</configuration>
```


#### # 댓글 비지니스 계층 구현
`/src/main/java/기본패키지/reply/service`패키지에 `ReplyService`인터페이스와 `ReplyServiceImpl`클래스를 생성해 아래와 같이 작성해준다.
**`ReplyService`**
```java
public interface ReplyService {

    List<ReplyVO> list(Integer articleNo) throws Exception;

    void create(ReplyVO replyVO) throws Exception;

    void update(ReplyVO replyVO) throws Exception;

    void delete(Integer replyNo) throws Exception;

}
```

**`ReplyServiceImpl`**
```java
@Service
public class ReplyServiceImpl implements ReplyService {

    private final ReplyDAO replyDAO;

    @Inject
    public ReplyServiceImpl(ReplyDAO replyDAO) {
        this.replyDAO = replyDAO;
    }

    @Override
    public List<ReplyVO> list(Integer articleNo) throws Exception {
        return replyDAO.list(articleNo);
    }

    @Override
    public void create(ReplyVO replyVO) throws Exception {
        replyDAO.create(replyVO);
    }

    @Override
    public void update(ReplyVO replyVO) throws Exception {
        replyDAO.update(replyVO);
    }

    @Override
    public void delete(Integer replyNo) throws Exception {
        replyDAO.delete(replyNo);
    }
}
```

## 2. Rest방식의 `ReplyController` 작성하기

#### # REST방식에서 쓰이는 주요 애너테이션들
- `@PathVariable` : URI의 경로에서 원하는 데이터를 추출하는 용도로 사용한다.
- `@RequestBody` : 전송된 JSON데이터를 객체로 변환해주는 애너테이션으로 @ModelAttribute와 유사한 역할을 하지만 JSON에서 사용한다는 점이 차이점이다.

#### # `ReplyController`클래스 생성 및 작성
`/src/main/java/기본패키지/reply/controller`패키지에 클래스를 생성하고, 아래와 같이 작성해준다.
```java
@RestController
@RequestMapping("/replies")
public class ReplyController {

    private final ReplyService replyService;

    @Inject
    public ReplyController(ReplyService replyService) {
        this.replyService = replyService;
    }

}
```

#### # 댓글 등록처리 메서드 작성
```java
@RequestMapping(value = "", method = RequestMethod.POST)
public ResponseEntity<String> register(@RequestBody ReplyVO replyVO) {
    ResponseEntity<String> entity = null;
    try {
        replyService.create(replyVO);
        entity = new ResponseEntity<>("regSuccess", HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }
    return entity;
}
```

#### # 댓글 목록 메서드 작성
```java
@RequestMapping(value = "/all/{articleNo}", method = RequestMethod.GET)
public ResponseEntity<List<ReplyVO>> list(@PathVariable("articleNo") Integer articleNo) {
    ResponseEntity<List<ReplyVO>> entity = null;
    try {
        entity = new ResponseEntity<>(replyService.list(articleNo), HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(HttpStatus.BAD_REQUEST);
    }
    return entity;
}
```

#### # 댓글 수정 처리 메서드 작성
```java

@RequestMapping(value = "/{replyNo}", method = {RequestMethod.PUT, RequestMethod.PATCH})
public ResponseEntity<String> update(@PathVariable("replyNo") Integer replyNo, @RequestBody ReplyVO replyVO) {
    ResponseEntity<String> entity = null;
    try {
        replyVO.setReplyNo(replyNo);
        replyService.update(replyVO);
        entity = new ResponseEntity<>("modSuccess", HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }
    return entity;
}
```

#### # `Overloaded POST` : 브라우저에서 `PUT`, `PATCH`, `DELETE`방식을 지원하기 위한 필터 추가
브라우저에 따라 `GET`과 `POST`방식을 지원하고, `PUT`, `PATCH`, `DELETE`방식은 지원하지 않는 경우가 발생할 수 있다. 해결책은 브라우저에서 `POST`방식으로 전송하고, 추가적인 정보를 이용해 `PUT`, `PATCH`, `DELETE`와 같은 정보를 함께 전송하는 것이다. 이것을 `Overloaded POST`라고 한다.
**`web.xml`**
```xml
<filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

#### # 댓글 삭제 처리 메서드 작성
```java
@RequestMapping(value = "/{replyNo}", method = RequestMethod.DELETE)
public ResponseEntity<String> delete(@PathVariable("replyNo") Integer replyNo) {
    ResponseEntity<String> entity = null;
    try {
        replyService.delete(replyNo);
        entity = new ResponseEntity<>("delSuccess", HttpStatus.OK);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }
    return entity;
}
```
