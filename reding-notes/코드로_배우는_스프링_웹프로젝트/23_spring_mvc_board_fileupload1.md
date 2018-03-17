
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
일반적인 파일 업로드 테스트 페이지는 `WEB-INF/view/tutorial` 디렉토리에 `upload_form.jsp`를 생성해서 사용한다. 기존의 프로젝트에서 쓰이는 양식의 내용을 쓰도록한다.

**Jasny Bootstrap Template 적용**
AdminLTE에서는 파일 업로드 입력폼 양식이 따로 없기 때문에 파일 업로드를 지원하는 부트스트랩 템플릿을 찾아서 적용했다. 그리고 이후에 회원 프로필 사진 업로드 기능 구현에도 쓰이게 된다.

http://www.jasny.net/bootstrap/getting-started/#download
템플릿 적용은 간단하다. 위의 링크에 들어가 템플릿의 js파일과 css파일을 다운받아 템플릿파일(js, css)을 넣어주고 적용시켜주기만 하면 된다.

http://www.jasny.net/bootstrap/javascript/#fileinput
그리고나서 파일 업로드 폼 예제가 여러 개가 있는데 그중에 파일을 업로드할 수 있는 예제를 붙여 넣고 아래와 같이 수정해주면 된다.

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

아래는 파일 입력 폼을 적용한 모습이다.
![fileInputForm]()

위 코드에서 가장 중요한 부분은 `<form>`태그의 속성 중에 `enctype`을 `multipart/form-data`으로 설정한 것이다. 이 설정이 없을 경우 파일 데이터가 전달되지 않으니 반드시 써줘야한다. 그리고 업로드 처리 결과 알림에 `<iframe>`을 사용하기 위해서 `<form>`태그의 `target`속성을 `<iframe>`의 `name`속성에 맞춰준다.


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
- `MultipartFile` : 객체를 이용하면 파일의 이름, 크기, 타입등과 같은 정보를 얻을 수 있다.
- `getRealUploadPath()`메서드 : 파일이 업로드 되는 상대경로를 추출해준다.
- `uploadFile()`메서드 : 파일을 업로드 처리하고, 최종적으로 저장된 파일명을 리턴해준다.

#### # `<iframe>`을 이용한 파일업로드 결과 처리
앞서 잠깐 언급한 것처럼 현재 페이지에서 파일 업로드가 제대로 되었는지 확인하기 위해 `<iframe>`을 사용해서 처리한다. `<form>`태그는 브라우저의 창에서 전송이 일어나기 때문에 화면 전환이 이루어지게 된다. `<form>`태그에 `target`속성을 주고, `<iframe>`을 이용하면 화면전환을 피하고, 파일이 업로드가 처리 되었는지 확인할 수 있다.

`<ifrmae>`을 이용한 파일 업로드는 아래와 같은 과정을 거치게 된다.

1. `<form>`태그 전송은 화면에 포함된 `<iframe>`으로 전송한다.
2. 결과 페이지(`upload_result.jsp`)는 `upload_form.jsp`의 `<iframe>`내에 포함되므로 실제로 화면 변화가 없다.
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
아래와 같이 `iframe`에 대한 스타일 속성 모두를 0px로 설정해준다. 이렇게 하면 iframe은 화면에 보이지 않게되어 AJAX처리와 같은 효과를 얻을 수 있다.
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
![fileSelect]()

파일 업로드 처리 후 파일명을 alert창을 통해 알려준다.
![afterFileUpload]()

alert창이 닫히고 나면 파일 입력폼을 초기화시켜준다.
![removeFileSelect]()

## 3. AJAX 방식의 파일 업로드 처리
#### # 파일 업로드 페이지
```html

```
