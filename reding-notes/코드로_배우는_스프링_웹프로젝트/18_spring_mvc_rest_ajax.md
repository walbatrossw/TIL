
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
---

# Spring MVC - RestController, AJAX

## 1. REST(Representational State Transfer)란?

#### # 기본 개념
하나의 URI는 하나의 고유한 리소스를 대표하도록 설계된다는 개념이다. REST방식은 특정한 URI는 반드시 그에 상응하는 데이터 자체라는 것을 의미하는 방식이다. 예를 들어 `/article/120`은 120번 게시물이라는 고유한 의미를 가지도록 설계하고, 이에 대한 처리는 GET, POST 방식과 같이 추가적인 정보를 통해 결정하게된다.

#### # REST API
외부에서 위와 같은 방식으로 특정 URI를 통해 사용자가 원하는 정보를 제공하는 방식이다. 최근 OPEN API에서 많이 사용되면서 REST방식으로 제공되는 외부연결 URI를 REST API라고 하고, REST 방식의 서비스 제공이 가능것을 "Restful"하다고 표현한다.

#### # 스프링에서의 REST
스프링의 경우 3버전부터 `@ResponseBody`애너테이션을 지원하게되면서 REST방식의 처리를 지원하고 있으며, 스프링4부터는 `@RestController`를 통해 구현이 가능하게 되었다.

## 2. `@RestController` 애너테이션
#### # `@RestController`
스프링4부터 사용이 가능한 `@RestController`애너테이션은 기존의 특정한 JSP와 같은 뷰를 만들어 내는 것이 목적이 아니라 REST방식의 데이터를 처리하기 위해 사용하는 애너테이션이다. 이 때 반환되는 데이터로 사용되는 것은 단순문자열, JSON, XML이다.

#### # `@Controller`, `@ResponseBody`
스프링3에서는 컨트롤러는 `@Controller`애너테이션만을 사용해서 처리했기 때문에 화면을 처리를 담당하는 JSP가 아닌 데이터자체를 서비스하려면 해당 메서드나 리턴타입에 `@ResponseBody`애너테이션을 추가하는 형태로 작성하였다.

## 3. RestController 예제

`@RestController`를 이용해서 예제 연습을 해보자.

#### # RestController 작성법
기존에는 `@Controller`애너테이션을 붙여줬지만 `RestController`를 이용하기 위해서는 `@RestController`를 붙여주면 된다.
```java
@RestController
@RequestMapping("/ajax/test")
public class AjaxController {

}
```

#### # 단순 문자열을 리턴할 경우
```java
@RequestMapping("/hello")
public String sayHello() {

    return "Hello World";

}
```
해당 URI를 요청하면 브라우저에 아래와 같이 출력이 되는 것을 확인 할 수 있다.

#### # 객체를 JSON으로 리턴할 경우

먼저 예제에 사용될 간단한 객체를 아래와 같이 작성해준다.
```java
public class SampleVO {

    private Integer sampleNo;
    private String firstName;
    private String lastName;

    // getter, setter, toString() 생략
}
```

위에서 작성한 클래스의 객체를 반환하는 메서드를 `RestController`에 추가해준다.
```java
@RequestMapping("/sendVO")
public SampleVO sendVO() {

    SampleVO sample = new SampleVO();
    sample.setSampleNo(1);
    sample.setFirstName("더블");
    sample.setLastName("에스");

    return sample;
}
```

해당 URI를 요청하면 브라우저에 아래와 같이 출력이 되는 것을 확인 할 수 있다.
![]()

만약 406에러가 발생한다면 `pom.xml`에 `jackson-databind`라이브러리가 추가되어있는지 확인해본다.
```xml
<!--JSON : jackson-databind 라이브러리-->
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.4</version>
</dependency>
```

#### # 컬렉션 타입(`List`)의 객체를 리턴할 경우
```java
@RequestMapping("/sendList")
public List<SampleVO> sendList() {

    List<SampleVO> samples = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        SampleVO sample = new SampleVO();
        sample.setSampleNo(i);
        sample.setFirstName("더블");
        sample.setLastName("에스" + i);
        samples.add(sample);
    }

    return samples;
}
```
해당 URI를 요청하면 브라우저에 아래와 같이 출력이 되는 것을 확인 할 수 있다.
![]()

#### # 컬렉션 타입(`Map`)의 객체를 리턴할 경우
```java
@RequestMapping("/sendMap")
public Map<Integer, SampleVO> sendMap() {

    Map<Integer, SampleVO> sampleMap = new HashMap<>();
    for (int i = 0; i < 10; i++) {
        SampleVO sample = new SampleVO();
        sample.setFirstName("더블");
        sample.setLastName("에스"+i);
        sample.setSampleNo(i);
        sampleMap.put(i, sample);
    }

    return sampleMap;
}
```
해당 URI를 요청하면 브라우저에 아래와 같이 출력이 되는 것을 확인 할 수 있다.
![]()

## 4. `ResponseEntity` 타입

#### `ResponseEntity`이란?
[HTTP 상태코드 정리 포스팅 참고](http://doublesprogramming.tistory.com/203)
`@RestController`는 별도의 뷰를 제공하지 않는 형태로 서비스를 실행하기 때문에 리턴 데이터가 예외적인 상황에서 문제가 발생할 수가 있다. 웹의 경우 HTTP 상태코드가 이러한 정보를 나타내는데 사용된다.

스프링에서 제공하는 `ResponseEntity`타입은 개발자가 직접 결과 데이터와 HTTP상태코드를 직접 제어할 수 있는 클래스다. `ResponseEntity`를 사용하면 404나 500같은 에러를 전송하고 싶은 데이터와 함께 전송할 수 있기 때문에 좀더 세밀한 제어가 가능해진다.

#### `ResponseEntity` 예제
```java
```
