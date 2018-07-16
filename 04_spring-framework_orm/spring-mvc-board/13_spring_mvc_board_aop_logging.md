
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

---

# Spring MVC 게시판 예제13  - AOP를 이용한 LogAdvice 작성

AOP 정리 포스팅 (http://doublesprogramming.tistory.com/115)
이전에도 AOP를 정리한 글을 포스팅한 적이 있지만 다시 복습하는 차원에서 다시 작성하려고한다.

## 1. AOP(Aspcet Orient Programming)

#### # AOP란?
AOP용어 자체를 한글로 바꾸면 측면 지향 프로그래밍이다. 측면이 개발에서 의미하는 것은 비지니스 로직은 아니지만, 반드시 해야할 작업을 말한다. 다시 쉽게 말하자면 정말 기능을 구현하는데 있어서 핵심적인 작업은 아니지만 반드시 해야 되는 공통된 작업이라고 할 수 있다. 이러한 것들을 횡단관심사(cross-concern)라고 한다. 횡단관심사는 시스템의 곳곳에서 공통으로 사용되지만, 그 자체가 목적이 아니다. 다만 시스템의 완성도를 높여주는 역할을 해준다. 예를 들면 보안처리나 로그 기록, 트랜잭션과 같은 번거로운 작업들을 말한다.

![](http://cfile28.uf.tistory.com/image/223A844A58BAC9020CEBD4)
*사진출처(http://addio3305.tistory.com/86)*

위 그림을 통해 설명하자면 일반적인 로직의 흐름에서는 계정, 게시판, 계좌이체를 처리할 때마다 동일하게 권한을 체크하고, 로그를 기록하고, 트랜잭션을 처리해줘야하기 때문에 동일한 코드가 반복적으로 삽입된다. 하지만 AOP는 이러한 횡단 관심사를 종단으로 삽입할 수 있게 한다. 기존에는 각각의 계층에서 직접 코드로 작성하여 처리하던 것들을 공통적인 관심을 처리하는 모듈을 분리해 개발한 뒤, 필요한 시점에 자동으로 소스코드가 삽입되도록 하는 것이다.

#### # AOP와 관련된 용어
```

외부 호출 -------------->  Proxy 객체 ---> Target 객체

```

AOP의 구현은 Porxy패턴이라는 방식을 통해 이루어진다. 외부에서 특정한 객체를 호출하면, 실제 객체(Target)를 감싸고 있는 바깥쪽 객체(Proxy)를 통해 호출이 전달된다. Proxy객체는 AOP의 기능이 적용된 상태에서 호출을 받아 사용되고, 실제 객체와 동일한 타입을 자동으로 생성할 수 있기 때문에 외부에서는 실제 객체와 동일한 타입으로 호출할 수 있다.

AOP를 이해하기 위해서는 아래의 용어의 이해가 필수적이다.

|용어|설명|
|---|---|
|Aspect|공통 관심사에 대한 추상적인 명칭, 예를 들어 로깅이나 보안, 트랜잭션과 같은 기능 자체에 대한 용어|
|Advice|실제로 기능을 구현한 객체|
|Join points|공통 관심사를 적용할 수 있는 대상, Spring AOP에서는 각 객체의 메서드가 해당됨|
|Pointcuts|여러 메서드 중 실제 Advice가 적용될 메서드|
|target|대상 메서드가 가지는 객체|
|Proxy|Advice가 적용되었을 때 만들어지는 객체|
|Introduction|target에는 없는 새로운 메서드나 인스턴스 변수를 추가하는 기능|
|Weaving|Advice나 target이 결합되어서 프록시 객체를 만드는 과정|

좀더 자세한 설명이 필요한 용어는 아래와 같다.

- **Advice** : 실제로 적용 시키고 싶은 코드 자체를 의미한다. 개발자가 만드는 것은 Aspcect가 아닌 클래스를 제작하고 `@Advice`애너테이션을 적용하는 것이다. 예를 들면 로그 출력기능, 파라미터 체크 기능 자체는 Aspect라는 용어로 부르지만 실제 구현 시에는 Adice를 제작한다고 표현한다.

- **Target** : 실제 비지니스 로직을 수행하는 객체를 의미한다. Aspect를 적용해야하는 대상 객체를 의미한다.

- **Join points** : 작성된 Advice가 작동되는 위치를 의미한다. 예를 들면 `ArticleService`의 등록, 수정, 삭제만을 골라서 Advice를 적용할 수 있는데 이 때 `ArticleService`의 모든 메서드는 JoinPoint가 된다.

- **Pointcuts** : 여러 Join points 중에서 Advice를 적용할 대상을 선택하는 정보를 의미한다. 이를 통해 특정 메서드는 Advice가 적용된 형태로 동작하게 된다.

#### # Advice의 종류
Advice의 타입은 아래와 같이 분류된다.
|타입|기능|
|---|---|
|Before Advice|target의 메서드 호출 전에 사용|
|After returning|target의 메서드 호출 이후에 사용|
|After throwing|target의 예외 발생 후에 적용|
|After|target의 메서드 호출 후 예외의 발생에 관계없이 적용|
|Around|target의 메서드 호출 이전과 이후 모두 적용|

## 2. AOP 적용을 위한 준비

#### # AOP와 관련된 라이브러리 추가 : `pom.xml`
Spring-AOP 라이브러리를 추가해준다.
```xml
<!-- AOP -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
```

AOP기능을 적용하기 위해서 AspectJ 언어의 문법을 이용하기 때문에 이와 관련된 라이브러리를 추가해준다.
```xml
<!-- AspectJ -->
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjrt</artifactId>
   <version>${org.aspectj-version}</version>
</dependency>
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjtools</artifactId>
   <version>${org.aspectj-version}</version>
</dependency>
```

#### # AOP 설정 : `dispatcher-servlet.xml`
`dispatcher-servlet.xml`에 아래와 같은 설정을 추가한다. 아래의 설정을 통해 자동으로 AspectJ 라이브러리를 통해 Porxy객체를 생성하는 용도로 사용된다.
```xml
<aop:aspectj-autoproxy/>
```

## 3. 게시판 예제에 AOP 적용하기

#### # AOP를 이용하여 로그 출력을 도와줄 `LogAdivice`클래스 작성
AOP를 이용하여 로그를 출력하는 기능을 처리해보자. `/src/java/main/기본패키지/commons/aop`패키지에 아래와 `LogAdvice`를 생성하고, 아래와 같이 코드를 작성해준다. 각 계층에서 로직이 실행될 때 마다 클래스 메서드명, 파라미터, 실행시간을 출력하도록 하였다.
```java
@Component
@Aspect
public class LogAdvice {

    private static final Logger logger = LoggerFactory.getLogger(LogAdvice.class);

    @Around("execution(* com.doubles.mvcboard..*Controller.*(..))"
            + " or execution(* com.doubles.mvcboard..service..*Impl.*(..))"
            + " or execution(* com.doubles.mvcboard..persistence..*Impl.*(..))")
    public Object logPrint(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {


        long start = System.currentTimeMillis();

        Object result = proceedingJoinPoint.proceed();

        String type = proceedingJoinPoint.getSignature().getDeclaringTypeName();
        String name = "";

        if (type.contains("Controller")) {
            name = "Controller : ";

        } else if (type.contains("Service")) {
            name = "Service : ";

        } else if (type.contains("DAO")) {
            name = "Persistence : ";
        }

        long end = System.currentTimeMillis();

        logger.info(name + type + "."+proceedingJoinPoint.getSignature().getName() + "()");
        logger.info("Argument/Parameter : " + Arrays.toString(proceedingJoinPoint.getArgs()));
        logger.info("Running Time : " + (end-start));
        logger.info("----------------------------------------------------------------");

        return result;
    }

}
```

위의 코드에서 주목해서 봐야할 점이나 알아두어야할 내용은 아래와 같다.
- `@Component` : 스프링 빈으로 인식되기 위한 애너테이션
- `@Aspect` : AOP 기능을 하는 클래스에 반드시 추가해야할 애너테이션
- `@Around` : 메서드 실행 전체를 앞, 뒤로 감싸서 특정한 기능을 실행할 수 있는 강력한 타입의 Advice
- `execution` : Pointcut을 지정하는 문법으로 AspectJ 언어 문법을 사용한다. AOP를 적용하는데 있어서 가장 주의해야할 부분
- **리턴타입 `Object`** : 다른 Advice와 달리 `@Around`는 반드시 리턴타입을 `Object`로 선언해줘야만 한다.
- `ProceedingJoinPoint` : Around타입의 Advice메서드의 파라미터로 사용되는 인터페이스, `JoinPoint`의 하위 인터페이스
- `proceed()` : 다음 Advice를 실행하거나, 실제 target객체의 메서드를 실행하는 기능을 가진 메서드
- `getSignature()`: 실행하는 대상 객체의 메서드에 대한 정보를 알아낼 때 사용한다.
- `getArgs` : 전달되는 모든 파라미터들을 `Object`의 배열로 가져온다.

#### # 특정 로직을 실행했을 때의 콘솔에 찍힌 로그
AOP를 적용하고 난 뒤 게시글을 작성하고, 게시글 목록으로 이동했을 때 콘솔에 출력된 로그 내용이다. 컨트롤러부터 시작해서 영속계층까지 실행된 모든 메서드와 파라미터, 실행시간을 출력해주고 있다.
```
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Controller : com.doubles.mvcboard.article.controller.ArticlePagingSearchController.writeGET()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : []
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 0
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Persistence : com.doubles.mvcboard.article.persistence.ArticleDAO.create()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [ArticleVO{articleNo=null, title='안녕하세요', content='반갑습니다 ^^', writer='더블에스', regDate=null, viewCnt=0}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 6
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Service : com.doubles.mvcboard.article.service.ArticleService.create()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [ArticleVO{articleNo=null, title='안녕하세요', content='반갑습니다 ^^', writer='더블에스', regDate=null, viewCnt=0}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 6
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Controller : com.doubles.mvcboard.article.controller.ArticlePagingSearchController.writePOST()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [ArticleVO{articleNo=null, title='안녕하세요', content='반갑습니다 ^^', writer='더블에스', regDate=null, viewCnt=0}, {}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 7
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Persistence : com.doubles.mvcboard.article.persistence.ArticleDAO.countSearchedArticles()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [SearchCriteria{searchType='null', keyword='null'}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 3
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Service : com.doubles.mvcboard.article.service.ArticleService.countSearchedArticles()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [SearchCriteria{searchType='null', keyword='null'}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 3
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Persistence : com.doubles.mvcboard.article.persistence.ArticleDAO.listSearch()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [SearchCriteria{searchType='null', keyword='null'}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 8
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Service : com.doubles.mvcboard.article.service.ArticleService.listSearch()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [SearchCriteria{searchType='null', keyword='null'}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 9
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Controller : com.doubles.mvcboard.article.controller.ArticlePagingSearchController.list()
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Argument/Parameter : [SearchCriteria{searchType='null', keyword='null'}, {msg=regSuccess, searchCriteria=SearchCriteria{searchType='null', keyword='null'}, org.springframework.validation.BindingResult.searchCriteria=org.springframework.validation.BeanPropertyBindingResult: 0 errors, articles=[ArticleVO{articleNo=1003, title='안녕하세요', content='반갑습니다 ^^', writer='더블에스', regDate=Wed Mar 14 17:06:26 KST 2018, viewCnt=0}, ArticleVO{articleNo=1001, title='asd', content='asd', writer='asd', regDate=Sat Mar 03 21:30:25 KST 2018, viewCnt=0}, ArticleVO{articleNo=1000, title='1000번째 글 제목입니다...', content='1000번재 글 내용입니다...', writer='user00', regDate=Wed Feb 28 18:19:52 KST 2018, viewCnt=0}, ArticleVO{articleNo=999, title='999번째 글 제목입니다...', content='999번재 글 내용입니다...', writer='user09', regDate=Wed Feb 28 18:19:52 KST 2018, viewCnt=0}, ArticleVO{articleNo=998, title='998번째 글 제목입니다...', content='998번재 글 내용입니다...', writer='user08', regDate=Wed Feb 28 18:19:52 KST 2018, viewCnt=0}, ArticleVO{articleNo=997, title='997번째 글 제목입니다...', content='997번재 글 내용입니다...', writer='user07', regDate=Wed Feb 28 18:19:52 KST 2018, viewCnt=0}, ArticleVO{articleNo=995, title='995번째 글 제목입니다...', content='995번재 글 내용입니다...', writer='user05', regDate=Wed Feb 28 18:19:52 KST 2018, viewCnt=0}, ArticleVO{articleNo=993, title='993번째 글 제목입니다...', content='993번재 글 내용입니다...', writer='user03', regDate=Wed Feb 28 18:19:52 KST 2018, viewCnt=0}, ArticleVO{articleNo=992, title='992번째 글 제목입니다...', content='992번재 글 내용입니다...', writer='user02', regDate=Wed Feb 28 18:19:52 KST 2018, viewCnt=0}, ArticleVO{articleNo=990, title='990번째 글 제목입니다...', content='990번재 글 내용입니다...', writer='user00', regDate=Wed Feb 28 18:19:52 KST 2018, viewCnt=0}], pageMaker=PageMaker{totalCount=996, startPage=1, endPage=10, prev=false, next=true, displayPageNum=10, criteria=SearchCriteria{searchType='null', keyword='null'}}}]
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - Running Time : 13
INFO : com.doubles.mvcboard.commons.aop.LogAdvice - ----------------------------------------------------------------
```
