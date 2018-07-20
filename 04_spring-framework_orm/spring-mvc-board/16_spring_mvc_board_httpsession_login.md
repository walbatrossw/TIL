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

#### # HttpSession

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

#### # 회원 테이블 생성
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
#### # 회원 클래스 작성
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

#### # 회원 영속 계층 구현
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

#### # 회원 서비스 계층 구현

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

#### # 회원가입 컨트롤러 작성

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

#### # 회원가입 및 로그인 페이지 작성

`/views/user/` 디렉토리에 회원가입 페이지(`register.jsp`)를 아래와 같이 작성한다.

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

`/views/user/` 디렉토리에 회원가입 페이지(`login.jsp`)를 아래와 같이 작성한다.

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


## 3. 로그인 처리를 위한 준비 작업

#### # 회원 테이블 생성
로그인 처리를 위해 회원 테이블을 아래와 같이 생성한다.

```sql
-- 회원 테이블
CREATE TABLE tbl_user (
  user_id VARCHAR(50) NOT NULL,
  user_pw VARCHAR(100) NOT NULL,
  user_name VARCHAR(100) NOT NULL,
  user_point INT NOT NULL DEFAULT 0,
  session_key VARCHAR(50) NOT NULL DEFAULT 'none',
  session_limit TIMESTAMP,
  user_img VARCHAR(100) NOT NULL DEFAULT 'user/default-user.png',
  user_email VARCHAR(50) NOT NULL,
  user_join_date TIMESTAMP NOT NULL DEFAULT NOW(),
  user_login_date TIMESTAMP NOT NULL DEFAULT NOW(),
  user_signature VARCHAR(200) NOT NULL DEFAULT '안녕하세요 ^^',
  PRIMARY KEY (user_id)
);
```

```sql
-- 회원 추가
intsert into tbl_user (user_id, user_pw, user_name, user_email) values ('doubles', '1234', 'doubles', 'doubles@mail.com');
intsert into tbl_user (user_id, user_pw, user_name, user_email) values ('user01', '1234', 'user01', 'user01@mail.com');
intsert into tbl_user (user_id, user_pw, user_name, user_email) values ('user02', '1234', 'user02', 'user02@mail.com');
intsert into tbl_user (user_id, user_pw, user_name, user_email) values ('user03', '1234', 'user03', 'user03@mail.com');
```

#### # 회원 클래스 작성

#### # 회원 영속 계층 구현

#### # 회원 서비스 계층 구현

## 3. 로그인 컨트롤러 작성

## 4. 로그인 Interceptor

#### # 로그인 Interceptor 클래스 작성

#### # 로그인 Interceptor 설정

## 5. 로그인 화면

