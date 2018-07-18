
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

---

# 스프링 MVC Interceptor

## 1. Filter와 Interceptor의 공통점과 차이점

#### # 공통점
Servlet 기술의 Filter와 스프링 MVC의 HandlerInterceptor는 특정 URI에 접근할 때 제어하는 용도로 사용된다

#### # 차이점
실행 시점에 속하는 영역(Context)에 차이점이 있다.

![spring-filter](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_interceptor/spring-filter.png?raw=true)

- Filter
  - 동일한 웹 어플리케이션의 영역 내에서 필요한 자원들을 활용
  - 웹 어플리케이션 내에서 동작하므로, 스프링 Context에 접근하기 어려움

![spring-interceptor](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_interceptor/spring-interceptor.png?raw=true)

- Interceptor
  - 스프링에서 관리되기 때문에 스프링 내의 모든 객체에 접근 가능
  - 스프링 Context 내에서 존재하므로 Context 내의 모든 객체를 활용 가능

HandlerInterceptor의 경우 스프링 빈으로 등록된 컨트롤러나 서비스 객체를 주입받아 사용할 수 있기 때문에 기존의 구조를 그대로 활용하면서 추가적인 작업이 가능하다.


## 2. Spring AOP와 HandlerInterceptor의 차이

특정 객체 동작 전에 또는 후에 처리는 이미 AOP를 통해 확인해보았지만, 일반적인으로 컨트롤러를 이용할 때에는 AOP의 `BeforeAdvice`등을 활용하기보다는 `HandlerInterceptor` 인터페이스 또는 `HandlerInterceptorAdaptor` 클래스를 활용하는 경우가 더 많다.

- AOP의 `Adivice` : `JoninPoint`나 `ProceedingJoinPoint` 등을 활용해서 호출 대상이 되는 메서드의 파라미터를 처리하는 방식
- Interceptor의 `HandlerInterceptor` : Filter와 유사하게 `HttpServletRequest`, `HttpServletResponse`를 파라미터로 받는 구조

일반적인 경우라면 Controller에서 DTO나 VO 타입을 주로 파라미터로 활용하고, Servlet API에 해당하는 `HttpServletRequest`와 `HttpServletResponse`를 활용하는 경우는 그다지 많지 않다.

HandlerInterceptor는 기존의 컨트롤러에서는 순수하게 필요한 파라미터와 결과 데이터를 만들어내고, 인터셉터를 이용해 웹과 관련된 처리를 도와주는 역할을 한다. HandlerInterceptor의 메서드는 아래와 같이 사용된다.

- `preHandle(request, response, handler)` : 지정된 컨트롤러의 동작 이전에 가로채는 역할로 사용
- `postHandle(request, response, handler, modelAndView)` : 지정된 컨트롤러의 동작 이후에 처리, Spring MVC의 FrontController인 DispatcherServlet이 화면을 처리하기 전에 동작
- `afterCompletion(request, response, handler, exception)` : DispatcherServletdml 화면처리가 완료된 상태에서 처리

`HandlerInterceptorAdaptor`는 `HandlerInterceptor`인터페이스를 구현한 추상 클래스로 설계되어 있는데 일반적으로 디자인패턴에서 Adaptor라는 용어가 붙게 되면 특정 인터페이스를 미리 구현해서 사용하기 쉽게 하는 용도인 경우가 많다. `HandlerInterceptorAdaptor`도 `HandlerInterceptor`를 쉽게 사용하기 위해서 인터페이스의 메서드를 미리 구현한 클래스이다.

## 3. Interceptor 예제

프로젝트에 Interceptor를 적용하기

#### # Interceptor 클래스 작성

예제를 위해 아래와 같이 `/tutorial/interceptor`패키지에 `SampleInterceptor` 클래스를 생성하고, `HandlerInterceptorAdapter`를 상속한다.

```java
public class SampleInterceptor extends HandlerInterceptorAdapter {
    
}
```

#### # Interceptor 설정

위에서 작성한 `SampleInterceptor`를 스프링에서 인식하기 위해 `dispatcher-servlet.xml`(`servlet-context.xml`)에 아래와 같이 설정을 해준다.
그리고 인터셉터를 사용할 요청 uri를 작성해주면된다.

```xml
<bean id="sampleInterceptor" class="com.doubles.mvcboard.tutorial.interceptor.SampleInterceptor"/>

<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/interceptor/doA"/>
        <mvc:mapping path="/interceptor/doB"/>
        <ref bean="sampleInterceptor"/>
    </mvc:interceptor>      
</mvc:interceptors>
```

#### # Interceptor를 위한 Controller, JSP 작성

간단한 테스트를 위해 `/tutorial/controller`패키지에 아래와 같이 Controller를 생성하고 코드를 작성해준다.

```java
@Controller
@RequestMapping("/interceptor")
public class InterceptorController {

    private static final Logger logger = LoggerFactory.getLogger(InterceptorController.class);

    @RequestMapping(value = "/doA", method = RequestMethod.GET)
    public String doA() {
        logger.info("doA() called");

        return "/tutorial/interceptor_test";
    }

    @RequestMapping(value = "/doB", method = RequestMethod.GET)
    public String doB(Model model) {
        logger.info("doB() called");
        model.addAttribute("result", "doB result data...");

        return "/tutorial/interceptor_test";
    }

}
```

`/views/tutorial`디렉토리에 `interceptor.jsp`를 생성하고 결과 페이지를 아래와 같이 작성해준다.
```html
<div class="col-lg-12">
    <div class="box box-primary">
        <div class="box-header with-border">
            <h3 class="box-title">인터셉터 결과 데이터</h3>
        </div>
        <div class="box-body">
            <a href="${path}/interceptor/doB">doB 페이지로 이동</a>
        </div>
        <div class="box-footer">
            <p>결과데이터 : ${result}</p>
        </div>
    </div>
</div>
```

