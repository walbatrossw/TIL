
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

# Spring MVC 게시판 예제12  - 댓글처리 : 프레젠테이션 계층 구현

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

![before]()

**JS코드를 이용한 댓글 목록 가져오기 테스트**
jQuery를 이용하여 특정 게시글(1000번째 게시글)의 댓글 목록을 배열 형태로 가져와 보자. `@RestController`의 경우 객체를 JSON방식으로 전달하기 때문에 jQuery를 이용해서 호출할 때는 `getJSON()`를 아래와 같이 사용한다.

```js
    var articleNo = 1000;
    $.getJSON("/replies/all/" + articleNo, function (data) {
       console.log(data);
    });
```

아래는 해당 페이지로 이동하고, 크롬 개발자 도구 콘솔창을 통해 확인해본 결과이다.

![replyt_list]()

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

![replyt_list2]()

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

![reply_register]()

그리고 알림창 확인 버튼을 클릭하고 나면 댓글 목록이 갱신되면서 방금 등록한 댓글의 내용이 출력되고, 입력했던 댓글내용과 등록자는 초기화가 된다.

![reply_register2]()

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

![modal]()

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

## 2. 게시판에 댓글 적용하기
지금까지 AJAX방식으로 댓글 기능을 구현하기 위한 화면 구현 연습을 했으니 실제로 게시판에 적용시켜보자.

#### # 조회 화면(`read.jsp`)의 수정
게시글에 댓글을 달기 위해서는 게시글 조회 페이지로 이동해야만 한다. 그렇기 때문에 게시글 조회 페이지에서 댓글 기능을 구현하게 된다. 앞서 연습해본 것과 마찬가지로 댓글 기능을 추가하기 위한 영역은 아래와 같이 2가지 영역으로 나누어진다.

- 댓글 입력 영역
- 댓글 목록/페이징 영역

이렇게 2가지의 영역을 아래와 같이 `read.jsp`에 추가시켜준다.

**댓글 입력 영역**

댓글 입력 영역은 간단하다. 댓글 내용과 댓글 작성자를 입력할 수 있게 `<textarea>`태그와 `<input>`로 이루어져있다.

```html
<div class="box box-warning">
    <div class="box-header with-border">
        <a class="link-black text-lg"><i class="fa fa-pencil"></i> 댓글작성</a>
    </div>
    <div class="box-body">
        <form class="form-horizontal">
            <div class="form-group margin">
                <div class="col-sm-10">
                    <textarea class="form-control" id="newReplyText" rows="3" placeholder="댓글내용..." style="resize: none"></textarea>
                </div>
                <div class="col-sm-2">
                    <input class="form-control" id="newReplyWriter" type="text" placeholder="댓글작성자...">
                </div>
                <hr/>
                <div class="col-sm-2">
                    <button type="button" class="btn btn-primary btn-block replyAddBtn"><i class="fa fa-save"></i> 저장</button>
                </div>
            </div>
        </form>
    </div>
</div>
```

**댓글 목록/페이징 영역**

댓글 목록/페이징 영역의 HTML코드가 약간 복잡하다. 지금 참고하고 있는 책의 예제와 약간 다른데 AdminLTE 템플릿에서 제공하는 `box`클래스 속성 중에서 `collapsed-box`클래스를 이용하여 `box`를 접거나 펼칠 수 있게 하였다. 즉, 쉽게 말하자면 댓글 목록을 펼치거나 접을 수 있게 구현하려고 한다. 댓글의 유무에 따라 댓글 목록을 펼칠 수 있는 버튼이 활성화/비활성화 되게 처리하고, 댓글 갯수도 함께 출력해줄 예정이다.

```html
<div class="box box-success collapsed-box">
    <%--댓글 유무 / 댓글 갯수 / 댓글 펼치기, 접기--%>
    <div class="box-header with-border">
        <a href="" class="link-black text-lg"><i class="fa fa-comments-o margin-r-5 replyCount"></i></a>
        <div class="box-tools">
            <button type="button" class="btn btn-box-tool" data-widget="collapse">
                <i class="fa fa-plus"></i>
            </button>
        </div>
    </div>
    <%--댓글 목록--%>
    <div class="box-body repliesDiv">

    </div>
    <%--댓글 페이징--%>
    <div class="box-footer">
        <div class="text-center">
            <ul class="pagination pagination-sm no-margin">

            </ul>
        </div>
    </div>
</div>
```

**현재까지의 댓글 영역의 모습**
아래는 아직은 구현이 되지 않은 댓글 영역의 모습이다.

