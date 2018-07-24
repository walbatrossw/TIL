
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

## 1. 쿠키를 이용하는 자동로그인 방식 구현

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

![user_login_remember_me1](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/user_login_remember_me1.png?raw=true)

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

![cookie_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/cookie_check.png?raw=true)

![cookie_check2](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/cookie_check2.png?raw=true)

![cookie_check3](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/cookie_check3.png?raw=true)

여기서 JSESSIONID는 Tomcat에서 발행된 세션 쿠키의 이름이고, loginCookie는 인터셉터에 의해서 만들어진 쿠키이다. 자세히 보면 세션 쿠키의 값과 동일한 값이 loginCookie에 기록된
것을 볼 수 있다.

#### 1.3 브라우저 종료 후 다시 접속

현재의 브라우저를 종료한 뒤 다시 해당 페이지를 쿠키가 유지되는 5분이내에 접속하면 아래와 같은 결과를 볼 수 있다.

![cookie_check4](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/cookie_check4.png?raw=true)

브라우저를 종료한 뒤 다시 접속하면 JSESSION은 변경되지만 일주일간 유효기간이 지정된 쿠키는 그 값을 유지하고 있는 것을 확인할 수 있다.

세션 쿠키는 브라우저가 종료될 때 같이 종료되기 때문에 매번 브라우저를 새로 실행하고, 접속하면 새롭게 만들어진다. 하지만 loginCookie의 경우 로그인할 때 브라우저를 이용해서 보관된다.

#### 1.4 자동로그인 구상

자동로그인의 처리는 Session과 Cookie를 이용하여 처리하도록 하는데 아래와 같이 4가지의 상황이 있을 수 있다.

|번호|HttpSession에 login 객체의 유무|loginCookie의 유무|상황|
|:---:|:---:|:---:|:---:|
|1|X|X|사용자 로그인이 필요, 비로그인 상태|
|2|O|X|사용자 로그인 상태(접속 중), 로그인 유지 선택 X|
|3|X|O|이전에 로그인 한 적이 있음, 7일사이에 로그인한 적 있음|
|4|O|O|사용자 로그인 상태(접속 중), 로그인 유지 선택 O|

위의 4가지 상황 중에서 자동 로그인이 필요한 상황은 3번이다. HttpSession에 login이란 이름으로 보관된 객체가 없지만, loginCookie가 존재한다. 이 경우는 인터셉터에서 설정한 7일의 기간 사이에
접속한 적이 있다는 것을 의미하므로, 과거의 로그인 시점에 기록된 정보를 이용해서 다시 HttpSession에 login이란 이름으로 `UserVO` 객체를 보관해줘야 한다.

#### 1.5 자동로그인을 위한 데이터베이스 설정 및 `LoginDTO`

위의 구상을 토대로 사용자가 loginCookie를 가지고 있다면 그 값은 과거에 로그인한 시점의 세션 아이디라는 것을 알 수 있다. 그래서 loginCookie의 값을 이용해서
데이터베이스에 `UserVO`의 정보를 읽어오고, 읽어본 `UserVO` 객체를 현재의 HttpSession에 보관하면 로그인이 된다. 이 과정을 정리하면 아래와 같다.

- 사용자가 로그인하면 데이터베이스에 현재의 Session ID값과 유효기간을 저장한다.
- 사용자가 로그인하지 않은 상태에서 쿠키를 가지고 접속하면 쿠키의 내용물을 추출한다.
- 쿠키의 내용물로 데이터베이스를 조회하고, 유효기간에 맞는 값인지 검증한다.
- 확인된 사용자는 Session에 로그인한 정보를 기록해서, 자동로그인 처리를 수행한다.

위의 내용을 구현하기 위해서 데이터베이스의 `tbl_user`의 컬럼들을 살펴보자. 이전에 회원가입과 로그인을 구현하면서 이미 자동로그인과 관련된 칼럼을 생성해두었다.

```sql
-- 회원 테이블
CREATE TABLE tbl_user (
  user_id VARCHAR(50) NOT NULL,
  user_pw VARCHAR(100) NOT NULL,
  user_name VARCHAR(100) NOT NULL,
  user_point INT NOT NULL DEFAULT 0,
  session_key VARCHAR(50) NOT NULL DEFAULT 'none',  -- session 아이디 보관
  session_limit TIMESTAMP,                          -- 자동로그인 유효시간 기록
  user_img VARCHAR(100) NOT NULL DEFAULT 'user/default-user.png',
  user_email VARCHAR(50) NOT NULL,
  user_join_date TIMESTAMP NOT NULL DEFAULT NOW(),
  user_login_date TIMESTAMP NOT NULL DEFAULT NOW(),
  user_signature VARCHAR(200) NOT NULL DEFAULT '안녕하세요 ^^',
  PRIMARY KEY (user_id)
);
```

#### 1.6 자동로그인 영속계층

`UserDAO` 인터페이스에 `sessionKey`와 `sessionLimit`를 업데이트하는 메서드(`keepLogin()`)와 `loginCookie`에 기록된 값으로 사용자의 정보를 조회하는 메서드(`checkUserWithSessionKey()`)를 추가하고,
`UserDAOImpl` 클래스에서 구현을 완료해준다.

```java
// 로그인 유지 처리
void keepLogin(String userId, String sessionId, Date sessionLimit) throws Exception;

// 세션키 검증
UserVO checkUserWithSessionKey(String value) throws Exception;
```

