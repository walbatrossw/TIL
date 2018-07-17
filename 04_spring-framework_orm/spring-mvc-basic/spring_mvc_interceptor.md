# 스프링 MVC Interceptor

## 1. Filter와 Interceptor의 공통점과 차이점

### 공통점
Servlet 기술의 Filter와 스프링 MVC의 HandlerInterceptor는 특정 URI에 접근할 때 제어하는 용도로 사용된다

### 차이점
실행 시점에 속하는 영역(Context)에 차이점이 있다.

![spring-filter]()

- Filter
  - 동일한 웹 어플리케이션의 영역 내에서 필요한 자원들을 활용
  - 웹 어플리케이션 내에서 동작하므로, 스프링 Context에 접근하기 어려움

![spring-interceptor]()

- Interceptor
  - 스프링에서 관리되기 때문에 스프링 내의 모든 객체에 접근 가능
  - 스프링 Context 내에서 존재하므로 Context 내의 모든 객체를 활용 가능

HandlerInterceptor의 경우 스프링 빈으로 등록된 컨트롤러나 서비스 객체를 주입받아 사용할 수 있기 때문에 기존의 구조를 그대로 활용하면서 추가적인 작업이 가능하다.


## 2. Spring AOP와 HandlerInterceptor의 차이
