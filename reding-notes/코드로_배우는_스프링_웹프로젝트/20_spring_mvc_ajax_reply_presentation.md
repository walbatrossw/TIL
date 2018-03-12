
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
- `<div class="modal fade" id="modifyModal" role="dialog"></div>` : 댓글 수정/삭제를 위한 Modal 영역

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
아래는 해당 페이지의 모습이다. 아직 댓글 목록과 페이징처리가 완료되지 않았기 때문에 하단의 2개의 빨간 박스는 비어있다. 추후에 댓글 목록과 페이징처리가 구현되면 차이점을 구분할 수 있다.
![html](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-11%2022-47-55.png?raw=true)

**JS코드를 이용한 댓글 목록 가져오기 테스트**
jQuery를 이용하여 특정 게시글(1000번째 게시글)의 댓글 목록을 배열 형태로 가져와 보자. `@RestController`의 경우 객체를 JSON방식으로 전달하기 때문에 jQuery를 이용해서 호출할 때는 `getJSON()`를 아래와 같이 사용한다.
```html
<script>
    var articleNo = 1000;
    $.getJSON("/replies/all/" + articleNo, function (data) {
       console.log(data);
    });
</script>
```
아래는 해당 페이지로 이동하고, 크롬 개발자 도구 콘솔창을 통해 확인해본 결과이다.
![list](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-11%2022-56-38.png?raw=true)

#### # 댓글 목록
댓글 목록을 출력하기 위한 함수 `getReplies()`를 정의하고, 위에서 댓글 목록 가져오기 테스트를 했던 것처럼 `getJSON()`함수를 통해 댓글 목록 객체를 JSON형식으로 받아온다. 받아온 댓글 객체를 `each()`함수를 통해 루프를 돌면서 `<li>`태그를 만들어낸다. `<li>`태그마다 댓글내용과 댓글작성자들이 출력되도록 한다. 아래의 코드에서 주목해서봐야할 점은 `<li>`태그 속성에 `data-replyNo`이다. `data-`로 시작되는 속성은 이름이나 갯수에 상관없이 id나 name속성을 대신해서 사용하기에 편리하다. 이 속성을 통해 앞으로 댓글 수정/삭제 처리시 댓글 번호를 세팅하는데 사용하게 된다.
```js
// 1000번째 게시글
var articleNo = 1000;

// 댓글 목록 호출
getReplies();

// 댓글 목록 출력 함수
function getReplies() {

    $.getJSON("/replies/all/" + articleNo, function (data) {

        console.log(data);

        var str = "";

        $(data).each(function () {
            str += "<li data-replyNo='" + this.replyNo + "' class='replyLi'>"
                +   "<p class='replyText'>" + this.replyText + "</p>"
                +   "<p class='replyWriter'>" + this.replyWriter + "</p>"
                +   "<button type='button' class='btn btn-xs btn-success' data-toggle='modal' data-target='#modifyModal'>댓글 수정</button>"
                + "</li>"
                + "<hr/>";

        });

        $("#replies").html(str);

    });

}
```

댓글 목록이 출력된 모습은 아래와 같다.
![list2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-11%2023-55-18.png?raw=true)

#### # 댓글 등록
댓글 저장 버튼 클릭을 할 경우, 이벤트 처리는 아래와 같이 작성해준다.
- jQuery를 이용하여 화면에서 입력된 변수를 처리하고, `$.ajax()`를 통해 서버를 호출한다.
- 전송하는 데이터는 JSON으로 구성된 문자열을 사용하고, 전송 받은 결과는 단순 문자열인 `regSuccess`이다.
- 이 문자열을 통해 댓글이 정상적으로 등록되었는지 확인할 수 있는 알림창을 띄워준다.
- 마지막으로 댓글 목록 출력 함수를 호출하여 댓글 목록을 갱신시키고, 댓글내용과 작성자 입력창은 초기화를 시켜준다.
```js
var articleNo = 1000;

// 댓글 목록 출력 함수 ...

// 댓글 저장 버튼 클릭 이벤트 발생시
$("#replyAddBtn").on("click", function () {

    // 화면으로부터 입력 받은 변수값의 처리
    var replyText = $("#newReplyText");
    var replyWriter = $("#newReplyWriter");

    var replyTextVal = replyText.val();
    var replyWriterVal = replyWriter.val();

    // AJAX 통신 : POST
    $.ajax({
        type : "post",
        url : "/replies",
        headers : {
            "Content-type" : "application/json",
            "X-HTTP-Method-Override" : "POST"
        },
        dataType : "text",
        data : JSON.stringify({
            articleNo : articleNo,
            replyText : replyTextVal,
            replyWriter : replyWriterVal
        }),
        success : function (result) {
            // 성공적인 댓글 등록 처리 알림
            if (result == "regSuccess") {
                alert("댓글 등록 완료!");
            }
            getReplies(); // 댓글 목록 출력 함수 호출
            replyText.val(""); // 댓글 내용 초기화
            replyWriter.val(""); // 댓글 작성자 초기화
        }
    });
});
```

