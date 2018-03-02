
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
---

# Spring MVC 게시판 예제 08 - 페이징처리 개선, 게시글 조회시 목록페이지 정보 유지

## 1. 페이징 처리 개선
![enhancement_before]()
이전 포스팅까지 페이징 처리 구현을 완료했다. 하지만 아쉬운 점은 위의 사진과 같이 GET파라미터을 `page`만 처리했기 때문에 `perPageNum`이나 그 이상의 정보를 전달할 수 없다는 점이다. 그래서 이번 포스팅에서 페이징 처리를 좀더 나은 방식으로 개선할 수 있는 방법들에 대해 정리해보자.
- `<a herf="/article/listPaging?page=2"/>` : 페이지 번호만을 전달
- `<a herf="/article/listPaging?page=2&perPageNum=10"/>` : 페이지 번호와 페이지당 출력할 게시글의 갯수도 함께 전달

#### # `UriComponentsBuilder`를 이용하는 방식
스프링 MVC에서 웹개발에 필요한 유틸리티 클래스들을 제공하고 있는데 그중에 URI를 작성할 때 도움을 주는 클래스는 `UriComponentsBuilder`와 `UriComponents`클래스가 있다. 그렇다면 `UriComponentsBuilder`를 사용하기 위한 간단한 테스트 코드를 작성해보자.
```java
@Test
public void testURI() throws Exception {

    UriComponents uriComponents = UriComponentsBuilder.newInstance()
            .path("/article/read")
            .queryParam("articleNo", 12)
            .queryParam("perPageNum", 20)
            .build();

    logger.info("/article/read?articleNo=12&perPageNum=20");
    logger.info(uriComponents.toString());

}
```
```
INFO : com.doubles.mvcboard.article.ArticleDAOTest - /article/read?articleNo=12&perPageNum=20
INFO : com.doubles.mvcboard.article.ArticleDAOTest - /article/read?articleNo=12&perPageNum=20
```
위와 같이 `UriComponents`클래스를 통해 `path`나 `query`에 해당하는 문자열을 추가해서 원하는 URI를 생성할 수가 있다. 위 테스트를 실행해보면 콘솔창에 동일한 URI가 생성되는 것을 확인할 수 있다.

```java
@Test
public void testURI2() throws Exception {

    UriComponents uriComponents = UriComponentsBuilder.newInstance()
            .path("/{module}/{page}")
            .queryParam("articleNo", 12)
            .queryParam("perPageNum", 20)
            .build()
            .expand("article", "read")
            .encode();

    logger.info("/article/read?articleNo=12&perPageNum=20");
    logger.info(uriComponents.toString());

}
```
```
INFO : com.doubles.mvcboard.article.ArticleDAOTest - /article/read?articleNo=12&perPageNum=20
INFO : com.doubles.mvcboard.article.ArticleDAOTest - /article/read?articleNo=12&perPageNum=20
```
그리고 만약 미리 필요한 경로를 설정해두고 `{module}`과 같은 경로를 `article`로 `{page}`를 `read`로 변경할 수도 있다. 테스트를 실행해보면 이것도 동일한 URI가 생성되는 것도 확인할 수가 있다.

이제 `PageMaker`클래스에 `makeQuery()`메서드를 추가하고, URI가 자동으로 생성하도록 처리해보자.
```java
public String makeQuery(int page) {
    UriComponents uriComponents = UriComponentsBuilder.newInstance()
            .queryParam("page", page)
            .queryParam("perPageNum", criteria.getPerPageNum())
            .build();

    return uriComponents.toUriString();
}
```
`list_paging.jsp`에서 게시글 제목, 페이지 번호, 다음, 이전의 `<a>`태그를 아래의 코드와 같이 수정한다.
```xml
<tr>
    <td>${article.articleNo}</td>
    <%--<td><a href="${path}/article/read?articleNo=${article.articleNo}">${article.title}</a></td>--%>
    <td>
      <a href="${path}/article/read${pageMaker.makeQuery(pageMaker.criteria.page)}&articleNo=${article.articleNo}">
        ${article.title}
      </a>
    </td>
    <td>${article.writer}</td>
    <td><fmt:formatDate value="${article.regDate}" pattern="yyyy-MM-dd a HH:mm"/></td>
    <td><span class="badge bg-red">${article.viewCnt}</span></td>
</tr>
```

