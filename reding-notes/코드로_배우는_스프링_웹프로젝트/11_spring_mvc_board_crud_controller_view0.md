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

#### 포스팅과 관련된 내용 포스팅 링크
|순서|포스팅 제목|
|:---:|---|
|1|[영속, 비지니스, 컨트롤러, 프레젠테이션 계층에 대한 정리 : 계층화 아키텍쳐(layered architecture)](http://doublesprogramming.tistory.com/194)|

---

# Spring MVC 게시판 예제 04 - 기본적인 CRUD 구현 : Control, Presentation 계층

## 1. Control, Presentation 계층 구현

#### # `ArticleController`
`/src/main/java/기본패키지/article/controller`패키지에 `ArticleController`를 생성하고, 아래와 같이 코드를 작성한다.
```java
@Controller
@RequestMapping("/article")
public class ArticleController {

    private static final Logger logger = LoggerFactory.getLogger(ArticleController.class);

    private final ArticleService articleService;

    @Inject
    public ArticleController(ArticleService articleService) {
        this.articleService = articleService;
    }

}
```
위 코드에서 주목할 점은 클래스 선언부의 `@RequestMapping("/article")`인데 공통의 경로를 `/article`로 인식하도록 설정한 것이다. 보통 컨트롤러는 하나의 기능을 가진 모듈의 대표적인 경로를 갖도록 하는 것이 좋다. 이후 작성할 메서드에 매핑할 URL의 길이를 줄여주는 효과도 있다.

#### # 등록구현 : URL(`/article/write`), Mapping(GET/POST)

##### 게시글 등록 페이지 이동, 등록 처리 매핑 : `ArticleController`
```java
// GET Mapping : 게시글 작성을 위한 페이지 매핑
@RequestMapping(value = "/write", method = RequestMethod.GET)
public String writeGET() {
    logger.info("write GET...");
    return "/article/write";
}
```
```java
// POST Mapping : 게시글 작성치리 매핑
@RequestMapping(value = "/write", method = RequestMethod.POST)
public String writePOST(ArticleVO articleVO, Model model) throws Exception {
    logger.info("write POST...");
    logger.info(articleVO.toString());
    articleService.create(articleVO);
    model.addAttribute("result", "success");
    return "/article/success";
}
```
게시글 작성페이지로 이동과 작성처리를 위한 URL매핑을 작성하고, `writePOST()`의 파라미터는 화면으로부터 받아온 모든 데이터를 `ArticleVO`로 수집하도록하는 부분과 향후 데이터를 전달항 가능성이 있으므로 `Model`클래스의 객체를 받도록 했다. 게시글 처리가 완료되면 게시글 목록페이지로 redirect되도록 할 예정인데 아직 게시글 목록 페이지가 구현되지 않았기 때문에 임시페이지인 `success.jsp`로 이동하도록 처리했다.

##### 게시글 등록을 위한 페이지 작성 : `write.jsp`
`/WEB-INF/views/article/`경로에 `write.jsp`를 생성하고, `home.jsp`의 내용을 전체를 복사해 붙여 넣는다. 그리고 나서 `include`의 모든 경로를 다음과 같이 변경해준다. `<%@ include file="../include/jsp파일명"%>`
마지막으로 `<section>`태그 안에 아래와 같이 게시글 입력폼 코드를 작성해준다.
```xml
<div class="col-lg-12">
    <form role="form" id="writeForm" method="post" action="${path}/article/write">
        <div class="box box-primary">
            <div class="box-header with-border">
                <h3 class="box-title">게시글 작성</h3>
            </div>
            <div class="box-body">
                <div class="form-group">
                    <label for="title">제목</label>
                    <input class="form-control" id="title" name="title" placeholder="제목을 입력해주세요">
                </div>
                <div class="form-group">
                    <label for="content">내용</label>
                    <textarea class="form-control" id="content" name="content" rows="30"
                              placeholder="내용을 입력해주세요" style="resize: none;"></textarea>
                </div>
                <div class="form-group">
                    <label for="writer">작성자</label>
                    <input class="form-control" id="writer" name="writer">
                </div>
            </div>
            <div class="box-footer">
                <button type="button" class="btn btn-primary"><i class="fa fa-list"></i> 목록</button>
                <div class="pull-right">
                    <button type="reset" class="btn btn-warning"><i class="fa fa-reply"></i> 초기화</button>
                    <button type="submit" class="btn btn-success"><i class="fa fa-save"></i> 저장</button>
                </div>
            </div>
        </div>
    </form>
</div>
```
위와 같이 작성한 `write.jsp`의 모습은 아래와 같다.
![write.jsp]()

##### 게시글 작성완료 후 성공알림을 위한 임시페이지 작성 : `success.jsp`
아직 게시글 목록 페이지를 구현하지 않았기 때문에 게시글 등록이 완료되면 이동할 임시페이지(`success.jsp`)를 작성해준다. 기본적인 내용은 `write.jsp`와 동일하고 `<section>`태그 안에 아래의 코드를 작성해준다.
```xml
<h1>
    게시판
    <small>입력성공 테스트 페이지</small>
</h1>
<ol class="breadcrumb">
    <li><i class="fa fa-edit"></i> article</li>
    <li class="active"><a href="${path}/article/write"> write</a></li>
</ol>
```
게시글을 작성하고 나면 아래와 같은 페이지로 이동하게 되고, DB를 확인하면 정상적으로 값이 저장된 것을 확인할 수 있다.
![success.jsp]()
![db]()

##### 게시글 작성완료 후 목록페이지로 리다이렉트 처리 : `ArticleController`,`list.jsp`
목록페이지로 redirect되도록 반환값을 `return "redirect:/article/list";`로 변경하고, 목록 페이지 매핑을 아래와 같이 작성해준다.
```java
// POST Mapping : 게시글 작성 처리 매핑
@RequestMapping(value = "/write", method = RequestMethod.POST)
public String writePOST(ArticleVO articleVO, RedirectAttributes redirectAttributes) throws Exception {

    logger.info("write POST...");
    logger.info(articleVO.toString());
    articleService.create(articleVO);
    redirectAttributes.addFlashAttribute("msg", "regSuccess");

    //return "/article/success";
    return "redirect:/article/list";
}
```
위 코드에서 주목해야할 점은 `writePOST()`의 매개변수가 `Model`에서 `RedirectAttributes`로 변경되었다는 점이다. `RedirectAttributes`객체는 리다이렉트 시점에 한번만 사용되는 데이터를 전송할 수 있는 `addFlashAttribute()`라는 메서드가 정의 되어있다. 게시글의 작성처리가 완료되고 리다이렉트 되는 시점에 게시글의 성공적인 등록을 알려주기 위해 딱 한번만 쓰여질 데이터를 저장하기 위해 변경된 것이다.
```java
// GET Mapping : 게시글 목록 페이지 이동
@RequestMapping(value = "/list", method = RequestMethod.GET)
public String list(Model model) throws Exception {
    logger.info("list ...");
    return "/article/list";
}
```

`write.jsp`와 같이 동일하게 jsp파일을 생성하고, `<section>`태그안에 아래와 같이 코드를 작성해준다.
```xml
<div class="col-lg-12">
    <div class="box box-primary">
        <div class="box-header with-border">
            <h3 class="box-title">게시글 목록</h3>
        </div>
        <div class="box-body">
            <table class="table table-bordered">
                <tbody>
                <tr>
                    <th style="width: 30px">#</th>
                    <th>제목</th>
                    <th style="width: 100px">작성자</th>
                    <th style="width: 60px">조회</th>
                    <th style="width: 150px">작성시간</th>
                </tr>
                <tr>
                    <td></td>
                    <td></td>
                    <td></td>
                    <td></td>
                    <td></td>
                </tr>
                </tbody>
            </table>
        </div>
        <div class="box-footer">
            <div class="pull-right">
                <button type="button" class="btn btn-success btn-flat" id="writeBtn">
                    <i class="fa fa-pencil"></i> 글쓰기
                </button>
            </div>
        </div>
    </div>
</div>
```
아래의 코드는 성공적으로 게시글의 등록이 되었음을 알려주기 위한 js코드이다. `list.jsp`의 `<script>`태그 안에 작성해준다.
```js
var result = '${msg}';
if (result == 'regSuccess') {
    alert("게시글 등록이 완료되었습니다.");
}
```
아래의 사진과 같이 게시글이 성공적으로 등록되고, 성공알림창이 뜨게된다.
![list.jsp]()

#### # 목록구현 : URL(`/article/list`), Mapping(GET)

##### 목록 컨트롤러 작성 완료하기 : `ArticleController`
`ArticleController`의 `list()`에서 `ArticleServie`연결 작업을 아래와 같이 완료해준다.
```java
@RequestMapping(value = "/list", method = RequestMethod.GET)
public String list(Model model) throws Exception {

    logger.info("list ...");
    model.addAttribute("articles", articleService.listAll());

    return "/article/list";
}
```

##### 목록 페이지 작성 완료하기 : `list.jsp`
목록페이지(`list.jsp`) 작성을 완료하기에 앞서 `JSTL`과 `EL`을 사용하기 위한 설정이 필요하다. `include`디렉토리에 있는 `head.jsp`의 최상단에 아래와 같이 코드를 추가해준다.

```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
<c:set var="path" value="${pageContext.request.contextPath}"/>
```

`JSTL`을 사용하여 `list.jsp`을 아래와 같이 코드 작성을 완료한다.
```xml
<div class="col-lg-12">
    <div class="box box-primary">
        <div class="box-header with-border">
            <h3 class="box-title">게시글 목록</h3>
        </div>
        <div class="box-body">
            <table class="table table-bordered">
                <tbody>
                <tr>
                    <th style="width: 30px">#</th>
                    <th>제목</th>
                    <th style="width: 100px">작성자</th>
                    <th style="width: 150px">작성시간</th>
                    <th style="width: 60px">조회</th>
                </tr>
                <c:forEach items="${articles}" var="article">
                <tr>
                    <td>${article.articleNo}</td>
                    <td><a href="${path}/article/read?articleNo=${article.articleNo}">${article.title}</a></td>
                    <td>${article.writer}</td>
                    <td><fmt:formatDate value="${article.regDate}" pattern="yyyy-MM-dd a HH:mm"/></td>
                    <td><span class="badge bg-red">${article.viewCnt}</span></td>
                </tr>
                </c:forEach>
                </tbody>
            </table>
        </div>
        <div class="box-footer">
            <div class="pull-right">
                <button type="button" class="btn btn-success btn-flat" id="writeBtn">
                    <i class="fa fa-pencil"></i> 글쓰기
                </button>
            </div>
        </div>
    </div>
</div>
```
아래의 사진은 목록페이지를 최종적으로 구현완료한 모습니다.
![list.jsp]()

#### # 조회구현 : URL(`article/read`), Mapping(GET)

##### 조회페이지 매핑 추가 : `ArticleController`
```java
@RequestMapping(value = "/read", method = RequestMethod.GET)
public String read(@RequestParam("articleNo") int articleNo, Model model) throws Exception {

    logger.info("read ...");
    model.addAttribute("article", articleService.read(articleNo));

    return "/article/read";
}
```

##### 조회페이지 작성 : `read.jsp`

```xml
<div class="col-lg-12">
    <div class="box box-primary">
        <div class="box-header with-border">
            <h3 class="box-title">글제목 : ${article.title}</h3>
        </div>
        <div class="box-body" style="height: 700px">
            ${article.content}
        </div>
        <div class="box-footer">
            <div class="user-block">
                <img class="img-circle img-bordered-sm" src="/dist/img/user1-128x128.jpg" alt="user image">
                <span class="username">
                    <a href="#">${article.writer}</a>
                </span>
                <span class="description"><fmt:formatDate pattern="yyyy-MM-dd a HH:mm" value="${article.regDate}"/></span>
            </div>
        </div>
        <div class="box-footer">
            <button type="submit" class="btn btn-primary listBtn"><i class="fa fa-list"></i> 목록</button>
            <div class="pull-right">
                <button type="submit" class="btn btn-warning modBtn"><i class="fa fa-edit"></i> 수정</button>
                <button type="submit" class="btn btn-danger delBtn"><i class="fa fa-trash"></i> 삭제</button>
            </div>
        </div>
    </div>
</div>
```

![read.jsp]()

#### # 수정구현 : URL(`article/modify`), Mapping(GET/POST)

#### # 삭제 : URL(`/article/delete`), Mapping(POST)

## 2. 예외처리