```java
// 로그인 유지 처리
@Override
public void keepLogin(String userId, String sessionId, Date sessionLimit) throws Exception {
    Map<String, Object> paramMap = new HashMap<>();
    paramMap.put("userId", userId);
    paramMap.put("sessionId", sessionId);
    paramMap.put("sessionLimit", sessionLimit);

    sqlSession.update(NAMESPACE + ".keepLogin", paramMap);
}

// 세션키 검증
@Override
public UserVO checkUserWithSessionKey(String value) throws Exception {
    return sqlSession.selectOne(NAMESPACE + ".checkUserWithSessionKey", value);
}
```

`userMapper.xml`에 아래와 같이 sql을 추가해준다.

로그인 유지를 선택한 경우 현재의 세션아이디와 로그인 유지기간을 갱신해준다.
```xml
<update id="keepLogin">
    UPDATE tbl_user
    SET session_key = #{sessionId}
        , session_limit = #{sessionLimit}
    WHERE user_id = #{userId}
</update>
```

로그인 시에 loginCookie 값과 `session_key`이 일치하는 회원의 정보를 가져온다.
```xml
<select id="checkUserWithSessionKey" resultMap="userVOResultMap">
    SELECT
        *
    FROM tbl_user
    WHERE session_key = #{value}
</select>
```

#### 1.7 자동로그인 서비스 계층

`UserService` 인터페이스에 로그인 유지 메서드(`keepLogin()`)와 loginCookie로 회원정보를 조회하는 메서드(`checkLoginBefore()`)를 추가하고 `UserServiceImpl` 클래스에서 구현을 완료한다.

```java
void keepLogin(String userId, String sessionId, Date next) throws Exception;

UserVO checkLoginBefore(String value) throws Exception;
```

```java
@Override
public void keepLogin(String userId, String sessionId, Date sessionLimit) throws Exception {
    userDAO.keepLogin(userId, sessionId, sessionLimit);
}

@Override
public UserVO checkLoginBefore(String value) throws Exception {
    return userDAO.checkUserWithSessionKey(value);
}
```

#### 1.8 자동로그인 컨트롤러
`UserLoginController`의 로그인 매핑 메서드(`loginPost()`)를 아래와 같이 수정해준다.

```java
// 로그인 처리
@RequestMapping(value = "/loginPost", method = RequestMethod.POST)
public void loginPOST(LoginDTO loginDTO, HttpSession httpSession, Model model) throws Exception {

    UserVO userVO = userService.login(loginDTO);

    if (userVO == null || !BCrypt.checkpw(loginDTO.getUserPw(), userVO.getUserPw())) {
        return;
    }

    model.addAttribute("user", userVO);

    // 로그인 유지를 선택할 경우
    if (loginDTO.isUseCookie()) {
        int amount = 60 * 60 * 24 * 7;  // 7일
        Date sessionLimit = new Date(System.currentTimeMillis() + (1000 * amount)); // 로그인 유지기간 설정
        userService.keepLogin(userVO.getUserId(), httpSession.getId(), sessionLimit);
    }

}
```

위 코드에서 주목해서 봐야할 점은 로그인 유지기간을 7일로 설정하고, 유지기간을 실제로 DB에 저장하도록 처리하는 것이다.


#### 1.9 자동로그인 인터셉터

이제 마지막으로 인터셉터를 통해 사용자가 접속을 할 경우 자동으로 로그인이 되도록 처리해보자. 책에서는 `AuthInterceptor`에 코드를 작성하였지만,
나의 경우 `RememberMeInterceptor` 클래스를 만들고 그 곳에 코드를 작성했다.

`RememberMeInterceptor` 클래스를 인터셉터로 스프링이 인식할 수 있도록 `dispatcher-servlet.xml`에 아래와 같이 설정을 추가한다.

```xml
<bean id="rememberMeInterceptor" class="com.doubles.mvcboard.commons.interceptor.RememberMeInterceptor"/>

<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**/"/>
        <ref bean="rememberMeInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

```java
public class RememberMeInterceptor extends HandlerInterceptorAdapter {

    private static final Logger logger = LoggerFactory.getLogger(RememberMeInterceptor.class);

    @Inject
    private UserService userService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        HttpSession httpSession = request.getSession();
        Cookie loginCookie = WebUtils.getCookie(request, "loginCookie");
        if (loginCookie != null) {
            UserVO userVO = userService.checkLoginBefore(loginCookie.getValue());
            if (userVO != null)
                httpSession.setAttribute("login", userVO);
        }

        return true;
    }
}
```

위의 코드를 설명하면 아래와 같다.

- 접속한 사용자가 `loginCookie`를 가지고 있다면, `loginCookie`의 정보를 이용해 사용자 정보가 존재하는지 확인한다.
- 사용자 정보가 존재할 경우, HttpSessin에 회원의 정보를 넣어준다.

#### 1.10 자동 로그인 테스트 확인

실제로 자동 로그인이 구현되었는지 확인해보자.

![remember_me_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/17_spring_mvc_board_remember_me/remember_me_check.png?raw=true)

위의 화면을 보면 로그인 유지를 선택하고 로그인을 하면 실제로 DB에 쿠키값과 로그인 유지기간에 저장되고, 브라우저를 종료한 뒤 다시 접속하면 정상적으로 로그인이 유지가 되는 것을 확인할 수 있다.

## 2. 로그아웃 처리

## 3. 로그인 상태에서 로그인 페이지, 회원가입 페이지 접근 제한하기