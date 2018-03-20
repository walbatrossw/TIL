
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

# Spring MVC 게시판 예제15 - 파일 업로드 구현 1 : 일반적인 방식, AJAX 방식 테스트

게시판에 파일 업로드를 기능을 추가하기 전에 일반적인 방식의 업로드와 AJAX방식의 업로드 구현을 연습해보자.

## 1. 파일 업로드를 위한 준비

우선 파일 업로드를 처리하기 위해서는 몇가지 설정이 필요하다. 가장 먼저 아래와 같이 파일 업로드관련 라이브러리를 추가해주고, 파일업로드 `MultipartResolver`설정을 해준다.

#### # 파일업로드 관련 라이브러리 추가 : `pom.xml`
`imgscalr`라이브러리는 큰 이미지 파일을 고정된 크기로 변환할 때 사용한다.
```xml
<!--파일 업로드 라이브러리-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.2</version>
</dependency>

<!--이미지 축소 라이브러리-->
<dependency>
    <groupId>org.imgscalr</groupId>
    <artifactId>imgscalr-lib</artifactId>
    <version>4.2</version>
</dependency>
```

#### # 파일업로드 Bean 설정 : `dispatcher-servlet.xml`
스프링에서 파일업로드를 처리하기 위해서는 파일 업로드로 들어오는 데이터를 처리하는 객체가 필요하다. 스프링에서 `multipartResolver`라고 하는 이 객체의 설정은 웹과 관련 있기 때문에 `dispatcher-servlet.xml`에서 설정해준다.
```xml
<!--파일 업로드 MultipartResolver 설정-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="10485760"/>
</bean>
```
`property`중에서 `maxUploadSize`를 통해 최대 크기를 10MB정도로 제한해준다.

#### # 한글파일명 필터 설정 : `web.xml`
한글파일명을 업로드했을 때 파일명이 깨져서 나오는 것을 방지하기 위해 아래와 같이 필터 설정이 되어 있는지 확인한다.
```xml
<!--한글 인코딩 설정-->
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 2. 일반적인 방식의 파일 업로드 처리
일반적인 방식의 파일 업로드는 파일을 선택하는 `<input type="file">`태그를 통해 원하는 파일을 선택해서 업로드하는 형태이다.

#### # 파일 업로드 페이지
가장 먼저 파일 업로드 페이지를 작성해보자.
일반적인 파일 업로드 테스트 페이지는 `WEB-INF/view/tutorial` 디렉토리에 `upload_form.jsp`를 생성해서 사용한다. 기존의 프로젝트에서 쓰이는 템플릿의 양식의 내용을 그대로 사용한다.

**Jasny Bootstrap Template 적용**
AdminLTE에서는 파일 업로드 템플릿이 없기 때문에 파일 업로드 부트스트랩 템플릿을 찾아서 적용했다. 그리고 이후에 회원 프로필 사진 업로드 기능 구현에도 쓰이게 된다.

http://www.jasny.net/bootstrap/getting-started/#download
템플릿 적용은 간단하다. 위의 링크에 들어가 템플릿파일을 다운받아 js, css파일을 프로젝트에 넣어주고, include파일(`head.jsp`, `plugin_js.jsp`)에 적용시켜주기만 하면된다.

http://www.jasny.net/bootstrap/javascript/#fileinput
그리고 나서 위의 페이지에서 파일 업로드 예제 중에서 하나를 참고해 아래와 같이 코드를 작성해주면 된다.

```html
<div class="col-lg-12">
    <form role="form" id="uploadForm" method="post" action="${path}/file/form/upload" enctype="multipart/form-data" target="imgFrame">
        <div class="box box-primary">
            <div class="box-header with-border">
                <h3 class="box-title">파일 업로드(일반) 입력폼</h3>
            </div>
            <!-- /.box-header -->
            <div class="box-body">

                <div class="fileinput fileinput-new input-group" data-provides="fileinput">
                    <div class="form-control" data-trigger="fileinput">
                        <i class="glyphicon glyphicon-file fileinput-exists"></i>
                        <span class="fileinput-filename"></span>
                    </div>
                    <span class="input-group-addon btn btn-default btn-file">
                        <span class="fileinput-new">파일 선택</span>
                        <span class="fileinput-exists">변경</span>
                        <input type="file" name="file"></span>
                    <a href="#" class="input-group-addon btn btn-default fileinput-exists" data-dismiss="fileinput">삭제</a>
                </div>

                <iframe name="imgFrame"></iframe>

            </div>
            <!-- /.box-body -->
            <div class="box-footer">
                <div class="form-group pull-right">
                    <button type="submit" class="btn btn-primary"><i class="fa fa-save"></i> 저장</button>
                </div>
            </div>
            <!-- /.box-footer -->
        </div>
    </form>
