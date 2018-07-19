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


## 2. 로그인 처리를 위한 준비 작업

#### # 회원 테이블 생성

#### # 회원 클래스 작성

#### # 회원 영속 계층 구현

#### # 회원 서비스 계층 구현

## 3. 로그인 컨트롤러 작성

## 4. 로그인 Interceptor

#### # 로그인 Interceptor 클래스 작성

#### # 로그인 Interceptor 설정

## 5. 로그인 화면

