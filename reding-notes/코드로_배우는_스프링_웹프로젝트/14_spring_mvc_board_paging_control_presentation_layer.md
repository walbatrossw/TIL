
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
---

# Spring MVC 게시판 예제 07 - 페이징처리 : Control, Presentation 계층

## 1. 페이징처리를 위한 컨트롤러 메서드 작성
#### #`ArticleController`
아래와 같이 페이징된 목록의 요청을 처리할 메서드를 아래와 같이 작성해준다.
```java
@RequestMapping(value = "/listCriteria", method = RequestMethod.GET)
public String listCriteria(Model model, Criteria criteria) throws Exception {
    logger.info("listCriteria ...");
    model.addAttribute("articles", articleService.listCriteria(criteria));
    return "/article/list_criteria";
}
```

## 2. 페이징처리를 위한 `JSP`페이지 작성
`/WEB-INF/views/article/`디렉토리에 `list_criteria.jsp`파일을 생성하고, `list.jsp`내용을 전체 복사해 붙여 넣어준다.

#### # 페이징 처리전의 게시글 목록과 처리 후의 게시글 목록 차이 비교

페이지 처리 전의 게시글 목록
![list.jsp](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-01%2020-58-17.png?raw=true)

페이징 처리 후의 게시글 목록1 : 기본값 페이지
![list_criteria.jsp1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-01%2020-58-46.png?raw=true)

페이징 처리 후의 게시글 목록2 : GET파라미터의 page 값이 4일때
![list_criteria.jsp2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-01%2021-01-37.png?raw=true)

페이징 처리 후의 게시글 목록3 : GET파라미터 page값이 4이고, perPageNum의 값이 20일때
![list_criteria.jsp3](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-01%2021-03-30.png?raw=true)

## 3. 목록 하단의 페이지 번호 출력을 위한 계산식 정리
위와 같이 매번 원하는 페이지로 이동하기 위해서 매번 URI를 직접 써서 이동할 수 없기 때문에 화면의 하단에 페이지번호가 출력되는 작업을 진행해보자.

#### # 하단 페이지 번호 출력을 위한 데이터들
화면 하단에 페이지 번호를 출력하기 위해서는 아래와 같은 데이터들이 필요하다.

```
// 시작 페이지 번호는 1
[1] [2] [3] [4] [5] ... [9] [10] [다음]
```
**시작 페이지 번호** : 하단에 출력할 페이지 번호의 갯수가 10이고, 현재 페이지 번호가 1~10사이라면 시작번호는 1이어야한다.
```
// 끝 페이지 번호는 7
[1] [2] [3] [4] [5] [6] [7]
```  
**끝 페이지 번호** : 만약 전체 게시글의 갯수가 만약 65개라면, 끝페이지 번호는 7이어야 한다.
**전체 게시글의 갯수** : 끝 페이지의 번호 계산을 위해서는 전체 게시글의 전체 갯수가 반드시 필요하다.
```
// 이전 페이지 링크
[이전] [11] [12] [13] [14] [15] ... [19] [20]
```
**이전 페이지 링크** : 시작 페이지 번호가 1이 아니라면 이전 페이지를 조회 할 수 있어야 한다.
```
// 다음 페이지 링크
[1] [2] [3] [4] [5] ... [9] [10] [다음]
```
**다음 페이지 링크** : 끝 페이지의 번호 이후에 더 많은 게시글이 존재한다면 다음 페이지를 조회 할 수 있어야 한다.

이제 각 데이터들이 어떻게 계산되는지 정리해보자.

#### # 끝 페이지 번호의 계산
시작페이지 번호부터 계산하는 것보다 끝 페이지 번호를 계산하는 것이 산술적으로 더 편할 수 있다.
```java
// 끝 페이지번호 = Math.ceil(현재페이지 / 페이지 번호의 갯수) * 페이지 번호의 갯수
int endPage = (int) (Math.ceil(criteria.getPage() / (double) displayPageNum) * displayPageNum);
```
예를 들어 위의 계산식을 통해 계산한 결과는 아래의 표와 같다.