```xml
<ul class="pagination">
    <c:if test="${pageMaker.prev}">
        <%--<li><a href="${path}/article/listPaging?page=${pageMaker.startPage - 1}">이전</a></li>--%>
        <li><a href="${path}/article/listPaging${pageMaker.makeQuery(pageMaker.startPage - 1)}">이전</a></li>
    </c:if>
    <c:forEach begin="${pageMaker.startPage}" end="${pageMaker.endPage}" var="idx">
        <li <c:out value="${pageMaker.criteria.page == idx ? 'class=active' : ''}"/>>
            <%-- <a href="${path}/article/listPaging?page=${idx}">${idx}</a> --%>
            <a href="${path}/article/listPaging${pageMaker.makeQuery(idx)}">${idx}</a>
        </li>
    </c:forEach>
    <c:if test="${pageMaker.next && pageMaker.endPage > 0}">
        <%--<li><a href="${path}/article/listPaging?page=${pageMaker.endPage + 1}">다음</a></li>--%>
        <li><a href="${path}/article/listPaging?${pageMaker.makeQuery(pageMaker.endPage + 1)}">다음</a></li>
    </c:if>
</ul>
```
이제 어떻게 바뀌었는지 크롬의 개발자 도구를 통해 수정한 화면의 페이지 링크를 확인해보면 이전과 다르게 `page`와 `perPageNum`이 같이 연동되는 것을 볼 수 있다.
![enhancement_uri]()


#### # `javascript`를 이용하는 방식
자바스크립트를 이용하는 방식은 페이지 번호를 클릭했을 때 이벤트를 제어하는 방식이다. 이벤트를 제어하기 위해서는 페이지 번호에 걸어둔 `<a>`태그의 `herf`속성을 단순히 페이지 번호만을 의미하도록 변경해줘야 한다.
```xml
<ul class="pagination">
    <c:if test="${pageMaker.prev}">
        <%--<li><a href="${path}/article/listPaging?page=${pageMaker.startPage - 1}">이전</a></li>--%>
        <%--<li><a href="${path}/article/listPaging${pageMaker.makeQuery(pageMaker.startPage - 1)}">이전</a></li>--%>
        <li><a href="${pageMaker.startPage - 1}">이전</a></li>
    </c:if>
    <c:forEach begin="${pageMaker.startPage}" end="${pageMaker.endPage}" var="idx">
        <li <c:out value="${pageMaker.criteria.page == idx ? 'class=active' : ''}"/>>
            <%--<a href="${path}/article/listPaging?page=${idx}">${idx}</a>--%>
            <%--<a href="${path}/article/listPaging${pageMaker.makeQuery(idx)}">${idx}</a>--%>
            <a href="${idx}">${idx}</a>
        </li>
    </c:forEach>
    <c:if test="${pageMaker.next && pageMaker.endPage > 0}">
        <%--<li><a href="${path}/article/listPaging?page=${pageMaker.endPage + 1}">다음</a></li>--%>
        <%--<li><a href="${path}/article/listPaging?${pageMaker.makeQuery(pageMaker.endPage + 1)}">다음</a></li>--%>
        <li><a href="${pageMaker.endPage + 1}">다음</a></li>
    </c:if>
</ul>
```
위와 같이 코드를 변경해주고, `<form>`태그를 추가해 페이지 이동시 `page`와 `perPageNum`값을 넘겨주도록 한다.
```xml
<form id="listPageForm">
    <input type="hidden" name="page" value="${pageMaker.criteria.page}">
    <input type="hidden" name="perPageNum" value="${pageMaker.criteria.perPageNum}">
</form>
```
이제 마지막으로 페이지 번호를 클릭하면 이벤트를 처리할 자바스크립트 코드를 작성해준다.
```js
$(".pagination li a").on("click", function (event) {
    event.preventDefault();

    var targetPage = $(this).attr("href");
    var listPageForm = $("#listPageForm");
    listPageForm.find("[name='page']").val(targetPage);
    listPageForm.attr("action", "/article/listPaging").attr("method", "get");
    listPageForm.submit();
});
```
`<a>`태그를 클릭하면 발생하는 이벤트는 `preventDefault()`를 통해 페이지 이동을 막는다. 그리고 이벤트가 발생한 `<a>`태그의 페이지 번호 값을 `<from>`태그의 `page`에 넣어 전송하게 된다.

실제로 구글 개발자도구를 통해 확인해보면 아래의 사진과 같이 `<a>`태그의 링크는 앞서 설명한 것처럼 페이지 번호만을 의미하도록 나온다.
![enhancement_js]()
