
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

---

# Spring MVC 게시판 예제 06 - 페이징처리 : Persistence, Business 계층

## 1. 페이징 구현 단계
페이징처리는 사용자에게 필요한 최소한의 데이터를 전송하기 위해 전체 데이터 중에서 일부분만을 보여주는 방식을 의미하는데 페이징 처리를 위해 아래와 같은 3단계를 거쳐 구현하게 된다.
- URI의 문자열을 조절해 원하는 페이지의 데이터가 출력되게 하는 1단계
- 목록 페이지 하단에 페이지 번호를 보여주고, 번호를 클릭하면 해당 페이지로 이동하는 2단계
- 목록 페이지에서 조회나 수정 작업을 한 뒤에 다시 원래의 목록 페이지로 이동할 수 있게 처리하는 3단계

## 2. 페이징 처리의 원칙
페이징 처리는 다음과 같은 원칙이 지켜져야한다.
- 페이징처리는 반드시 GET방식만을 이용한다.
- 페이징처리가 되면 조회 화면에서 반드시 목록으로 이동할 수 있어야 한다. 예를 들면 게시판의 4페이지를 보다가 특정게시물을 조회하고, 목록으로 이동할 경우 다시 4페이지로 이동할 수 있어야 한다.
- 페이징처리는 반드시 필요한 페이지 번호만을 제공해야한다. 만약 페이지당 10개의 게시글을 출력하고, 전체 데이터가 41건의 게시글이 있다면 5페이지까지 화면에 출력되어야 하며, 더 많은 데이터가 존재할 경우 다음, 이전과 같은 버튼이 존재해야한다.

## 3. 페이징 처리를 위한 더미데이터 넣기
페이징 처리가 제대로 되었는지 확인하기 위해서는 많은 양의 데이터가 존재해야한다. 그래서 이전에 `ArticleDAOTest`클래스의 게시글 작성 테스트메서드인 `testCreate()`를 아래와 같이 수정한 뒤 테스트를 진행하면 1000건의 데이터가 DB에 저장된다.
```java
@Test
public void testCreate() throws Exception {

    for (int i = 1; i <= 1000; i++) {
        ArticleVO articleVO = new ArticleVO();
        articleVO.setTitle(i+ "번째 글 제목입니다...");
        articleVO.setContent(i+ "번재 글 내용입니다...");
        articleVO.setWriter("user0"+(i%10));

        articleDAO.create(articleVO);
    }

}
```
`COUNT`쿼리를 통해 전체 데이터의 수를 확인해보면 아래와 같이 1000건의 게시글이 DB저장된 것을 확인해볼수 있다.
```sql
SELECT COUNT(*) FROM tbl_article;
```

쿼리 수행 결과

![count_query_result](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/06_spring_mvc_board_paging_persistence_business_layer/count_query_result.png?raw=true)

## 4. 페이징 처리를 위한 SQL : `LIMIT`
DB에 따라 페이징 처리를 위한 방법은 각기 다른데 MySQL의 경우 `LIMIT`를 ORACLE의 경우는 `ROWNUM`을 이용한다. 본 예제의 경우 MySQL을 사용하기 때문에 `LIMIT`을 이용할 것이다.
```sql
-- MySQL LIMIT
SELECT *
FROM 테이블명
LIMIT 시작데이터, 출력할 데이터갯수
```
`LIMIT`을 사용하는 방법은 간단하다. 기존의 `SELECT`쿼리에 `LIMIT`키워드를 써주고 첫번째는 시작데이터를 두번째는 출력할 데이터의 갯수를 써주면 된다. 만약 게시글을 10건씩 출력하고, 첫번째 페이지를 출력하고 싶다면 아래와 같이 쿼리를 작성해주면 된다.
```SQL
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
LIMIT 0, 10
```
쿼리 수행 결과
![LIMIT](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-02-28%2018-48-31.png?raw=true)

## 5. 페이징 처리를 위한 영속(Persistence) 계층 구현

#### # `ArticleDAO`인터페이스 : 페이징 처리와 관련된 메서드 추가
```java
List<ArticleVO> listPaging(int page) throws Exception;
```

#### # `ArticleDAOImpl`클래스 : 메서드 구현
`ArticleDAO`인터페이스에서 선언한 추상메서드를 `ArticleDAOImpl`클래스에서 아래와 같이 구현해준다.
```java
@Override
public List<ArticleVO> listPaging(int page) throws Exception {

    if (page <= 0) {
        page = 1;
    }

    page = (page - 1) * 10;

    return sqlSession.selectList(NAMESPACE + ".listPaging", page);
}
```
파라미터 `page`의 값이 0보다 작은 음수값이 들어올 수 없게 위와 같이 조건문을 작성해준다.

