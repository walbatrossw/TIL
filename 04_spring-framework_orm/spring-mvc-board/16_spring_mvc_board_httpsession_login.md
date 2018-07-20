# Spring MVC 게시판 예제 16 - HttpSession을 이용하는 로그인 처리

앞서 살펴본 Interceptor에서 통해 HttpSession 객체를 이용하여 게시판에 로그인을 처리하는 것을 정리해보자.

## 1. HttpSession과 로그인

웹에서 로그인의 가장 기본적인 방식은 HttpSession 객체를 이용해서 사용자의 정보를 보관하고, 필요한 경우 사용하거나 수정하는 방식이다.

HttpSession의 동작은 세션 쿠키를 통해 이루어진다. 서버는 접속한 브라우저에게 고유한 세션쿠키를 전달하고, 매번 브라우저에서 서버를 호출할 때 세션 쿠키를 가지고
다니기 때문에, 이를 마치 열쇠처럼 사용해서 필요한 데이터를 보관한다.

세션쿠키가 열쇠라면 HttpSession은 열쇠가 필요한 잠금장치가 되어있는 상자와 유사하다. 이 상자들이 모여있는 공간을 세션 저장소(Session Repository)라고 하는데,
너무나 많은 세션이 존재하면 서버의 성능에 영향을 미치기 때문에 서버는 일정시간 이상 사용되지 않는 상자들을 정리하는 기능을 가지고 있다. `web.xml`에서 HttpSession의
timeout을 지정할 수 있다.

session을 이요하는 방식의 핵심은 HttpSession을 이용해서 원하는 객체를 보관할 수 있다는 점이다. 사용자는 항상 열쇠에 해당하는 세션쿠키를 가지고 접근하고, 서버의 내부에
상자가 필요한 객체를 보관하기 때문에 안전하다는 장점을 가지고 있다.

session에 보관된 객체는 JSP에서 EL을 이용해서 자동으로 추적하는 방식을 사용한다. 예를 들면 `${name}`은 page -> request -> session -> application의 순서대로 원하는
데이터를 검색한다. 이와 같은 방식으로 동작하기 때문에 JSP를 개발하는 개발자는 자신이 사용하는 변수가 request에 존재하는 것인지, session에 존재하는 것인지 고민하지 않아도
된다.

#### 1.1 HttpSession

