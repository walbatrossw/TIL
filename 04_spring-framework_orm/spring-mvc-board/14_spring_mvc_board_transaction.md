
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
|11|[AJAX 댓글처리 : Persistence, Business, Control 계층](http://doublesprogramming.tistory.com/205)|
|12|[AJAX 댓글처리 : Presentation 계층](http://doublesprogramming.tistory.com/206)|
|13|[AOP를 이용한 LogAdvice 작성](http://doublesprogramming.tistory.com/207)|

---

# Spring MVC 게시판 예제14  - 게시글 조회수, 댓글 갯수의 출력 구현과 트랜잭션 처리

## 1. 트랜잭션(Transaction)

#### # 트랜잭션이란?
관련 포스팅 참조 (http://doublesprogramming.tistory.com/124)

#### # 스프링의 트랜잭션 처리
스프링에서는 트랜잭션을 처리하는 방식은 기본적으로 XML을 사용해 선언하는 방식과 애너테이션을 활용하는 방식으로 나누어진다.

- **XML을 사용하는 경우** : 별도의 `treansaction-context.xml`파일을 이용해 XML로 작성해서 처리한다.
- **애너테이션을 활용하는 경우** : 주로 비지니스 계층의 Service인터페이스를 구현한 클래스에 애너테이션을 붙여 처리한다.

게시판 예제에서는 애너테이션을 활용하여 다음과 같은 로직을 트랜잭션 처리를 한다.

- 게시글에 댓글이 추가/삭제되면, 댓글 갯수를 갱신(1씩 증가/감소)한다.
- 게시글을 조회하면 게시글 조회수를 1씩 증가시킨다.

## 2. 트랙잭션 처리를 위한 준비

#### # 트랜잭션 라이브러리 추가 : `pom.xml`

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
```

#### # 트랜잭션 매너저 설정 : `applicationContext.xml`
트랜잭션 매니저를 설정은 보통 DataSource의 설정이 존재하는 곳에 함께 적용하는 것이 편리하다. `<tx:annotation-driven/>`은 `@Treansactional`애너테이션을 이용한 트랜잭션 관리가 가능하게 하는 설정이다.

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<tx:annotation-driven/>
```

## 3. 게시글의 댓글에 따른 트랜잭션 처리
게시글 테이블에 댓글의 갯수를 의미하는 칼럼을 추가하고, 댓글 등록/삭제시 댓글의 갯수를 갱신하는 작업을 처리하도록 한다.

#### # 댓글 갯수 칼럼 추가
게시글 테이블에 댓글 갯수 칼럼을 추가하고, 게시글 등록일자와 조회수 칼럼명도 수정해준다.(이 작업은 기존의 칼럼명이 맘에 들지 않아서 변경하는 것일 뿐 트랜잭션작업과 무관하다.)

```sql
-- 댓글 갯수 칼럼 추가
ALTER TABLE tbl_article ADD COLUMN reply_cnt int DEFAULT 0;

-- 등록일자 칼럼명 변경 : 트랜잭션 작업과 무관
ALTER TABLE tbl_article CHANGE COLUMN regdate reg_date int DEFAULT 0;

-- 조회수 칼럼명 변경 : 트랜잭션 작업과 무관
ALTER TABLE tbl_article CHANGE COLUMN viewcnt view_cnt int DEFAULT 0;
```

#### # 게시글 도메인 클래스 수정
게시글 도메인 클래스인 `ArticleVO`에 멤버변수로 `replyCnt`를 추가해준다.

```java
public class ArticleVO {

    private Integer articleNo;
    private String title;
    private String content;
    private String writer;
    private Date regDate;
    private int viewCnt;
    private int replyCnt; // 댓글 갯수 추가

    // getter, setter, toString() 생략...
}
```

#### # SQL Mapper 수정 : `articleMapper.xml`
댓글 갯수 칼럼이 필요한 SQL문과 `resultMap`에 댓글 갯수 칼럼을 추가시켜주고, 변경한 칼럼명은 수정해준다.

```sql
<select id="listSearch" resultMap="ArticleResultMap">
    <![CDATA[
    SELECT
        article_no,
        title,
        content,
        writer,
        reg_date,
        view_cnt,
        reply_cnt
    FROM tbl_article
    WHERE article_no > 0
    ]]>
      <include refid="search"/>
    <![CDATA[
    ORDER BY article_no DESC, reg_date DESC
    LIMIT #{pageStart}, #{perPageNum}
    ]]>
</select>
```

```xml
<resultMap id="ArticleResultMap" type="ArticleVO">
    <id property="articleNo" column="article_no"/>
    <result property="title" column="title" />
    <result property="content" column="content" />
    <result property="writer" column="writer" />
    <result property="regDate" column="reg_date" />
    <result property="viewCnt" column="view_cnt" />
    <result property="replyCnt" column="reply_cnt" />
</resultMap>
```

#### # 게시글, 댓글 영속 계층 수정
**`ArticleDAO`/`ArticleDAOImpl` 댓글 갯수 갱신 메서드 선언 및 구현**
```java
// ArticleDAO
void updateReplyCnt(Integer articleNo, int amount) throws Exception;
```
```java
// ArticleDAOImpl
@Override
public void updateReplyCnt(Integer articleNo, int amount) throws Exception {

    Map<String, Object> paramMap = new HashMap<>();
    paramMap.put("articleNo", articleNo);
    paramMap.put("amount", amount);

    sqlSession.update(NAMESPACE + ".updateReplyCnt",paramMap);
}
```

**`articleMapper.xml` 댓글 갯수 갱신 SQL 작성**

```sql
<update id="updateReplyCnt">
    UPDATE tbl_article
    SET reply_cnt = reply_cnt + #{amount}
    WHERE article_no = #{articleNo}
</update>
```


**`ReplyDAO`/`ReplyDAOImpl` 댓글의 게시글 번호 조회 메서드 선언 및 구현**
`ReplyDAO`에는 댓글이 삭제될 때 해당 게시물의 번호를 조회하는 기능을 추가한다. 이 기능을 통해 조회한 게시글의 번호를 가지고 댓글 갯수를 갱신하는 작업을 수행하게 된다.
```java
// ReplyDAO
int getArticleNo(Integer replyNo) throws Exception;
```

```java
// ReplyDAOImpl
@Override
public int getArticleNo(Integer replyNo) throws Exception {
    return sqlSession.selectOne(NAMESPACE + ".getArticleNo", replyNo);
}
```

**`replyMapper.xml` 댓글의 게시글 번호 조회 SQL 작성**

```sql
<select id="getArticleNo" resultType="int">
    SELECT
        article_no
    FROM tbl_reply
    WHERE reply_no = #{replyNo}
</select>
```

#### # 댓글 비지니스 계층의 수정 및 트랜잭션 적용 : `ReplyServiceImpl`

ArticleDAO인터페이스를 생성자를 통해 의존성 주입시켜준다.
```java
private final ReplyDAO replyDAO;

private final ArticleDAO articleDAO;

@Inject
public ReplyServiceImpl(ReplyDAO replyDAO, ArticleDAO articleDAO) {
    this.replyDAO = replyDAO;
    this.articleDAO = articleDAO;
}
```

댓글 등록/삭제 메서드를 아래와 같이 수정해준다. 댓글이 추가되거나 삭제되면, 게시글의 댓글 갯수를 갱신하는 작업을 동시에 수행하기 때문에 `@Transactional`애너테이션을 통해 두가지의 작업을 트랜잭션 처리해준다.

댓글이 추가되면 댓글 갯수를 1씩 증가 시켜준다.
```java
// 댓글 등록
@Transactional // 트랜잭션 처리
@Override
public void addReply(ReplyVO replyVO) throws Exception {
    replyDAO.create(replyVO); // 댓글 등록
    articleDAO.updateReplyCnt(replyVO.getArticleNo(), 1); // 댓글 갯수 증가
}
```
댓글이 삭제되면 댓글 갯수를 1씩 감소 시킨다.
```java
// 댓글 삭제
@Transactional // 트랜잭션 처리
@Override
public void removeReply(Integer replyNo) throws Exception {
    int articleNo = replyDAO.getArticleNo(replyNo); // 댓글의 게시물 번호 조회
    replyDAO.delete(replyNo); // 댓글 삭제
    articleDAO.updateReplyCnt(articleNo, -1); // 댓글 갯수 감소
}
```

#### # 게시글 목록 페이지 수정 : `list.jsp`
게시글 목록 페이지에서 게시글 제목 옆에 아래와 같이 댓글 갯수를 출력하도록 코드를 작성해준다.
```html
<td>
    <a href="${path}/article/paging/search/read${pageMaker.makeSearch(pageMaker.criteria.page)}&articleNo=${article.articleNo}">
            ${article.title}
    </a>
    <span class="badge bg-teal"><i class="fa fa-comment-o"></i> + ${article.replyCnt}</span>
</td>
```

## 3. 게시글의 조회에 따른 트랜잭션 처리
게시글을 조회하면 조회수를 의미하는 칼럼인 `viewCnt`를 1씩 증가시키는 작업을 처리하도록 한다.

#### # 게시글 영속 계층의 조회수 증가 메서드 선언 및 구현

**`ArticleDAO`**
```java
void updateViewCnt(Integer articleNo) throws Exception;
```

**`ArticleDAOImpl`**
```java
@Override
public void updateViewCnt(Integer articleNo) throws Exception {
    sqlSession.update(NAMESPACE + ".updateViewCnt", articleNo);
}
```

**`articleMapper.xml`**
```sql
<update id="updateViewCnt">
    UPDATE tbl_article
    SET view_cnt = view_cnt + 1
    WHERE article_no = #{articleNo}
</update>
```

#### # 게시글 비지니스 계층의 수정 및 트랜잭션 적용 : `ArticleServiceImpl`
```java
@Transactional(isolation = Isolation.READ_COMMITTED)
@Override
public ArticleVO read(Integer articleNo) throws Exception {
    articleDAO.updateViewCnt(articleNo);
    return articleDAO.read(articleNo);
}
```

## 4. 최종 기능 구현 모습 확인
댓글 추가/삭제에 따른 댓글 갯수 변화와 게시글 조회에 따른 조회수 증가 모습은 아래와 같다.

![check](https://raw.githubusercontent.com/walbatrossw/TIL/master/04_spring-framework_orm/spring-mvc-board/img/14_spring_mvc_board_transaction/check.png)

## 5. 정리
게시글의 조회수 변화와 댓글 갯수의 갱신작업을 트랜잭션을 통해 처리했는데 앞서 포스팅한 내용에 비해 간단한 내용들이어서 따로 정리할 것은 없다. 다만 불만족스러운 부분은 게시글의 조회수 중복증가 방지처리를 하지 않았다는 점이다. 이 부분은 앞으로 로그인처리를 구현하면서 Session을 통해 조회수 중복증가를 막아보도록 할 예정이다.