#### # `articleMapper.xml` : `LIMIT`쿼리 작성
게시글을 페이지당 10개씩 출력하기 위해 아래와 같이 `LIMIT`쿼리를 작성해준다.
```sql
<select id="listPaging" resultMap="ArticleResultMap">
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
    LIMIT #{page}, 10
    ]]>
</select>
```

## 6. 페이징 처리 SQL 테스트

#### # `ArticleDAOTest`
`ArticleDAOTest`클래스에 아래와 같이 페이징 처리 SQL테스트 코드를 작성해준다.
```java
@Test
public void testListPaging() throws Exception {

    int page = 3;

    List<ArticleVO> articles = articleDAO.listPaging(page);

    for (ArticleVO article : articles) {
        logger.info(article.getArticleNo() + ":" + article.getTitle());
    }

}
```

#### # 테스트 결과
테스트 결과 아래와 같이 출력되었다.
```
|-----------|-----------------|-----------------|-------|----------------------|--------|
|article_no |title            |content          |writer |regdate               |viewcnt |
|-----------|-----------------|-----------------|-------|----------------------|--------|
|980        |980번째 글 제목입니다... |980번재 글 내용입니다... |user00 |2018-02-28 18:19:52.0 |0       |
|979        |979번째 글 제목입니다... |979번재 글 내용입니다... |user09 |2018-02-28 18:19:52.0 |0       |
|978        |978번째 글 제목입니다... |978번재 글 내용입니다... |user08 |2018-02-28 18:19:52.0 |0       |
|977        |977번째 글 제목입니다... |977번재 글 내용입니다... |user07 |2018-02-28 18:19:52.0 |0       |
|976        |976번째 글 제목입니다... |976번재 글 내용입니다... |user06 |2018-02-28 18:19:52.0 |0       |
|975        |975번째 글 제목입니다... |975번재 글 내용입니다... |user05 |2018-02-28 18:19:52.0 |0       |
|974        |974번째 글 제목입니다... |974번재 글 내용입니다... |user04 |2018-02-28 18:19:52.0 |0       |
|973        |973번째 글 제목입니다... |973번재 글 내용입니다... |user03 |2018-02-28 18:19:52.0 |0       |
|972        |972번째 글 제목입니다... |972번재 글 내용입니다... |user02 |2018-02-28 18:19:52.0 |0       |
|971        |971번째 글 제목입니다... |971번재 글 내용입니다... |user01 |2018-02-28 18:19:52.0 |0       |
|-----------|-----------------|-----------------|-------|----------------------|--------|

INFO : jdbc.resultset - 1. ResultSet.next() returned false
INFO : jdbc.resultset - 1. ResultSet.close() returned void
INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@70cf32e3
INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@5a59ca5e
INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
INFO : jdbc.audit - 1. PreparedStatement.close() returned
INFO : jdbc.connection - 1. Connection closed
INFO : jdbc.audit - 1. Connection.close() returned
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 980:980번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 979:979번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 978:978번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 977:977번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 976:976번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 975:975번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 974:974번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 973:973번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 972:972번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 971:971번째 글 제목입니다...
```

## 7. 페이징 처리를 도와줄 `Criteria`클래스 작성

#### # 문제점?
지금까지 작성했던 `articleMapper.xml`의 SQL과 `ArticleDAOImpl`코드의 문제점(?)을 한번 살펴보자.

- `articleMapper.xml` : 만약 한 페이지에 보여지는 데이터가 10개가 아니라면 `LIMIT`구문의 마지막에 10이라는 숫자는 변경되어야만 한다.
- `ArticleDAOImpl` : 매번 원하는 페이지를 처리할 때마다 계산을 해야한다.

이러한 문제점를 해결하기 위해서 `listPaging()`의 파라미터를 2개로 받는 방법이 있다. 하지만 매번 2개의 적절한 데이터를 직접 넘겨 줘야하는 불편함이 발생한다. 그래서 페이징 처리를 도와줄 `Criteria`클래스 생성하고, 페이징 처리의 기준이 되는 변수들을 하나의 객체로 처리하면 보다 편리하게 사용할 수 있다. 또한 이후에 추가사항이 발생하더라도 메서드의 파라미터를 늘리지 않고, 객체에 필드를 추가함으로써 보다 관리가 용이하다.

#### # SQL Mappper의 규칙
`Criteria`클래스를 작성하기 전에 `MyBatis`의 SQL Mappper의 공통적인 규칙에 대해 알아보자.
`#{page}`와 같은 파라미터를 사용할 때 SQL Mapper는 내부적으로 `page`속성의 `getter`에 해당하는 `getPage()`를 호출하게 된다. 예를 들어 아래와 같은 SQL이 있다고 가정해보자.
```sql
SELECT *
FROM tbl_article
WHERE article_no > 0
ORDER BY article_no DESC
LIMIT #{pageStart}, #{perPageNum}
```
위의 SQL은 `pageStart`, `perPageNum`라는 인라인 파라미터를 2개 가지고 있는데, SQL을 실행하게 되면 파라미터로 전달된 객체의 `getPageStart()`, `getPerPageNum()`이라는 메서드를 각각 호출하게 된다.

