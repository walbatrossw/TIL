본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.

본 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](http://doublesprogramming.tistory.com/171?category=667155) 포스팅을 참고해주시면 감사하겠습니다.

# SpringMVC + MyBatis

앞서 포스팅한 [MyBatis 설정 및 테스트](http://doublesprogramming.tistory.com/173?category=667155)에 이어서 MyBatis를 이용하여 회원을 입력하고, 조회(아이디, 아이디+비밀번호)하는 간단한 기능을 구현해보고, 제대로 구현되었는지 테스트까지 진행해보자.

## 01) MyBatis의 개발 방식 정리

MyBatis는 JDBC에서 개발자가 직접 처리하는 `PreparedStatement`의 '?'에 대한 설정이나 `ResultSet`을 이용한 처리가 이루어지기 때문에 기존 방식에 비해 개발 생산성이 좋아진다.

- iBatis : 개발자가 모든 SQL을 XML로 작성하고, SQL문을 사용하는 DAO클래스를 설계해서 SQL문을 호출하는 방식의 코드를 작성
- MyBatis : iBatis에서 발전된 형태로 애너테이션을 지원하고, 인터페이스와 애너테이션을 통해 SQL문을 설정하고 처리할 수 있음
- MyBatis를 이용할 때 SQL문을 사용하는 방식의 종류

  1. XML만을 이용하는 방식
      - 장점 : SQL문은 별도의 XML로 작성되기 때문에 SQL문의 수정이나 유지보수에 적합
      - 단점 : 개발시 코드의 양이 많아지고, 복잡성 증가

  2. 애너테이션과 인터페이스만을 이용하는 방식
      - 장점 : 별도의 DAO없이도 개발이 가능하기 때문에 생산성 증가
      - 단점 : SQL문을 애너테이션으로 작성하기 때문에 매번 수정이 일어나는 경우 다시 컴파일

  3. 인터페이스와 XML로 작성된 SQL문을 활용하는 방식
      - 장점 : 간단한 SQL문은 애너테이션으로, 복잡한 SQL문은 XML로 처리하므로, 상황에 따라 유연하게 처리
      - 단점 : 개발자에 따라 개발방식의 차이가 존재하기 때문에 유지보수가 중요한 프로젝트의 경우 부적합

## 02) XML만을 이용하는 방식 구현해보기

앞서 언급한 것처럼 XML만을 이용하는 방식은 SQL문을 완전히 분리해서 처리하기 때문에 SQL문의 변경이 일어날 때, 대처가 수월해 유지보수에 적합한 장점을 가지고 있다. 그래서 대부분의 프로젝트에서 XML만을 이용해서 SQL문을 작성하고, 별도의 DAO를 만드는 방식을 선호한다고 한다.

자 그럼 이제 본격적으로 구현한 것을 정리해보자.

#### 1. DB(MySQL) 테이블 생성 : 기존에 생성한 스키마 `spring_ex`에 회원 테이블을 추가해준다.
```sql
CREATE TABLE tbl_member (
  userid     VARCHAR(50) NOT NULL, -- 회원아이디
  userpw     VARCHAR(50) NOT NULL, -- 회원비밀번호
  username   VARCHAR(50) NOT NULL, -- 회원이름
  email      VARCHAR(100), -- 회원이메일
  regdate    TIMESTAMP DEFAULT now(), -- 가입일자
  updatedate TIMESTAMP DEFAULT now(), -- 수정일자
  PRIMARY KEY (userid)  -- 기본키 : userid
)
```

#### 2. 도메인 객체를 위한 클래스 설계 : domain패키지를 생성하고 회원을 의미하는 `MemberVO` 클래스를 작성한다.
```java
public class MemberVO {

    private String userid; // 회원아이디
    private String userpw; // 회원 비밀번호
    private String username; // 회원이름
    private String email; // 이메일
    private Date regdate; // 가입일자
    private Date updatedate; // 수정일자

    // getter, setter, toString() ...
}
```
#### 3. DAO 인터페이스 작성 : persistence 패키지를 생성하고 `MemberDAO` 인터페이스 생성하고, 기능을 처리할 메서드를 작성해준다.
```java
public interface MemberDAO {

    // 현재시간체크
    public String getTime();

    // 회원 입력
    public void insertMember(MemberVO memberVO);

    // 회원아이디로 조회
    public MemberVO readMember(String userid) throws Exception;

    // 회원아이디, 비밀번호로 조회
    public MemberVO readWithPW(String userid, String userpw) throws Exception;

}
```
#### 4. XML Mapper 작성 : /main/resources/mappers/ 디렉토리에 `memberMapper.xml`를 아래와 같이 작성해준다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.doubles.ex00.mapper.MemberMapper">

    <select id="getTime" resultType="string">
        SELECT NOW()
    </select>

    <insert id="insertMember">
        INSERT INTO tbl_member (
            userid
            , userpw
            , username
            , email
        ) VALUES (
            #{userid}
            , #{userpw}
            , #{username}
            , #{email}
        )
    </insert>

    <select id="selectMember" resultType="com.doubles.ex00.domain.MemberVO">
        SELECT *
        FROM tbl_member
        WHERE userid = #{userid}
    </select>

    <select id="readWithPW" resultType="com.doubles.ex00.domain.MemberVO">
        SELECT *
        FROM tbl_member
        WHERE userid = #{userid} AND userpw = #{userpw}
    </select>

</mapper>
```
* `<mapper namespace="속성">`은 클래스의 패키지와 유사한 용도로, MyBatis내에서 원하는 SQL문을 찾아서 실행할 때 동작하므로 설정에 신경을 써줘야한다.

#### 5. `myBatis-Spring`에서 XML Mapper 인식시키기 : `applicationContext.xml`(IntelliJ), `root-context.xml`(STS)에 아래와 같이 Mapper의 경로를 작성해준다.
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:/mybatis-config.xml"/>
    <!-- Mapper 경로 -->
    <property name="mapperLocations" value="classpath:mappers/**/*Mapper.xml"/>
</bean>
```
* `value="classpath:mappers/**/*Mapper.xml` : 추후에 추가될 여러 디렉토리에 존재하는 Mapper들을 모두 인식할 수 있도록 작성한다.

#### 6. `SqlSessionTemplate` 설정 : `SqlSessionTemplate`는 `SqlSessionFactory`를 생성자로 주입하도록 설정해준다.
```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
    <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>
```
* `SqlSessionTemplate` : 보통 DAO의 작업에서 가장 번거로운 작업은 DB와 연결을 맺고, 작업을 완료한 뒤 연결을 종료하는 작업인데 `mybatis-spring`라이브러리는 `SqlSessionTemplate`라는 클래스를 제공하여 직접 연결하고, 연결을 종료하는 반복적인 작업을 줄여준다.

#### 7. DAO 인터페이스 구현하기 : 앞서 작성한 `MemberDAO`인터페이스를 구현하는 클래스를 작성하고 `SqlSessionTemplate`를 주입받아 사용한다.
```java
@Repository
public class MemberDAOImpl implements MemberDAO {
    @Inject
    SqlSession sqlSession;

    private static final String NAMESPACE = "com.doubles.ex00.mapper.MemberMapper";

    @Override
    public String getTime() {
        return sqlSession.selectOne(NAMESPACE + ".getTime");
    }

    @Override
    public void insertMember(MemberVO memberVO) {
        sqlSession.insert(NAMESPACE + ".insertMember", memberVO);
    }

    @Override
    public MemberVO readMember(String userid) throws Exception {
        return (MemberVO) sqlSession.selectOne(NAMESPACE + ".selectMember", userid);
    }

    @Override
    public MemberVO readWithPW(String userid, String userpw) throws Exception {
        Map<String, Object> paraMap = new HashMap<String, Object>();
        paraMap.put("userid", userid);
        paraMap.put("userpw", userpw);
        return sqlSession.selectOne(NAMESPACE + ".readWithPW", paraMap);
    }
}
```
* 위의 코드에서 주목해야할 점은 클래스 선언부에 작성된 `@Repository`애너테이션이다. DAO를 Spring에 인식시키기 위해 사용한다.

#### 7. Spring에 Bean으로 등록하기 : `applicationContext.xml`(IntelliJ), `root-context.xml`(STS)에 아래와 같이 작성해준다.
```xml
<context:component-scan base-package="com.doubles.ex00.persistence"/>
```
* `MemberDAOImpl`을 `@Repository`애너테이션을 통해 DAO로 인식시키더라도 Spring이 해당패키지를 스캔하지 않으면 Bean으로 등록되지 못하기 때문에 반드시 작성해줘야한다. 이 부분을 놓쳐서 계속 오류를 찾느라고 며칠을 고생하기 때문에 반드시 기억해두자.

#### 8. MyBatis 로그(`log4jdbc-log4j2`) : MyBatis의 로그를 보다 자세히 조사할 수 있도록 로그를 설정해준다.
* `log4jdbc-log4j2` 라이브러리 추가
  ```xml
  <dependency>
      <groupId>org.bgee.log4jdbc-log4j2</groupId>
      <artifactId>log4jdbc-log4j2-jdbc4</artifactId>
      <version>1.16</version>
  </dependency>
  ```
* `applicationContext.xml`(IntelliJ), `root-context.xml`(STS) : `DataSource`부분을 아래와 같이 수정
  ```xml
  <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"/>
      <property name="url" value="jdbc:log4jdbc:mysql://127.0.0.1:3306/spring_ex?useSSL=false"/>
      <property name="username" value="아이디"/>
      <property name="password" value="비밀번호"/>
  </bean>
  ```
* `log4jdbc.log4j2.properties` : /src/main/resources 디렉토리에 아래와 같이 파일을 생성하고 작성
  ```
  log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
  ```
* `logback.xml` : /src/main/resources 디렉토리에 아래와 같이 파일을 생성하고 작성
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <configuration>
      <include resource="org/springframework/boot/logging/logback/base.xml"/>
      <!-- log4jdbc-log4j2 -->
      <logger name="jdbc.sqlonly"        level="DEBUG"/>
      <logger name="jdbc.sqltiming"      level="INFO"/>
      <logger name="jdbc.audit"          level="WARN"/>
      <logger name="jdbc.resultset"      level="ERROR"/>
      <logger name="jdbc.resultsettable" level="ERROR"/>
      <logger name="jdbc.connection"     level="INFO"/>
  </configuration>
  ```

#### 9. 테스트 코드 작성 : 각각의 기능이 제대로 구현됬는지 확인하기 위해 테스트 코드를 작성해보자.
* `log4j.xml` : test/resources 디렉토리에 아래와 같이 작성해야 위에 설정한 로그가 정상적으로 출력
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE log4j:configuration PUBLIC "-//APACHE//DTD LOG4J 1.2//EN" "log4j.dtd">
  <log4j:configuration xmlns:log4j="http://jakarta.apache.org/log4j/">

      <!-- Appenders -->
      <appender name="console" class="org.apache.log4j.ConsoleAppender">
          <param name="Target" value="System.out"/>
          <layout class="org.apache.log4j.PatternLayout">
              <param name="ConversionPattern" value="%-5p: %c - %m%n"/>
          </layout>
      </appender>

      <!-- Application Loggers -->
      <logger name="com.doubles.ex00">
          <level value="info"/>
      </logger>

      <!-- 3rdparty Loggers -->
      <logger name="org.springframework.core">
          <level value="info"/>
      </logger>

      <logger name="org.springframework.beans">
          <level value="info"/>
      </logger>

      <logger name="org.springframework.context">
          <level value="info"/>
      </logger>

      <logger name="org.springframework.web">
          <level value="info"/>
      </logger>

      <!-- Root Logger -->
      <root>
          <priority value="info"/>
          <appender-ref ref="console"/>
      </root>

  </log4j:configuration>
  ```

* `MemberDAOTest` 클래스 : test/java/하위패키지에 아래와 같이 테스트 코드를 작성
  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring/applicationContext.xml"})
  public class MemberDAOTest {

      @Inject
      private MemberDAO memberDAO;

      // 현재시간 출력 테스트
      @Test
      public void testTime() throws Exception {
          System.out.println(memberDAO.getTime());
      }

      // 회원입력 테스트
      @Test
      public void testInsertMember() throws Exception {
          MemberVO memberVO = new MemberVO();
          memberVO.setUserid("user00");
          memberVO.setUserpw("user00");
          memberVO.setUsername("user00");
          memberVO.setEmail("user00@mail.com");

          memberDAO.insertMember(memberVO);
      }

      // 회원조회 테스트 1 : 아이디
      @Test
      public void testReadMember() throws Exception {
          memberDAO.readMember("user00");
      }

      // 회원조회 테스트 2 : 아이디 + 비밀번호
      @Test
      public void testReadWithPW() throws Exception {
          memberDAO.readWithPW("user00", "user00");
      }
  }
  ```
* 테스트 완료 후 콘솔화면 : 아래와 같이 일련의 과정이 깔끔하게 출력된다.
  * `testTime()`
    ```
    INFO : jdbc.connection - 1. Connection opened
    INFO : jdbc.audit - 1. Connection.new Connection returned
    INFO : jdbc.audit - 1. Connection.getAutoCommit() returned true
    INFO : jdbc.audit - 1. PreparedStatement.new PreparedStatement returned
    INFO : jdbc.audit - 1. Connection.prepareStatement(SELECT NOW()) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@4686afc2
    INFO : jdbc.sqlonly - SELECT NOW()

    INFO : jdbc.sqltiming - SELECT NOW()
     {executed in 0 msec}
    INFO : jdbc.audit - 1. PreparedStatement.execute() returned true
    INFO : jdbc.resultset - 1. ResultSet.new ResultSet returned
    INFO : jdbc.audit - 1. PreparedStatement.getResultSet() returned net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy@6340e5f0
    INFO : jdbc.resultset - 1. ResultSet.getMetaData() returned com.mysql.jdbc.ResultSetMetaData@1ffaf86 - Field level information:
    	com.mysql.jdbc.Field@6574a52c[catalog=,tableName=,originalTableName=,columnName=NOW(),originalColumnName=,mysqlType=12(FIELD_TYPE_DATETIME),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
    INFO : jdbc.resultset - 1. ResultSet.getType() returned 1003
    INFO : jdbc.resultset - 1. ResultSet.next() returned true
    INFO : jdbc.resultset - 1. ResultSet.getString(NOW()) returned 2017-11-26 22:23:43.0
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultsettable -
    |----------------------|
    |now()                 |
    |----------------------|
    |2017-11-26 22:23:43.0 |
    |----------------------|

    INFO : jdbc.resultset - 1. ResultSet.next() returned false
    INFO : jdbc.resultset - 1. ResultSet.close() returned void
    INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@644baf4a
    INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@7526515b
    INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
    INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
    INFO : jdbc.audit - 1. PreparedStatement.close() returned
    INFO : jdbc.connection - 1. Connection closed
    INFO : jdbc.audit - 1. Connection.close() returned
    2017-11-26 22:23:43.0
    ```
  * `testInsertMember()`
    ```
    INFO : jdbc.connection - 1. Connection opened
    INFO : jdbc.audit - 1. Connection.new Connection returned
    INFO : jdbc.audit - 1. Connection.getAutoCommit() returned true
    INFO : jdbc.audit - 1. PreparedStatement.new PreparedStatement returned
    INFO : jdbc.audit - 1. Connection.prepareStatement(INSERT INTO tbl_member (
                userid
                , userpw
                , username
                , email
            ) VALUES (
                ?
                , ?
                , ?
                , ?
            )) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@537f60bf
    INFO : jdbc.audit - 1. PreparedStatement.setString(1, "user00") returned
    INFO : jdbc.audit - 1. PreparedStatement.setString(2, "user00") returned
    INFO : jdbc.audit - 1. PreparedStatement.setString(3, "user00") returned
    INFO : jdbc.audit - 1. PreparedStatement.setString(4, "user00@mail.com") returned
    INFO : jdbc.sqlonly - INSERT INTO tbl_member ( userid , userpw , username , email ) VALUES ( 'user00' , 'user00'
    , 'user00' , 'user00@mail.com' )

    INFO : jdbc.sqltiming - INSERT INTO tbl_member ( userid , userpw , username , email ) VALUES ( 'user00' , 'user00'
    , 'user00' , 'user00@mail.com' )
     {executed in 4 msec}
    INFO : jdbc.audit - 1. PreparedStatement.execute() returned false
    INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned 1
    INFO : jdbc.audit - 1. PreparedStatement.close() returned
    INFO : jdbc.connection - 1. Connection closed
    INFO : jdbc.audit - 1. Connection.close() returned
    ```
  * `testReadMember()`
    ```
    INFO : jdbc.connection - 1. Connection opened
    INFO : jdbc.audit - 1. Connection.new Connection returned
    INFO : jdbc.audit - 1. Connection.getAutoCommit() returned true
    INFO : jdbc.audit - 1. PreparedStatement.new PreparedStatement returned
    INFO : jdbc.audit - 1. Connection.prepareStatement(SELECT *
            FROM tbl_member
            WHERE userid = ?) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@4686afc2
    INFO : jdbc.audit - 1. PreparedStatement.setString(1, "user00") returned
    INFO : jdbc.sqlonly - SELECT * FROM tbl_member WHERE userid = 'user00'

    INFO : jdbc.sqltiming - SELECT * FROM tbl_member WHERE userid = 'user00'
     {executed in 0 msec}
    INFO : jdbc.audit - 1. PreparedStatement.execute() returned true
    INFO : jdbc.resultset - 1. ResultSet.new ResultSet returned
    INFO : jdbc.audit - 1. PreparedStatement.getResultSet() returned net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy@6340e5f0
    INFO : jdbc.resultset - 1. ResultSet.getMetaData() returned com.mysql.jdbc.ResultSetMetaData@1ffaf86 - Field level information:
    	com.mysql.jdbc.Field@6574a52c[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=userid,originalColumnName=userid,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= PRIMARY_KEY BINARY, charsetIndex=83, charsetName=UTF-8]
    	com.mysql.jdbc.Field@6c1a5b54[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=userpw,originalColumnName=userpw,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
    	com.mysql.jdbc.Field@1c7696c6[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=username,originalColumnName=username,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
    	com.mysql.jdbc.Field@60099951[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=email,originalColumnName=email,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
    	com.mysql.jdbc.Field@20140db9[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=regdate,originalColumnName=regdate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
    	com.mysql.jdbc.Field@1e6a3214[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=updatedate,originalColumnName=updatedate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
    INFO : jdbc.resultset - 1. ResultSet.getType() returned 1003
    INFO : jdbc.resultset - 1. ResultSet.next() returned true
    INFO : jdbc.resultset - 1. ResultSet.getString(userid) returned user00
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getString(userpw) returned user00
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getString(username) returned user00
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getString(email) returned user00@mail.com
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getTimestamp(regdate) returned 2017-11-26 22:27:40.0
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getTimestamp(updatedate) returned 2017-11-26 22:27:40.0
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultsettable -
    |-------|-------|---------|----------------|----------------------|----------------------|
    |userid |userpw |username |email           |regdate               |updatedate            |
    |-------|-------|---------|----------------|----------------------|----------------------|
    |user00 |user00 |user00   |user00@mail.com |2017-11-26 22:27:40.0 |2017-11-26 22:27:40.0 |
    |-------|-------|---------|----------------|----------------------|----------------------|

    INFO : jdbc.resultset - 1. ResultSet.next() returned false
    INFO : jdbc.resultset - 1. ResultSet.close() returned void
    INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@54c5a2ff
    INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@6d4d66d2
    INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
    INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
    INFO : jdbc.audit - 1. PreparedStatement.close() returned
    INFO : jdbc.connection - 1. Connection closed
    INFO : jdbc.audit - 1. Connection.close() returned
    ```
  * `testReadWithPW()`
    ```
    INFO : jdbc.connection - 1. Connection opened
    INFO : jdbc.audit - 1. Connection.new Connection returned
    INFO : jdbc.audit - 1. Connection.getAutoCommit() returned true
    INFO : jdbc.audit - 1. PreparedStatement.new PreparedStatement returned
    INFO : jdbc.audit - 1. Connection.prepareStatement(SELECT *
            FROM tbl_member
            WHERE userid = ? AND userpw = ?) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@1e0b4072
    INFO : jdbc.audit - 1. PreparedStatement.setString(1, "user00") returned
    INFO : jdbc.audit - 1. PreparedStatement.setString(2, "user00") returned
    INFO : jdbc.sqlonly - SELECT * FROM tbl_member WHERE userid = 'user00' AND userpw = 'user00'

    INFO : jdbc.sqltiming - SELECT * FROM tbl_member WHERE userid = 'user00' AND userpw = 'user00'
     {executed in 2 msec}
    INFO : jdbc.audit - 1. PreparedStatement.execute() returned true
    INFO : jdbc.resultset - 1. ResultSet.new ResultSet returned
    INFO : jdbc.audit - 1. PreparedStatement.getResultSet() returned net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy@45099dd3
    INFO : jdbc.resultset - 1. ResultSet.getMetaData() returned com.mysql.jdbc.ResultSetMetaData@6574a52c - Field level information:
    	com.mysql.jdbc.Field@6c1a5b54[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=userid,originalColumnName=userid,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= PRIMARY_KEY BINARY, charsetIndex=83, charsetName=UTF-8]
    	com.mysql.jdbc.Field@1c7696c6[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=userpw,originalColumnName=userpw,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
    	com.mysql.jdbc.Field@60099951[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=username,originalColumnName=username,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
    	com.mysql.jdbc.Field@20140db9[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=email,originalColumnName=email,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
    	com.mysql.jdbc.Field@1e6a3214[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=regdate,originalColumnName=regdate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
    	com.mysql.jdbc.Field@368247b9[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=updatedate,originalColumnName=updatedate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
    INFO : jdbc.resultset - 1. ResultSet.getType() returned 1003
    INFO : jdbc.resultset - 1. ResultSet.next() returned true
    INFO : jdbc.resultset - 1. ResultSet.getString(userid) returned user00
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getString(userpw) returned user00
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getString(username) returned user00
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getString(email) returned user00@mail.com
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getTimestamp(regdate) returned 2017-11-26 22:27:40.0
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultset - 1. ResultSet.getTimestamp(updatedate) returned 2017-11-26 22:27:40.0
    INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
    INFO : jdbc.resultsettable -
    |-------|-------|---------|----------------|----------------------|----------------------|
    |userid |userpw |username |email           |regdate               |updatedate            |
    |-------|-------|---------|----------------|----------------------|----------------------|
    |user00 |user00 |user00   |user00@mail.com |2017-11-26 22:27:40.0 |2017-11-26 22:27:40.0 |
    |-------|-------|---------|----------------|----------------------|----------------------|

    INFO : jdbc.resultset - 1. ResultSet.next() returned false
    INFO : jdbc.resultset - 1. ResultSet.close() returned void
    INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@6d4d66d2
    INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@2a265ea9
    INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
    INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
    INFO : jdbc.audit - 1. PreparedStatement.close() returned
    INFO : jdbc.connection - 1. Connection closed
    INFO : jdbc.audit - 1. Connection.close() returned

    ```