|페이지 번호의 갯수|현재 페이지|계산식|끝 페이지 번호|
|---|---|---|---|
|10|3|Math.ceil(3/10) * 10|10|
|10|1|Math.ceil(1/10) * 10|10|
|10|20|Math.ceil(20/10) * 10|20|
|10|21|Math.ceil(21/10) * 10|30|
|20|20|Math.ceil(20/20) * 20|20|
|20|21|Math.ceil(21/20) * 20|40|

#### # 시작 페이지 번호의 계산
끝페이지 번호를 구했다면 시작 페이지 번호를 계산하는 것은 매우 쉽다.
```java
// 시작 페이지 번호 = (끝 페이지 번호 - 페이지 번호의 갯수) + 1
int startPage = (endPage - displayPageNum) + 1;
```
끝 페이지 번호에서 페이지 번호의 갯수를 빼고 1을 더해 주기만 하면 된다. 예를 들어 계산한 결과는 아래의 표와 같다.

|끝 페이지 번호|페이지 번호의 갯수|계산식|시작 페이지 번호|
|---|---|---|---|
|10|10|(10-10)+1|1|
|20|10|(20-10)+1|11|
|30|10|(30-10)+1|21|
|40|20|(40-20)+1|21|
|60|30|(40-20)+1|31|

#### # 전체 게시글의 갯수와 끝 페이지 번호의 보정
끝 페이지 번호는 실제 게시글의 전체 갯수와 관련되어 있기 때문에 다시 한번 계산을 통해 값을 보정해야할 필요가 있다.
```java
// 끝 페이지 번호 계산
int endPage = (int) (Math.ceil(criteria.getPage() / (double) displayPageNum) * displayPageNum);

// 100개의 게시글을 20개씩 보여줄 경우 끝 페이지 번호는 5이어야 함
// 그런데 계산 결과 값은 20???
20 = Math.ceil(1/20) * 20;

// 계산식의 결과 값을 보정하기 위한 또 다른 계산식이 필요하다!
```
예를 들어 100개의 게시글을 10개씩 보여준다면 끝페이지 번호는 10이 되어야 하고, 20개씩 보여주는 경우는 5가 되어야 한다. 그러데 20개씩 보여줄 경우를 계산해보면 20이 나오게 된다. 이렇게 잘못 계산된 내용을 보정하기 위해 아래의 코드를 통해 다시 계산을 하고, 결과 값을 비교한 최종적으로 계산한 결과 값을 끝페이지 번호로 저장하게 된다.
```java
// 끝 페이지 번호 보정 계산식
// 끝 페이지 번호 = Math.ceil(전체 게시글 갯수 / 페이지 당 출력할 게시글의 갯수)
int tempEndPage = (int) (Math.ceil(totalCount / (double) criteria.getPerPageNum()));

// 원하던 결과 같이 나왔다
5 = Math.ceil(100/20);

// 이전의 결과 값과 보정된 결과 값을 비교 후, 보정한 결과 값을 페이지 끝 번호 변수에 저장
if (endPage > tempEndPage) {
  endPage = tempEndPage;
}
```

#### # 이전과 다음 링크의 계산
이전 링크의 경우 시작 페이지 번호가 1인지 아닌지 검사하는 것으로 충분하다. 삼항 연산자를 통해 1이면 `false`값을 아니면 `true`값을 가지도록 하면 된다.
```java
boolean prev = startPage == 1 ? false : true;
```

다음 링크의 경우는 예를 들어 설명해보겠다. 만약 페이지 당 출력할 페이지 번호의 갯수가 10이고, 끝 페이지 번호가 10인 상황에서 전체 게시글의 숫자가 101이라면 다음 링크는 `true`가 되어야 한다. 이 것을 삼항 연산자로 표현하면 다음과 같다.
```java
// 다음 링크 = 끝페이지 * 페이지 당 출력할 게시글의 갯수 >= 전체 게시글의 갯수 ? : false : true;
boolean next = endPage * criteria.getPerPageNum() >= totalCount ? false : true;
// true = 10 * 10 >= 101 ? false : true;
```

## 4. 페이징 처리를 위한 계산 클래스 설계
위에서 정리한 계산식을 직접 JSP에서 처리할 수 있지만, 좀 더 편리하게 사용하기 위해서 별도의 클래스를 설계하여 처리하는 것이 좋다. 이렇게 클래스를 설계하여 처리하면 페이징이 필요한 모든 곳에서 사용할 수 있기 때문에 재사용에 장점을 가지게 된다.

