
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
|8|[Spring MVC Interceptor](http://doublesprogramming.tistory.com/210)|


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
|14|[댓글 갯수, 게시글 조회수 구현, 트랜잭션처리](http://doublesprogramming.tistory.com/208)|
|15|[AJAX방식의 게시판 첨부파일 기능 구현](http://doublesprogramming.tistory.com/209)|
|16|[HttpSession을 이용하는 로그인 처리](http://doublesprogramming.tistory.com/211)|

# Spring MVC 게시판 예제 17 - 자동로그인 기능 및 로그아웃 구현

자동로그인을 구현해보자.

## 1. 쿠키를 이용하는 자동로그인 방식

자동 로그인을 처리하기 전에 앞서 사용자가 로그인한 후 쿠키를 만들어 브라우저로 전송하고, 다시 서버에 접속할 때 쿠키가 전달되는지 확인해보자.

#### 1.1 로그인유지를 위한 체크박스
로그인 처리 구현하면서 `login.jsp`에 체크박스를 이미 만들어 두었다.

```html
<div class="col-xs-8">
    <div class="checkbox icheck">
        <label>
            <input type="checkbox" name="useCookie"> 로그인유지
        </label>
    </div>
</div>
```

![user_login](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_login.png?raw=true)

#### 1.2 LoginInterceptor에서의 쿠키 생성하기

이전에 작성한 `LoginInterceptor` 클래스의 경우 `preHandle()`메서드를 이용해서 HttpSession에 `UserVO`타입의 객체를 보관한다. 이 것을 수정해 중간에 쿠키를 생성하고,
`HttpServletResponse`에 담아서 전송하도록 코드를 수정해준다.

```java
@Override
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    HttpSession httpSession = request.getSession();
    ModelMap modelMap = modelAndView.getModelMap();
    Object userVO =  modelMap.get("user");

    if (userVO != null) {
        logger.info("new login success");
        httpSession.setAttribute(LOGIN, userVO);
        //response.sendRedirect("/");
        
        if (request.getParameter("useCookie") != null) {
            logger.info("remember me...");
            // 쿠키 생성
            Cookie loginCookie = new Cookie("loginCookie", httpSession.getId());
            loginCookie.setPath("/");
            loginCookie.setMaxAge(60*60*24*7);
            // 전송
            response.addCookie(loginCookie);
        }

        Object destination = httpSession.getAttribute("destination");
        response.sendRedirect(destination != null ? (String) destination : "/");
    }

}
```

위의 코드를 보면 사용자가 로그인 유지를 선택한 경우 쿠키를 생성하고, `loginCookie`라는 이름의 변수에 현재 session의 아이디값을 보관한다. 여기서 session 아이디는
session cookie의 값을 의미한다. session cookie의 경우 브라우저를 종료하면 사라지지만, 작성하는 loginCookie의 경우 오랜시간 보관하기 위해 `setMaxAge()`메서드를
이용한다. 그리고 최종적으로 만들어진 쿠키는 `HttpServletResponse`에 담겨서 전송된다.

위 코드를 이용해서 로그인한 후에 게시물의 여러 페이지에서 매번 브라우저에 `loginCookie`가 전달되는 것을 개발자 도구에서 확인할 수 있다.

![]()



