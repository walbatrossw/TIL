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

## 1. 기본 게시판 CRUD URI
게시글을 등록, 수정, 삭제, 조회, 목록 기능의 URI을 정리해보면 아래와 같다.

|방식|URI|설명|
|---|---|---|
|GET|/article/write|게시글의 등록페이지|
|POST|/article/write|게시글의 등록처리|
|GET|/article/list|게시글의 목록페이지|
|GET|/article/read?articleNo=게시글번호|게시글의 조회|
|GET|/article/modify?articleNo=게시글번호|게시글의 수정페이지|
|POST|/article/modify|게시글의 수정처리|
|POST|/article/remove|게시글의 삭제처리|

## 2. JSP파일 수정사항 및 생성
Control, Presentation계층을 구현하기에 앞서 미리 설정해야할 사항들을 정리해보자. 앞서 포스팅을 하면서 누락했던 사항들도 함께 정리했다.

#### # `Include`페이지 수정 : `left_column.jsp`, `head.jsp`
`JSTL`과 `EL`을 사용하기 위해 `head.jsp`의 상단에 아래와 같이 코드를 추가시켜준다.
```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
<c:set var="path" value="${pageContext.request.contextPath}"/>
```
그리고 템플릿의 왼쪽 사이드 메뉴를 사용하기 위해 `left_column.jsp`의 코드를 아래와 같이 수정해준다.
```xml
<ul class="sidebar-menu" data-widget="tree">
    <li class="header">메뉴</li>
    <li class="active"><a href="${path}/article/write"><i class="fa fa-edit"></i> <span>게시글 작성</span></a></li>
    <li><a href="${path}/article/list"><i class="fa fa-list"></i> <span>게시글 목록</span></a></li>
</ul>
```

#### # `jsp`파일 생성, `include`경로 변경
`WEB-INF/views/article`디렉토리에 등록(`write.jsp`), 조회(`read.jsp`), 수정(`modify.jsp`), 목록(`list.jsp`) `jsp`파일을 생성해준다. 그리고 `home.jsp`의 전체 코드를 복사해 붙여넣고, `include`의 모든 경로를 아래와 같이 수정해준다.
```xml
<%@ include file="../include/jsp파일명"%>
```


## 3. 게시글 관련 컨트롤러 생성하기
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
위 코드에서 주목할 점은 클래스 선언부의 `@RequestMapping("/article")`인데 공통의 경로를 `/article`로 인식하도록 설정한 것이다. 보통 컨트롤러는 하나의 기능을 가진 모듈의 대표적인 경로를 갖도록 하는 것이 좋다. 이후 작성할 메서드에 매핑할 URL의 길이를 줄여주는 장점도 가지고 있다.

이제 본격적으로 등록, 목록, 조회, 수정, 삭제 순으로 기능을 구현해보자.

## 4. 등록 구현

#### # 게시글 등록관련 메서드 작성
`ArticleController`에 게시글 등록 페이지 이동을 위한 메서드(`writeGET()`)와 등록 처리를 위한 메서드(`wrtiePOST()`)는 아래와 같이 작성해준다.
```java
// 게시글 등록페이지 매핑
@RequestMapping(value = "/write", method = RequestMethod.GET)
public String writeGET() {

    logger.info("write GET...");

    return "/article/write";
}
```

```java
// 게시글 등록처리 매핑
@RequestMapping(value = "/write", method = RequestMethod.POST)
public String writePOST(ArticleVO articleVO, Model model) throws Exception {

    logger.info("write POST...");
    logger.info(articleVO.toString());
    articleService.create(articleVO);
    model.addAttribute("msg", "regSuccess");

    return "/article/success";
}
```
`writePOST()`의 파라미터는 화면으로부터 받아온 모든 데이터를 `ArticleVO`로 수집 하도록하는 부분과 향후 데이터를 전달할 가능성이 있으므로 `Model`클래스의 객체를 받도록 했다. 게시글 처리가 완료되면 게시글 목록페이지로 `redirect`가 되도록 할 예정인데 아직 게시글 목록 페이지가 구현되지 않았기 때문에 임시페이지인 `success.jsp`을 생성하여 정상적으로 게시글이 등록되었는지 확인할 수 있도록 하였다.

#### # 게시글 등록 페이지 : `write.jsp`
게시글 등록페이지인 `write.jsp`의 `<section>`태그 안에 아래와 같이 코드를 작성해준다.
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
아래의 사진은 게시글 작성 페이지의 모습이다.
![write.jsp]()
아래의 사진은 게시글이 정상적으로 등록되고, `success.jsp`페이지로 이동한 결과이다.
![regSuccess]()

#### # 등록처리 메서드 수정
이제는 게시글 등록이 완료되면 `success.jsp`로 이동하지 않고, `redirect`결과페이지인 `list.jsp`로 이동하도록 `writePOST()`메서드의 리턴값을 `"redirect:/article/list";`로 변경한다. 그리고 아래의 코드에서 주목 해야할 점은 `writePOST()`의 매개변수 중에서 `Model`이 `RedirectAttributes`로 변경되었다는 점이다. `RedirectAttributes`객체는 `redirect`되는 시점에 딱 한번만 사용될 데이터를 전송할 수 있는 `addFlashAttribute()`라는 메서드가 정의 되어있다. 이 메서드를 통해 게시글의 등록처리가 완료되고 `redirect` 되는 시점에 딱 한번만 게시글이 성공적으로 등록되었다는 것을 알리는 데이터가 전송된다.

