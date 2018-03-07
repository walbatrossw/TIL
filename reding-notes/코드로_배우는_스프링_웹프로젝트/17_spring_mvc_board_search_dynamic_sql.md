
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
|3|[기본적인 CRUD 구현 : Persistence, Business 계층](http://doublesprogramming.tistory.com/195)|
|4|[기본적인 CRUD 구현 : Control, Presentation 계층](http://doublesprogramming.tistory.com/196)|
|5|[예외처리](http://doublesprogramming.tistory.com/197)|
|6|[페이징처리 : Persistence, Business 계층](http://doublesprogramming.tistory.com/198)|
|7|[페이징처리 : Control, Presentation 계층](http://doublesprogramming.tistory.com/199)|
|8|[페이징처리 개선, 목록페이지 정보 유지](http://doublesprogramming.tistory.com/200)|
|9|[프로젝트 구조 변경 및 수정사항](http://doublesprogramming.tistory.com/201)|
---

# Spring MVC 게시판 예제 10 - 검색처리, 동적SQL

## 1. 검색에 필요한 데이터와 `SearchCriteria`클래스 작성하기

#### # 검색에 필요한 데이터
페이징처리와 함께 검색 기능을 구현하기 위해서는 아래와 같이 4가지 정보가 필요하다.

- 현재 페이지 번호(`page`)
- 페이지 당 출력할 게시글의 갯수(`perPageNum`)
- 검색 조건(`searchType`)
- 검색 키워드(`keyword`)

위의 정보 중에서 `page`와 `perPageNum`의 경우 앞서 페이징 처리를 구현하면서 사용한 `Criteria`클래스에 이미 멤버변수로 선언해 사용하고 있다. 하지만  `searchType`과 `keyword`는 어디에도 사용한 적이 없기 때문에 새로 추가해야만 사용이 가능하다. 그래서 `Criteria`클래스에 추가적으로 멤버변수로 선언해 사용하면 된다. 그러나 검색 처리 구현 이전의 코드와의 차이점을 구분하기 위해 `Criteria`에 멤버변수를 추가적으로 선언하는 방식 대신 `Criteria`클래스를 상속받은 `SearchCriteria`라는 클래스를 새로 생성해 새로운 멤버 변수로 작성해준다. 이렇게 함으로써 기존의 코드와 차이점을 구분하는 것 뿐만아니라, 기존코드의 수정을 최소화 할 수 있게된다.

```
SearchCriteria(자손클래스) -------------> Criteria(부모클래스)
```

#### # `SearchCriteria`클래스 작성하기
앞서 설명한 것처럼 `SearchCriteria`클래스를 `src/main/java/기본패키지/commons/paging`패키지에 생성하고, 검색조건과 검색키워드를 멤버변수로 선언해준다.
```java
public class SearchCriteria extends Criteria {

    private String searchType;
    private String keyword;

    // Getter, Setter, toString() 생략
}
```

## 2. 컨트롤러(`ArticlePagingSearchController`) 작성하기
이전 포스팅에서 정리한 것처럼 기존의 컨트롤러에 메서드를 추가하거나, 수정하는 방식이 아닌 새로운 컨트롤러를 생성해서 기존 코드와 차이점을 구분할 수 있도록 `src/main/java/기본패키지/article/controller`패키지에 클래스 새로 생성해 아래와 같이 작성해준다.
```java
@Controller
@RequestMapping("/article/paging/search")
public class ArticlePagingSearchController {

    private static final Logger logger = LoggerFactory.getLogger(ArticlePagingSearchController.class);

    private final ArticleService articleService;

    @Inject
    public ArticlePagingSearchController(ArticleService articleService) {
        this.articleService = articleService;
    }

    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public String list(@ModelAttribute("searchCriteria") SearchCriteria searchCriteria,
                       Model model) throws Exception {

        logger.info("search list() called ...");

        PageMaker pageMaker = new PageMaker();
        pageMaker.setCriteria(searchCriteria);
        pageMaker.setTotalCount(articleService.countArticles(searchCriteria));

        model.addAttribute("articles", articleService.listCriteria(searchCriteria));
        model.addAttribute("pageMaker", pageMaker);

        return "article/search/list";
    }

}
```
이전에 작성했던 컨트롤러와 비교해보면 `list()`의 매개변수의 타입이 `Criteria`에서 `SearchCriteria`로 변경되었다는 것 외에는 차이점은 없다.

## 3. 목록 페이지(`list.jsp`) : 검색창 만들기, 페이지이동URI 수정, 검색버튼 동작 처리
`/WEB-INF/views/article/search`디렉토리를 생성하고, 기존의 `list.jsp`를 복사하여 붙어 넣어준다. 이것도 마찬가지로 기존의 `list.jsp`파일을 수정하지 않고, 새로 만들어주는 이유는 기존의 코드와 차이점을 구분하기 위해서이다.

#### # 검색창 만들기
게시글 페이지 번호 영역 바로 아래에 `<div class="box-footer"></div>`을 작성하고, 안에 아래와 같이 코드를 작성한다.
```xml
<div class="box-footer">
    <div class="form-group col-sm-2">
        <select class="form-control" name="searchType" id="searchType">
            <option value="n" <c:out value="${searchCriteria.searchType == null ? 'selected' : ''}"/>>:::::: 선택 ::::::</option>
            <option value="t" <c:out value="${searchCriteria.searchType eq 't' ? 'selected' : ''}"/>>제목</option>
            <option value="c" <c:out value="${searchCriteria.searchType eq 'c' ? 'selected' : ''}"/>>내용</option>
            <option value="w" <c:out value="${searchCriteria.searchType eq 'w' ? 'selected' : ''}"/>>작성자</option>
            <option value="tc" <c:out value="${searchCriteria.searchType eq 'tc' ? 'selected' : ''}"/>>제목+내용</option>
            <option value="cw" <c:out value="${searchCriteria.searchType eq 'cw' ? 'selected' : ''}"/>>내용+작성자</option>
            <option value="tcw" <c:out value="${searchCriteria.searchType eq 'tcw' ? 'selected' : ''}"/>>제목+내용+작성자</option>
        </select>
    </div>
    <div class="form-group col-sm-10">
        <div class="input-group">
            <input type="text" class="form-control" name="keyword" id="keywordInput" value="${searchCriteria.keyword}" placeholder="검색어">
            <span class="input-group-btn">
                <button type="button" class="btn btn-primary btn-flat" id="searchBtn">
                    <i class="fa fa-search"></i> 검색
                </button>
            </span>
        </div>
    </div>
    <div class="pull-right">
        <button type="button" class="btn btn-success btn-flat" id="writeBtn">
            <i class="fa fa-pencil"></i> 글쓰기
        </button>
    </div>
</div>
```
검색 조건은 `selectbox`를 통해 사용자가 값을 선택할 수 있도록 구현하는데 값이 가지는 의미는 아래와 같다.

- 검색조건 없음 : `n`
- 제목 : `t`
- 내용 : `c`
- 작성자 : `w`
- 제목 or 내용 : `tc`
- 내용 or 작성자 : `cw`
- 제목 or 내용 or 작성자 : `tcw`

```
검색한 게시글 목록 페이지 ------> 특정 게시글 조회/수정/삭제 처리 ------> 검색한 게시글 목록 페이지
```
그리고 검색된 목록에서 사용자가 특정게시글을 조회하거나 수정/삭제한 뒤 다시 검색한 게시글 목록으로 돌아가기 위해서는 검색한 목록의 정보(검색조건, 검색키워드)를 유지할 필요가 있다. 그래서 정보를 `list.jsp`에 다시 세팅해주도록 jstl의 `<c:out>`을 이용한다.

#### # 페이지 이동 URI 수정
기존의 게시글 목록 페이지 번호와 게시글 제목에는 아래와 같이 URI링크가 걸려있는데 검색한 게시글의 목록 정보를 유지하기 위해서는 `&searchType=값&keyword=값`과 같은 GET방식으로 URI에 붙이는 작업이 필요하다.

- 게시글 목록 페이지 번호의 링크 : `<a href="/article/paging/search/list?page=2&pagePageNum=10">`
- 게시글 제목의 링크 : `<a href="/article/paging/read?page=1&perPageNum=10&articleNo=1000">`

**`PageMaker` 클래스에 URI 자동생성 메서드 추가하기**
`makeSearch()`메서드에 검색조건과 검색키워드를 처리를 추가로 작성해주고, `encoding()`메서드에서는 검색키워드를 인코딩처리를 위한 로직을 구현해준다.
```java
public String makeSearch(int page) {

    UriComponents uriComponents = UriComponentsBuilder.newInstance()
            .queryParam("page", page)
            .queryParam("pagePageNum", criteria.getPerPageNum())
            .queryParam("searchType", ((SearchCriteria) criteria).getSearchType())
            .queryParam("keyword", encoding(((SearchCriteria) criteria).getKeyword()))
            .build();

    return uriComponents.toUriString();
}

private String encoding(String keyword) {
    if (keyword == null || keyword.trim().length() == 0) {
        return "";
    }

    try {
        return URLEncoder.encode(keyword, "UTF-8");
    } catch (UnsupportedEncodingException e) {
        return "";
    }
}
```
**`list.jsp`의 페이지 번호 링크 수정**
위에서 새로 추가한 메서드를 호출하여 목록 페이지의 번호 링크에 검색조건과 검색키워드가 추가된 URI가 생성되도록 아래와 같이 수정해준다.
```xml
<div class="box-footer">
    <div class="text-center">
        <ul class="pagination">
            <c:if test="${pageMaker.prev}">
                <li><a href="${path}/article/paging/search/list${pageMaker.makeSearch(pageMaker.startPage - 1)}">이전</a></li>
            </c:if>
            <c:forEach begin="${pageMaker.startPage}" end="${pageMaker.endPage}" var="idx">
                <li <c:out value="${pageMaker.criteria.page == idx ? 'class=active' : ''}"/>>
                    <a href="${path}/article/paging/search/list${pageMaker.makeSearch(idx)}">${idx}</a>
                </li>
            </c:forEach>
            <c:if test="${pageMaker.next && pageMaker.endPage > 0}">
                <li><a href="${path}/article/paging/search/list?${pageMaker.makeSearch(pageMaker.endPage + 1)}">다음</a></li>
            </c:if>
        </ul>
    </div>
</div>
```
**`list.jsp`의 게시글 조회 링크 수정**
게시글 제목에 걸린 게시글 조회 링크도 아래와 같이 수정해준다.
```xml
<td>
  <a href="${path}/article/paging/search/read${pageMaker.makeSearch(pageMaker.criteria.page)}&articleNo=${article.articleNo}">
    ${article.title}
  </a>
</td>
```

#### # 검색버튼 동작 처리
검색버튼 클릭 이벤트가 발생하면 기존의 URI에 검색조건과 검색키워드를 GET방식으로 붙여 요청하게 처리한다. 그리고 `encodeURIComponent()`은 URI로 데이터를 전달하기 위해서 문자열을 인코딩해준다.
```js
$(document).ready(function () {

    $("#searchBtn").on("click", function (event) {
        self.location =
            "/article/paging/search/list${pageMaker.makeQuery(1)}"
            + "&searchType=" + $("select option:selected").val()
            + "&keyword=" + encodeURIComponent($("#keywordInput").val());
    });
});
```

#### # 검색처리를 위한 목록페이지 완성 모습
![](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/20180306_190759.png?raw=true)

## 4. 검색 처리를 위한 영속 계층 구현 : MyBatis 동적 SQL

#### # `ArticleDAO`인터페이스
검색된 목록과 검색된 게시글의 갯수를 리턴하는 추상 메서드를 선언해준다.
```java
List<ArticleVO> listSearch(SearchCriteria searchCriteria) throws Exception;

int countSearchedArticles(SearchCriteria searchCriteria) throws Exception;
```

#### # `ArticleDAOImpl`클래스
`ArticleDAO`인터페이스에 선언된 추상메서드를 오버라이드하여 구현해준다.
```java
@Override
public List<ArticleVO> listSearch(SearchCriteria searchCriteria) throws Exception {
    return sqlSession.selectList(NAMESPACE + ".listSearch", searchCriteria);
}

@Override
public int countSearchedArticles(SearchCriteria searchCriteria) throws Exception {
    return sqlSession.selectOne(NAMESPACE + ".countSearchedArticles", searchCriteria);
}
```


#### # MyBatis의 표현식
화면에서 사용자가 선택한 검색조건에 따라 검색요청을 처리하는 SQL문이 달라지기 때문에 이를 처리하기 위해서는 Mybatis SQL Mapper에서 동적 SQL문을 작성해줘야한다.  동적 SQL문을 작성하기 위해 표현식을 간단하게 정리해보자.

- `if` : 코드로 작성할 때의 if구문에 대한 처리
  ```xml
  <if test="title != null">
    AND title like #{like}
  </if>
  ```
- `choose(when, otherwise)` : switch와 같은 상황에 대한 처리
  ```xml
  <choose>
    <when test="title != null">
      AND title like #{like}
    </when>  
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND feathered = 1
    </otherwise>
  </choose>
  ```
- `trim(where, set)` : 로직을 처리하면서 필요한 구문을 변경
  ```xml
  <trim prefix="WHERE" prefixOverrides="AND|OR">
    ...
  </trim>
  ```
- `foreach` : 컬렉션에 대한 순환처리
  ```xml
  <foreach item="item" index="index" collection="list" open="(" seperator="," close="">
    #{item}
  </foreach>
  ```


#### # `articleMapper.xml`에서 동적 SQL문 작성
`<if>`문을 통해 동적 SQL문을 통해 상황에 맞게 검색을 처리하도록 코드를 작성해준다. 그리고 SQL문이 중복되어 사용될 경우 `<include>`를 통해 SQL 중복을 제거하고, SQL의 재사용을 가능하게 처리해준다.
```sql
<select id="listSearch" resultMap="ArticleResultMap">
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
    ]]>
      <include refid="search"/>
    <![CDATA[
    ORDER BY article_no DESC, regdate DESC
    LIMIT #{pageStart}, #{perPageNum}
    ]]>
</select>

<select id="countSearchedArticles" resultType="int">
    <![CDATA[
    SELECT
        COUNT(article_no)
    FROM tbl_article
    WHERE article_no > 0
    ]]>
    <include refid="search"/>
</select>

<sql id="search">
    <if test="searchType != null">
        <if test="searchType == 't'.toString()">
            AND title LIKE CONCAT('%', #{keyword}, '%')
        </if>
        <if test="searchType == 'c'.toString()">
            AND content LIKE CONCAT('%', #{keyword}, '%')
        </if>
        <if test="searchType == 'w'.toString()">
            AND writer LIKE CONCAT('%', #{keyword}, '%')
        </if>
        <if test="searchType == 'tc'.toString()">
            AND (
                title LIKE CONCAT('%', #{keyword}, '%')
                OR content LIKE CONCAT('%', #{keyword}, '%')
            )
        </if>
        <if test="searchType == 'cw'.toString()">
            AND (
                content LIKE CONCAT('%', #{keyword}, '%')
                OR writer LIKE CONCAT('%', #{keyword}, '%')
            )
        </if>
        <if test="searchType == 'tcw'.toString()">
            AND (
                title LIKE CONCAT('%', #{keyword}, '%')
                OR content LIKE CONCAT('%', #{keyword}, '%')
                OR writer LIKE CONCAT('%', #{keyword}, '%')
            )
        </if>
    </if>
</sql>
```


#### # 동적 SQL 테스트

**테스트 코드**
```java
@Test
public void testDynamic1() throws Exception {

    SearchCriteria searchCriteria = new SearchCriteria();
    searchCriteria.setPage(1);
    searchCriteria.setKeyword("999");
    searchCriteria.setSearchType("t");

    logger.info("======================");

    List<ArticleVO> articles = articleDAO.listSearch(searchCriteria);

    for (ArticleVO article : articles) {
        logger.info(article.getArticleNo() + " : " + article.getTitle());
    }

    logger.info("======================");

    logger.info("searched articles count : " + articleDAO.countSearchedArticles(searchCriteria));
}
```
**테스트 결과**
```
INFO : com.doubles.mvcboard.article.ArticleDAOTest - ======================
INFO : jdbc.connection - 1. Connection opened
INFO : jdbc.audit - 1. Connection.new Connection returned
INFO : jdbc.audit - 1. Connection.getAutoCommit() returned true
INFO : jdbc.audit - 1. PreparedStatement.new PreparedStatement returned
INFO : jdbc.audit - 1. Connection.prepareStatement(SELECT
            article_no,
            title,
            content,
            writer,
            regdate,
            viewcnt
        FROM tbl_article
        WHERE article_no > 0
          AND title LIKE CONCAT('%', ?, '%')
        ORDER BY article_no DESC, regdate DESC
        LIMIT ?, ?) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@2657d4dd
INFO : jdbc.audit - 1. PreparedStatement.setString(1, "999") returned
INFO : jdbc.audit - 1. PreparedStatement.setInt(2, 0) returned
INFO : jdbc.audit - 1. PreparedStatement.setInt(3, 10) returned
INFO : jdbc.sqlonly - SELECT article_no, title, content, writer, regdate, viewcnt FROM tbl_article WHERE article_no
> 0 AND title LIKE CONCAT('%', '999', '%') ORDER BY article_no DESC, regdate DESC LIMIT 0,
10

INFO : jdbc.sqltiming - SELECT article_no, title, content, writer, regdate, viewcnt FROM tbl_article WHERE article_no
> 0 AND title LIKE CONCAT('%', '999', '%') ORDER BY article_no DESC, regdate DESC LIMIT 0,
10
 {executed in 10 msec}
INFO : jdbc.audit - 1. PreparedStatement.execute() returned true
INFO : jdbc.resultset - 1. ResultSet.new ResultSet returned
INFO : jdbc.audit - 1. PreparedStatement.getResultSet() returned net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy@475c9c31
INFO : jdbc.resultset - 1. ResultSet.getMetaData() returned com.mysql.jdbc.ResultSetMetaData@5276d6ee - Field level information:
	com.mysql.jdbc.Field@71687585[catalog=spring_ex,tableName=tbl_article,originalTableName=tbl_article,columnName=article_no,originalColumnName=article_no,mysqlType=3(FIELD_TYPE_LONG),flags= AUTO_INCREMENT PRIMARY_KEY, charsetIndex=63, charsetName=US-ASCII]
	com.mysql.jdbc.Field@1807f5a7[catalog=spring_ex,tableName=tbl_article,originalTableName=tbl_article,columnName=title,originalColumnName=title,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@1b919693[catalog=spring_ex,tableName=tbl_article,originalTableName=tbl_article,columnName=content,originalColumnName=content,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY BLOB, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@7fb4f2a9[catalog=spring_ex,tableName=tbl_article,originalTableName=tbl_article,columnName=writer,originalColumnName=writer,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@4dc27487[catalog=spring_ex,tableName=tbl_article,originalTableName=tbl_article,columnName=regdate,originalColumnName=regdate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
	com.mysql.jdbc.Field@6a4f1a55[catalog=spring_ex,tableName=tbl_article,originalTableName=tbl_article,columnName=viewcnt,originalColumnName=viewcnt,mysqlType=3(FIELD_TYPE_LONG),flags=, charsetIndex=63, charsetName=US-ASCII]
INFO : jdbc.resultset - 1. ResultSet.getType() returned 1003
INFO : jdbc.resultset - 1. ResultSet.next() returned true
INFO : jdbc.resultset - 1. ResultSet.getInt(article_no) returned 999
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultset - 1. ResultSet.getString(title) returned 999번째 글 제목입니다...
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultset - 1. ResultSet.getString(content) returned 999번재 글 내용입니다...
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultset - 1. ResultSet.getString(writer) returned user09
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultset - 1. ResultSet.getTimestamp(regdate) returned 2018-03-06 19:01:55.0
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultset - 1. ResultSet.getInt(viewcnt) returned 0
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultsettable -
|-----------|-----------------|-----------------|-------|----------------------|--------|
|article_no |title            |content          |writer |regdate               |viewcnt |
|-----------|-----------------|-----------------|-------|----------------------|--------|
|999        |999번째 글 제목입니다... |999번재 글 내용입니다... |user09 |2018-03-06 19:01:55.0 |0       |
|-----------|-----------------|-----------------|-------|----------------------|--------|

INFO : jdbc.resultset - 1. ResultSet.next() returned false
INFO : jdbc.resultset - 1. ResultSet.close() returned void
INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@3aa078fd
INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@d23e042
INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
INFO : jdbc.audit - 1. PreparedStatement.close() returned
INFO : jdbc.connection - 1. Connection closed
INFO : jdbc.audit - 1. Connection.close() returned
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 999 : 999번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - ======================
INFO : jdbc.connection - 2. Connection opened
INFO : jdbc.audit - 2. Connection.new Connection returned
INFO : jdbc.audit - 2. Connection.getAutoCommit() returned true
INFO : jdbc.audit - 2. PreparedStatement.new PreparedStatement returned
INFO : jdbc.audit - 2. Connection.prepareStatement(SELECT
            COUNT(article_no)
        FROM tbl_article
        WHERE article_no > 0
        AND title LIKE CONCAT('%', ?, '%')) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@2254127a
INFO : jdbc.audit - 2. PreparedStatement.setString(1, "999") returned
INFO : jdbc.sqlonly - SELECT COUNT(article_no) FROM tbl_article WHERE article_no > 0 AND title LIKE CONCAT('%', '999',
'%')

INFO : jdbc.sqltiming - SELECT COUNT(article_no) FROM tbl_article WHERE article_no > 0 AND title LIKE CONCAT('%', '999',
'%')
 {executed in 3 msec}
INFO : jdbc.audit - 2. PreparedStatement.execute() returned true
INFO : jdbc.resultset - 2. ResultSet.new ResultSet returned
INFO : jdbc.audit - 2. PreparedStatement.getResultSet() returned net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy@51891008
INFO : jdbc.resultset - 2. ResultSet.getMetaData() returned com.mysql.jdbc.ResultSetMetaData@2f953efd - Field level information:
	com.mysql.jdbc.Field@f68f0dc[catalog=,tableName=,originalTableName=,columnName=COUNT(article_no),originalColumnName=,mysqlType=8(FIELD_TYPE_LONGLONG),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
INFO : jdbc.resultset - 2. ResultSet.getType() returned 1003
INFO : jdbc.resultset - 2. ResultSet.next() returned true
INFO : jdbc.resultset - 2. ResultSet.getInt(COUNT(article_no)) returned 1
INFO : jdbc.resultset - 2. ResultSet.wasNull() returned false
INFO : jdbc.resultsettable -
|------------------|
|count(article_no) |
|------------------|
|1                 |
|------------------|

INFO : jdbc.resultset - 2. ResultSet.next() returned false
INFO : jdbc.resultset - 2. ResultSet.close() returned void
INFO : jdbc.audit - 2. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@24c1b2d2
INFO : jdbc.audit - 2. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@7dc19a70
INFO : jdbc.audit - 2. PreparedStatement.getMoreResults() returned false
INFO : jdbc.audit - 2. PreparedStatement.getUpdateCount() returned -1
INFO : jdbc.audit - 2. PreparedStatement.close() returned
INFO : jdbc.connection - 2. Connection closed
INFO : jdbc.audit - 2. Connection.close() returned
INFO : com.doubles.mvcboard.article.ArticleDAOTest - searched articles count : 1
```
위에서 작성한 동적 SQL문이 제대로 작동하는지 확인하기 위해 테스트 코드를 작성하고, 확인해보자.


## 5. 비지니스 계층 구현
검색처리를 위한 컨트롤러와 영속계층을 연결해주는 비지니스 로직은 아래와 같이 작성해준다.
#### # `ArticleService`인터페이스
```java
List<ArticleVO> listSearch(SearchCriteria searchCriteria) throws Exception;

int countSearchedArticles(SearchCriteria searchCriteria) throws Exception;
```

#### # `ArticleServiceImpl`클래스
```java
@Override
public List<ArticleVO> listSearch(SearchCriteria searchCriteria) throws Exception {
    return articleDAO.listSearch(searchCriteria);
}

@Override
public int countSearchedArticles(SearchCriteria searchCriteria) throws Exception {
    return articleDAO.countSearchedArticles(searchCriteria);
}
```

## 6. 컨트롤러(`ArticlePagingSearchController`) 수정

#### # 목록 페이지 매핑 메서드 수정
동적 SQL을 통해 검색처리된 게시글의 목록이 출력되게 아래와 같이 코드를 수정해준다.
```java
@RequestMapping(value = "/list", method = RequestMethod.GET)
public String list(@ModelAttribute("searchCriteria") SearchCriteria searchCriteria,
                   Model model) throws Exception {

    logger.info("search list() called ...");

    PageMaker pageMaker = new PageMaker();
    pageMaker.setCriteria(searchCriteria);
    //pageMaker.setTotalCount(articleService.countArticles(searchCriteria));
    pageMaker.setTotalCount(articleService.countSearchedArticles(searchCriteria));

    //model.addAttribute("articles", articleService.listCriteria(searchCriteria));
    model.addAttribute("articles", articleService.listSearch(searchCriteria));
    model.addAttribute("pageMaker", pageMaker);

    return "article/search/list";
}
```

#### # 조회 페이지 매핑 메서드 수정
게시글의 검색정보가 유지되도록 `read()`메서드의 매개변수 타입을 `SearchCriteria`로 변경한다.
```java
// 조화 페이지
@RequestMapping(value = "/read", method = RequestMethod.GET)
public String read(@RequestParam("articleNo") int articleNo,
                   @ModelAttribute("searchCriteria") SearchCriteria searchCriteria,
                   Model model) throws Exception {

    logger.info("search read() called ...");
    model.addAttribute("article", articleService.read(articleNo));

    return "article/search/read";
}
```
#### # 수정 페이지 매핑, 수정처리 매핑 메서드 수정
게시글의 검색정보가 유지되도록 `modifyGET()`메서드의 매개변수 타입을 `SearchCriteria`로 변경한다.
```java
// 수정 페이지
@RequestMapping(value = "/modify", method = RequestMethod.GET)
public String modifyGET(@RequestParam("articleNo") int articleNo,
                        @ModelAttribute("searchCriteria") SearchCriteria searchCriteria,
                        Model model) throws Exception {

    logger.info("search modifyGet() called ...");
    logger.info(searchCriteria.toString());
    model.addAttribute("article", articleService.read(articleNo));

    return "article/search/modify";
}
```

수정처리가 완료되고, 목록페이지로 리다이렉트될 때 검색정보가 유지될 수 있도록 아래와 같이 검색조건과 검색키워드를 `redirectAttributes.addAttribute()`에 저장해준다.
```java
// 수정 처리
@RequestMapping(value = "/modify", method = RequestMethod.POST)
public String modifyPOST(ArticleVO articleVO,
                         SearchCriteria searchCriteria,
                         RedirectAttributes redirectAttributes) throws Exception {

    logger.info("search modifyPOST() called ...");
    articleService.update(articleVO);
    redirectAttributes.addAttribute("page", searchCriteria.getPage());
    redirectAttributes.addAttribute("perPageNum", searchCriteria.getPerPageNum());
    redirectAttributes.addAttribute("searchType", searchCriteria.getSearchType());
    redirectAttributes.addAttribute("keyword", searchCriteria.getKeyword());
    redirectAttributes.addFlashAttribute("msg", "modSuccess");

    return "redirect:/article/paging/search/list";
}
```
#### # 삭제 처리 매핑 메서드 수정
수정처리와 마찬가지로 삭제처리가 완료되고, 목록페이지로 리다이렉트될 때 검색정보가 유지될 수 있도록 아래와 같이 검색조건과 검색키워드를 `redirectAttributes.addAttribute()`에 저장해준다.
```java
// 삭제 처리
@RequestMapping(value = "/remove", method = RequestMethod.POST)
public String remove(@RequestParam("articleNo") int articleNo,
                     SearchCriteria searchCriteria,
                     RedirectAttributes redirectAttributes) throws Exception {

    logger.info("search remove() called ...");
    articleService.delete(articleNo);
    redirectAttributes.addAttribute("page", searchCriteria.getPage());
    redirectAttributes.addAttribute("perPageNum", searchCriteria.getPerPageNum());
    redirectAttributes.addAttribute("searchType", searchCriteria.getSearchType());
    redirectAttributes.addAttribute("keyword", searchCriteria.getKeyword());
    redirectAttributes.addFlashAttribute("msg", "delSuccess");

    return "redirect:/article/paging/search/list";
}
```
## 7. 조회/삭제, 수정 페이지 수정

#### # 조회 페이지, 삭제처리(`read.jsp`)

**HTML코드**
`<input>`타입의 `hidden`속성으로 검색조건과 검색키워드를 추가해준다.
```xml
<div class="box-footer">
    <form role="form" method="post">
        <input type="hidden" name="articleNo" value="${article.articleNo}">
        <input type="hidden" name="page" value="${searchCriteria.page}">
        <input type="hidden" name="perPageNum" value="${searchCriteria.perPageNum}">
        <input type="hidden" name="searchType" value="${searchCriteria.searchType}">
        <input type="hidden" name="keyword" value="${searchCriteria.keyword}">
    </form>
    <button type="submit" class="btn btn-primary listBtn"><i class="fa fa-list"></i> 목록</button>
    <div class="pull-right">
        <button type="submit" class="btn btn-warning modBtn"><i class="fa fa-edit"></i> 수정</button>
        <button type="submit" class="btn btn-danger delBtn"><i class="fa fa-trash"></i> 삭제</button>
    </div>
</div>
```

**JS코드**
각각 버튼의 클릭이벤트가 발생하면 해당 페이지로 이동하기 위해 `attr()`로 속성을 제어해준다.
```js
$(document).ready(function () {

    var formObj = $("form[role='form']");
    console.log(formObj);

    $(".modBtn").on("click", function () {
        formObj.attr("action", "/article/paging/search/modify");
        formObj.attr("method", "get");
        formObj.submit();
    });

    $(".delBtn").on("click", function () {
        formObj.attr("action", "/article/paging/search/remove");
        formObj.submit();
    });

    $(".listBtn").on("click", function () {
        formObj.attr("action", "/article/paging/search/list");
        formObj.attr("method", "get");
        formObj.submit();
    });

});
```

#### # 수정 페이지(`modify.jsp`)

**HTML코드**
조회페이지와 마찬가지로 수정페이지에서도 `<input>`타입의 `hidden`속성으로 검색조건과 검색키워드를 추가해준다.
```xml
<div class="box-body">
    <input type="hidden" name="articleNo" value="${article.articleNo}">
    <input type="hidden" name="page" value="${searchCriteria.page}">
    <input type="hidden" name="perPageNum" value="${searchCriteria.perPageNum}">
    <input type="hidden" name="searchType" value="${searchCriteria.searchType}">
    <input type="hidden" name="keyword" value="${searchCriteria.keyword}">
    <div class="form-group">
        <label for="title">제목</label>
        <input class="form-control" id="title" name="title" placeholder="제목을 입력해주세요" value="${article.title}">
    </div>
    <div class="form-group">
        <label for="content">내용</label>
        <textarea class="form-control" id="content" name="content" rows="30"
                  placeholder="내용을 입력해주세요" style="resize: none;">${article.content}</textarea>
    </div>
    <div class="form-group">
        <label for="writer">작성자</label>
        <input class="form-control" id="writer" name="writer" value="${article.writer}" readonly>
    </div>
</div>

```

**JS코드**
목록 버튼 클릭이벤트가 발생하면 검색한 목록페이지로 이동하기 위해 유지된 정보값들을 아래와 같이 GET방식으로 URI에 작성해준다.
```js
$(document).ready(function () {

    var formObj = $("form[role='form']");
    console.log(formObj);

    $(".modBtn").on("click", function () {
        formObj.submit();
    });

    $(".cancelBtn").on("click", function () {
        history.go(-1);
    });

    $(".listBtn").on("click", function () {
        self.location = "/article/paging/search/list?page=${searchCriteria.page}"
            + "&perPageNum=${searchCriteria.perPageNum}"
            + "&searchType=${searchCriteria.searchType}"
            + "&keyword=${searchCriteria.keyword}";
    });

});
```

## 8. 최종적으로 구현된 검색처리 모습

#### # 99로 게시글 검색 이후 목록페이지
![list.jsp2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/20180307_154125.png?raw=true)

#### # 게시글 검색 이후 조회 페이지
![read.jsp](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/20180307_154202.png?raw=true)

#### # 게시글 검색 이후 수정 페이지
![modify.jsp](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/20180307_154221.png?raw=true)

#### # 수정 처리후 목록페이지로 리다이렉트
![list.jsp3](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/20180307_154251.png?raw=true)

#### # 삭제 처리후 목록페이지로 리다이렉트
![list.jsp4](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/20180307_154320.png?raw=true)