[Interface HttpSession 링크](https://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpSession.html)

아래의 내용은 위의 Servlet API에서 HttpSession에 대한 내용을 번역하고 정리한 내용이다.

**HttpSession은 둘 이상의 페이지의 요청 또는 웹사이트를 방문한 사용자를 식별하고 해당 사용자에 대한 정보를 저장하는 방법을 제공한다.**

- 서블릿 컨테이너는 Http 클라이언트와 Http 서버 사이의 세션을 만드는데 사용한다.
- Session은 사용자의 둘 이상의 연결 또는 페이지 요청을 통해 지정된 기간 동안 유지된다.
- Session은 보통 웹사이트를 여러번 방문하는 하나의 사용자에 해당한다.
- 서버는 쿠키를 사용하거나 URL 다시쓰기 등과 같은 여러가지 방법으로 session을 유지할 수 있다.

HttpSession은 서블릿이 다음 작업을 수행하도록 허용한다.

- Session 식별자, 생성시간, 마지막 접근 시간과 같은 세션에 대한 정보를 조작하고 볼 수 있다.
- 객체를 Session에 바인딩하여 여러 사용자 연결에서 사용자 정보가 유지되도록 한다.

어플리케이션이 Session에 객체를 저장하거나 제거할 때, Session은 객체가 `HttpSessionBindingListener`인터페이스 구현여부를 확인한다. 이 경우 서블릿은 객체가 Session에 바인딩
되거나 바인딩 해제되었음을 바인딩 메서드의 실행이 완료된 후에 알려준다. 해제되거나 만료된 Session은 Session이 해제되거나 만료된 후에 알려준다.

컨테이너가 분산 컨테이너 설정에서 VM간의 세션을 옮겨갈 때 `HttpSessionActivationListener`인터페이스를 구현하는 모든 Session의 속성을 알려준다.

서블릿은 쿠키를 의도적으로 해제할 때와 같이 클라이언트가 Session에 참여하지 않는 경우를 처리할 수 있어야 한다. 클라이언트가 Session에 참여할 때까지 `isNew()` 메서드는 `true`를 반환한다.
만약 클라이언트가 Session에 참여하지 않기로 선택한 경우에는 `getSession()` 메서드는 각 요청에 대해 다른 Session을 반환하고, `isNew()`메서드는 항상 `true`를 반환한다.

Session의 정보는 현재 웹 어플리케이션(Servlet Context)의 범위만 지정되기 때문에 하나의 Context에 저장된 정보는 직접적으로 다른 Context에서는 볼 수가 없다.

## 2. 로그인 처리를 위한 회원가입 기능 구현

로그인 처리를 위해서 먼저 회원가입 기능을 먼저 구현해보자. 현재 보고있는 책에서는 따로 회원가입에 대한 내용이 없다. 그래서 직접 구현한 내용을 정리하였다.

#### 2.1 회원 테이블 생성
회원가입과 로그인을 처리하기 위해 아래와 같이 회원 테이블을 생성한다. 기본적으로 회원 테이블의 칼럼에는 아이디, 비밀번호, 이름, 이메일이 있고, 추후에 구현될 회원포인트 기능을 위한 칼럼과 로그인 유지를
위한 `session_key`, `session_limit` 칼럼이 있다. 그리고 회원의 프로필 이미지 칼럼, 가입일자, 로그인일자, 서명이 있다.

```sql
-- 회원 테이블
CREATE TABLE tbl_user (
  user_id VARCHAR(50) NOT NULL,
  user_pw VARCHAR(100) NOT NULL,
  user_name VARCHAR(100) NOT NULL,
  user_email VARCHAR(50) NOT NULL,
  user_point INT NOT NULL DEFAULT 0,
  session_key VARCHAR(50) NOT NULL DEFAULT 'none',
  session_limit TIMESTAMP,
  user_img VARCHAR(100) NOT NULL DEFAULT 'user/default-user.png',
  user_join_date TIMESTAMP NOT NULL DEFAULT NOW(),
  user_login_date TIMESTAMP NOT NULL DEFAULT NOW(),
  user_signature VARCHAR(200) NOT NULL DEFAULT '안녕하세요 ^^',
  PRIMARY KEY (user_id)
);
```
#### 2.2 회원 클래스 작성
`기본패키지/user/domain`패키지에 회원 클래스를 아래와 같이 작성해준다.
```java
public class UserVO {

    private String userId;
    private String userPw;
    private String userName;
    private String userEmail;
    private Date userJoinDate;
    private Date userLoginDate;
    private String userSignature;
    private String userImg;
    private int userPoint;

    // getter, setter, toString 생략 
}
```

#### 2.3 회원 영속 계층 구현
`기본패키지/user/persistence`패키지에 `UserDAO` 인터페이스와 `UserDAOImpl`클래스를 아래와 같이 작성해준다.

```java
public interface UserDAO {
    
    // 회원가입 처리
    void register(UserVO userVO) throws Exception;
    
}
```

```java
@Repository
public class UserDAOImpl implements UserDAO {
    
    private static final String NAMESPACE = "com.doubles.mvcboard.mappers.user.UserMapper";
    
    private final SqlSession sqlSession;

    @Inject
    public UserDAOImpl(SqlSession sqlSession) {
        this.sqlSession = sqlSession;
    }
    
    // 회원가입처리
    @Override
    public void register(UserVO userVO) throws Exception {
        sqlSession.insert(NAMESPACE + ".register", userVO);
    }
    
}
```

`/resources/mappers/user`경로에 `userMapper.xml`을 생성하고 아래와 같이 작성해준다. `tbl_user`테이블의 칼럼과 `UserVO`클래스의 필드명이 불일치하기 때문에 각각을 매칭시켜주기 위해 `resultMap`도
아래와 같이 작성해준다. insert 퀴리의 경우 문제가 발생하지 않지만 select 쿼리의 경우 문제가 발생하기 때문에 미리 작성한 것이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.doubles.mvcboard.mappers.user.UserMapper">
    
    <insert id="register">
        INSERT INTO tbl_user (
            user_id
            , user_pw
            , user_name
            , user_email
        ) VALUES (
            #{userId}
            , #{userPw}
            , #{userName}
            , #{userEmail}
        )
    </insert>
    
</mapper>
```

#### 2.4 회원 서비스 계층 구현

`기본패키지/user/service`패키지에 `UserService`인터페이스와 `UserServiceImpl`클래스를 아래와 같이 작성해준다.

```java
public interface UserService {
    
    // 회원 가입 처리
    void register(UserVO userVO) throws Exception;
    
}
```

```java
@Service
public class UserServiceImpl implements UserService {

    private final UserDAO userDAO;

    @Inject
    public UserServiceImpl(UserDAO userDAO) {
        this.userDAO = userDAO;
    }
    
    // 회원 가입 처리
    @Override
    public void register(UserVO userVO) throws Exception {
        userDAO.register(userVO);
    }
    
}
```

#### 2.5 회원가입 컨트롤러 작성

회원관련 uri를 통합해서 하나의 컨트롤러에 모든 매핑시키는 것도 좋지만 그렇게 되면 코드가 많아지고, 구분이 어려워지기 때문에
회원 가입페이지, 가입처리, 탈퇴 uri만을 매핑하는 컨트롤러를 따로 만들어 사용한다.

`기본패키지/user/controller` 패키지에 `UserRegisterController` 클래스를 생성하고, 아래와 같이 코드를 작성해준다.

```java
@Controller
@RequestMapping("/user")
public class UserRegisterController {

    private final UserService userService;

    @Inject
    public UserRegisterController(UserService userService) {
        this.userService = userService;
    }

    // 회원가입 페이지
    @RequestMapping(value = "/register", method = RequestMethod.GET)
    public String registerGET() throws Exception {
        return "/user/register";
    }

    // 회원가입 처리
    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public String registerPOST(UserVO userVO, RedirectAttributes redirectAttributes) throws Exception {

        String hashedPw = BCrypt.hashpw(userVO.getUserPw(), BCrypt.gensalt());
        userVO.setUserPw(hashedPw);
        userService.register(userVO);
        redirectAttributes.addFlashAttribute("msg", "REGISTERED");

        return "redirect:/user/login";
    }
    
}
```

위의 코드에서 주목해서 봐야할 점은 회원가입 처리 매핑 메서드인 `registerPost()`이다. 파라미터로 넘어온 회원의 객체정보(`UserVO`) 중에서 비밀번호(`userPw`)를 암호화 하는 작업을 수행한다.
이렇게 하는 이유는 회원으로부터 받은 정보 중에서 비밀번호는 반드시 DB에 암호화해서 보관해야 보안에 비교적 안전하기 때문이다. `BCrypt.hashpw()`메서드는 위와 같이 첫번째 파라미터에는 암호화할
비밀번호를 두번째 파라미터는 `BCrypt.gensalt()`를 받고 암호화된 비밀번호를 리턴해준다. 이렇게 암호화된 비밀번호를 다시 회원 객체에 저장하고 서비스의 회원가입 메서드를 호출하면 된다.

그리고 최종적으로 회원가입 처리가 완료되면 최종적으로 성공메서지를 가지고 로그인페이지로 리다이렉트 하게된다.

암호화 처리를 위해 BCrypt를 사용하기 위해서는 `pom.xml`에 아래와 같이 의존성을 추가시켜줘야한다.

```xml
<!--비밀번호 암호화 -->
<!-- https://mvnrepository.com/artifact/org.mindrot/jbcrypt -->
<dependency>
    <groupId>org.mindrot</groupId>
    <artifactId>jbcrypt</artifactId>
    <version>0.4</version>
</dependency>
```

#### 2.6 회원가입 및 로그인 페이지 작성

`/views/user/` 디렉토리에 회원가입 페이지(`register.jsp`)를 생성하고, 아래와 같이 작성한다.

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html>
<%@ include file="../include/head.jsp" %>
<body class="hold-transition register-page">
<div class="register-box">
    <div class="register-logo">
        <a href="${path}/">
            <b>DoubleS</b>&nbsp MVC-BOARD
        </a>
    </div>

    <div class="register-box-body">
        <p class="login-box-msg">회원가입 페이지</p>

        <form action="${path}/user/register" method="post">
            <div class="form-group has-feedback">
                <input type="text" name="userId" class="form-control" placeholder="아아디">
                <span class="glyphicon glyphicon-exclamation-sign form-control-feedback"></span>
            </div>
            <div class="form-group has-feedback">
                <input type="text" name="userName" class="form-control" placeholder="이름">
                <span class="glyphicon glyphicon-user form-control-feedback"></span>
            </div>
            <div class="form-group has-feedback">
                <input type="email" name="userEmail" class="form-control" placeholder="이메일">
                <span class="glyphicon glyphicon-envelope form-control-feedback"></span>
            </div>
            <div class="form-group has-feedback">
                <input type="password" name="userPw" class="form-control" placeholder="비밀번호">
                <span class="glyphicon glyphicon-lock form-control-feedback"></span>
            </div>
            <div class="form-group has-feedback">
                <input type="password" class="form-control" placeholder="비밀번호 확인">
                <span class="glyphicon glyphicon-log-in form-control-feedback"></span>
            </div>
            <div class="row">
                <div class="col-xs-8">
                    <div class="checkbox icheck">
                        <label>
                            <input type="checkbox"> 약관에 <a href="#">동의</a>
                        </label>
                    </div>
                </div>
                <div class="col-xs-4">
                    <button type="submit" class="btn btn-primary btn-block btn-flat">가입</button>
                </div>
            </div>
        </form>

        <div class="social-auth-links text-center">
            <p>- 또는 -</p>
            <a href="#" class="btn btn-block btn-social btn-facebook btn-flat">
                <i class="fa fa-facebook"></i> 페이스북으로 가입
            </a>
            <a href="#" class="btn btn-block btn-social btn-google btn-flat">
                <i class="fa fa-google-plus"></i> 구글 계정으로 가입
            </a>
        </div>

        <a href="${path}/user/login" class="text-center">로그인</a>
    </div>
    <!-- /.form-box -->
</div>
<!-- /.register-box -->

<%@ include file="../include/plugin_js.jsp" %>
<script>
    $(function () {
        $('input').iCheck({
            checkboxClass: 'icheckbox_square-blue',
            radioClass: 'iradio_square-blue',
            increaseArea: '20%' // optional
        });
    });
</script>
</body>
</html>
```

![user_register](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_register.png?raw=true)

`/views/user/` 디렉토리에 로그인 페이지(`login.jsp`)를 생성하고, 아래와 같이 작성한다.

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html>
<%@ include file="../include/head.jsp" %>
<body class="hold-transition login-page">
<div class="login-box">
    <div class="login-logo">
        <a href="${path}/">
            <b>DoubleS</b>&nbsp MVC-BOARD
        </a>
    </div>
    <!-- /.login-logo -->
    <div class="login-box-body">
        <p class="login-box-msg">로그인 페이지</p>

        <form action="${path}/user/loginPost" method="post">
            <div class="form-group has-feedback">
                <input type="text" name="userId" class="form-control" placeholder="아아디">
                <span class="glyphicon glyphicon-exclamation-sign form-control-feedback"></span>
            </div>
            <div class="form-group has-feedback">
                <input type="password" name="userPw" class="form-control" placeholder="비밀번호">
                <span class="glyphicon glyphicon-lock form-control-feedback"></span>
            </div>
            <div class="row">
                <div class="col-xs-8">
                    <div class="checkbox icheck">
                        <label>
                            <input type="checkbox" name="useCookie"> 로그인유지
                        </label>
                    </div>
                </div>
                <!-- /.col -->
                <div class="col-xs-4">
                    <button type="submit" class="btn btn-primary btn-block btn-flat">
                        <i class="fa fa-sign-in"></i> 로그인
                    </button>
                </div>
                <!-- /.col -->
            </div>
        </form>

        <div class="social-auth-links text-center">
            <p>- 또는 -</p>
            <a href="#" class="btn btn-block btn-social btn-facebook btn-flat">
                <i class="fa fa-facebook"></i> 페이스북으로 로그인
            </a>
            <a href="#" class="btn btn-block btn-social btn-google btn-flat">
                <i class="fa fa-google-plus"></i> 구글 계정으로 로그인
            </a>
        </div>
        <!-- /.social-auth-links -->

        <a href="#">비밀번호 찾기</a><br>
        <a href="${path}/user/register" class="text-center">회원가입</a>

    </div>
    <!-- /.login-box-body -->
</div>
<!-- /.login-box -->

<%@ include file="../include/plugin_js.jsp" %>
<script>

    var msg = "${msg}";
    if (msg === "REGISTERED") {
        alert("회원가입이 완료되었습니다. 로그인해주세요~");
    } else if (msg == "FAILURE") {
        alert("아이디와 비밀번호를 확인해주세요.");
    }

    $(function () {
        $('input').iCheck({
            checkboxClass: 'icheckbox_square-blue',
            radioClass: 'iradio_square-blue',
            increaseArea: '20%' // optional
        });
    });
</script>
</body>
</html>
```

![user_login](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_login.png?raw=true)

#### 2.7 회원가입 처리 확인

회원가입 처리가 제대로 되는지 확인해보자.

`/user/register` uri를 요청하여 회원가입 페이지로 이동한 뒤 아이디, 이름, 이메일, 비밀번호를 작성하고 가입 버튼을 누른다.

![user_register_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_register_check.png?raw=true)

회원가입이 성공적으로 처리가 되면, `/user/login` uri로 리다이렉트 되고, 아래와 같이 성공 메시지가 나타나게 된다.

![user_register_check2](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_register_check2.png?raw=true)

DB를 확인해보면 비밀번호가 제대로 암호화가 되었는지도 확인할 수 있다.
![user_register_check3](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_register_check3.png?raw=true)


## 3. 로그인 구현

#### 3.1 로그인 처리를 위한 DTO 클래스 작성

`기본클래스/user/domain`에 `LoginDTO`클래스를 생성하고, 아래와 같이 코드를 작성한다. `LoginDTO`의 용도는 로그인화면으로부터 전달되는 회원의 데이터(아이디, 비밀번호)를 수집하는 용도로 사용한다.

```java
public class LoginDTO {

    private String userId;
    private String userPw;
    private boolean useCookie;

    // getter, setter, toString 생략
}
```

**VO(Value Object)와 DTO(Data Transfer Object)의 차이점**

일반적으로 컨트롤러에 전달되는 데이터를 수집하는 용도로 VO를 사용하는 경우도 있고, DTO를 사용하는 경우도 있는데 두 용어에 대해서 공통점과 차이점에 대해 알아보자.

- 공통점
    - DTO와 VO의 용도는 데이터 수집과 전달에 사용할 수 있다.
    - 파라미터나 리턴 타입으로 사용하는 것이 가능하다.
- 차이점
    - VO의 경우 보다 데이터베이스와 거리가 가깝기 때문에 VO는 테이블 구조를 이용해서 작성되는 경우가 많다.
    - DTO의 경우 화면에 거리가 가깝기 때문에 화면에서 전달되는 데이터를 수집하는 용도로 사용하는 경우가 많다.


#### 3.2 회원 영속 계층 구현

`UserDAO`인터페이스에 로그인 메서드를 추가하고, `UserDAOImpl`클래스에서 로그인 메서드를 구현한다.

```java
// 로그인 처리
UserVO login(LoginDTO loginDTO) throws Exception;
```

```java
// 로그인 처리
@Override
public UserVO login(LoginDTO loginDTO) throws Exception {
    return sqlSession.selectOne(NAMESPACE + ".login", loginDTO);
}
```

그리고 `userMapper.xml`에서 회원의 아이디로 회원정보를 select하는 쿼리를 작성해준다. 물론 책에서는 `where` 조건절을 아이디와 비밀번호 둘다 가지고 쿼리를 작성하지만 나의 경우
비밀번호 암호화를 처리했기때문에 컨트롤러에서 비밀번호 검증처리가 들어간다. 그래서 비밀번호는 제외하고 아이디만으로 select하도록 작성했다.

```xml
<select id="login" resultMap="userVOResultMap">
    SELECT
      *
    FROM tbl_user
    WHERE user_id = #{userId}
</select>
```

#### 3.3 회원 서비스 계층 구현

`UserService`인터페이스에 로그인 메서드를 추가하고, `UserServceImpl`클래스에서 로그인 메서드를 아래와 같이 구현한다.

```java
UserVO login(LoginDTO loginDTO) throws Exception;
```

```java
@Override
public UserVO login(LoginDTO loginDTO) throws Exception {
    return userDAO.login(loginDTO);
}
```

#### 3.4 로그인 컨트롤러 작성

서비스 계층까지 구현이 완료되었는데 나머지 컨트롤러와 인터셉터를 적용하는 작업을 해야한다. 이 때 가장 중요한 것은 컨트롤러에서 HttpSessin 객체를 처리할 것인지 인터셉터에서 HttpSessin을
처리할 것인지 정해야한다.

스프링 MVC는 컨트롤러에서 필요한 모든 작원을 파라미터에서 수집해서 처리하기 때문에 `HttpServletRequest`나 `HttpSession`과 같은 자원들 역시 파라미터로 처리해도 문제가 없다. 그래서 컨트롤러에서는
되도록 순수하게 데이터를 만들어내는데 집중하고, 인터셉터를 이용해 HttpSessin을 처리하도록 작성해준다.

`기본패키지/user/controller` 패키지에 `UserLoginController` 클래스를 생성하고, 아래와 같이 코드를 작성한다. 이전에도 언급했지만 하나의 컨트롤러엣 회원관련 uri를 매핑하는 메서드가 많아지면
코드의 양이 많아지고, 구분하기 어렵기때문에 로그인 처리를 담당하는 컨트롤러를 따로 작성해주었다.

```java
@Controller
@RequestMapping("/user")
public class UserLoginController {

    private final UserService userService;

    @Inject
    public UserLoginController(UserService userService) {
        this.userService = userService;
    }

    // 로그인 페이지
    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public String loginGET(@ModelAttribute("loginDTO") LoginDTO loginDTO) {
        return "/user/login";
    }

    // 로그인 처리
    @RequestMapping(value = "/loginPost", method = RequestMethod.POST)
    public void loginPOST(LoginDTO loginDTO, HttpSession httpSession, Model model) throws Exception {

        UserVO userVO = userService.login(loginDTO);

        if (userVO == null || !BCrypt.checkpw(loginDTO.getUserPw(), userVO.getUserPw())) {
            return;
        }

        model.addAttribute("user", userVO);
        
    }
    
}
```

위 코드에서 주목해서 봐야할 점은 로그인처리 메서드(`loginPost()`)인데 처리과정을 정리하면 아래와 같다.

- 화면으로부터 받은 데이터(회원아이디, 비밀번호) 중에서 아이디를 통해 select한 회원 정보를 변수 `userVO`에 담는다.
- `userVO`가 `null`이거나 비밀번호를 `BCrypt.checkpw()`를 통해 검증해서 맞지않으면 메서드를 종료시킨다.
- 비밀번호가 일치하면 model에 `userVO`를 `user`란 이름의 변수에 저장한다.

#### 3.5 로그인 Interceptor 클래스 작성 및 설정

`UserLoginController`에서 HttpSession과 관련된 작업이 처리되지 않았기 때문에 HttpSession과 관련된 모든 설정은 인터셉터에서 처리한다. `기본패키지/commons/interceptor` 패키지에 `LoginInterceptor`를
생성하고, 아래와 같이 코드를 작성한다.

```java
public class LoginInterceptor extends HandlerInterceptorAdapter {

    private static final String LOGIN = "login";
    private static final Logger logger = LoggerFactory.getLogger(LoginInterceptor.class);

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

        HttpSession httpSession = request.getSession();
        ModelMap modelMap = modelAndView.getModelMap();
        Object userVO =  modelMap.get("user");

        if (userVO != null) {
            logger.info("new login success");
            httpSession.setAttribute(LOGIN, userVO);
            response.sendRedirect("/");
        }

    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        HttpSession httpSession = request.getSession();
        // 기존의 로그인 정보 제거
        if (httpSession.getAttribute(LOGIN) != null) {
            logger.info("clear login data before");
            httpSession.removeAttribute(LOGIN);
        }

        return true;
    }
}
```

`LoginInterceptor`의 `postHandle()` 메서드는 httpSession에 컨트롤러에서 저장한 `user`를 저장하고, `/`로 리다이렉트를 한다. 그리고 `preHandle()` 메서드는 기존의 로그인 정보가 있을 경우 초기화하는 역할을
수행한다.

앞서 작성한 로그인 인터셉터 클래스를 스프링에서 인터셉터로 인식시키기 위해 아래와 같이 `dispatcher-servlet.xml`(`servlet-context.xml`)에 코드를 작성해준다.

```xml
<bean id="loginInterceptor" class="com.doubles.mvcboard.commons.interceptor.LoginInterceptor"/>