#### # `Criteria`클래스 작성

`src/main/java/기본패키지/commons/paging`패키지를 생성하고, `Criteria`클래스를 아래와 같이 작성한다.
```java
public class Criteria {

    private int page;
    private int perPageNum;

    public Criteria() {
        this.page = 1;
        this.perPageNum = 10;
    }

    public void setPage(int page) {

        if (page <= 0) {
            this.page = 1;
            return;
        }

        this.page = page;
    }

    public int getPage() {
        return page;
    }

    public void setPerPageNum(int perPageNum) {

        if (perPageNum <= 0 || perPageNum > 100) {
            this.perPageNum = 10;
            return;
        }

        this.perPageNum = perPageNum;
    }

    public int getPerPageNum() {
        return this.perPageNum;
    }

    public int getPageStart() {
        return (this.page - 1) * perPageNum;
    }

    // toString() 생략...
}
```
`Criteria`클래스는 페이징 처리의 기준이 되는 되는 변수들를 처리하기 위해 작성되었는데 코드의 내용을 살펴보자.
- `page` : 현재 페이지 번호
- `perPageNum` : 페이지 당 출력되는 게시글의 갯수
- `Criteria()` : 기본생성자, 현재페이지를 1, 페이지 당 출력할 게시글의 갯수를 10으로 기본 세팅
- `set`메서드 : 음수와 같은 잘못된 값이 들어오지 않도록 validation체크를 통해 적절한 값으로 세팅
- `get`메서드 : SQL Mapper가 사용할 `get`메서드를 정의

위 코드에서 가장 주목해서 봐야할 점은 `getPageStart()`인데 SQL Mapper의 `LIMIT`구문에서 현재 페이지의 게시글의 시작위치를 지정할 때 사용한다. 예를 들어 10개씩 출력할 경우, 3페이지는 SQL이 `LIMIT 20, 10`과 같은 형태가 되어야 한다. 아래는 20을 얻기 위한 계산 공식이다.
```
현재 페이지의 시작 게시글 번호 = (현재 페이지번호 - 1) * 페이지 당 출력할 게시글의 갯수
```

## 8. 영속(Persistence)계층 수정 및 테스트
매개변수를 `Criteria`타입의 변수로 가진 게시글 페이징 목록 메서드를 인터페이스에 선언하고, 클래스에서 구현해준다. SQL Mapper도 위와 같이 작성해준다.

#### # `ArticleDAO`인터페이스
아래와 같이 추상 메서드를 추가시킨다.
```java
List<ArticleVO> listCriteria(Criteria criteria) throws Exception;
```

#### # `ArticleDAOImpl`클래스
인터페이스에 선언되 추상메서드를 아래와 같이 구현해준다.
```java
@Override
public List<ArticleVO> listCriteria(Criteria criteria) throws Exception {
    return sqlSession.selectList(NAMESPACE + ".listCriteria", criteria);
}
```

#### # `articleMapper.xml`
아래와 같이 select 쿼리를 작성해준다.
```sql
<select id="listCriteria" resultMap="ArticleResultMap">
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
    LIMIT #{pageStart}, #{perPageNum}
    ]]>
</select>
```