</div>
```
위 코드에서 가장 중요한 부분은 `<form>`태그의 속성 중에 `enctype`을 `multipart/form-data`으로 설정한 것이다. 이 설정이 없을 경우 파일 데이터가 전달되지 않으니 반드시 써줘야한다. 그리고 업로드 처리 결과 알림에 `<iframe>`을 사용하기 위해서 `<form>`태그의 `target`속성을 `<iframe>`의 `name`속성에 맞춰준다.

아래는 파일 입력 폼을 적용한 모습이다.
![fileInputForm](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-17%2016-41-34.png?raw=true)

#### # 일반적인 파일 업로드 컨트롤러
입력페이지가 준비되었으니 파일 업로드 컨트롤러를 작성해보자. 일반적인 파일 업로드에서는 파일을 파일을 다운하거나 삭제/수정처리 작업은 제외하고 단순히 파일을 업로드하는 것만 구현해준다. 이어서 나오는 AJAX방식의 파일업로드 처리에서 파일 다운, 삭제/수정처리를 정리한다.

```java
@Controller
@RequestMapping("/file/form")
public class UploadFormController {

    private static final Logger logger = LoggerFactory.getLogger(UploadFormController.class);

    @RequestMapping(value = "/uploadPage", method = RequestMethod.GET)
    public String uploadPage() {

        return "tutorial/upload_form";
    }

    @RequestMapping(value = "/upload", method = RequestMethod.POST)
    public String uploadFile(MultipartFile file, Model model, HttpServletRequest request) throws Exception {

        logger.info("==================== FILE UPLOAD ===========================");
        logger.info("original file name : " + file.getOriginalFilename());
        logger.info("file size : " + file.getSize());
        logger.info("contentType : " + file.getContentType());
        logger.info("============================================================");

        // 업로드 디렉토리 추출
        String uploadPath = getRealUploadPath(request);

        // 파일업로드 메서드 호출, 리턴값 : 업로드된 파일명
        String savedFileName = uploadFile(uploadPath, file.getOriginalFilename(), file.getBytes());

        // 업로드된 파일명
        model.addAttribute("savedFileName", savedFileName);

        // 결과페이지
        return "tutorial/upload_result";

    }

    // 업로드 디렉토리
    private String getRealUploadPath(HttpServletRequest request) {
        return request.getSession().getServletContext().getRealPath("/resources/upload/");
    }

