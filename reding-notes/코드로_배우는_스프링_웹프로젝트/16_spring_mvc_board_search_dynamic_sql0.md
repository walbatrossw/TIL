
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
|6|[SpringMVC + MyBatis](http://doublesprogramming.tistory.com/176)|

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
|8|[페이징처리 개선, 목록페이지 정보 유지](http://doublesprogramming.tistory.com/198)|
---

# Spring MVC 게시판 예제 08 - 검색처리

이전 포스팅까지 게시글 목록을 페이징 처리를 완료했는데 이제 검색조건을 선택하고, 검색 키워드를 입력해 검색을 할 수 있는 기능을 추가해보자. 검색처리는 게시글의 목록 페이지에서 검색 조건의 항목인 제목, 내용, 작성자를 기준으로 아래와 같은 6가지의 경우에 따라 결과가 보여지게 구현할 것인데 이것을 구현하기 위해 MyBatis의 동적 SQL을 이용할 것이다.

- 제목
- 내용
- 작성자
- 제목 or 내용
- 내용 or 작성자
- 작성자 or 내용 or 작성자

그리고 추가적으로 목록 페이지의 정보를 유지하는 것처럼 검색 조건과 검색 키워드를 가지고 조회, 수정/삭제 페이지를 이동할 수 있도록 구현할 것이다.

## 1. 검색처리를 위한 데이터와 클래스
검색을 처리하고, 목록페이지를 출력하기 위해서는 아래와 같은 정보가 필요하다.

- 현재 페이지 번호
- 목록 페이지 당 출력할 게시글의 갯수
- 검색 조건의 종류
- 검색 키워드

위의 정보 중에서 현재 페이지 번호와 목록 페이지당 출력할 게시글의 갯수는 이미 `Criteria`클래스에서 사용할 수 있게 구현을 해놓았기 때문에 추가적으로 검색 조건의 종류와 검색 키워드를 추가해서 구현하면 된다. `Criteria`클래스에 필드를 추가하는 방식으로 구현할 수도 있지만, `Criteria`클래스를 상속받는 `SerachCriteria`라는 클래스를 만들어 사용하는 방식으로 구현해보겠다. 이렇게 구현하게 되면 기존의 코드의 변경을 최소화하면서 검색기능 추가할 수 있다.

`/src/main/java/기본패키지/commons/paging`패키지를 생성하고, 아래와 같이 클래스를 생성해 코드를 작성해준다.
```java
public class SearchCriteria extends Criteria {

    private String searchType;
    private String keyword;

    // getter(), setter(), toString() 생략...
}
```

## 2. 검색관련 컨트롤러 작성

## 3. 검색을 위한 JSP작성

## 4. MyBatis 동적 SQL

## 5. `ArticleService`와 `SearchArticleController` 수정

## 6. 조회, 수정, 삭제 페이지 처리

## 7. 등록 페이지 처리

## 8. 결과 확인
