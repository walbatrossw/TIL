
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
---

# Spring MVC - 댓글처리 : 프레젠테이션 계층 구현

## 1. 화면에서 AJAX 호출 테스트

#### # 댓글 테스트를 위한 컨트롤러 작성
AJAX 댓글 테스트를 위해 임시 테스트 페이지를 매핑할 컨트롤러와 메서드를 아래와 같이 작성해준다.
```java
@Controller
@RequestMapping("/reply")
public class ReplyTestController {

    @RequestMapping("/test")
    public String replyAjaxTest() {
        return "/tutorial/reply_test";
    }

}
```

#### # 댓글 테스트를 위한 JSP페이지 작성

**HTML 코드**
`/WEB-INF/views/tutorial`디렉토리에 `reply_test.jsp`을 기존의 페이지들과 같이 생성하고, `<section>`태그 안에 구현할 내용을 아래와 같이 작성해준다. 아래 코드에서 각 영역을 구분해보면 아래와 같다.

- `<div class="box-body"></div>` : 댓글 입력 영역
- `<ul id="replies"></ul>` : 댓글 목록 영역
- `<ul class="pagination pagination-sm no-margin"></ul>` : 댓글 페이징 영역
- `<div class="modal fade" id="modifyModal" role="dialog"></div>` : 댓글 수정/삭제를 위한 모달창 영역

```html
<section class="content container-fluid">
  <div class="col-lg-12">
    <div class="box box-primary">
        <div class="box-header with-border">
            <h3 class="box-title">댓글 작성</h3>
        </div>
        <div class="box-body">
            <div class="form-group">
                <label for="newReplyText">댓글 내용</label>
                <input class="form-control" id="newReplyText" name="replyText" placeholder="댓글 내용을 입력해주세요">
            </div>
            <div class="form-group">
                <label for="newReplyWriter">댓글 작성자</label>
                <input class="form-control" id="newReplyWriter" name="replyWriter" placeholder="댓글 작성자를 입력해주세요">
            </div>
        </div>
        <div class="box-footer">
            <ul id="replies">

            </ul>
        </div>
        <div class="box-footer">
            <div class="text-center">
                <ul class="pagination pagination-sm no-margin">

                </ul>
            </div>
        </div>
    </div>
  </div>

  <div class="modal fade" id="modifyModal" role="dialog">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal">&times;</button>
                <h4 class="modal-title">댓글 수정창</h4>
            </div>
            <div class="modal-body">
                <div class="form-group">
                    <label for="replyNo">댓글 번호</label>
                    <input class="form-control" id="replyNo" name="replyNo" readonly>
                </div>
                <div class="form-group">
                    <label for="replyText">댓글 내용</label>
                    <input class="form-control" id="replyText" name="replyText" placeholder="댓글 내용을 입력해주세요">
                </div>
                <div class="form-group">
                    <label for="replyWriter">댓글 작성자</label>
                    <input class="form-control" id="replyWriter" name="replyWriter" readonly>
                </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default pull-left" data-dismiss="modal">닫기</button>
                <button type="button" class="btn btn-success modalModBtn">수정</button>
                <button type="button" class="btn btn-danger modalDelBtn">삭제</button>
            </div>
        </div>
    </div>
  </div>

</section>
```
아래는 해당 페이지의 모습이다. 아직 댓글 목록과 페이징처리가 완료되지 않았기 때문에 하단의 2개의 빨간 박스는 비어있다. 추후에 댓글 목록과 페이징처리가 구현되면 차이점을 구분할 수 있게된다.
**JS코드를 이용한 댓글 목록 가져오기 테스트**
jQuery를 이용하여 특정 게시글(1000번째 게시글)의 댓글 목록을 리스트형태로 가져와 보자. `@RestController`의 경우 객체를 JSON방식으로 전달하기 때문에 jQuery를 이용해서 호출할 때는 `getJSON()`를 아래와 같이 사용한다.
```html
<script>
    var articleNo = 1000;
    $.getJSON("/replies/all/" + articleNo, function (data) {
       console.log(data);
    });
</script>
```
아래는 해당 페이지로 이동하고, 크롬 개발자 도구 콘솔창을 통해 확인해본 결과이다.
![]()

#### # 댓글 목록
```html

```
#### # 댓글 등록

#### # 댓글 조회 및 수정/삭제

#### # 전체 페이징 처리

## 2. 게시글에 댓글 적용하기

#### # 조회 화면의 수정

#### # Handlebars를 이용한 템플릿

#### # 댓글 목록

#### # 댓글 댓글 등록

#### # 댓글 수정/삭제