클래스 작성에 앞서 필요한 데이터 변수들을 혼동하지 않도록 다시 한번 체크해보자.

**외부에서 입력되는 데이터**
- `page` : 현재 페이지의 번호
- `perPageNum` : 페이지당 출력할 게시글의 갯수

**DB에서 계산되는 데이터**
- `totalCount` : 전체 게시글의 갯수

**계산식을 통해 만들어지는 데이터**
- `startPage` : 시작 페이지 번호
- `endPage` : 끝 페이지 번호
- `prev` : 이전 링크
- `next` : 다음 링크

#### # `PageMaker`클래스 작성
이제 하단의 페이지 번호 출력을 도와줄 클래스를 아래와 같이 작성한다.
```java
public class PageMaker {

    private int totalCount;
    private int startPage;
    private int endPage;
    private boolean prev;
    private boolean next;

    private int displayPageNum = 10;

    private Criteria criteria;

    public void setCriteria(Criteria criteria) {
        this.criteria = criteria;
    }

    public void setTotalCount(int totalCount) {
        this.totalCount = totalCount;
        calcData();
    }

    private void calcData() {

        endPage = (int) (Math.ceil(criteria.getPage() / (double) displayPageNum) * displayPageNum);

        startPage = (endPage - displayPageNum) + 1;

        int tempEndPage = (int) (Math.ceil(totalCount / (double) criteria.getPerPageNum()));

        if (endPage > tempEndPage) {
            endPage = tempEndPage;
        }

        prev = startPage == 1 ? false : true;

        next = endPage * criteria.getPerPageNum() >= totalCount ? false : true;

    }
}
```
위 코드에서 주목해서 봐야할 점은 아래와 같이 2가지이다.
- `displayPageNum` : 하단의 페이지 번호의 갯수를 의미한다.
- `calcData()` : 게시글의 전체 갯수가 설정되는 시점에 `calcData()`메서드를 호출하여 필요한 데이터들를 계산한다.

## 5. 컨트롤러와 JSP페이지 작성

#### # `ArticleController`
`ArticleController`에 페이지 번호 출력처리가 된 목록페이지를 처리할 메서드를 아래와 같이 작성해준다.
```java
@RequestMapping(value = "/listPaging", method = RequestMethod.GET)
public String listPaging(Model model, Criteria criteria) throws Exception {
    logger.info("listPaging ...");

    PageMaker pageMaker = new PageMaker();
    pageMaker.setCriteria(criteria);
    pageMaker.setTotalCount(1000);

    model.addAttribute("articles", articleService.listCriteria(criteria));
    model.addAttribute("pageMaker", pageMaker);

    return "/article/list_paging";
}
```
위의 코드에 눈여겨 볼 점은 아래와 같다.
- `Criteria`, `Model`타입의 변수 `criteria`와 `model`를 파라미터로 사용한다.
- `Model`객체를 사용하여 `PageMaker`에서 계산한 결과 값을 저장한다.
- 아직 영속계층에서 전체 게시글의 갯수를 구하는 로직을 구현하지 않았기 때문에 `setTotalCount()`의 매개변수는 1000을 임의로 넣어주었다.

#### # `list_paging.jsp`
`/WEB-INF/views/article/`디렉토리에 `list_paging.jsp`파일을 생성하고, `list.jsp`의 내용을 전체 복사한 뒤 붙여 넣는다. 그리고 `<div class="box-body"></div>`태그 바로 밑에 추가적으로 아래의 코드를 작성해준다.
```xml
<div class="box-footer">
    <div class="text-center">
        <ul class="pagination">
            <c:if test="${pageMaker.prev}">
                <li><a href="${path}/article/listPaging?page=${pageMaker.startPage - 1}">이전</a></li>
            </c:if>
            <c:forEach begin="${pageMaker.startPage}" end="${pageMaker.endPage}" var="idx">
                <li <c:out value="${pageMaker.criteria.page == idx ? 'class=active' : ''}"/>>
                    <a href="${path}/article/listPaging?page=${idx}">${idx}</a>
                </li>
            </c:forEach>
            <c:if test="${pageMaker.next && pageMaker.endPage > 0}">
                <li><a href="${path}/article/listPaging?page=${pageMaker.endPage + 1}">다음</a></li>
            </c:if>
        </ul>
    </div>
</div>
```
위의 코드에서 살펴볼 점을 정리한 내용은 아래와 같다.
- `JSTL`의 `<c:if>` 조건문을 통해 이전 링크와 다음 링크의 활성/비활성 처리를 하였다.
- `<c:forEach>` 반복문을 통해 `pageMaker`클래스에서 계산된 페이지 번호를 출력해준다.
- `<c:out>`에서 삼항연산자를 통해 `<li>`태그의 속성을 제어하여 페이지 번호들 중에서 현재 페이지 번호임을 알 수 있게 색을 변경한다.