#### # `SampleInterceptor` 코드 작성

`SampleInterceptor`에서 `preHandle()`과 `postHandle()`을 이용해서 콘솔 출력 코드를 아래와 같이 작성해준다.

```java
public class SampleInterceptor extends HandlerInterceptorAdapter {
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        
        System.out.println("post handle....");
        
   }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        System.out.println("pre handle....");

        return true;
    }
}
```

#### # 실행 및 결과

`/interceptor/doA` 경로로 이동하면 아래와 같이 콘솔창에 로그가 찍히는 것을 확인할 수 있다.

```
pre handle....
INFO : com.doubles.mvcboard.tutorial.controller.InterceptorController - doA() called
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Controller : com.doubles.mvcboard.tutorial.controller.InterceptorController.doA()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : []
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Return Value : /tutorial/interceptor_test
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 5
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
post handle....
```

## 4. Interceptor의 request, response 활용

인터셉터를 좀더 적극적으로 활용하면 컨트롤러가 웹에서 필요한 자원을 처리할 때 겪는 불편을 해소하는데 도움이 된다.

#### # `preHandle()`의 `Object` 파라미터

`preHandle()` 메서드의 경우 다음과 같은 3개의 파라미터를 사용한다.

- `HttpServletRequest request`
- `HttpServletResponse response`
- `Object handler`

마지막 `handler`는 현재 실행하려는 메서드 자체를 의미하는데, 이를 활용해 현재 실행되는 컨트롤러를 파악하거나 추가적인 메서드를 실행하는 등의 작업이 가능하다.

아래의 코드는 이전에 작성한 예제의 `SampleInterceptor`의 `preHandle()`을 이용하여 현재 실행되는 컨트롤러와 메서드 정보를 파악할 수 있게 작성했다.

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

    System.out.println("pre handle....");

    HandlerMethod method = (HandlerMethod) handler;
    Method methodObj = method.getMethod();

    System.out.println("Bean : " + method.getBean());
    System.out.println("Method : " + methodObj);

    return true;
}
```

파라미터로 넘어온 `handler`는 `HandlerMethod`타입으로 형변환한 후 원래 메서드와 객체를 확인 할 수 있다. 아래는 실행 결과 콘솔창에 출력된 결과이다.

```
pre handle....
Bean : com.doubles.mvcboard.tutorial.controller.InterceptorController@6ef34a13  
Method : public java.lang.String com.doubles.mvcboard.tutorial.controller.InterceptorController.doA()  
INFO : com.doubles.mvcboard.tutorial.controller.InterceptorController - doA() called
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Controller : com.doubles.mvcboard.tutorial.controller.InterceptorController.doA()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : []
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Return Value : /tutorial/interceptor_test
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 1
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
post handle....
pre handle....
Bean : com.doubles.mvcboard.tutorial.controller.InterceptorController@6ef34a13  
Method : public java.lang.String com.doubles.mvcboard.tutorial.controller.InterceptorController.doB(org.springframework.ui.Model)
INFO : com.doubles.mvcboard.tutorial.controller.InterceptorController - doB() called
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Controller : com.doubles.mvcboard.tutorial.controller.InterceptorController.doB()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [{result=doB result data...}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Return Value : /tutorial/interceptor_test
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 1
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
post handle....
```

#### # `postHandle()`을 이용해 추가적인 작업 수행하기

`postHandle()` 메서드의 경우 컨트롤러 메서드의 실행이 끝나고, 아직 화면처리가 되지 않은 상태이므로 필요하다면 메서드 실행 이후에 추가적인 작업이 가능하다.

예를 들어 특정 메서드의 실행결과를 `HttpSession` 객체에 같이 담아야할 경우, 컨트롤러에서는 `Model`객체에 결과 데이터를 저장하고, 인터셉터의 `postHandle()`에서
이를 이용해 `HttpSession`에 결과를 담는다면 컨트롤러에서 `HttpSession`을 따로 처리할 필요가 없게 된다.

아래의 코드는 컨트롤러에서 `result`라는 변수가 저장되었다면 `HttpSession` 객체에 이를 보관하는 예이다.

```java
@Override
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    System.out.println("post handle....");

    Object result = modelAndView.getModel().get("result");

    if (result != null) {
        request.getSession().setAttribute("result", result);
        response.sendRedirect("/interceptor/doA");
    }
}
```

컨트롤러의 코드에서 `/doB`를 호출하면, `result`라는 이름으로 하나의 문자열이 보관된다. 이후에 `postHandle()` 메서드에서 `result`라는 변수가 `ModelAndView`에 존재하면
이를 추출해 `HttpSession`에 추가한다. 그리고 최종적으로 `/doA`페이지로 리다이렉트한다.

실제로 화면에서 `/doB`를 호출하면 `HttpSession`에 보관된 `result`변수에 저장한 객체를 가지고 `/doA`로 이동하게 된다.

![interceptor_ex1](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_interceptor/interceptor_ex1.png?raw=true)

![interceptor_ex2](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-basic/img/spring_mvc_interceptor/interceptor_ex2.png?raw=true)

첫번째 사진은 `/doA`에서 `/doB`를 요청하기 전이고, 두번째 사진은 `/doB` 요청을 처리하고 리다이렉트되어 `/doA`로 이동한 모습이다. 위에서 설명한 것처럼 리다이렉트 되기 전에
인터셉터에서 `result`변수에 담긴 문자열을 `HttpSession`에 보관하여 `/doA`로 이동하게 되고 화면에 결과 데이터가 보여지게 된다.