    // 파일 업로드 처리
    private String uploadFile(String uploadPath,
                              String originalFileName,
                              byte[] fileData) throws Exception {
        // 중복파일 저장을 방지하기 위해
        UUID uuid = UUID.randomUUID();
        String savedFileName = uuid.toString() + "_" + originalFileName;

        // 파일 객체 생성
        File target = new File(uploadPath, savedFileName);

        // 파일 데이터를 복사
        // 데이터가 담긴 바이트 배열을 파일객체에 기록
        FileCopyUtils.copy(fileData, target);

        return savedFileName;
    }

}
```

위의 코드에서 중요한 부분을 정리해보면 다음과 같다.
- `uploadFile()`메서드 : 파라미터에 `MultipartFile`타입이 선언되어 있는데 `MultipartFile`은 POST방식으로 들어온 파일 데이터를 의미한다.
- `MultipartFile` : 객체를 이용하면 파일의 이름(`getOriginalFilename()`), 크기(`getSize()`), 타입(`getContentType()`)등과 같은 정보를 얻을 수 있다.
- `getRealUploadPath()`메서드 : 파일이 업로드 되는 상대경로를 추출해준다.
- `uploadFile()`메서드 : 파일을 업로드 처리하고, 최종적으로 저장된 파일명을 리턴해준다.

#### # `<iframe>`을 이용한 파일업로드 결과 처리
앞서 잠깐 언급한 것처럼 현재 페이지에서 파일 업로드가 제대로 되었는지 확인하기 위해 `<iframe>`을 사용해서 처리한다. `<form>`태그는 브라우저의 창에서 전송이 일어나기 때문에 화면 전환이 이루어지게 된다. `<form>`태그에 `target`속성을 주고, `<iframe>`을 이용하면 화면전환을 피하고, 파일이 업로드가 처리 되었는지 확인할 수 있다.

`<ifrmae>`을 이용한 파일 업로드는 아래와 같은 과정을 거치게 된다.

1. `<form>`태그 전송은 입력페이지(`upload_form.jsp`)에 포함된 `<iframe>`으로 전송한다.
2. 결과 페이지(`upload_result.jsp`)는 입력페이지(`upload_form.jsp`)의 `<iframe>`내에 포함되므로 실제로 화면 변화가 없다.
3. 결과 페이지에서 다시 바깥 페이지(`upload_form.jsp`)의 JS함수를 호출하고, alert창을 띄운다.

**결과 페이지 : `<iframe>`페이지(`upload_result.jsp`) 작성**
`upload_result.jsp`는 별내용 없이 아래와 같이 간단한 내용만 추가하면된다.
```js
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page session="false" %>

<script>
    // 업로드된 파일명
    var result = "${savedFileName}";
    // 부모창의 함수 호출
    parent.addFilePath(result);
</script>
```

**입력페이지 : `upload_form.jsp`수정**
아래와 같이 `iframe`에 대한 스타일 속성 모두를 0px로 설정해준다. 이렇게 하면 iframe은 입력화면에 보이지 않게 되어  AJAX처리와 같은 효과를 얻을 수 있다.
```css
iframe {
    width: 0px;
    height: 0px;
    border: 0px;
}
```
결과 페이지(`upload_result.jsp`)에서 호출할 함수를 선언해준다. iframe에서 이 함수를 호출하면 파일명을 alert창을 통해 출력해준다.
```js
function addFilePath(msg) {

    // 파일명
    alert(msg);

    // 파일 입력폼 초기화
    $("#uploadForm").each(function(){
        this.reset();
    });

}
```

#### # 파일 업로드 처리 확인
파일을 선택하고 난 후 모습이다.
![fileSelect](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-17%2017-49-06.png?raw=true)

파일 업로드 처리 후 파일명을 alert창을 통해 알려준다. `<iframe>`을 사용했기 때문에 페이지전환은 이루어지지 않는다.
![afterFileUpload](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-17%2020-38-15.png?raw=true)

alert창이 닫히고 나면 파일 입력폼을 초기화시켜준다.
![removeFileSelect](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-17%2020-38-35.png?raw=true)

실제로 파일이 해당 디렉토리에 업로드 되었는지 확인해보면 다음과 같이 정상적으로 업로드 된 것을 알 수 있다.
![saveDirectory](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-17%2020-41-55.png?raw=true)

## 3. AJAX 방식의 파일 업로드 처리

이번 예제에서는 AJAX를 이용해 드래그 앤 드랍 방식으로 파일 업로드 기능을 구현해보자. 그리고 업로드한 파일이 이미지이면 썸네일을 생성하여 작은 이미지를 화면에 출력하고, 일반파일이면 다운로드할 수 있도록 처리한다. 마지막으로 삭제버튼을 클릭하면 업로드한 파일이 삭제되도록 처리한다.

#### # 파일 업로드 구현

##### 1. AJAX 업로드 페이지
`WEB-INF/views/toturial/`디렉토리에 `upload_ajax.jsp`를 기존의 jsp파일을 복사해서 아래와 같이 css, html, js 코드를 작성해준다.
```css
/* 파일업로드 영역 */
.fileDrop {
    width: 100%;
    height: 200px;
    border: 2px dotted #0b58a2;
}

