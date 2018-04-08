
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
---

# Spring MVC - 인터셉터(Interceptor)

## 1. Filter와 Interceptor의 공통점과 차이점

#### # 공통점
Servlet기술의 Filter와 Spring의 HandlerInterceptor는 특정 URI에 접근을 제어하는 용도로 사용하는 점에서 공통점을 가지고 있다.

#### # 차이점
Filter와 Interceptor의 차이점은 실행 시점의 속하는 영역(Context)에 있다.
- Filter : 동일한 웹 애플리케이션의 영역 내에서 필요한 자원들을 활용
- Interceptor : 스프링 내의 모든 객체(빈)에 접근이 가능

HandlerInterceptor의 경우 스프링의 빈으로 등록된 컨트롤러나 서비스 객체들을 주입받아 사용할 수 있기 때문에 기존의 구조를 그대로 활용하면서 추가적인 작업이 가능하다.

## 2. AOP와 HandlerInterceptor의 차이점

#### # 파라미터의 차이
- AOP : Advice의 경우 JoinPoint나 ProceedingJoinPoint 등을 활용하여 호출 대상이 되는 메서드의 파라미터를 처리
- HandlerInterceptor : Filter와 유사하게 HttpServletRequest, HttpServletResponse를 파라미터로 받는 구조
