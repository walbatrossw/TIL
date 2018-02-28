
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
![COUNT]()

## 4. 페이징 처리를 위한 SQL : `LIMIT`
DB에 따라 페이징 처리를 위한 방법은 각기 다른데 MySQL의 경우 `LIMIT`를 ORACLE의 경우는 `ROWNUM`을 이용한다. 본 예제의 경우 MySQL을 사용하기 때문에 `LIMIT`을 이용할 것이다.
```sql
-- MySQL LIMIT
SELECT SQL
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
![LIMIT]()