<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/user/loginPost"/>
        <ref bean="loginInterceptor"/>
    </mvc:interceptor>
<mvc:interceptors>
```

#### 3.6 로그인 화면

이미 회원가입 처리를 구현하면서 로그인 화면을 이미 작성했고, 결과 페이지에 해당하는 `loginPost.jsp`를 아래와 같이 작성해준다. 컨트롤러에서 만약 회원정보가 없거나, 비밀번호가 불일치한다면 `loginPost.jsp`로
이동하여 아이디와 비밀번호를 확인하라는 메시지와 함께 로그인페이지로 다시 이동하게 처리하였다.

```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <script>
        alert("아이디와 비밀번호를 확인해주세요.");
        self.location = "/user/login";
    </script>
</body>
</html>

```

그리고 실제로 로그인이 되었는지 사용자 알 수 있게 include한 페이지들(`main_header.jsp`, `left_column.jsp`)를 아래와 같이 수정해준다.

**`main_header.jsp`**

```html
<%-- Header Navbar --%>
<nav class="navbar navbar-static-top" role="navigation">
    <a href="#" class="sidebar-toggle" data-toggle="push-menu" role="button">
        <span class="sr-only">Toggle navigation</span>
    </a>
    <div class="navbar-custom-menu">
        <ul class="nav navbar-nav">
            <c:if test="${not empty login}">
                <li class="dropdown user user-menu">
                    <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                        <img src="/${login.userImg}" class="user-image" alt="User Image">
                        <span class="hidden-xs">${login.userName}</span>
                    </a>
                    <ul class="dropdown-menu">
                        <li class="user-header">
                            <img src="/${login.userImg}" class="img-circle" alt="User Image">
                            <p>${login.userName}
                                <small>
                                    가입일자 : <fmt:formatDate value="${login.userJoinDate}" pattern="yyyy-MM-dd"/>
                                </small>
                            </p>
                        </li>
                        <li class="user-body">
                            <div class="row">
                                <div class="col-xs-4 text-center">
                                    <a href="#">게시글</a>
                                </div>
                                <div class="col-xs-4 text-center">
                                    <a href="#">추천글</a>
                                </div>
                                <div class="col-xs-4 text-center">
                                    <a href="#">북마크</a>
                                </div>
                            </div>
                        </li>
                        <li class="user-footer">
                            <div class="pull-left">
                                <a href="${path}/user/info" class="btn btn-default btn-flat"><i
                                        class="fa fa-info-circle"></i><b> 내 프로필</b></a>
                            </div>
                            <div class="pull-right">
                                <a href="${path}/user/logout" class="btn btn-default btn-flat"><i
                                        class="glyphicon glyphicon-log-out"></i><b> 로그아웃</b></a>
                            </div>
                        </li>
                    </ul>
                </li>
            </c:if>
            <c:if test="${empty login}">
                <li class="dropdown user user-menu">
                    <a href="#" class="dropdown-toggle" data-toggle="dropdown">
                        <img src="${path}/user/default-user.png" class="user-image" alt="User Image">
                        <span class="hidden-xs">회원가입 또는 로그인</span>
                    </a>
                    <ul class="dropdown-menu">
                        <li class="user-header">
                            <img src="/dist/img/default-user.png" class="img-circle" alt="User Image">
                            <p>
                                <b>회원가입 또는 로그인해주세요</b>
                                <small></small>
                            </p>
                        </li>
                        <li class="user-footer">
                            <div class="pull-left">
                                <a href="${path}/user/register" class="btn btn-default btn-flat"><i
                                        class="fa fa-user-plus"></i><b> 회원가입</b></a>
                            </div>
                            <div class="pull-right">
                                <a href="${path}/user/login" class="btn btn-default btn-flat"><i
                                        class="glyphicon glyphicon-log-in"></i><b> 로그인</b></a>
                            </div>
                        </li>
                    </ul>
                </li>
            </c:if>
        </ul>
    </div>
