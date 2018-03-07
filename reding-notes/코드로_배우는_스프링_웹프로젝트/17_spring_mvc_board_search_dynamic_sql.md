
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

# Spring MVC 게시판 예제 08 - 검색처리, 동적SQL

## 1. 검색에 필요한 데이터와 `SearchCriteria`클래스 작성하기
완전한 검색 기능을 구현하기 위해서는 아래와 같이 4가지의 정보가 필요하다.

- 현재 페이지 번호(`page`)
- 페이지 당 출력할 게시글의 갯수(`perPageNum`)
- 검색 조건(`searchType`)
- 검색 키워드(`keyword`)

위의 정보 중에서 `page`와 `perPageNum`의 경우 페이징처리를 구현하면서 사용한 `Criteria`클래스에 이미 멤버변수로 선언이 되있다. 하지만  `searchType`과 `keyword`는 어디에도 사용한 적이 없기 때문에 새로 추가해줘야한다. `Criteria`클래스의 멤버변수로 추가해 사용해도 되지만 이전의 코드와 차이점을 구분하기 위해 `Criteria`를 상속받은 `SearchCriteria`라는 클래스에 멤버 변수로 작성해준다. 이렇게 하면 기존 코드의 수정을 최소화할 수 있다는 장점도 생기게 된다.
```
SearchCriteria(자손클래스) -------------> Criteria(부모클래스)
```

#### # `SearchCriteria`클래스 생성
`src/main/java/기본패키지/commons/paging`패키지에 아래와 같이 검색조건과 검색 키워드를 멤버변수로 가지는 클래스를 작성한다.
```java
public class SearchCriteria extends Criteria {

    private String searchType;
    private String keyword;

    // Getter, Setter, toString() 생략
}
```

## 2. 컨트롤러(`ArticlePagingSearchController`) 작성
`src/main/java/기본패키지/article/controller`패키지에 컨트롤러 클래스를 작성해준다.
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

    @RequestMapping(value = "/write", method = RequestMethod.GET)
    public String writeGET() throws Exception {

        logger.info("search writeGET() called ...");

        return "article/search/write";
    }

    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public String list(@ModelAttribute("searchCriteria") SearchCriteria searchCriteria,
                       Model model) throws Exception {

        logger.info("search list() called ...");

        PageMaker pageMaker = new PageMaker();
        pageMaker.setCriteria(searchCriteria);
        pageMaker.setTotalCount(articleService.countSearchedArticles(searchCriteria));

        model.addAttribute("articles", articleService.listSearch(searchCriteria));
        model.addAttribute("pageMaker", pageMaker);

        return "article/search/list";
    }

}
```
이전의 컨트롤러와 비교해보면 `list()`의 매개변수의 타입이 `Criteria`에서 `SearchCriteria`로 변경되었다는 것 외에는 차이점은 없다.

## 3. 목록 페이지 : 검색창 만들기, `searchType`와 `keyword`링크 처리

#### # 검색창 만들기

`/WEB-INF/views/article/search`디렉토리를 생성하고, 기존의 `list.jsp`를 복사하여 붙어 넣어준다. 기존의 `jsp`파일을 수정하지 않고, 새로 만들어주는 이유는 기존의 코드와 차이점을 구분하기 위해서이다.

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
검색 조건은 selectbox를 통해 값을 선택하는데 값이 가지는 의미는 아래와 같다.

- 검색조건 없음 : `n`
- 제목 : `t`
- 내용 : `c`
- 작성자 : `w`
- 제목 or 내용 : `tc`
- 내용 or 작성자 : `cw`
- 제목 or 내용 or 작성자 : `tcw`

```
검색한 목록 페이지 ------> 특정 게시글 조회/수정/삭제 처리 ------> 검색한 목록 페이지
```
그리고 검색된 목록에서 사용자가 특정게시글을 조회하거나 수정/삭제한 뒤 다시 목록으로 돌아가기 위해서는 검색한 목록의 정보(검색조건, 검색키워드)를 유지할 필요가 있다. 그래서 jstl의 `<c:out>`을 이용하여 정보를 `list.jsp`에 다시 세팅해준다.

#### # `searchType`과 `keyword`링크 처리
게시글 목록의 페이지번호의 링크와 게시글 조회 페이지 링크의 URI에 `searchType`과 `keyword`을 붙이기 위한 작업을 아래와 같이 해준다.

**`PageMaker` 클래스**
`makeSearch()`메서드를 추가해 URI가 자동으로 생성되게 처리해주고, `encoding()`는 검색키워드를 `UTF-8`로 변환해주도록 작성한다.
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
```xml
<td>
  <a href="${path}/article/paging/search/read${pageMaker.makeSearch(pageMaker.criteria.page)}&articleNo=${article.articleNo}">
    ${article.title}
  </a>
</td>
```

#### # 검색처리를 위한 목록페이지 완성 모습

## 4. MyBatis 동적 SQL

## 5.
