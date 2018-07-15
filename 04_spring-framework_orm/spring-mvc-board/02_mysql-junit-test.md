# Spring MVC - MySQL 연결테스트

본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.

그리고 본 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정 파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](http://doublesprogramming.tistory.com/171?category=667155) 포스팅을 참고해주시면 감사하겠습니다.


포스팅하고 있는 현재 프로젝트의 예제가 혹시 필요하신 분은 github(https://github.com/walbatrossw/spring-mvc-ex)를 통해 얻으실 수 있습니다.

---

## 1. MySQL 설치 및 DB추가

이전의 스프링 예제에서는 Oracle을 사용했지만 앞으로 예제는 MySQL을 사용하기 때문에 MySQL을 설치하고, 스키마(DB)를 추가해주었다.
```sql
-- spring_ex 스키마(데이터베이스) 생성
CREATE SCHEMA `spring_ex` DEFAULT CHARACTER SET utf8 COLLATE utf8_bin;
```

이제 java를 이용해 JDBC연결이 정상적으로 이루어지는지 테스트를 하기 앞서 기본적으로 `jUnit`에 대해 간단히 정리해보자.

## 2. jUnit - 기본적으로 알아야할 사항
* `@Test` : 테스트할 내용을 메서드 안에 작성하고 `@Test` 애너테이션을 추가하면 테스트용 코드로 간주하고 테스트를 진행
* `@Before` : 모든 테스트 작업에 앞서 준비되어야 하는 내용을 작성해서 메서드에 추가하는 애너테이션
* `@After` : 테스트 작업이 끝난 후 자동으로 실행되는 메서드에 추가하는 애너테이션
* `org.junit.Assert.assertxxx` : 테스트 중에 발생되는 값을 확신하는 용도로 테스트 중간엔 특정 값이나 상태를 예상하고, 체크하는 용도

## 3. MySQL 라이브러리 추가
`pom.xml`에 MySQL라이브러리를 추가해준다.
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.41</version>
</dependency>
```

## 3. JDBC 연결 테스트 코드 작성
`test/java/하위패키지`에 테스트 클래스를 생성하고 아래와 같이 작성한다.
```java
public class MySQLConnectionTest {

    private static final String DRIVER = "com.mysql.jdbc.Driver";
    private static final String URL = "jdbc:mysql://127.0.0.1:3306/spring_ex?useSSL=false&serverTimezone=Asia/Seoul";
    private static final String USER = "아이디";
    private static final String PASSWORD = "비밀번호";

    @Test
    public void testConnection() throws Exception {
        Class.forName(DRIVER);
        try(Connection connection = DriverManager.getConnection(URL, USER, PASSWORD)) {
            System.out.println(connection);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

정상적으로 테스트가 완료되었다면 콘솔에 아래와 같이 출력될 것이다.

```
com.mysql.jdbc.JDBC4Connection@f5f2bb7 // Connection 객체 생성
```