</nav>
```

**`left_column.jsp`**

```html
<!-- Sidebar user panel (optional) -->
<div class="user-panel">
    <c:if test="${empty login}">
        <div class="pull-left image">
            <img src="${path}/user/default-user.png" class="img-circle" alt="User Image">
        </div>
        <div class="pull-left info">
            <p>Guest</p>
                <%-- Status --%>
            <a href="#"><i class="fa fa-circle text-danger"></i> OFFLINE</a>
        </div>
    </c:if>
    <c:if test="${not empty login}">
        <div class="pull-left image">
            <img src="/${login.userImg}" class="img-circle" alt="User Image">
        </div>
        <div class="pull-left info">
            <p>${login.userName}</p>
                <%-- Status --%>
            <a href="#"><i class="fa fa-circle text-success"></i> ONLINE</a>
        </div>
    </c:if>
</div>
```

#### 3.7 로그인 처리 확인

이제 로그인 처리가 제대로 작동하는지 확인해보자. 로그인 페이지로 이동한 뒤 아이디와 비밀번호를 입력하고, 로그인 버튼을 누른다.

![user_login_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_login_check.png?raw=true)

로그인이 제대로 처리되었다면 메인페이지(`/`)로 이동하게 되고, session에 `user`정보를 저장하여 아래와 같이 로그인한 회원의 정보가 보이게 된다.

![user_login_check2](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_login_check2.png?raw=true)

로그인 이전의 모습과 비교해보면 차이점을 알 수 있다.

![user_login_check3](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/user_login_check3.png?raw=true)

#### 3.8 권한 Interceptor 클래스 및 설정

이전에 작성한 `LoginInterceptor`는 로그인한 사용자에 대한 정보를 HttpSessin에 보관처리를 담당했는데, 지금 작성하는 인터셉터는 특정 경로에 접근할 경우
현재 사용자의 로그인 여부를 체크하는 역할을 수행한다.

`LoginInterceptor`와 마찬가지로 `기본패키지/commons/interceptor` 패키지에 `AuthInterceptor` 클래스를 생성하고 아래와 같이 코드를 작성한다.

```java
public class AuthInterceptor extends HandlerInterceptorAdapter {

