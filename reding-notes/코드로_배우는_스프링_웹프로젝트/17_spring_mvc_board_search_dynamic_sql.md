
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

이제 게시글 검색 처리를 구현해보자. 일반적으로 검색기능의 구현은 검색조건과 검색키워드로 이루어진다. 검색 조건의 경우 select박스를 통해 조건을 선택할 수 있도록 하고, 게시글의 제목, 내용, 작성자를 기준으로 아래와 같은 6가지의 경우의 수에 따라 검색결과를 반환하도록 구현할 예정이다.

- 제목
- 내용
- 작성자
- 제목 or 내용
- 내용 or 작성자
- 작성자 or 내용 or 작성자

그리고 페이징 처리를 구현할 때와 마찬가지로 조회, 수정, 삭제 시에도 검색된 목록의 정보를 유지하도록 처리하는 것까지 같이 구현한다.

## 1. 검색처리를 위한 데이터와 클래스
게시글의 검색결과를 반환하기 위해서는 다음과 같은 정보가 필요하다.

- 현재 페이지 번호 : `Criteria`클래스 멤버변수
- 목록 페이지 당 출력할 게시글의 갯수 : `Criteria`클래스 멤버변수
- 검색 조건 : `SearchCriteria`클래스 멤버변수
- 검색 키워드 : `SearchCriteria`클래스 멤버변수

위의 정보 중에서 "현재 페이지 번호"와 "목록 페이지 당 출력할 게시글의 갯수"는 이미 `Criteria`클래스에 선언되있고, "검색 조건"과 "검색 키워드"를 추가해주면 된다. 기존의 `Criteria`클래스에 "검색 조건"과 "검색 키워드"를 작성해줘도 되지만, `Criteria`클래스를 상속받는 `SerachCriteria`클래스를 생성하고, 그 클래스에 멤버변수로 선언한다. 이렇게 상속을 통해 클래스를 작성하고, 사용하면 기존의 코드의 변경을 최소화할 수 있기 때문이다.
```
SearchCriteria(자손클래스) ---> Criteria(부모클래스)
```

#### # `SerachCriteria`클래스
`/src/main/java/기본패키지/commons/paging`패키지를 생성하고, 아래와 같이 `Criteria`클래스를 상속받는 `SearchCriteria`클래스를 만들어준다.
```java
public class SearchCriteria extends Criteria {

    private String searchType;
    private String keyword;

    // getter(), setter(), toString() 생략...
}
```

## 2. 검색관련 컨트롤러 작성
`/src/main/java/기본패키지/article/controller`패키지에 `ArticlePagingSearchController`클래스를 생성하고, 아래와 같이 작성해준다.
#### # `ArticlePagingSearchController` 클래스
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
}
```

#### # 게시글 작성 매핑 메서드
```java
// 게시글 작성 페이지
@RequestMapping(value = "/write", method = RequestMethod.GET)
public String writeGET() throws Exception {

    logger.info("search writeGET() called ...");

    return "article/search/write";
}
```
```java
// 게시글 등록 처리
@RequestMapping(value = "/write", method = RequestMethod.POST)
public String writePOST() throws Exception {

    logger.info("search writePOST() called ...");

    return "redirect:/article/paging/search/list";
}
```

#### # 게시글 목록 매핑 메서드
```java
// 게시글 목록
@RequestMapping(value = "/list", method = RequestMethod.GET)
public String list(@ModelAttribute("searchCriteria") SearchCriteria searchCriteria,
                   Model model) throws Exception {

    logger.info("list ...");

    PageMaker pageMaker = new PageMaker();
    pageMaker.setCriteria(searchCriteria);
    pageMaker.setTotalCount(articleService.countSearchedArticles(searchCriteria));

    model.addAttribute("articles", articleService.listSearch(searchCriteria));
    model.addAttribute("pageMaker", pageMaker);

    return "article/search/list";
}
```

#### # 게시글 조회 매핑 메서드
```java
// 게시글 조회
@RequestMapping(value = "/read", method = RequestMethod.GET)
public String read(@RequestParam("articleNo") int articleNo,
                   @ModelAttribute("SearchCriteria") SearchCriteria searchCriteria,
                   Model model) throws Exception {

    logger.info("search read() called ...");
    model.addAttribute("article", articleService.read(articleNo));

    return "article/paging/read";
}
```

#### # 게시글 수정 매핑 메서드
```java
// 게시글 수정 페이지
@RequestMapping(value = "/modify", method = RequestMethod.GET)
public String modifyGET(@RequestParam("articleNo") int articleNo,
                        @ModelAttribute("SearchCriteria") SearchCriteria searchCriteria,
                        Model model) throws Exception {

    logger.info("search modifyGet() called ...");
    model.addAttribute("article", articleService.read(articleNo));

    return "article/paging/modify";
}
```
```java
// 게시글 수정 처리
@RequestMapping(value = "/modify", method = RequestMethod.POST)
public String modifyPOST(ArticleVO articleVO,
                         SearchCriteria searchCriteria,
                         RedirectAttributes redirectAttributes) throws Exception {

    logger.info("search modifyPOST() called ...");
    articleService.update(articleVO);
    redirectAttributes.addAttribute("page", searchCriteria.getPage());
    redirectAttributes.addAttribute("perPageNum", searchCriteria.getPerPageNum());
    redirectAttributes.addFlashAttribute("msg", "modSuccess");

    return "redirect:/article/paging/list";
}
```
#### # 게시글 삭제 매핑 메서드
```java
```

## 3. 검색을 위한 JSP작성

## 4. MyBatis 동적 SQL

## 5. `ArticleService`와 `SearchArticleController` 수정

## 6. 조회, 수정, 삭제 페이지 처리

## 7. 등록 페이지 처리

## 8. 결과 확인