```java
@RequestMapping(value = "/write", method = RequestMethod.POST)
public String writePOST(ArticleVO articleVO,
                        RedirectAttributes redirectAttributes) throws Exception {

    logger.info("write POST...");
    logger.info(articleVO.toString());
    articleService.create(articleVO);
    redirectAttributes.addFlashAttribute("msg", "regSuccess");

    return "redirect:/article/list";
}
```

`list.jsp`의 하단에 `<script>`태그를 추가하고 `js`코드를 아래와 같이 작성해주면 게시글 등록처리 완료 `alert`창이 뜨게된다.
```js
var result = "${msg}";
if (result == "regSuccess") {
    alert("게시글 등록이 완료되었습니다.");
}
```

## 5. 목록 구현
이제 게시글 목록 구현을 완료해보자.

#### # 게시글 목록 메서드 작성

```java
// 게시글 목록 페이지 매핑
@RequestMapping(value = "/list", method = RequestMethod.GET)
public String list(Model model) throws Exception {

    logger.info("list ...");
    model.addAttribute("articles", articleService.listAll());

    return "/article/list";
}
```

#### # 게시글 등록 페이지 : `write.jsp`
`<section>`태그 안에 아래와 같이 코드를 작성해준다.
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
위의 코드에서 주목해야할 점은 게시글의 제목에 `<a>`태그로 조회페이지로 이동할 수 있도록 링크를 처리한 것과 게시글의 작성시간을 `JSTL`의 `fmt`태그를 통해 날짜,시간의 포맷을 변경한 것이다.

아래의 사진은 게시글이 정상적으로 등록되고 게시글 목록페이지로 이동한 결과이다.
![regSuccess]()
![list.jsp]()

## 6. 조회 구현

#### # 게시글 조회 메서드 작성
```java
// 게시글 조회 페이지 매핑
@RequestMapping(value = "/read", method = RequestMethod.GET)
public String read(@RequestParam("articleNo") int articleNo,
                   Model model) throws Exception {

    logger.info("read ...");
    model.addAttribute("article", articleService.read(articleNo));

    return "/article/read";
}
```

#### # 게시글 조회 페이지 작성
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
            <form role="form" method="post">
                <input type="hidden" name="articleNo" value="${article.articleNo}">
            </form>
            <button type="submit" class="btn btn-primary listBtn"><i class="fa fa-list"></i> 목록</button>
            <div class="pull-right">
                <button type="submit" class="btn btn-warning modBtn"><i class="fa fa-edit"></i> 수정</button>
                <button type="submit" class="btn btn-danger delBtn"><i class="fa fa-trash"></i> 삭제</button>
            </div>
        </div>
    </div>
</div>
```
위의 코드에서 눈여겨봐야할 점은 수정페이지, 삭제처리 링크처리인데 `<a>`를 사용하지 않고, `jQuery`를 통해 페이지를 이동하도록 `<script>`태그 안에 아래와 같이 작성해준다.
```js
$(document).ready(function () {

    var formObj = $("form[role='form']");
    console.log(formObj);

    $(".modBtn").on("click", function () {
        formObj.attr("action", "/article/modify");
        formObj.attr("method", "get");
        formObj.submit();
    });

    $(".delBtn").on("click", function () {
       formObj.attr("action", "/article/remove");
       formObj.submit();
    });

    $(".listBtn").on("click", function () {
       self.location = "/article/list"
    });

});
```


아래의 그림은 조회 페이지로 이동한 결과이다.
![read.jsp]()


## 7. 수정 구현

#### # 수정관련 메서드 작성
```java
// 수정페이지 매핑
@RequestMapping(value = "/modify", method = RequestMethod.GET)
public String modifyGET(@RequestParam("articleNo") int articleNo,
                        Model model) throws Exception {

    logger.info("modifyGet ...");
    model.addAttribute("article", articleService.read(articleNo));

    return "/article/modify";
}
```
```java
// 수정처리 매핑
@RequestMapping(value = "/modify", method = RequestMethod.POST)
public String modifyPOST(ArticleVO articleVO,
                         RedirectAttributes redirectAttributes) throws Exception {

    logger.info("modifyPOST ...");
    articleService.update(articleVO);
    redirectAttributes.addFlashAttribute("msg", "modSuccess");

    return "redirect:/article/list";
}
```

#### # 수정페이지 작성
```xml
<div class="col-lg-12">
    <form role="form" id="writeForm" method="post" action="${path}/article/modify">
        <div class="box box-primary">
            <div class="box-header with-border">
                <h3 class="box-title">게시글 작성</h3>
            </div>
            <div class="box-body">
                <input type="hidden" name="articleNo" value="${article.articleNo}">
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
            <div class="box-footer">
                <button type="button" class="btn btn-primary"><i class="fa fa-list"></i> 목록</button>
                <div class="pull-right">
                    <button type="button" class="btn btn-warning cancelBtn"><i class="fa fa-trash"></i> 취소</button>
                    <button type="submit" class="btn btn-success modBtn"><i class="fa fa-save"></i> 수정 저장</button>
                </div>
            </div>
        </div>
    </form>
</div>
```
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
        self.location = "/article/list"
    });

});
```

## 8. 삭제 구현
#### # 게시글 삭제 메서드 작성
```java
// 게시글 삭제 매핑
@RequestMapping(value = "/remove", method = RequestMethod.POST)
public String remove(@RequestParam("articleNo") int articleNo,
                     RedirectAttributes redirectAttributes) throws Exception {

    logger.info("remove ...");
    articleService.delete(articleNo);
    redirectAttributes.addFlashAttribute("msg", "delSuccess");

    return "redirect:/article/list";
}
```
