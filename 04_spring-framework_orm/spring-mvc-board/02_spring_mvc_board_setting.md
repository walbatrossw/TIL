본 포스팅은 [코드로 배우는스프링 웹프로젝트](http://www.yes24.com/24/goods/19720776?scode=032&OzSrank=1)를 참조하여 작성한 내용입니다. 개인적으로 학습한 내용을 복습하기 위한 내용이기 때문에 내용상 오류가 있을 수 있습니다. 기존의 Spring MVC 관련 포스팅들이 제대로 정리되지 않은 것 같아 처음부터 차분히 정리하면서 포스팅을 진행하고 있습니다.

그리고 본 포스팅의 예제는 STS 또는 Eclipse를 사용하지 않고 IntelliJ를 통해 구현하고 있습니다. 그래서 기존의 STS에서 생성된 Spring 프로젝트의 스프링관련 설정 파일명과 프로젝트 구조가 약간 다를 수 있습니다. [IntelliJ를 통한 Spring MVC 프로젝트 생성](http://doublesprogramming.tistory.com/171?category=667155) 포스팅을 참고해주시면 감사하겠습니다.

포스팅하고 있는 현재 프로젝트의 예제가 혹시 필요하신 분은 깃주소(https://github.com/walbatrossw/spring-mvc-ex)를 통해 얻으실 수 있습니다.

---

#### Spring-MVC 기본 개념 및 테스트 예제 관련 포스팅 링크

|순서|포스팅 제목|
|:---:|---|
|1|[IntelliJ에서 Spring MVC Project 생성하기](http://doublesprogramming.tistory.com/171)|
|2|[Spring MVC - MySQL 연결테스트](http://doublesprogramming.tistory.com/172)|
|3|[Spring MVC - MyBatis 설정 및 테스트](http://doublesprogramming.tistory.com/173)|
|4|[Spring MVC 구조](http://doublesprogramming.tistory.com/174)|
|5|[Spring MVC - Controller 작성 연습, WAS없이 Controller 테스트 해보기](http://doublesprogramming.tistory.com/175)|
|6|[SpringMVC + MyBatis](http://doublesprogramming.tistory.com/176)|

#### Spring-MVC 게시판 예제 관련 포스팅 링크
|순서|포스팅 제목|
|:---:|---|
|1|[IntelliJ에서 Spring MVC Project 생성하기](http://doublesprogramming.tistory.com/171)|

---

# Spring-MVC 게시판 예제 01 - 프로젝트 생성 및 세팅

앞서 위와 같이 여러 번의 포스팅에 걸쳐서 Spring-MVC에 대해 정리해보았다. 아직 정리가 부족한 부분은 포스팅을 하면서 추가적으로 정리할 예정이다. 이제는 본격적으로 Spring-MVC 게시판 만들기 예제를 직접 구현해보면서 각각의 내용들을 다시 한번 정리해보자. 서두에서 언급한 것과 같이 IntelliJ에서 Spring MVC 프로젝트를 생성했기 때문에 기존에 STS나 Eclipse의 스프링 설정 관련 `xml`파일명이 다르거나, 디렉토리 구조가 약간 다를 수 있다.

## 1. Spring MVC 프로젝트 생성 및 라이브러리 설정

#### # [IntelliJ에서 Spring MVC Project 생성하기](http://doublesprogramming.tistory.com/171) 포스팅 참고

#### # `pom.xml` 라이브러리 추가 및 설정
`spring-jdbc`, `spring-test` `MySQL`, `MyBatis`, `MyBatis-Spring`, `log4jdbc-log4j2`, `jackson-databind` 라이브러리를 추가한다. `servlet`의 버전은 3.1로 변경한다. `Junit`의 버전 4.12로 변경한다.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.doubles</groupId>
    <artifactId>mvc-board</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>mvc-board Maven Webapp</name>
    <url>http://maven.apache.org</url>

    <properties>
        <java-version>1.8</java-version>
        <org.springframework-version>4.3.12.RELEASE</org.springframework-version>
        <org.aspectj-version>1.6.10</org.aspectj-version>
        <org.slf4j-version>1.6.6</org.slf4j-version>
    </properties>

    <dependencies>
        <!--Spring-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${org.springframework-version}</version>
            <exclusions>
                <!-- Exclude Commons Logging in favor of SLF4j -->
                <exclusion>
                    <groupId>commons-logging</groupId>
                    <artifactId>commons-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${org.springframework-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${org.springframework-version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${org.springframework-version}</version>
        </dependency>

        <!--MySQL 라이브러리-->
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.41</version>
        </dependency>

        <!--MyBatis 라이브러리-->
        <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.1</version>
        </dependency>

        <!--MyBatis-Spring 라이브러리-->
        <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.0</version>
        </dependency>

        <!--MyBatis 로그-->
        <!-- https://mvnrepository.com/artifact/org.bgee.log4jdbc-log4j2/log4jdbc-log4j2-jdbc4 -->
        <dependency>
            <groupId>org.bgee.log4jdbc-log4j2</groupId>
            <artifactId>log4jdbc-log4j2-jdbc4</artifactId>
            <version>1.16</version>
        </dependency>

        <!--JSON : jackson-databind 라이브러리-->
        <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.8.4</version>
        </dependency>

        <!-- AspectJ -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>${org.aspectj-version}</version>
        </dependency>

        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${org.slf4j-version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${org.slf4j-version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${org.slf4j-version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.15</version>
            <exclusions>
                <exclusion>
                    <groupId>javax.mail</groupId>
                    <artifactId>mail</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>javax.jms</groupId>
                    <artifactId>jms</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>com.sun.jdmk</groupId>
                    <artifactId>jmxtools</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>com.sun.jmx</groupId>
                    <artifactId>jmxri</artifactId>
                </exclusion>
            </exclusions>
            <scope>runtime</scope>
        </dependency>

        <!-- @Inject -->
        <dependency>
            <groupId>javax.inject</groupId>
            <artifactId>javax.inject</artifactId>
            <version>1</version>
        </dependency>

        <!-- Servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>mvc-board</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.5.1</version>
                <configuration>
                    <source>${java-version}</source>
                    <target>${java-version}</target>
                    <compilerArgument>-Xlint:all</compilerArgument>
                    <showWarnings>true</showWarnings>
                    <showDeprecation>true</showDeprecation>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.2.1</version>
                <configuration>
                    <mainClass>org.test.int1.Main</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

## 2. Web관련 설정 및 한글 인코딩 설정

#### # `web.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

    <display-name>Archetype Created Web Application</display-name>

    <!--DataSource, MyBatis 설정 xml 경로-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring-config/applicationContext.xml</param-value>
    </context-param>

    <!--리스너-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!--servlet dispatcher 설정 xml 경로-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring-config/dispatcher-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!--servlet mapping 설정-->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

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

</web-app>

```

## 3. DataSource, MyBatis설정, MyBatis 로그 설정

#### # `applicationContext.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--DataSource default 설정-->
    <!--<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">-->
    <!--<property name="driverClassName" value="com.mysql.jdbc.Driver"/>-->
    <!--<property name="url" value="jdbc:mysql://127.0.0.1:3306/spring_ex?useSSL=false"/>-->
    <!--<property name="username" value="doubles"/>-->
    <!--<property name="password" value="qazwsx12"/>-->
    <!--</bean>-->

    <!--DataSource log4jdbc-log4j2 설정-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"/>
        <property name="url" value="jdbc:log4jdbc:mysql://127.0.0.1:3306/spring_ex?useSSL=false"/>
        <property name="username" value="doubles"/>
        <property name="password" value="qazwsx12"/>
    </bean>

    <!--SqlSessionFactory 설정 : dataSource를 참조, mybatis-config.xml 경로설정-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="classpath:/mybatis-config.xml"/>
        <property name="mapperLocations" value="classpath:mappers/**/*Mapper.xml"/>
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
        <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>

    <context:component-scan base-package="com.doubles.mvcboard"/>

</beans>
```

#### # `mybatis-config.xml`
`src/main/resoures` 디렉토리에 파일 생성, 아래와 같이 코드를 작성해준다.
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <typeAliases>
        <package name="com.doubles.mvcboard"/>
    </typeAliases>

</configuration>
```

#### # `log4j.xml`
`src/main/resoures` 디렉토리에 파일 생성, 아래와 같이 작성해준다.
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
        <priority value="warn"/>
        <appender-ref ref="console"/>
    </root>

</log4j:configuration>
```

#### # `log4jdbc.log4j2.properties`
`src/main/resoures` 디렉토리에 파일 생성, 아래와 같이 작성해준다.
```
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```

#### # `logback.xml`
`src/main/resoures` 디렉토리에 파일 생성, 아래와 같이 작성해준다.
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

## 4. MVC 관련 설정
#### # `dispatcher-servlet.xml`
아래와 같이 코드를 작성해준다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--애너테이션 인식-->
    <mvc:annotation-driven/>

    <!--정적자원 매핑-->
    <mvc:resources mapping="/resources/**" location="/resources/"/>

    <!--viewResolver 설정-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>


    <context:component-scan base-package="com.doubles.mvcboard"/>

</beans>
```

## 5. package, resoures 구성 및 완성된 기본 프로젝트의 형태
#### # package 구성과 resoures에 추가된 설정파일
아래와 같이 기능별로 각각의 패키지를 구성해준다.
```
main
 ├── java
 |     └── 기본 패키지(com.doubles.mvcboard)
 |              ├── article : 게시글 관련
 |              ├── commons : 공통으로 사용되는 클래스 모음(aop, exception, interceptor, util)
 |              ├── reply   : 댓글 관련
 |              ├── tutorial : 연습 예제 관련
 |              ├── upload : 업로드 관련
 |              └── user : 회원 관련
 |                   |
 |                기능별 패키지 하위 패키지에 각각 controller, domain, persistence, service 패키지 생성
 |                   |
 |                   ├── controller : Spring-Mvc의 Controller 패키지
 |                   ├── domain : VO(Value Object)가 사용하는 패키지
 |                   ├── persistence : MyBatis의 DAO 인터페이스와 구현 클래스 패키지
 |                   └── service : Service 인터페이스와 구현 클래스 패키지
 └── resoures : log 관련 설정 xml (log4j.xml, log4jdbc.log4j2.properties, logback.xml), MyBatis 설정 xml (mybatis-config.xml)
          └── mappers : MyBatis XML Mapper 위치, 하위에 각 기능별로 디렉토리 생성

```

#### # 완성된 기본 프로젝트 형태
![end](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%BD%94%EB%93%9C%EB%A1%9C_%EB%B0%B0%EC%9A%B0%EB%8A%94_%EC%8A%A4%ED%94%84%EB%A7%81_%EC%9B%B9%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/photo/20180224_210006.png?raw=true)

## 6. 마무리

프로젝트를 생성하고, 기본 설정을 하는 것까지 정리해보았다. STS에서는 프로젝트를 생성하고 설정하는 과정이 이것보다는 조금 빠를 수 있지만 IntelliJ에서 직접 설정을 해보면서 하나하나 무슨 역할을 하고 어떻게 작동하는지 좀더 공부해볼 수 있는 계기가 되었다. 물론 공부한 내용을 여기에 쓰기에는 너무 길어져서 세세한 내용들은 생략한 경우가 있는데 이것들은 따로 또 정리해보자.