/* 이벤트를 통해 생성된 파일의 삭제버튼 */
small {
    margin-left: 3px;
    font-weight: bold;
    color: #0b58a2;
}
```
```html
<section class="content container-fluid">
    <div class="col-lg-12">
        <div class="box box-primary">
            <div class="box-header with-border">
                <h3 class="box-title">파일 업로드 (Ajax) 입력폼</h3>
            </div>
            <!-- /.box-header -->
            <div class="box-body">

                <div class="fileDrop">
                    <br/>
                    <br/>
                    <br/>
                    <br/>
                    <p class="text-center">파일을 드래그해주세요.</p>
                </div>
                <hr/>
                <div class="uploadedList"></div>
            </div>
            <!-- /.box-body -->
            <div class="box-footer">
                <div class="form-group pull-right">
                    <button type="submit" class="btn btn-primary"><i class="fa fa-save"></i> 저장</button>
                </div>
            </div>
            <!-- /.box-footer -->
        </div>
    </div>
</section>
```
```js
// 파일 드래그 앤 드랍 이벤트 처리
$(".fileDrop").on("drop dragenter dragover", function (event) {
    // 기본 이벤트 처리 방지
    event.preventDefault();
    // 이벤트와 같이 전달된 데이터들을 가져와 files에 저장
    var files = event.originalEvent.dataTransfer.files;
    // 단일 파일업로드이기 때문에 첫번째 파일을 file에 저장
    var file = files[0];
    // 콘솔에 출력
    console.log(file);

    // FormData객체를 통해 form태그처럼 파일데이터를 전송 : IE의 경우 버전이 10이상만 지원
    var formData = new FormData();
    // FormData에 데이터를 key, value의 형태로 저장
    formData.append("file", file);

    // $.ajax()를 이용해서 formData객체에 저장된 파일데이터를 전송하기 위해서는 processData, contentType의 속성을 false로 지정해야함
    // processData의 기본값은 true로 "application/x-www-form-urlencoded"타입으로 전송, 다른형식의 데이터를 전송하기 위해서는 false를 지정해줘야한다.
    // contentType의 기본값은 true로 "application/x-www-form-urlencoded"타입으로 전송, 파일의 경우 "multipart/form-data"방식으로 전송해야기 때문에 false로 지정해줘야 한다.
    $.ajax({
        url: "/file/ajax/upload",
        data: formData,
        dataType: "text",
        processData: false,
        contentType: false,
        type: "POST",
        success: function (data) {
            alert(data);
        }
    });
});
```

아래는 위의 코드를 작성하고 난 뒤의 모습이다.
![]()

##### 2. AJAX 업로드 컨트롤러 생성, 업로드 페이지 매핑/ 업로드 처리
AJAX방식으로 파일 업로드를 처리할 컨트롤러를 아래와 같이 작성해준다.

아래의 코드는 업로드 페이지로 매핑하기 위한 메서드이다. 일반적인 컨트롤러에서는 리턴타입을 `String`으로 하면 view페이지를 의미하지만, Rest컨트롤러의 경우 `String`리턴타입은 단순히 값을 의미한다. 그래서 Rest컨트롤러에서 파일업로드 페이지로 이동하기 위해 `ModelAndView`리턴타입을 선언해주고 `ViewName`을 지정해주었다.
```java
@RestController
@RequestMapping("/file/ajax")
public class UploadAjaxController {

    private static final Logger logger = LoggerFactory.getLogger(UploadAjaxController.class);

