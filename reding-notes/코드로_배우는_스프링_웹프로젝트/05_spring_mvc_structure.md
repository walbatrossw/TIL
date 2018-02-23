# Spring MVC 구조

본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.

그리고 본 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정 파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](http://doublesprogramming.tistory.com/171?category=667155) 포스팅을 참고해주시면 감사하겠습니다.

포스팅하고 있는 현재 프로젝트의 예제가 혹시 필요하신 분은 github(https://github.com/walbatrossw/spring-mvc-ex)를 통해 얻으실 수 있습니다.

---

## 1. Model 2
Model 2 방식은 흔히 MVC구조를 응용한 방식으로 **화면과 데이터 처리를 분리해서 재사용이 가능하도록 하는 구조** 라고 할 수 있다.

- **Model** : 데이터 혹은 데이터를 처리하는 영역을 의미
- **View** : 결과 화면을 만들어 내는 데 사용하는 자원을 의미
- **Controller** : 웹의 요청을 처리하는 존재로 View와 Model사이의 중간 통신역할을 수행

개발자와 웹퍼블리셔의 영역을 분리할 수 있고, Controller의 URI를 통해 View를 제어하기 때문에 View의 교체나 변경과 같은 유지보수 사용된다.

## 2. Front Controller 패턴
**전체 로직의 일부만을 Controller가 처리하도록 변경** 되었다. 이것을 **위임**(**Delegation**)이라고 하는데 **전체 로직의 일부를 Controller에게 위임하고, 모든 흐름의 제어는 앞 쪽의 Front Controller가 담당** 하게 된다. 이러한 구조를 사용하게 될 경우 개발자가 작성하는 Controller는 전체 로직의 일부분만을 처리하는 형태가 되기 때문에 개발자가 작성해야하는 전체 코드는 줄어들게 된다.

## 3. Spirng MVC 구조
#### # Spring MVC의 요청에 대한 흐름
![spring_stream](http://cfile2.uf.tistory.com/image/99CC3D335A1934810C9AC4)
1. 사용자의 요청을 Front Controller에 전달
2. 전달된 요청은 적절한 Controller를 찾아 호출
3. Controller는 해당 Service 객체를 찾아 호출
4. Service객체는 DAO객체를 이용하여 원하는 Data를 요청
5. DAO객체는 MyBatis를 이용하는 Mapper를 통해 작업을 수행(CRUD)
6. Service를 통해 처리한 데이터를 Controller에게 전달
7. Controller는 다시 스프링 MVC쪽으로 데이터를 전달

#### # SpringMVC VS, 개발자
|SpringMVC가 처리하는 작업|개발자가 직접 해야하는 작업|
|---|---|
|URI를 분석해서 적절한 Controller를 찾는 작업|특정URI에 동작하는 Controller를 설계하는 작업|
|Controller에 필요한 메서드를 호출하는 작업|Service / DAO 객체 생성|
|Controller의 결과 데이터를 View로 전달하는 작업|Controller 내에 원하는 결과를 메서드로 설계|
|적절한 View를 찾는 작업|View에서 전달받은 데이터의 출력|

## 4. Spring MVC 주요 애너테이션
|애너테이션|기능|
|---|---|
|`@Controller`|Controller 객체임을 명시|
|`@Repository`|DAO 객채임을 명시|
| `@Service` | Service 객체임을 명시|
| `@RequestMapping` | 특정 URI에 매칭되는 클래스나, 메서드임을 명시|
| `@RequestParam` | 요청에서 특정한 파라미터 값을 찾아낼 때 사용|
| `@RequestHeader` | 요청에서 특정 HTTP 헤더 정보를 추출할 때 사용|
| `@PathVariable` | 현재의 URI에서 원하는 정보를 추출할 때 사용|
| `@CookieValue` | 현재 사용자의 쿠키가 존재하는 경우 쿠키의 이름을 이용해서 쿠키 값을 추출|
| `@ModelAttribute` | 자동으로 해당 객체를 뷰까지 전달하도록 만드는 애너테이션|
| `@SessionAttribute`|세션상에서 모델의 정보를 유지하고 싶은 경우 사용|
| `@InitBinder`|파라미터를 수집해서 객체로 만들 경우|
| `@ResponseBody`|리턴타입이 HTTP의 응답메시지로 전송|
| `@RequestBody`| 요청 문자열이 그대로 파라미터로 전달|