아래는 댓글내용과 댓글 작성자를 입력하고 저장버튼을 클릭하고 난 직후의 모습이다. 댓글 등록이 제대로 처리되고 나서 댓글 등록이 완료되었다는 알림창이 뜨게 되는 것을 확인 할 수 있다.
![register1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2000-30-41.png?raw=true)

그리고 알림창 확인 버튼을 클릭하고 나면 댓글 목록이 갱신되면서 방금 등록한 댓글의 내용이 출력되고, 입력했던 댓글내용과 등록자는 초기화가 된다.
![register2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2000-31-07.png?raw=true)

#### # 댓글 조회 및 수정/삭제
`getReplies()`함수를 통해 생성된 `<li>`태그 안의 수정버튼을 클릭하면 Modal창이 뜨고, 클릭한 댓글의 값들(댓글번호, 댓글내용, 댓글작성자)이 `<input>`태그 value값으로 세팅된다.

**댓글 수정/삭제 Modal 버튼**
```html
<button type='button' class='btn btn-xs btn-success' data-toggle='modal' data-target='#modifyModal'>
  댓글 수정
</button>
```

**댓글 수정/삭제 Modal 영역**
```html
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
```

**댓글의 값들 세팅**
jQuery코드에서 주목해서 봐야할 것은 클릭 이벤트 선택자가 `<ul>`의 id인 `replies`로 되어있다는 점이다. 단순히 클릭이 발생한 수정버튼을 선택자로 지정하면 될 것 같지만 Ajax통신 후에 생기는 요소이기 때문에 이벤트처리가 되지 않는다. 그래서 AJA통신 이전에 존재하는 `<ul>`을 이용해서 이벤트를 등록해야만 한다.
```js
$("#replies").on("click", ".replyLi button", function () {
    var reply = $(this).parent();

    var replyNo = reply.attr("data-replyNo");
    var replyText = reply.find(".replyText").text();
    var replyWriter = reply.find(".replyWriter").text();

    $("#replyNo").val(replyNo);
    $("#replyText").val(replyText);
    $("#replyWriter").val(replyWriter);

});
```

아래를 보면 댓글 수정 버튼 클릭시 모달창이 나오고, 해당 댓글의 값들이 함께 출력되는 것을 확인할 수 있다.
![modal](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2019-02-17.png?raw=true)

**댓글 삭제 호출하기**
Modal의 삭제버튼을 클릭할 경우, 이벤트 처리는 아래와 같이 작성해준다.

- jQuery를 이용하여 댓글번호(`replyNo`)를 처리하고, `$.ajax()`를 통해 서버를 호출한다.
- 데이터를 전송하고, 전송받은 결과는 단순 문자열인 `delSuccess`이다.
- 정상적으로 삭제되었음을 확인할 수 있는 알림창을 띄워준다.
- 마지막으로 Modal창을 닫고, 댓글 목록을 갱신시켜준다.

```js
$(".modalDelBtn").on("click", function () {

    // 댓글 번호
    var replyNo = $(this).parent().parent().find("#replyNo").val();

    // AJAX통신 : DELETE
    $.ajax({
        type : "delete",
        url : "/replies/" + replyNo,
        headers : {
            "Content-type" : "application/json",
            "X-HTTP-Method-Override" : "DELETE"
        },
        dataType : "text",
        success : function (result) {
            console.log("result : " + result);
            if (result == "delSuccess") {
                alert("댓글 삭제 완료!");
                $("#modifyModal").modal("hide"); // Modal 닫기
                getAllList(); // 댓글 목록 갱신
            }
        }
    });

});
```

**댓글 수정 호출하기**
Modal의 수정버튼을 클릭할 경우, 이벤트 처리는 아래와 같이 작성해준다.

- jQuery를 이용하여 댓글번호(`replyNo`)와 수정한 댓글내용(`replyText`)를 처리하고, `$.ajax()`를 통해 서버를 호출한다.
- 전송하는 데이터는 JSON으로 구성된 문자열로 사용하고, 전송받은 결과는 단순 문자열인 `modSuccess`이다.
- 정상적으로 수정되었음을 확인할 수 있는 알림창을 띄워준다.
- 마지막으로 Modal창을 닫고, 댓글 목록을 갱신시켜준다.