    // AJAX 파일업로드 페이지 매핑
    @RequestMapping(value = "/uploadPage", method = RequestMethod.GET)
    public ModelAndView uploadPage() {

        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("tutorial/upload_ajax");

        return modelAndView;
    }
}
```

이제 컨트롤러에 업로드 처리 메서드를 추가해보자. 컨트롤러까지 파일 데이터가 제대로 전달되었는지 로그를 찍어서 확인해보자. 아래의 코드에서 주목해서 봐야할 점은 `@RequestMapping`애너테이션의 `produces`속성이다. 업로드된 파일명이 한글일 경우 깨지지 않고 정상적으로 전달하기 위한 설정이다.
```java
// 파일 업로드 처리
@RequestMapping(value = "/upload", method = RequestMethod.POST, produces = "text/plain;charset=UTF-8")
public ResponseEntity<String> uploadFile(MultipartFile file, HttpServletRequest request) {

    logger.info("==================== FILE UPLOAD ===========================");
    logger.info("original file name : " + file.getOriginalFilename());
    logger.info("file size : " + file.getSize());
    logger.info("contentType : " + file.getContentType());
    logger.info("============================================================");

    return new ResponseEntity<>(file.getOriginalFilename(), HttpStatus.CREATED);
}
```

AJAX 파일업로드 페이지에서 한글이름의 파일을 업로드해보면 아래와 같이 파일이 컨트롤러까지 제대로 전송된 것을 확인해볼 수 있다.
```
INFO : com.doubles.mvcboard.tutorial.controller.UploadAjaxController - ==================== FILE UPLOAD ===========================
INFO : com.doubles.mvcboard.tutorial.controller.UploadAjaxController - original file name : 한글이름_사진파일.jpg
INFO : com.doubles.mvcboard.tutorial.controller.UploadAjaxController - file size : 1644685
INFO : com.doubles.mvcboard.tutorial.controller.UploadAjaxController - contentType : image/jpeg
INFO : com.doubles.mvcboard.tutorial.controller.UploadAjaxController - ============================================================
```

##### 3. 파일 업로드 처리 관련 클래스들 만들기 : `UploadFileUtils`, `MediaUtils`
앞서 일반적인 방식의 파일 업로드 처리를 구현할 때처럼 AJAX 업로드 컨트롤러에서 파일업로드 처리를 직접 하지 않았다. 그 이유는 파일업로드 처리시 주의해야할 점들이 많고, 게시글 작성 뿐만아니라 다른 기능을 추가할 때도 파일업로드 기능을 재사용할 수 있도록 파일업로드 클래스를 따로 만들어주기 위해서이다.

파일 업로드 처리 클래스를 만들기 전에 먼저 서버에 파일을 저장할 때 발생할 수 있는 문제점과 해결방법에 대해서 알아보자.

- **파일이름의 중복 문제** : UUID를 통해 고유값을 생성하고, 파일의 이름 앞에 붙여서 문제를 해결한다.
- **파일의 저장 경로 문제** : 특정 디렉토리에 너무나 많은 파일을 업로드할 경우 성능문제가 발생하기 때문에 날짜별로 구성해서 해결한다.
- **이미지 파일일 경우 화면에 출력할 때 보여지는 파일의 크기** : 썸네일이미지 파일을 생성해 화면에 출력해준다.

위에서 알아본 내용을 고려하여 파일 업로드 클래스를 설계한다면 파일업로드 시에 **자동적으로 날짜별 폴더가 생성** 되어야 하고, **파일명은 UUID를 통해 중복되지 않게 변경** 되어야 하며, **이미지 파일일 경우 썸네일 이미지를 생성** 해주도록 해야한다. 이를 위해`/src/java/기본패키지/commons/util`패키지에 `UploadFileUtils`(파일업로드 처리), `MediaUtils`(이미지 파일타입 판별)클래스를 아래와 같이 생성해준다.
```java
// 파일 업로드 처리 클래스
public class UploadFileUtils {
    // UUID를 이용하여 파일명 중복방지 처리
    // 파일 업로드 날짜별 폴더 생성
    // 파일 업로드 처리
    // 이미지 파일일 경우 썸네일이미지 파일 생성
}
```
```java
// 이미지 파일 타입 판별 클래스
public class MediaUtils {
    // 이미지 파일 타입 판별 처리
}
```

`UploadFileUtils`클래스는 스프링의 `FileCopyUtils`클래스와 유사하게 파일 업로드 메서드를 `static`메서드로 선언하여 인스턴스를 생성하지 않고 바로 호출하여 사용할 수 있도록 할 것이다. 그리고 `MediaUtils`클래스는 썸네일이미지를 생성하기 위해서 업로드된 파일이 이미지파일 여부를 판별할 수 있도록 구현할 예정이다.

이제 `UploadFileUtils`클래스에 파일 업로드 메서드를 작성해보자.
```java
// 파일 업로드 처리 메서드
public static String uploadFile(String originalFileName, byte[] fileData, HttpServletRequest request) throws Exception {

    // 저장파일명 : 중복 방지 처리
    String uuidFileName = getUuidFileName(originalFileName);

    // 파일 업로드 경로 설정
    String rootPath = getRootPath(originalFileName, request); // 기본경로 추출(이미지 or 일반파일)
    String datePath = getDatePath(rootPath); // 날짜 경로, 날짜 경로 생성처리

    // 서버에 파일 저장
    File target = new File(rootPath + datePath, uuidFileName); // 파일 객체 생성
    FileCopyUtils.copy(fileData, target); // 파일 객체에 파일 데이터 복사

    // 업로드 경로 + 파일명
    String uploadedFileName = null;

    // 이미지 파일일 경우 썸네일이미지 생성
    if (MediaUtils.getMediaType(getFormatName(originalFileName)) != null) {
        uploadedFileName = makeThumbnail(rootPath, datePath, uuidFileName);
    } else { // 일반 파일일 경우
        uploadedFileName = makeIcon(rootPath, datePath, uuidFileName);
    }

    return uploadedFileName;
}
```
위의 메서드는 아래와 같은 흐름으로 `UploadFileUtils`에 추가적으로 선언한 메서드들을 호출하면서 5번과 6번 사이에 파일 업로드를 처리하게 된다.

1. 업로드 파일의 중복방지 처리  
2. 파일의 종류(이미지파일 or 일반파일)인지 판별
3. 파일의 종류에 따라 기본 저장 경로(`/images` or `/files`)를 추출
4. 날짜별 경로 추출 : ex) `/2018/03/01`
5. 기본 경로에 날짜별 경로를 폴더를 생성 : ex) `/images/2018/03/01`, `/files/2018/03/01`
6. 이미지 파일이면 썸네일 이미지를 생성, 저장경로+파일명 리턴 or 일반 파일이면 저장경로+파일명만 리턴

위에서 설명한 파일 업로드 관련 메서드들은 아래와 같다.

```java
// 1. 중복파일명 방지 메서드
private static String getUuidFileName(String originalFileName) {
    return UUID.randomUUID().toString() + "_" + originalFileName;
}
```

```java
// 2. 파일 확장자 추출 메서드
private static String getFormatName(String fileName) {
    return fileName.substring(fileName.lastIndexOf(".") + 1);
}
```

```java
// 3. 파일업로드 기본경로 추출 메서드
public static String getRootPath(String fileName, HttpServletRequest request) {
    if (MediaUtils.getMediaType(getFormatName(fileName)) != null) {
        // 이미지 파일 경로
        return request.getSession().getServletContext().getRealPath("/resources/upload/images/");
    }
    // 일반파일 경로
    return request.getSession().getServletContext().getRealPath("/resources/upload/files/");
}
```

```java
// 4. 파일업로드 날짜별 경로 추출메서드
private static String getDatePath(String uploadPath) {

    Calendar calendar = Calendar.getInstance();
    String yearPath = File.separator + calendar.get(Calendar.YEAR); // 년도
    String monthPath = yearPath + File.separator + new DecimalFormat("00").format(calendar.get(Calendar.MONTH) + 1); // 월
    String datePath = monthPath + File.separator + new DecimalFormat("00").format(calendar.get(Calendar.DATE)); // 일자
    makeDir(uploadPath, yearPath, monthPath, datePath);

    return datePath;
}
```

```java
// 5. 기본경로 + 날짜별 경로 폴더 생성 메서드
private static void makeDir(String uploadPath, String... paths) {
    // 날짜별 경로가 이미 존재하면 메서드 종료
    if (new File(uploadPath + paths[paths.length - 1]).exists())
        return;

    for (String path :  paths) {

        File dirPath = new File(uploadPath + path);

        if (!dirPath.exists())
            dirPath.mkdir();

    }
}
```

```java
// 6-1. 썸네일 이미지 생성
private static String makeThumbnail(String uploadRootPath, String datePath, String fileName) throws Exception {

    // 원본이미지를 메모리상에 로딩
    BufferedImage originalImg = ImageIO.read(new File(uploadRootPath + datePath, fileName));
    // 원본이미지를 축소
    BufferedImage thumbnailImg = Scalr.resize(originalImg, Scalr.Method.AUTOMATIC, Scalr.Mode.FIT_TO_HEIGHT, 100);
    // 썸네일파일명 생성
    String thumbnailName = uploadRootPath + datePath + File.separator + "s_" + fileName;
    // 파일 객체생성
    File newFile = new File(thumbnailName);
    // 파일 확장자 추출
    String formatName = getFormatName(fileName);
    // 썸네일 파일 저장
    ImageIO.write(thumbnailImg, formatName.toUpperCase(), newFile);
    // \을 / 치환하고, 썸네일 저장 경로 + 파일명을 리턴
    return thumbnailName.substring(uploadRootPath.length()).replace(File.separatorChar, '/');
}
```

```java
// 6-2. 파일 아이콘
private static String makeIcon(String uploadRootPath, String datePath, String fileName) throws Exception {

    String iconName = uploadRootPath + datePath + File.separator +  fileName;
    // \을 / 치환하고, 썸네일 저장 경로 + 파일명을 리턴
    return iconName.substring(uploadRootPath.length()).replace(File.separatorChar, '/');
}
```

그리고 이미지 파일의 타입을 판별하는 `MediaUtils`클래스의 코드는 아래와 같다.
```java
public class MediaUtils {