![before_reply](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-12%2023-32-09.png?raw=true)

#### # Handlebars를 이용한 JS템플릿 적용
댓글 기능을 구현하는데 있어서 가장 핵심적인 부분은 댓글이 추가되거나 수정, 삭제되더라도 지속적으로 댓글 목록이 갱신되어 화면에 출력 되어야하는 것이다. 목록의 출력은 `<div>`태그가 반복적으로 구성되고, 하나의 `<div>` 안의 댓글의 정보들이 채워지는 방식으로 동작하게 된다. 이러한 작업은 문자열로 이루어지기 때문에 상당히 번거롭고, 지저분한 코드가 만들어지게 된다. 앞서 본 연습에서도 `str`변수에 html코드를 계속 붙여가면서 작성해야하는 번거로움이 있었던 것처럼 말이다. 하지만 자바스크립트 템플릿을 통해 좀더 깔끔한 코드를 작성할 수 있고 가독성 향상에도 도움이 된다. 자바스크립트 템플릿 종류에는 JS Render, Mustache, Mustache를 기반으로 한 Handlebars, Hogan 등이 있다. 이 예제에서는 Handlebars를 사용할 것이다.

Handlebars의 구체적인 사용법을 알고 싶다면 아래를 통해 참조하면 된다.
https://handlebarsjs.com/

**Handlebars 라이브러리 추가**