```js
$(".modalModBtn").on("click", function () {

    // 댓글 선택자
    var reply = $(this).parent().parent();
    // 댓글번호
    var replyNo = reply.find("#replyNo").val();
    // 수정한 댓글내용
    var replyText = reply.find("#replyText").val();

    // AJAX통신 : PUT
    $.ajax({
        type : "put",
        url : "/replies/" + replyNo,
        headers : {
            "Content-type" : "application/json",
            "X-HTTP-Method-Override" : "PUT"
        },
        data : JSON.stringify(
            {replyText : replyText}
        ),
        dataType : "text",
        success : function (result) {
            console.log("result : " + result);
            if (result == "modSuccess") {
                alert("댓글 수정 완료!");
                $("#modifyModal").modal("hide"); // Modal 닫기
                getReplies(); // 댓글 목록 갱신
            }
        }
    });

});
```

**댓글 삭제/수정 구현 결과 확인**
댓글 삭제버튼 클릭하면 댓글 삭제 처리가 되고, 성공적으로 처리가 되었다는 알림창이 뜬다.
![delete1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2020-55-52.png?raw=true)

그리고 Modal이 닫히고, 댓글 목록이 갱신된다.
![delete2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2020-56-08.png?raw=true)

댓글 수정버튼 클릭하면 댓글 수정 처리가 되고, 성공적으로 처리가 되었다는 알림창이 뜬다.
![update1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2020-58-43.png?raw=true)

그리고 Modal이 닫히고, 댓글 목록이 갱신된다.
![update2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2020-59-00.png?raw=true)

#### # 댓글 목록 페이징 처리

**댓글 목록 페이지 / 댓글 목록 페이지 번호 출력**
댓글 목록을 화면에서 페이징 처리 해보자. 페이징 처리된 댓글 목록을 출력하기 위한 함수 `getRepliesPaging()`와 댓글 목록 페이지 번호를 출력하기 위한 함수 `printPageNumbers()`를 선언하고, 아래와 같이 작성해준다. 그리고 이전에 `getReplies()`함수를 호출한 것들은 `getRepliesPaging()`로 변경해준다.
```js
// 댓글 목록 페이징 함수
function getRepliesPaging(page) {

    $.getJSON("/replies/" + articleNo + "/" + page, function (data) {
        console.log(data);

        var str = "";

        $(data.replies).each(function () {
            str += "<li data-replyNo='" + this.replyNo + "' class='replyLi'>"
                +  "<p class='replyText'>" + this.replyText + "</p>"
                +  "<p class='replyWriter'>" + this.replyWriter + "</p>"
                +  "<button type='button' class='btn btn-xs btn-success' data-toggle='modal' data-target='#modifyModal'>댓글 수정</button>"
                +  "</li>"
                +  "<hr/>";
        });

        $("#replies").html(str);

        // 페이지 번호 출력 호출
        printPageNumbers(data.pageMaker);

    });

}

// 댓글 목록 페이지 번호 출력 함수
function printPageNumbers(pageMaker) {

    var str = "";

    // 이전 버튼 활성화
    if (pageMaker.prev) {
        str += "<li><a href='"+(pageMaker.startPage-1)+"'>이전</a></li>";
    }

    // 페이지 번호
    for (var i = pageMaker.startPage, len = pageMaker.endPage; i <= len; i++) {
        var strCalss = pageMaker.criteria.page == i ? 'class=active' : '';
        str += "<li "+strCalss+"><a href='"+i+"'>"+i+"</a></li>";
    }

    // 다음 버튼 활성화
    if (pageMaker.next) {
        str += "<li><a href='"+(pageMaker.endPage + 1)+"'>다음</a></li>";
    }

    $(".pagination-sm").html(str);
}
```

**댓글 목록 페이지 번호 클릭 이벤트처리**
페이지번호가 출력되었으니 각 페이지번호에 대한 클릭 이벤트를 처리해주자. `<a>`태그에서 페이지 번호를 추출해서 `getRepliesPaging()`를 호출해준다.
```js

// 목록페이지 번호 변수 선언, 1로 초기화(첫번째 페이지)
var replyPageNum = 1;

// 목록페이지 번호 클릭 이벤트
$(".pagination").on("click", "li a", function (event) {

    event.preventDefault();
    replyPageNum = $(this).attr("href"); // 목록 페이지 번호 추출
    getRepliesPaging(replyPageNum); // 목록 페이지 호출

});
```

**댓글 페이징처리 구현 모습**
![paging1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2021-22-19.png?raw=true)
![paging2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2021-23-09.png?raw=true)

## 2. 게시글에 댓글 적용하기

#### # 조회 화면의 수정

#### # Handlebars를 이용한 템플릿

#### # 댓글 목록

#### # 댓글 댓글 등록

#### # 댓글 수정/삭제
