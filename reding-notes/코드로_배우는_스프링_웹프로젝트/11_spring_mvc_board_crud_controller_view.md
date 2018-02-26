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
|6|[SpringMVC + MyBatis](http://doublesprogramming.tistory.com/176)|

#### Spring-MVC 게시판 예제  이전 포스팅 링크
|순서|포스팅 제목|
|:---:|---|
|1|[Intellij를 이용한 Spring-MVC 프로젝트 생성 및 세팅](http://doublesprogramming.tistory.com/177)|
|2|[Bootstrap 템플릿 세팅 (AdminLTE)](http://doublesprogramming.tistory.com/178)|

---

# Spring MVC 게시판 예제 03 - 기본적인 CRUD 구현

## 1. 게시판 테이블 생성
기본적인 CRUD를 구현하기에 앞서 `spring_ex`스키마에 아래와 같이 `tbl_article`테이블을 생성해준다.
```sql
-- 게시판 테이블
CREATE TABLE tbl_article (
  article_no INT NOT NULL AUTO_INCREMENT,     -- 게시글 번호
  title VARCHAR(200) NOT NULL,                -- 게시글 제목
  content TEXT NULL,                          -- 게시글 내용
  writer VARCHAR(50) NOT NULL,                -- 게시글 작성자
  regdate TIMESTAMP NOT NULL DEFAULT NOW(),   -- 게시글 등록시간
  viewcnt INT DEFAULT 0,                      -- 게시글 조회수
  PRIMARY KEY (article_no)                    -- 게시글 기본키
)
```

## 2. 영속(`DAO`)계층, 비지니스(`Service`) 계층 구현하기

#### # `ArticleVO`클래스 작성
테이블의 구조를 객체화 시킬 때 사용할 `ArticleVO`클래스를 `src/main/java/기본패키지/article/domain`패키지 경로에 작성해준다.
```java
public class ArticleVO {

    private Integer articleNo;

    private String title;

    private String content;

    private String writer;

    private Date regDate;

    private int viewCnt;

    // getter, setter, toString() 생략
}
```

#### # `ArticleDAO`인터페이스 작성
`src/main/java/기본패키지/article/persistence`패키지 경로에 아래와 같이 작성해준다.
```java
public interface ArticleDAO {

    public void create(ArticleVO articleVO) throws Exception;

    public ArticleVO read(Integer articleNo) throws Exception;

    public void update(ArticleVO articleVO) throws Exception;

    public void delete(Integer articleNo) throws Exception;

    public List<ArticleVO> listAll() throws Exception;

}
```

#### # `ArticleDAO`인터페이스를 구현한 `ArticleDAOImpl`클래스 작성
`src/main/java/기본패키지/article/persistence`패키지 경로에 아래와 같이 작성해준다.
```java
@Repository
public class ArticleDAOImpl implements ArticleDAO {

    private static final String NAMESPACE = "com.doubles.mvcboard.mappers.article.ArticleMapper";

    private final SqlSession sqlSession;

    @Inject
    public ArticleDAOImpl(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public void create(ArticleVO articleVO) throws Exception {
        sqlSession.insert(NAMESPACE + ".create", articleVO);
    }

    @Override
    public ArticleVO read(Integer articleNo) throws Exception {
        return sqlSession.selectOne(NAMESPACE + ".read", articleNo);
    }

    @Override
    public void update(ArticleVO articleVO) throws Exception {
        sqlSession.update(NAMESPACE + ".update", articleVO);
    }

    @Override
    public void delete(Integer articleNo) throws Exception {
        sqlSession.delete(NAMESPACE + ".delete", articleNo);
    }

    @Override
    public List<ArticleVO> listAll() throws Exception {
        return sqlSession.selectList(NAMESPACE + ".listAll");
    }
}
```
- `field injection`, `constructor injection`, `setter injection` 정리
- `@Repository`애너테이션
- `NAMESPACE` 정리

#### # `articleMapper.xml` 작성
`src/main/resources/mappers/article/`경로에 아래와 같이 작성해준다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.doubles.mvcboard.mappers.article.ArticleMapper">

    <insert id="create">
        INSERT INTO tbl_article (
            article_no
            , title
            , content
            , writer
            , regdate
            , viewcnt
        ) VALUES (
            #{articleNo}
            , #{title}
            , #{content}
            , #{writer}
            , #{regDate}
            , #{viewCnt}
        )
    </insert>

    <select id="read" resultType="ArticleVO">
        SELECT
            article_no AS articleNo
            , title
            , content
            , writer
            , regdate
            , viewcnt
        FROM
            tbl_article
        WHERE article_no = #{articleNo}
    </select>

    <update id="update">
        UPDATE tbl_article
        SET
            title = #{title}
            , content = #{content}
        WHERE
            article_no = #{articleNo}
    </update>

    <delete id="delete">
        DELETE FROM tbl_article
        WHERE article_no = #{articleNo}
    </delete>

    <select id="listAll" resultType="ArticleVO">
        <![CDATA[
        SELECT
            article_no,
            title,
            content,
            writer,
            regdate,
            viewcnt
        FROM tbl_article
        WHERE article_no > 0
        ORDER BY article_no DESC, regdate DESC
        ]]>
    </select>

    <resultMap id="ArticleResultMap" type="ArticleVO">
        <id property="articleNo" column="article_no"/>
        <result property="title" column="title" />
        <result property="content" column="content" />
        <result property="writer" column="writer" />
        <result property="regDate" column="regdate" />
        <result property="viewCnt" column="viewcnt" />
    </resultMap>

</mapper>
```

- `resultType` 정리
- `resultMap` 정리

#### # `ArticleDAO`테스트
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring-config/applicationContext.xml"})
public class ArticleDAOTest {

    private static final Logger logger = LoggerFactory.getLogger(ArticleDAOTest.class);

    @Inject
    private ArticleDAO articleDAO;

    @Test
    public void testCreate() throws Exception {
        ArticleVO article = new ArticleVO();
        article.setTitle("새로운 글 작성 테스트 제목");
        article.setContent("새로운 글 작성 테스트 내용");
        article.setWriter("새로운 글 작성자");
        articleDAO.create(article);
    }

    @Test
    public void testRead() throws Exception {
        logger.info(articleDAO.read(1).toString());
    }

    @Test
    public void testUpdate() throws Exception {
        ArticleVO article = new ArticleVO();
        article.setArticleNo(1);
        article.setTitle("글 수정 테스트 제목");
        article.setContent("글 수정 테스트 내용");
        articleDAO.update(article);
    }

    @Test
    public void testDelete() throws Exception {
        articleDAO.delete(1);
    }
}
```
#### # `Service`(비지니스 계층)작성

## 2. 컨트롤러(`Controller`), 프레젠테이션(`View`) 계층 구현하기

#### # 등록구현

#### # 목록구현

#### # 조회구현

#### # 삭제구현

#### # 삭제/수정구현

## 2. 예외처리