Handlebars를 사용하기 위해서는 jQuery와 Handlebar라이브러리를 추가해줘야하는데 이미 jQuery는 추가 되어있기 때문에
`WEB-INF/view/include/plugin_js.jsp`에 Handlebar라이브러리만 추가해준다.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/handlebars.js/4.0.11/handlebars.min.js"></script>
```

Handlebars를 사용하기 위한 준비는 마쳤다. 이제 댓글 목록, 등록, 수정/삭제 조회 기능을 구현해보자.

#### # 댓글 목록

**템플릿 코드**
댓글 목록을 출력하기 위해 템플릿 코드를 아래와 같이 작성해준다. 이 템플릿 코드는 하나의 댓글을 구성하는 부분이다.

```html
<script id="replyTemplate" type="text/x-handlebars-template">
    {{#each.}}
    <div class="post replyDiv" data-replyNo={{replyNo}}>
        <div class="user-block">
            <img class="img-circle img-bordered-sm" src="/dist/img/user1-128x128.jpg" alt="user image">
            <span class="username">
                <a href="#">{{replyWriter}}</a>
                <a href="#" class="pull-right btn-box-tool replyDelBtn" data-toggle="modal" data-target="#delModal">
                    <i class="fa fa-times"> 삭제</i>
                </a>
                <a href="#" class="pull-right btn-box-tool replyModBtn" data-toggle="modal" data-target="#modModal">
                    <i class="fa fa-edit"> 수정</i>
                </a>
            </span>
            <span class="description">{{prettifyDate regDate}}</span>
        </div>
        <div class="oldReplyText">{{{escape replyText}}}</div>
        <br/>
    </div>
    {{/each}}
</script>
```
위의 코드에서 주목해서 봐야할 점은 아래와 같다.
- Handlebars의 `{{#each.}}{{/each}}`은 JSTL의 `<c:forEach>`처럼 배열의 루프를 처리하기 위해 사용한다.
- 댓글 등록일자의 `prettifyDate`와 댓글 내용의 `escape`은 Handlebars의 확장기능을 사용한 것으로 아래의 JS코드에서 처리해준다.

**댓글 목록 JS코드**
```js
$(document).ready(function () {

    var articleNo = "${article.articleNo}";  // 현재 게시글 번호
    var replyPageNum = 1; // 댓글 페이지 번호 초기화

    // 댓글 내용 : 줄바꿈/공백처리
    Handlebars.registerHelper("escape", function (replyText) {
        var text = Handlebars.Utils.escapeExpression(replyText);
        text = text.replace(/(\r\n|\n|\r)/gm, "<br/>");
        text = text.replace(/( )/gm, "&nbsp;");
        return new Handlebars.SafeString(text);
    });

    // 댓글 등록일자 : 날짜/시간 2자리로 맞추기
    Handlebars.registerHelper("prettifyDate", function (timeValue) {
        var dateObj = new Date(timeValue);
        var year = dateObj.getFullYear();
        var month = dateObj.getMonth() + 1;
        var date = dateObj.getDate();
        var hours = dateObj.getHours();
        var minutes = dateObj.getMinutes();
        // 2자리 숫자로 변환
        month < 10 ? month = '0' + month : month;
        date < 10 ? date = '0' + date : date;
        hours < 10 ? hours = '0' + hours : hours;
        minutes < 10 ? minutes = '0' + minutes : minutes;
        return year + "-" + month + "-" + date + " " + hours + ":" + minutes;
    });

    // 댓글 목록 함수 호출
    getReplies("/replies/" + articleNo + "/" + replyPageNum);

    // 댓글 목록 함수
    function getReplies(repliesUri) {
        $.getJSON(repliesUri, function (data) {
            printReplyCount(data.pageMaker.totalCount);
            printReplies(data.replies, $(".repliesDiv"), $("#replyTemplate"));
            printReplyPaging(data.pageMaker, $(".pagination"));
        });
    }

    // 댓글 갯수 출력 함수
    function printReplyCount(totalCount) {

        var replyCount = $(".replyCount");
        var collapsedBox = $(".collapsed-box");

        // 댓글이 없으면
        if (totalCount === 0) {
            replyCount.html(" 댓글이 없습니다. 의견을 남겨주세요");
            collapsedBox.find(".btn-box-tool").remove();
            return;
        }

        // 댓글이 존재하면
        replyCount.html(" 댓글목록 (" + totalCount + ")");
        collapsedBox.find(".box-tools").html(
            "<button type='button' class='btn btn-box-tool' data-widget='collapse'>"
            + "<i class='fa fa-plus'></i>"
            + "</button>"
        );

    }

    // 댓글 목록 출력 함수
    function printReplies(replyArr, targetArea, templateObj) {
        var replyTemplate = Handlebars.compile(templateObj.html());
        var html = replyTemplate(replyArr);
        $(".replyDiv").remove();
        targetArea.html(html);
    }

    // 댓글 페이징 출력 함수
    function printReplyPaging(pageMaker, targetArea) {
        var str = "";
        if (pageMaker.prev) {
            str += "<li><a href='" + (pageMaker.startPage - 1) + "'>이전</a></li>";
        }
        for (var i = pageMaker.startPage, len = pageMaker.endPage; i <= len; i++) {
            var strClass = pageMaker.criteria.page == i ? "class=active" : "";
            str += "<li " + strClass + "><a href='" + i + "'>" + i + "</a></li>";
        }
        if (pageMaker.next) {
            str += "<li><a href='" + (pageMaker.endPage + 1) + "'>다음</a></li>";
        }
        targetArea.html(str);
    }

    // 댓글 페이지 번호 클릭 이벤트
    $(".pagination").on("click", "li a", function (event) {
        event.preventDefault();
        replyPageNum = $(this).attr("href");
        getReplies("/replies/" + articleNo + "/" + replyPageNum);
    });
}
```

댓글 목록에 관련된 JS코드가 상당히 많은 편인데 각각의 코드를 설명하면 아래와 같다.
- 전역변수(`articleNo`, `replyPageNum`) 선언 : JS코드 전체에서 쓰이는 현재 게시글의 번호와 댓글 페이지 번호 변수를 선언해준다.
- Handlebars 확장 기능 : `Handlebars.registerHelper()`를 통해 댓글 내용의 줄바꿈/공백처리와 댓글 등록일자의 날짜 시간을 변환시켜준다.
- 댓글 목록함수(`getReplies()`) : 댓글 목록 함수를 통해 특정 게시글의 전체 댓글을 가져와 댓글갯수, 댓글목록, 댓글 페이징처리 함수를 호출해준다.
- 댓글 갯수 출력 함수(`printReplyCount()`) : 댓글이 없으면 `collapsedBox`를 비활성화 시키고, 1개 이상이면 활성화 시킨다.
- 댓글 페이징 출력 함수(`printReplyPaging()`) : 기존에 페이징 처리했던 것처럼 동일하게 다음, 이전, 페이지번호를 출력하는 함수이다.
- 댓글 페이지 번호 클릭시 해당 페이지로 이동하기 위한 이벤트 처리를 해준다.

#### # 댓글 등록
```js
// 댓글 저장 버튼 클릭 이벤트
$(".replyAddBtn").on("click", function () {

    // 입력 form 선택자
    var replyWriterObj = $("#newReplyWriter");
    var replyTextObj = $("#newReplyText");
    var replyWriter = replyWriterObj.val();
    var replyText = replyTextObj.val();

    // 댓글 입력처리 수행
    $.ajax({
        type : "post",
        url : "/replies/",
        headers : {
            "Content-Type" : "application/json",
            "X-HTTP-Method-Override" : "POST"
        },
        dataType : "text",
        data : JSON.stringify({
            articleNo : articleNo,
            replyWriter : replyWriter,
            replyText : replyText
        }),
        success: function (result) {
            console.log("result : " + result);
            if (result === "regSuccess") {
                alert("댓글이 등록되었습니다.");
                replyPageNum = 1;  // 페이지 1로 초기화
                getReplies("/replies/" + articleNo + "/" + replyPageNum); // 댓글 목록 호출
                replyTextObj.val("");   // 댓글 입력창 공백처리
                replyWriterObj.val("");   // 댓글 입력창 공백처리
            }
        }
    });
});
```
댓글 등록 처리는 목록처리에 비해 간단한데 코드에 설명은 아래와 같다.
- 댓글 저장 버튼 클릭 이벤트가 발생하면 jQuery 선택자를 통해 입력된 값들을 각각 변수에 담고 JSON으로 변환시켜 AJAX POST방식으로 값을 넘겨준다.
- 댓글 등록처리가 성공적으로 처리되면 결과값으로 받은 문자열(`regSuccess`)을 통해 알림창을 띄워준다.
- 페이지 번호는 1로 초기화하여 댓글 목록을 다시 호출해 첫번째 목록화면으로 이동한다.
- 마지막으로 댓글내용과 댓글 작성자 입력칸은 비워지게 처리를 해준다.

#### # 댓글 수정/삭제

**HTML 코드 : 댓글 수정/삭제를 위한 Modal 추가**

댓글 수정/삭제는 Modal창을 띄워서 처리하도록 구현하기 위해 아래와 같이 html코드를 추가해준다.

```html
<%--댓글 수정 modal 영역--%>
<div class="modal fade" id="modModal">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span></button>
                <h4 class="modal-title">댓글수정</h4>
            </div>
            <div class="modal-body" data-rno>
                <input type="hidden" class="replyNo"/>
                <%--<input type="text" id="replytext" class="form-control"/>--%>
                <textarea class="form-control" id="replyText" rows="3" style="resize: none"></textarea>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default pull-left" data-dismiss="modal">닫기</button>
                <button type="button" class="btn btn-primary modalModBtn">수정</button>
            </div>
        </div>
    </div>
</div>

<%--댓글 삭제 modal 영역--%>
<div class="modal fade" id="delModal">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span></button>
                <h4 class="modal-title">댓글 삭제</h4>
                <input type="hidden" class="rno"/>
            </div>
            <div class="modal-body" data-rno>
                <p>댓글을 삭제하시겠습니까?</p>
                <input type="hidden" class="rno"/>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-default pull-left" data-dismiss="modal">아니요.</button>
                <button type="button" class="btn btn-primary modalDelBtn">네. 삭제합니다.</button>
            </div>
        </div>
    </div>
</div>
```
**JS 코드 : 댓글 수정/삭제 이벤트 처리**
```js
// 댓글 수정을 위해 modal창에 선택한 댓글의 값들을 세팅
$(".repliesDiv").on("click", ".replyDiv", function (event) {
    var reply = $(this);
    $(".replyNo").val(reply.attr("data-replyNo"));
    $("#replyText").val(reply.find(".oldReplyText").text());
});

// modal 창의 댓글 수정버튼 클릭 이벤트
$(".modalModBtn").on("click", function () {
    var replyNo = $(".replyNo").val();
    var replyText = $("#replyText").val();
    $.ajax({
        type : "put",
        url : "/replies/" + replyNo,
        headers : {
            "Content-Type" : "application/json",
            "X-HTTP-Method-Override" : "PUT"
        },
        dataType : "text",
        data: JSON.stringify({
            replyText : replyText
        }),
        success: function (result) {
            console.log("result : " + result);
            if (result === "modSuccess") {
                alert("댓글이 수정되었습니다.");
                getReplies("/replies/" + articleNo + "/" + replyPageNum); // 댓글 목록 호출
                $("#modModal").modal("hide"); // modal 창 닫기
            }
        }
    })
});

// modal 창의 댓글 삭제버튼 클릭 이벤트
$(".modalDelBtn").on("click", function () {
    var replyNo = $(".replyNo").val();
    $.ajax({
        type: "delete",
        url: "/replies/" + replyNo,
        headers: {
            "Content-Type" : "application/json",
            "X-HTTP-Method-Override" : "DELETE"
        },
        dataType: "text",
        success: function (result) {
            console.log("result : " + result);
            if (result === "delSuccess") {
                alert("댓글이 삭제되었습니다.");
                getReplies("/replies/" + articleNo + "/" + replyPageNum); // 댓글 목록 호출
                $("#delModal").modal("hide"); // modal 창 닫기
            }
        }
    });
});
```
댓글 수정/삭제 이벤트 처리에 대한 코드의 설명은 아래와 같다.
- 댓글 수정 버튼클릭 이벤트가 발생하면 수정 Modal창에 댓글번호, 댓글 내용을 입력폼에 세팅해준다.
- 수정 Modal창의 수정버튼 클릭 이벤트가 발생하면, 댓글번호, 댓글내용을 jQuery선택자를 통해 변수(`replyNo`, `replyText`)에 담고, AJAX PUT방식으로 값을 넘겨준다.
- 댓글 삭제버튼 클릭 이벤트가 발생하면 삭제 Modal창에 댓글번호를 변수(`replyNo`)에 담고, AJAX DELETE방식으로 값을 넘겨준다.
- 댓글 수정/삭제 처리가 성공적으로 처리되고 결과값으로 받은 문자열(`modSuccess`, `delSuccess`)을 통해 알림창을 띄워주고, 댓글의 목록을 갱신해준다.
- 마지막으로 Modal창을 닫아주면 댓글 수정/삭제처리가 종료된다.

#### # 댓글 기능 구현 모습
**댓글 목록**
댓글이 없을 경우

![box1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2014-40-59.png?raw=true)

댓글이 있을 경우(댓글 목록 접기)

![box2_1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2014-42-29.png?raw=true)

댓글이 있을 경우(댓글 목록 펼치기)

![box2_2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2014-44-34.png?raw=true)

**댓글 입력**

댓글 저장 버튼 클릭하고, 댓글 저장 처리 후 알림창을 띄운 모습

![add1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2014-47-53.png?raw=true)

댓글 목록 갱신, 댓글 입력창 초기화

![add2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2014-48-08.png?raw=true)

**댓글 수정**

댓글 수정 버튼 클릭시 Modal창이 뜨고, 기존의 댓글을 수정

![mod1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2015-00-30.png?raw=true)

Modal창의 댓글 수정버튼 클릭하고, 댓글 수정 처리 후 알림창을 띄운 모습

![mod2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2015-01-21.png?raw=true)

댓글 수정 처리 후 댓글 목록 갱신

![mod3](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2015-01-38.png?raw=true)

**댓글 삭제**

댓글 삭제 버튼 클릭시 Modal창의 모습

![del1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2015-03-07.png?raw=true)

Modal창의 댓글 삭제버튼을 클릭하고, 댓글 삭제 처리 후 알림창을 띄운 모습

![del2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2015-03-53.png?raw=true)

댓글 삭제 처리 후 댓글 목록 갱신

![del3](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-13%2015-04-38.png?raw=true)

## 3. 정리
Rest방식으로 AJAX를 통해 댓글 기능 구현을 완료하였다. 일반적으로 `@Controller`를 사용하여 작성하는 방식보다 `@RestController`를 통해 작성하는 방식의 경우 클라이언트에 작성해야할 코드가 많아졌다. 물론 JS템플릿을 통해 코드의 양이 줄어들긴했지만 여전히 많은 양의 코드를 작성해줘야만 했다. 덕분에 설명해야할 포스팅의 양도 많아졌다. 그래서 지금까지 댓글 화면을 구현한 내용 중에서 설명을 빼먹은 부분이나 기억해두어야할 내용들을 간단하게 정리해보자.

- `$.getJSON(uri, 익명함수(결과데이터))` : 서버로부터 원하는 URI를 호출하고, 결과 데이터인 배열객체를 가져와 익명함수의 매개변수에 넣어준다. 익명함수의 내부에는 결과 데이터인 배열객체를 목록형태의 html코드로 출력하는 변환하는 로직을 작성해준다.
- `$.ajax()` : 각각의 기능에 해당하는 적절한 데이터를 JSON으로 변환해 POST, PUT, DELETE 방식으로 서버에 값을 넘겨준다.
- `JSON.stringify()` : 자바스크립트 데이터를 JSON문자열로 변환시켜준다.
- `success : 익명함수(결과데이터){}` : AJAX처리가 성공적으로 처리된 이후 처리할 로직을 작성해준다.
- Handlebars : JS템플릿으로 보다 깔끔하고, 가독성있는 코드를 작성하게 도와준다.
- `Handlebars.registerHelper()` : Handlebars의 확장기능으로 사용자가 데이터를 원하는 방식으로 처리하거나, 변환할 수 있도록 도와준다.
