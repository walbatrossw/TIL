
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
---

# Spring MVC 게시판 예제 07 - 페이징처리 : Control, Presentation 계층

## 1. 페이징처리를 위한 컨트롤러 메서드 작성
#### #`ArticleController`
아래와 같이 페이징된 목록의 요청을 처리할 메서드를 아래와 같이 작성해준다.
```java
@RequestMapping(value = "/listCriteria", method = RequestMethod.GET)
public String listCriteria(Model model, Criteria criteria) throws Exception {
    logger.info("listCriteria ...");
    model.addAttribute("articles", articleService.listCriteria(criteria));
    return "/article/list_criteria";
}
```

## 2. 페이징처리를 위한 `JSP`페이지 작성
`/WEB-INF/views/article/`디렉토리에 `list_criteria.jsp`파일을 생성하고, `list.jsp`내용을 복사해 붙여 넣어준다.

#### # 페이징 처리전의 게시글 목록과 처리 후의 게시글 목록

페이지 처리 전의 게시글 목록
![list.jsp]()

페이징 처리 후의 게시글 목록1
![list_criteria.jsp1]()

페이징 처리 후의 게시글 목록2 : GET파라미터 page의 값 4
![list_criteria.jsp2]()

페이징 처리 후의 게시글 목록3 : GET파라미터 page의 값 4와 perPageNum의 값 20
![list_criteria.jsp3]()