#### # 페이지 번호 출력까지 한 목록 페이지 화면
첫 페이지와 이전 링크 비활성화
![1](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-02%2000-34-31.png?raw=true)

중간 페이지와 이전, 다음 링크 활성화
![2](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-02%2000-36-00.png?raw=true)

끝 페이지와 다음 링크 비활성화
![3](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/2018-03-02%2000-36-23.png?raw=true)

## 6. 페이징 처리를 위한 전체 게시글의 갯수 구하기
#### # `ArticleDAO`인터페이스, `ArticleDAOImpl`클래스 메서드 추가
아래와 같이 인터페이스에 추상메서드를 추가하고, 구현 클래스에서 메서드를 구현해준다.
```java
int countArticles(Criteria criteria) throws Exception;
```
```java
@Override
public int countArticles(Criteria criteria) throws Exception {
    return sqlSession.selectOne(NAMESPACE + ".countArticles", criteria);
}
```

#### # `articleMapper.xml` SQL작성
아래와 같이 SQL Mapper에 `COUNT`쿼리를 작성해준다.
```sql
<select id="countArticles" resultType="int">
    <![CDATA[
    SELECT
        COUNT(article_no)
    FROM tbl_article
    WHERE article_no > 0
    ]]>
</select>
```

#### # `ArticleService`인터페이스, `ArticleServiceImpl`클래스 메서드 추가
아래와 같이 인터페이스에 추상메서드를 추가하고, 구현 클래스에서 메서드를 구현해준다.
```java
int countArticles(Criteria criteria) throws Exception;
```
```java
@Override
public int countArticles(Criteria criteria) throws Exception {
    return articleDAO.countArticles(criteria);
}
```

#### # `ArticleController` 수정
이전에 임의로 전체 게시글의 숫자를 임의로 넣어 줬던 것을 아래와 같이 변경해준다.
```java
@RequestMapping(value = "/listPaging", method = RequestMethod.GET)
public String listPaging(Model model, Criteria criteria) throws Exception {
    logger.info("listPaging ...");

    PageMaker pageMaker = new PageMaker();
    pageMaker.setCriteria(criteria);
    // 수정
    pageMaker.setTotalCount(articleService.countArticles(criteria));

    model.addAttribute("articles", articleService.listCriteria(criteria));
    model.addAttribute("pageMaker", pageMaker);

    return "/article/list_paging";
}
```

## 7. 페이징 처리 간단 요약 정리
지금까지 페이징 처리구현을 위해 컨트롤러와 뷰까지 구현을 완료해보았는데 기억해두어야할 사항들을 간단 요약 정리해보자.
- 페이지 번호 출력처리를 위한 데이터들 : 시작 페이지 번호, 끝 페이지 번호, 전체 게시글의 갯수, 이전 링크, 다음 링크
- 끝 페이지 번호 계산식 : `Math.ceil(현재 페이지 번호 / 페이지 번호의 갯수) * 페이지번호의 갯수`
- 시작 페이지 번호 계산식 : `(끝 페이지 번호 - 페이지 번호의 갯수) + 1`
- 끝 페이지 번호의 보정 계산식 : `Math.ceil(전체 게시글의 갯수 / 페이지 당 출력할 게시글의 갯수)`
- 이전 링크 활성/비활성 계산식 : `시작 페이지 번호 == 1 ? fales : true`
- 다음 링크 활성/비활성 계산식 : `끝 페이지 번호 * 페이지 당 출력할 게시글 갯수 >= 전체 게시글의 갯수 ? fales : true`
