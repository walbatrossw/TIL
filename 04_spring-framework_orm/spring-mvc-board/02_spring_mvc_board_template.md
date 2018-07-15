

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

---

## 1. Bootstrap 템플릿 다운로드 및 세팅

스프링 프로젝트를 생성하고 기본적인 세팅을 마무리 했으니 Bootstrap 템플릿을 이용해서 화면을 구성해보자. 이전에 포스팅한 게시판 예제에서는 화면 전체의 레이아웃이나 CSS를 직접 구성했었는데 Bootstrap으로 만들어진 템플릿을 가져다 사용하면 좀더 깔끔하고 좋은 화면을 구성할 수 있다. Bootstrap 템플릿은 구글링을 통해서 무료로 얻을 수 있다. 그 중에서도 [AdminLTE](https://adminlte.io/themes/AdminLTE/index2.html)을 이용해 게시판 예제를 완성해 보려고 한다. 원래 이 템플릿은 관리자 페이지를 위해 만들어진 것인데 게시판 예제에 이 템플릿을 선택한 이유는 현재 보고 있는 책의 예제에도 이 템플릿을 추천하기도 했고, 개인 프로젝트에서도 이 템플릿을 사용해봤기 때문이다. 다양한 js플러그인(fullcalendar, ChartJS, CK Editor, DataTables, 등등)을 제공해주기 때문에 따로 검색해서 적용할 필요 없이 템플릿의 문서를 보고 잘 쓰기만 하면 된다. 물론 여기서 제공하는 플러그인이 없을 경우는 직접 적용해야하기도 하지만 일단 기본적인 플러그인들이 함께 첨부되있어 사용하기에 편리하기 때문에 이 템플릿을 추천한다.

AdminLTE 다운로드 페이지(https://adminlte.io/docs/2.4/installation)에서 템플릿 다운로드한 뒤에 압축해제 후 `bower_component`, `dist`, `plugins` 폴더를 복사 프로젝트의 `/webapp/resources` 디렉토리에 붙여넣는다.

![template_folder1](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/03_spring_mvc_board_template/template_folder1.png?raw=true)

![template_folder2](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/03_spring_mvc_board_template/template_folder2.png?raw=true)

views디렉토리에 `home.jsp`를 생성하고, 압축 해제한 템플릿 폴더의 `starter.html`파일 전체 코드를 복사한 뒤 붙여넣는다. 그리고 `dispathcer-servlet.xml`에 정적자원 디렉토리를 등록해준다.

```xml
<resources mapping="/bower_components/**" location="/resources/bower_components/"/>
<resources mapping="/plugins/**" location="/resources/plugins/"/>
<resources mapping="/dist/**" location="/resources/dist/"/>
```

`HomeController`를 아래와 같이 작성한 뒤 tomcat서버를 구동시킨다.

```java
@Controller
public class HomeController {

    @RequestMapping("/")
    public String home() {
        return "home";
    }

}
```

크롬 브라우저 개발자도구의 Network탭에서 정적자원 요청이 제대로 이루어지는지 확인한다.

![network_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/03_spring_mvc_board_template/network_check.png?raw=true)

## 2. Include 처리
Include 관련 포스팅 : [JSP include(지시자와 액션태그)](http://doublesprogramming.tistory.com/64)

만약 현재 `home.jsp`처럼 많은 코드를 반복적으로 모든 페이지마다 작성한다면 수정할 코드가 있을 때마다 모든 페이지를 동일하게 수정해 줘야하는 번거로움이 있다. 그래서 `views`디렉토리 하위에 `include`디렉토리를 생성하여 각각 부분별로 `jsp`파일을 만들어주고 include해주는게 좋다. `home.jsp`페이지의 내용을 5개 부분으로 나누어 `jsp`파일을 생성해 `home.jsp`에 include 해주자.

![includes](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/03_spring_mvc_board_template/includes.png?raw=true)
- `head.jsp` : `meta`태그, `css`태그를 작성해줄 곳
- `main_header.jsp` : 페이지의 main header에 해당하는 부분
- `left_column.jsp` : 페이지의 left side column에 해당하는 부분
- `main_footer.jsp` : 페이지의 main footer에 해당하는 부분
- `plugin_js.jsp` : js플러그인 태그를 작성해줄 곳

아래 사진은 브라우저 상에서 보이는 부분을 나누어 본 것으로 그외에 `head.jsp`, `plugin_js.jsp`는 css와 js태그를 추가할 용도로 사용한다.

![includes2](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/03_spring_mvc_board_template/includes2.png?raw=true)

아래 사진의 코드는 내용이 상당히 길기 때문에 collaspe한 상태이다. include할 내용을 복사해서 해당하는 `jsp`파일에 붙여 넣어주고 include를 해주면된다. 왼쪽의 line number를 보더라도 310에서 52로 줄어든 것을 볼 수 있다. 이제 페이지 본문내용의 코드는 `<div class="content-wrapper">`에 작성해주기만 하면된다.

include 처리하기 전 `home.jsp`

![before_include](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/03_spring_mvc_board_template/before_include.png?raw=true)

include 처리하기 후 `home.jsp`

![after_include](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/03_spring_mvc_board_template/after_include.png?raw=true)

크롬 브라우저 개발자도구의 Network탭에서 정상적으로 화면이 나오는지 다시 확인해보면 이상없이 잘 나오는것을 확인해볼 수 있다.
![include_network_check](https://github.com/walbatrossw/TIL/blob/master/04_spring-framework_orm/spring-mvc-board/img/03_spring_mvc_board_template/include_network_check.png?raw=true)


## 3. 마무리
지금까지 프로젝트를 생성하고, Bootstrap 템플릿을 적용해보았는데 다음 포스팅에서는 본격적으로 기본적인 게시물의 입력, 수정, 삭제, 조회, 목록 기능을 구현해볼 예정이다.