#### # `ArticleDAOTest`
아래와 같이 테스트 코드를 작성하고 테스트를 실행해 원하는 페이지의 출력값이 콘솔창에 나오는지 확인해본다.
```java
@Test
public void testListCriteria() throws Exception {
    Criteria criteria = new Criteria();
    criteria.setPage(3);
    criteria.setPerPageNum(20);

    List<ArticleVO> articles = articleDAO.listCriteria(criteria);

    for (ArticleVO article : articles) {
        logger.info(article.getArticleNo() + " : " + article.getTitle());
    }
}
```
```
|-----------|-----------------|-----------------|-------|----------------------|--------|
|article_no |title            |content          |writer |regdate               |viewcnt |
|-----------|-----------------|-----------------|-------|----------------------|--------|
|960        |960번째 글 제목입니다... |960번재 글 내용입니다... |user00 |2018-02-28 18:19:52.0 |0       |
|959        |959번째 글 제목입니다... |959번재 글 내용입니다... |user09 |2018-02-28 18:19:52.0 |0       |
|958        |958번째 글 제목입니다... |958번재 글 내용입니다... |user08 |2018-02-28 18:19:52.0 |0       |
|957        |957번째 글 제목입니다... |957번재 글 내용입니다... |user07 |2018-02-28 18:19:52.0 |0       |
|956        |956번째 글 제목입니다... |956번재 글 내용입니다... |user06 |2018-02-28 18:19:52.0 |0       |
|955        |955번째 글 제목입니다... |955번재 글 내용입니다... |user05 |2018-02-28 18:19:52.0 |0       |
|954        |954번째 글 제목입니다... |954번재 글 내용입니다... |user04 |2018-02-28 18:19:52.0 |0       |
|953        |953번째 글 제목입니다... |953번재 글 내용입니다... |user03 |2018-02-28 18:19:52.0 |0       |
|952        |952번째 글 제목입니다... |952번재 글 내용입니다... |user02 |2018-02-28 18:19:52.0 |0       |
|951        |951번째 글 제목입니다... |951번재 글 내용입니다... |user01 |2018-02-28 18:19:52.0 |0       |
|950        |950번째 글 제목입니다... |950번재 글 내용입니다... |user00 |2018-02-28 18:19:52.0 |0       |
|949        |949번째 글 제목입니다... |949번재 글 내용입니다... |user09 |2018-02-28 18:19:52.0 |0       |
|948        |948번째 글 제목입니다... |948번재 글 내용입니다... |user08 |2018-02-28 18:19:52.0 |0       |
|947        |947번째 글 제목입니다... |947번재 글 내용입니다... |user07 |2018-02-28 18:19:52.0 |0       |
|946        |946번째 글 제목입니다... |946번재 글 내용입니다... |user06 |2018-02-28 18:19:52.0 |0       |
|945        |945번째 글 제목입니다... |945번재 글 내용입니다... |user05 |2018-02-28 18:19:52.0 |0       |
|944        |944번째 글 제목입니다... |944번재 글 내용입니다... |user04 |2018-02-28 18:19:52.0 |0       |
|943        |943번째 글 제목입니다... |943번재 글 내용입니다... |user03 |2018-02-28 18:19:52.0 |0       |
|942        |942번째 글 제목입니다... |942번재 글 내용입니다... |user02 |2018-02-28 18:19:52.0 |0       |
|941        |941번째 글 제목입니다... |941번재 글 내용입니다... |user01 |2018-02-28 18:19:52.0 |0       |
|-----------|-----------------|-----------------|-------|----------------------|--------|

INFO : jdbc.resultset - 1. ResultSet.next() returned false
INFO : jdbc.resultset - 1. ResultSet.close() returned void
INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@6f53b8a
INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@5c80cf32
INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
INFO : jdbc.audit - 1. PreparedStatement.close() returned
INFO : jdbc.connection - 1. Connection closed
INFO : jdbc.audit - 1. Connection.close() returned
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 960 : 960번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 959 : 959번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 958 : 958번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 957 : 957번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 956 : 956번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 955 : 955번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 954 : 954번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 953 : 953번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 952 : 952번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 951 : 951번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 950 : 950번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 949 : 949번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 948 : 948번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 947 : 947번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 946 : 946번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 945 : 945번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 944 : 944번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 943 : 943번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 942 : 942번째 글 제목입니다...
INFO : com.doubles.mvcboard.article.ArticleDAOTest - 941 : 941번째 글 제목입니다...
INFO : org.springframework.context.support.GenericApplicationContext - Closing org.springframework.context.support.GenericApplicationContext@3c0f93f1: startup date [Thu Mar 01 18:03:18 KST 2018]; root of context hierarchy
```
## 9. 비지니스(Business) 계층 구현
`ArticleService`는 아직까지는 큰 역할이 없다. 단지 `ArticleController`와 `ArticleDAO`를 연결하는 작업만을 담당하고 있다. `ArticleService`인터페이스에 페이징 목록 메서드를 추가하고, `ArticleServiceImpl`클래스에서 메서드를 구현을 완료해준다.

#### # `ArticleService`인터페이스
```java
List<ArticleVO> listCriteria(Criteria criteria) throws Exception;
```

#### # `ArticleServiceImpl`클래스
```java
@Override
public List<ArticleVO> listCriteria(Criteria criteria) throws Exception {
    return articleDAO.listCriteria(criteria);
}
```

## 10. 간단 요약 정리
이번에는 게시글 목록의 페이징 처리의 영속, 비지니스 계층까지 구현해보았다. 지금까지의 내용 중에서 기억해야할 것들을 간단 요약 정리해보자.
- 페이징 처리를 위한 SQL 키워드 : `LIMIT 시작데이터, 출력할 데이터의 갯수`
- SQL Mapper의 규칙 : 객체 파라미터를 사용할 때 내부적으로 `get`메서드를 통해 필요한 값을 가져온다.
- `Criteria`클래스
  - `set`메서드의 내부에 값의 유효성검사 정의
  - 현재 페이지의 시작 데이터를 가져오기 위한 계산식
    ```
    현재 페이지의 시작 데이터 = (현재 페이지 번호 - 1) 페이지 당 출력할 데이터의 갯수
    ```