    private static Map<String, MediaType> mediaTypeMap;

    // 클래스 초기화 블럭
    static {
        mediaTypeMap = new HashMap<>();
        mediaTypeMap.put("JPG", MediaType.IMAGE_JPEG);
        mediaTypeMap.put("GIF", MediaType.IMAGE_GIF);
        mediaTypeMap.put("PNG", MediaType.IMAGE_PNG);
    }

    // 파일 타입 판별
    public static MediaType getMediaType(String formatName) {
        return mediaTypeMap.get(formatName.toUpperCase());
    }

}
```

##### 4. 파일 업로드 컨트롤러 수정
파일 업로드를 위한 관련 클래스 작성이 완료되었으니 파일업로드 컨트롤러의 파일 업로드 메서드를 아래와 같이 수정한다.
```java
// 파일 업로드 처리
@RequestMapping(value = "/upload", method = RequestMethod.POST, produces = "text/plain;charset=UTF-8")
public ResponseEntity<String> uploadFile(MultipartFile file, HttpServletRequest request) {

    ResponseEntity<String> entity = null;

    try {
        String uploadedFileName = UploadFileUtils.uploadFile(file.getOriginalFilename(), file.getBytes(), request);
        entity = new ResponseEntity<>(uploadedFileName, HttpStatus.CREATED);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(HttpStatus.BAD_REQUEST);
    }

    return entity;
}
```

##### 5. 파일업로드 확인

이미지 파일을 업로드했을 경우 원본 이미지 파일과 썸네일 이미지 파일이 저장된 것을 확인할 수 있다.
![image]()

일반 파일을 업로드했을 경우 일반 파일만 저장된 것을 확인할 수 있다.
![file]()

#### # 업로드 파일 화면 출력 구현
파일을 드래그 앤 드랍으로 업로드를 하고 난 뒤에 업로드한 파일을 바로 화면에 출력해주기 위해서 아래와 같은 작업이 이루어져야 한다.

- 업로드된 파일 이름을 가지고, 화면에 태그를 생성해서 추가하는 작업
- 컨트롤러에서 특정 경로의 파일 데이터를 화면에 전송하는 작업

위의 작업 중에서 특정 경로의 파일을 다시 브라우저로 전송해주는 작업의 경우 주의해야할 점은 MINE타입이다. 요청된 파일이 이미지이면 포맷에 맞는 MINE타입을 지정해야하고, 일반파일인 경우 다운로드를 처리할 수 있어야 한다.

##### 1. 파일 업로드 컨트롤러 수정 : 파일 출력 메서드 작성
```java
// 업로드 파일 출력 처리
@RequestMapping(value = "/display", method = RequestMethod.GET)
public ResponseEntity<byte[]> displayFile(String fileName, HttpServletRequest request) throws IOException {

    ResponseEntity<byte[]> entity = null;
    InputStream inputStream = null;

    try {
        HttpHeaders httpHeaders = UploadFileUtils.getHttpHeaders(fileName); // Http 헤더 설정 가져오기
        String rootPath = UploadFileUtils.getRootPath(fileName, request); // 업로드 기본경로 경로
        inputStream = new FileInputStream(rootPath + fileName); // 해당 경로의 파일 읽어오기
        entity = new ResponseEntity<>(IOUtils.toByteArray(inputStream), httpHeaders, HttpStatus.CREATED);
    } catch (Exception e) {
        e.printStackTrace();
        entity = new ResponseEntity<>(HttpStatus.BAD_REQUEST);
    } finally {
        inputStream.close();
    }

    return entity;
}
```

##### 2. `UploadFileUtils`클래스에 파일 출력을 위한 메서드 추가
```java
// 파일 출력을 위한 Header 설정
public static HttpHeaders getHttpHeaders(String fileName) throws Exception {

    // 파일타입 확인
    MediaType mediaType = MediaUtils.getMediaType(getFormatName(fileName));

    HttpHeaders httpHeaders = new HttpHeaders();

    if (mediaType != null) {
        httpHeaders.setContentType(mediaType);
    } else {
        fileName = fileName.substring(fileName.indexOf("_") + 1);
        httpHeaders.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        httpHeaders.add("Content-Disposition", "attachment; filename=\"" +
                new String(fileName.getBytes("UTF-8"), "ISO-8859-1")+"\"");
    }

    return httpHeaders;
}
```

##### 2. AJAX 업로드 페이지에서 첨부파일 출력하기

```js
if (checkImageType(data)) {
    str = "<div>"
            + "<a href='display?fileName="+getImageLink(data)+"'>"
                + "<img src='display?fileName=" + data + "'/>"
            + "</a>"
            + "<small data-src="+data+">X</small>"
        + "</div>"
        + "<hr/>";
} else {
    str = "<div>"
            + "<a href='display?fileName="+data+"'>"
                + "<i class='fa fa-file'></i> "
                + getOriginalName(data)
            + "</a><small data-src="+data+">X</small>"
        + "</div>"
        + "<hr/>";
}
$(".uploadedList").append(str);
```

```js
function checkImageType(fileName) {

    var pattern = /jpg$|gif$|png$|jpge$/i;

    return fileName.match(pattern);
}
```
```js
function getOriginalName(fileName) {
    if (checkImageType(fileName)) {
        return;
    }
    var idx = fileName.indexOf("_") + 1;
    return fileName.substr(idx);
}
```
```js
function getImageLink(fileName) {
    if (!checkImageType(fileName)) {
        return;
    }
    var front = fileName.substr(0, 12); // /년/월/일 경로 추출
    var end = fileName.substr(14);      // _s 썸네일 표시 제거
    return front + end;
}
```

#### # 첨부 파일 삭제 구현

##### 1.