    private static final Logger logger = LoggerFactory.getLogger(AuthInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        HttpSession httpSession = request.getSession();

        if (httpSession.getAttribute("login") == null) {
            logger.info("current user is not logged");
            response.sendRedirect("/user/login");
            return false;
        }

        return true;
    }
}
```

`preHandle()`메서드는 현재 사용자가 로그인한 상태여부를 확인하고, 컨트롤러를 호출할 것인지 아닌지를 결정한다. 그리고 만약 로그인하지 않은 사용자라면
로그인 페이지(`/user/login`)으로 리다이렉트하게 된다.


`AuthInterceptor`클래스를 인터셉터로 인식하기 위해 `dispatcher-servlet.xml`(`servlet-context.xml`)에서 아래와 같이 코드를 작성해준다. 게시물 입력, 수정, 삭제, 회원 정보 페이지 요청에는
권한 인터셉터가 작동하도록 아래와 같이 매핑을 해주었다.

```xml
<bean id="authInterceptor" class="com.doubles.mvcboard.commons.interceptor.AuthInterceptor"/>

<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/article/paging/search/write"/>
        <mvc:mapping path="/article/paging/search/modify"/>
        <mvc:mapping path="/article/paging/search/remove"/>
        <mvc:mapping path="/user/info"/>
        <ref bean="authInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 3.9 권한 Interceptor 처리 확인

![auth_interceptor_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/16_spring_mvc_board_httpsession_login/auth_interceptor_check.gif?raw=true)

위의 화면을 보면 로그인하지 않은 상태에서 게시글 쓰기 버튼을 누르거나, 게시글 쓰기 uri를 주소창에 쓰고 페이지로 이동하려고 하면 아래의 화면과 같이 로그인 화면으로 이동하게 된다.