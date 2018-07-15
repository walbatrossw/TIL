본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.

그리고 본 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정 파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](http://doublesprogramming.tistory.com/171?category=667155) 포스팅을 참고해주시면 감사하겠습니다.


포스팅하고 있는 현재 프로젝트의 예제가 혹시 필요하신 분은 github(https://github.com/walbatrossw/spring-mvc-ex)를 통해 얻으실 수 있습니다.

---

# SpringMVC + MyBatis, DAO구현 테스트

앞서 포스팅한 [MyBatis 설정 및 테스트](http://doublesprogramming.tistory.com/173)에 이어서 MyBatis를 이용하여 회원을 입력하고, 조회(아이디, 아이디+비밀번호)하는 간단한 기능을 구현해보고, 제대로 구현되었는지 테스트까지 진행해보자.


## 1. MyBatis의 개발 방식 간단 정리
MyBatis는 JDBC에서 개발자가 직접 처리하는 `PreparedStatement`의 `?`에 대한 설정이나 `ResultSet`을 이용한 처리가 이루어지기 때문에 기존의 방식에 비해 생산성이 좋다. MyBatis의 이전 버전인 iBatis는 개발자가 모든 SQL을 XML로 작성하고, SQL문을 사용하는 DAO클래스를 설계해서 SQL문을 호출하는 방식으로 작성했지만 MyBatis의 경우 애너테이션을 지원하고, 인터페이스와 애너테이션을 통해 SQL문을 설정하고 처리할 수 있게 되었다.
MyBatis를 이용할때 SQL문을 사용하는 방식의 종류는 다음과 같다.

|방식|장점|단점|
|---|---|---|
|XML만을 사용하는 방식|SQL문은 별도의 XML로 작성되기 때문에 SQL문의 수정이나 유지보수에 적합|개발시 코드의 양이 많아지고, 복잡성 증가|
|애너테이션과 인터페이스만을 이용하는 방식|별도의 DAO없이도 개발이 가능하기 때문에 생산성 증가|SQL문을 애너테이션으로 작성하기 때문에 매번 수정이 일어나는 경우 다시 컴파일해야 됨|
|인터페이스와 XML로 작성된 SQL문을 활용하는 방식|간단한 SQL문은 애너테이션으로 복잡한 SQL문은 XML로 처리하므로, 상황에 따라 유연하게 처리|개발자에 따라 개발방식의 차이가 존재하기 때문에 유지보수가 중요한 프로젝트의 경우 부적합|

## 2. `XML`만을 이용하는 방식 구현하기
앞서 언급한 것처럼 XML만을 이용하는 방식은 SQL문을 완전히 분리해서 처리하기 때문에 SQL문의 변경이 일어날 때, 대처가 수월해 유지보수에 적합한 장점을 가지고 있다. 그래서 대부분의 프로젝트에서 XML만을 이용해서 SQL문을 작성하고, 별도의 DAO를 만드는 방식을 선호한다고 한다.

이제 본격적으로 `xml`만을 이용하는 방식을 통해 간단한 회원가입로직을 구현해보자.
#### # DB(MySQL) 테이블 생성
기존에 앞서 생성한 DB `spring_ex`에 회원 테이블을 추가시킨다.
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

#### # 도메인 객체를 위한 클래스 설계
domain패키지를 생성하고, 회원을 의미하는 `MemberVO`클래스를 아래와 같이 작성해준다.
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
#### # `DAO`(Data Access Object)인터페이스 작성
persistence패키지를 생성하고 `MemberDAO` 인터페이스를 생성하고, 간단한 회원관련 기능을 처리할 메서드를 메서드를 작성해준다.
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
#### # XML Mapper 작성
`/main/resources/mappers/tutorial`디렉토리와 `memberMapper.xml`파일을 생성하고 아래와 같이 `mapper`를 작성해준다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.doubles.mvcboard.mappers.tutorial.MemberMapper">

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

    <select id="selectMember" resultType="com.doubles.mvcboard.tutorial.domain.MemberVO">
        SELECT *
        FROM tbl_member
        WHERE userid = #{userid}
    </select>

    <select id="readWithPW" resultType="com.doubles.mvcboard.tutorial.domain.MemberVO">
        SELECT *
        FROM tbl_member
        WHERE userid = #{userid} AND userpw = #{userpw}
    </select>

</mapper>
```
여기서 가장 주의해야할 것은 `mapper`태그의 `namespace`속성에 가장 신경을 써줘야한다. 클래스의 패키지와 유사한 용도로, MyBatis내에서 원하는 SQL문을 찾아서 실행할 때 동작하기 때문이다.

#### # `myBaits-Spring`에서 XML Mapper 인식시키기
MyBatis가 XML Mapper를 인식할 수 있도록 `applicationContext.xml`에 아래와 같이 Mapper의 경로를 추가해준다. 여기서 `applicationContext.xml`은 sts에서의 `root-context.xml`파일과 동일한 역할을 수행하고 있다.
```xml
<!--SqlSessionFactory 설정 : dataSource를 참조, mybatis-config.xml 경로설정-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:/mybatis-config.xml"/>
        <!-- mapper 경로 추가 -->
        <property name="mapperLocations" value="classpath:mappers/**/*Mapper.xml"/>
    </bean>
```

#### # `SqlSessionTemplate` 설정
MyBatis에서 `DAO`를 이용하는 경우 `SqlSessionTemplate`을 이용해 `DAO`를 구현하기 때문에 `DAO`인터페이스를 구현하기에 앞서 `SqlSessionTemplate`설정을 먼저 해줘야한다. `DAO`작업에서 가장 번거로운 작업은 데이터베이스와 연결을 맺고, 작업이 완료된 후 연결을 `close()`하는 작업인데 `mybatis-spring`라이브러리는 이것을 처리할 수 있는 `SqlSessionTemplate`이라는 클래스를 통해 DB를 연결하고, 종료하는 작업을 처리 할 수있다.
`SqlSessionTemplate`은 `SqlSession`인터페이스를 구현한 클래스로 기본적인 트랜잭션의 관리나 쓰레드 처리의 안정성을 보장해주고, DB의 연결과 종료를 책임진다.

`SqlSessionTemplate`설정은 `applicationContext.xml`에서 `SqlSessionFactory`를 생성자로 주입해서 아래와 같이 설정해준다.
```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
    <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
</bean>
```

#### # `DAO`인터페이스 구현 클래스 작성
앞서 작성한 `MemberDAO`인터페이스를 구현하는 클래스를 작성하고, `SqlSessionTemplate`을 주입받아 사용한다.
```java
@Repository
public class MemberDAOImpl implements MemberDAO {

    @Inject
    SqlSession sqlSession;

    private static final String NAMESPACE = "com.doubles.mvcboard.mappers.tutorial.MemberMapper";

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
        return sqlSession.selectOne(NAMESPACE + ".selectMember", userid);
    }

    @Override
    public MemberVO readWithPW(String userid, String userpw) throws Exception {
        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("userid", userid);
        paramMap.put("userpw", userpw);
        return sqlSession.selectOne(NAMESPACE + ".readWithPW", paramMap);
    }

}
```
#### # Spring에서 Bean등록
`MemberDAOImpl`에 `@Repository`애너테이션 설정을 했더라도, 스프링에서 해당 패키지를 스캔하지 않으면 스프링의 빈으로 등록되지 못하기 때문에 아래와 같이`applicationContext.xml`에 작성을 반드시 해줘야한다.
```xml
<context:component-scan base-package="com.doubles.mvcboard"/>
```

#### # MyBatis로그(`log4jdbc-log4j2`)
MyBatis를 사용해 개발을 하다보면 잘못된 SQL이나 잘못된 속성명으로 인해 예외가 발생하는 경우가 있는데, 이러한 경우를 대비해 MyBatis의 로그를 보다 자세히 조사할 수 있도록 설정해주는 것이 좋다. 보다 자세한 로그를 보기 위해서는 `log4jdbc-log4j2`라이브러리를 추가해준다.
```xml
<dependency>
    <groupId>org.bgee.log4jdbc-log4j2</groupId>
    <artifactId>log4jdbc-log4j2-jdbc4</artifactId>
    <version>1.16</version>
</dependency>
```
그리고 나서 `applicationContext.xml`에서 약간의 수정이 필요한데 `dataSource`설정에서 `driverClassName`과 `url`의 `value`를 아래와 같이 변경해준다.
```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <!-- 변경된 코드 -->
    <property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"/>
    <property name="url" value="jdbc:log4jdbc:mysql://127.0.0.1:3306/spring_ex?useSSL=false"/>
    <!-- 변경된 코드 -->
    <property name="username" value="아이디"/>
    <property name="password" value="비밀번호"/>
</bean>
```
`log4jdbc-log4j2`가 제대로 동작하기 위해서는 별도의 로그관련 설정이 추가적으로 필요하다. `/src/main/resources`의 디렉토리에 `log4jdbc.log4j2.properties`와 `logback.xml`파일을 생성해주고, 아래와 같이 코드를 작성해준다.
```
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```

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

마지막으로 `/src/main/resources/`디렉토리의 `log4j.xml`를 `/src/test/resource/`디렉토리에 동일하게 파일을 생성하고 아래와 같이 작성해줘야 로그가 제대로 출력된다.
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
    <logger name="com.doubles.mvcboard">
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

#### # 테스트 코드 작성하기
이제 `MemberDAOImpl`에서 구현한 기능들이 제대로 동작하는지 확인하기 위해 테스트 코드를 작성해보자.
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"file:src/main/webapp/WEB-INF/spring-config/applicationContext.xml"})
public class MemberDAOTest {

    @Inject
    private MemberDAO memberDAO;

    // 현재시간 출력 테스트
    @Test
    public void testTime() throws Exception {
        System.out.println(memberDAO.getTime());
    }

    // 회원 가입 테스트
    @Test
    public void testInsertMember() throws Exception {
        MemberVO memberVO = new MemberVO();
        memberVO.setUserid("user00");
        memberVO.setUserpw("user00");
        memberVO.setUsername("user00");
        memberVO.setEmail("user00@mail.com");

        memberDAO.insertMember(memberVO);
    }

    // 회원 조회 테스트 1 : 아이디만
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

이제 테스트를 돌려보면 아래와 같이 각각의 메서드 테스크 결과가 콘솔창에 깔끔하게 로그가 출력되는 것을 볼 수 있다.
```
// testTime() 테스트 결과

INFO : jdbc.connection - 1. Connection opened
INFO : jdbc.audit - 1. Connection.new Connection returned
INFO : jdbc.audit - 1. Connection.getAutoCommit() returned true
INFO : jdbc.audit - 1. PreparedStatement.new PreparedStatement returned
INFO : jdbc.audit - 1. Connection.prepareStatement(SELECT NOW()) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@7161d8d1
INFO : jdbc.sqlonly - SELECT NOW()

INFO : jdbc.sqltiming - SELECT NOW()
 {executed in 1 msec}
INFO : jdbc.audit - 1. PreparedStatement.execute() returned true
INFO : jdbc.resultset - 1. ResultSet.new ResultSet returned
INFO : jdbc.audit - 1. PreparedStatement.getResultSet() returned net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy@3a7442c7
INFO : jdbc.resultset - 1. ResultSet.getMetaData() returned com.mysql.jdbc.ResultSetMetaData@4b013c76 - Field level information:
	com.mysql.jdbc.Field@53fb3dab[catalog=,tableName=,originalTableName=,columnName=NOW(),originalColumnName=,mysqlType=12(FIELD_TYPE_DATETIME),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
INFO : jdbc.resultset - 1. ResultSet.getType() returned 1003
INFO : jdbc.resultset - 1. ResultSet.next() returned true
INFO : jdbc.resultset - 1. ResultSet.getString(NOW()) returned 2018-02-24 18:10:32.0
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultsettable -
|----------------------|
|now()                 |
|----------------------|
|2018-02-24 18:10:32.0 |
|----------------------|

INFO : jdbc.resultset - 1. ResultSet.next() returned false
INFO : jdbc.resultset - 1. ResultSet.close() returned void
INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@18271936
INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@606e4010
INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
INFO : jdbc.audit - 1. PreparedStatement.close() returned
INFO : jdbc.connection - 1. Connection closed
INFO : jdbc.audit - 1. Connection.close() returned
2018-02-24 18:10:32.0
```

```
// testInsertMember() 테스트 결과

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
        )) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@235ecd9f
INFO : jdbc.audit - 1. PreparedStatement.setString(1, "user00") returned
INFO : jdbc.audit - 1. PreparedStatement.setString(2, "user00") returned
INFO : jdbc.audit - 1. PreparedStatement.setString(3, "user00") returned
INFO : jdbc.audit - 1. PreparedStatement.setString(4, "user00@mail.com") returned
INFO : jdbc.sqlonly - INSERT INTO tbl_member ( userid , userpw , username , email ) VALUES ( 'user00' , 'user00'
, 'user00' , 'user00@mail.com' )

INFO : jdbc.sqltiming - INSERT INTO tbl_member ( userid , userpw , username , email ) VALUES ( 'user00' , 'user00'
, 'user00' , 'user00@mail.com' )
 {executed in 15 msec}
INFO : jdbc.audit - 1. PreparedStatement.execute() returned false
INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned 1
INFO : jdbc.audit - 1. PreparedStatement.close() returned
INFO : jdbc.connection - 1. Connection closed
INFO : jdbc.audit - 1. Connection.close() returned
```

```
// testReadMember() 테스트 결과

INFO : jdbc.connection - 1. Connection opened
INFO : jdbc.audit - 1. Connection.new Connection returned
INFO : jdbc.audit - 1. Connection.getAutoCommit() returned true
INFO : jdbc.audit - 1. PreparedStatement.new PreparedStatement returned
INFO : jdbc.audit - 1. Connection.prepareStatement(SELECT *
        FROM tbl_member
        WHERE userid = ?) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@7161d8d1
INFO : jdbc.audit - 1. PreparedStatement.setString(1, "user00") returned
INFO : jdbc.sqlonly - SELECT * FROM tbl_member WHERE userid = 'user00'

INFO : jdbc.sqltiming - SELECT * FROM tbl_member WHERE userid = 'user00'
 {executed in 1 msec}
INFO : jdbc.audit - 1. PreparedStatement.execute() returned true
INFO : jdbc.resultset - 1. ResultSet.new ResultSet returned
INFO : jdbc.audit - 1. PreparedStatement.getResultSet() returned net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy@3a7442c7
INFO : jdbc.resultset - 1. ResultSet.getMetaData() returned com.mysql.jdbc.ResultSetMetaData@4b013c76 - Field level information:
	com.mysql.jdbc.Field@53fb3dab[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=userid,originalColumnName=userid,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= PRIMARY_KEY BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@cb0755b[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=userpw,originalColumnName=userpw,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@33065d67[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=username,originalColumnName=username,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@712625fd[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=email,originalColumnName=email,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@7bba5817[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=regdate,originalColumnName=regdate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
	com.mysql.jdbc.Field@742ff096[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=updatedate,originalColumnName=updatedate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
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
INFO : jdbc.resultset - 1. ResultSet.getTimestamp(regdate) returned 2018-02-24 18:12:20.0
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultset - 1. ResultSet.getTimestamp(updatedate) returned 2018-02-24 18:12:20.0
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultsettable -
|-------|-------|---------|----------------|----------------------|----------------------|
|userid |userpw |username |email           |regdate               |updatedate            |
|-------|-------|---------|----------------|----------------------|----------------------|
|user00 |user00 |user00   |user00@mail.com |2018-02-24 18:12:20.0 |2018-02-24 18:12:20.0 |
|-------|-------|---------|----------------|----------------------|----------------------|

INFO : jdbc.resultset - 1. ResultSet.next() returned false
INFO : jdbc.resultset - 1. ResultSet.close() returned void
INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@48075da3
INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@68c9133c
INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
INFO : jdbc.audit - 1. PreparedStatement.close() returned
INFO : jdbc.connection - 1. Connection closed
INFO : jdbc.audit - 1. Connection.close() returned

```

```
// testReadWithPW() 테스트 결과

INFO : jdbc.connection - 1. Connection opened
INFO : jdbc.audit - 1. Connection.new Connection returned
INFO : jdbc.audit - 1. Connection.getAutoCommit() returned true
INFO : jdbc.audit - 1. PreparedStatement.new PreparedStatement returned
INFO : jdbc.audit - 1. Connection.prepareStatement(SELECT *
        FROM tbl_member
        WHERE userid = ? AND userpw = ?) returned net.sf.log4jdbc.sql.jdbcapi.PreparedStatementSpy@74e28667
INFO : jdbc.audit - 1. PreparedStatement.setString(1, "user00") returned
INFO : jdbc.audit - 1. PreparedStatement.setString(2, "user00") returned
INFO : jdbc.sqlonly - SELECT * FROM tbl_member WHERE userid = 'user00' AND userpw = 'user00'

INFO : jdbc.sqltiming - SELECT * FROM tbl_member WHERE userid = 'user00' AND userpw = 'user00'
 {executed in 1 msec}
INFO : jdbc.audit - 1. PreparedStatement.execute() returned true
INFO : jdbc.resultset - 1. ResultSet.new ResultSet returned
INFO : jdbc.audit - 1. PreparedStatement.getResultSet() returned net.sf.log4jdbc.sql.jdbcapi.ResultSetSpy@4be29ed9
INFO : jdbc.resultset - 1. ResultSet.getMetaData() returned com.mysql.jdbc.ResultSetMetaData@53fb3dab - Field level information:
	com.mysql.jdbc.Field@cb0755b[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=userid,originalColumnName=userid,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= PRIMARY_KEY BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@33065d67[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=userpw,originalColumnName=userpw,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@712625fd[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=username,originalColumnName=username,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@7bba5817[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=email,originalColumnName=email,mysqlType=253(FIELD_TYPE_VAR_STRING),flags= BINARY, charsetIndex=83, charsetName=UTF-8]
	com.mysql.jdbc.Field@742ff096[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=regdate,originalColumnName=regdate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
	com.mysql.jdbc.Field@75437611[catalog=spring_ex,tableName=tbl_member,originalTableName=tbl_member,columnName=updatedate,originalColumnName=updatedate,mysqlType=7(FIELD_TYPE_TIMESTAMP),flags= BINARY, charsetIndex=63, charsetName=US-ASCII]
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
INFO : jdbc.resultset - 1. ResultSet.getTimestamp(regdate) returned 2018-02-24 18:12:20.0
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultset - 1. ResultSet.getTimestamp(updatedate) returned 2018-02-24 18:12:20.0
INFO : jdbc.resultset - 1. ResultSet.wasNull() returned false
INFO : jdbc.resultsettable -
|-------|-------|---------|----------------|----------------------|----------------------|
|userid |userpw |username |email           |regdate               |updatedate            |
|-------|-------|---------|----------------|----------------------|----------------------|
|user00 |user00 |user00   |user00@mail.com |2018-02-24 18:12:20.0 |2018-02-24 18:12:20.0 |
|-------|-------|---------|----------------|----------------------|----------------------|

INFO : jdbc.resultset - 1. ResultSet.next() returned false
INFO : jdbc.resultset - 1. ResultSet.close() returned void
INFO : jdbc.audit - 1. PreparedStatement.getConnection() returned net.sf.log4jdbc.sql.jdbcapi.ConnectionSpy@68c9133c
INFO : jdbc.audit - 1. Connection.getMetaData() returned com.mysql.jdbc.JDBC4DatabaseMetaData@7a35b0f5
INFO : jdbc.audit - 1. PreparedStatement.getMoreResults() returned false
INFO : jdbc.audit - 1. PreparedStatement.getUpdateCount() returned -1
INFO : jdbc.audit - 1. PreparedStatement.close() returned
INFO : jdbc.connection - 1. Connection closed
INFO : jdbc.audit - 1. Connection.close() returned
```
