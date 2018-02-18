# Ch02. JPA 시작

## 1. 라이브러리와 JPA 설정

- `pom.xml` 메이븐 설정 파일
    ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns="http://maven.apache.org/POM/4.0.0"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>

          <groupId>jpabook</groupId>
          <artifactId>ch02-jpa-start1</artifactId>
          <version>1.0-SNAPSHOT</version>

          <properties>
              <!--자바 버전 설정-->
              <java.version>1.6</java.version>
              <!--프로젝트 인코딩 설정-->
              <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
              <project.report.outputEncoding>UTF-8</project.report.outputEncoding>

              <!--JPA 하이버네이트 버전-->
              <hiberate.version>4.3.10.Final</hiberate.version>
              <!--H2 데이터베이스 버전-->
              <h2db.version>1.4.187</h2db.version>
          </properties>

          <dependencies>
              <!--JPA, 하이버네이트-->
              <dependency>
                  <groupId>org.hibernate</groupId>
                  <artifactId>hibernate-entitymanager</artifactId>
                  <version>${hiberate.version}</version>
              </dependency>
              <!--h2 데이터베이스-->
              <dependency>
                  <groupId>com.h2database</groupId>
                  <artifactId>h2</artifactId>
                  <version>${h2db.version}</version>
              </dependency>
          </dependencies>

          <build>
              <plugins>
                  <plugin>
                      <groupId>org.apache.maven.plugins</groupId>
                      <artifactId>maven-compiler-plugin</artifactId>
                      <version>3.1</version>
                      <configuration>
                          <source>${java.version}</source>
                          <target>${java.version}</target>
                      </configuration>
                  </plugin>
              </plugins>
          </build>

      </project>
    ```

- `persistance.xml` 설정
    ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <!--JPA는 persitance.xml을 사용하여 필요한 설정 정보를 관리-->
      <persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">
          <persistence-unit name="jpabook">
              <properties>

                  <!-- 필수 속성 -->
                  <!--javax.persistence로 시작하는 속성은 JPA 표준속성으로 특정 구현체에 종속되지 X-->
                  <!--JPA 표준 속성-->
                  <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/> <!--JDBC 드라이버-->
                  <property name="javax.persistence.jdbc.user" value="sa"/> <!--데이터베이스 접속 id-->
                  <property name="javax.persistence.jdbc.password" value=""/> <!--데이터베이스 접속 pw-->
                  <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/> <!--데이터베이스 접속 URL-->
                  <!--하이버네이트 속성-->
                  <!--hibernate로 시작하는 속성은 하이버네이트 전용 속성으므로 하이버네이트에서만 사용가능-->
                  <!--데이터베이스 방언 : 데이터베이스별로 SQL문법과 함수가 조금씩 다른 것을 의미-->
                  <!--H2 데이터베이스 방언-->
                  <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />
                  <!--
                  오라클 10g 데이터베이스 방언
                  <property name="hibernate.dialect" value="org.hibernate.dialect.Oracle10gDialect" />
                  -->
                  <!--
                  Mysql 데이터베이스 방언
                  <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5InnoDBDialect" />
                  -->

                  <!-- 옵션 -->
                  <property name="hibernate.show_sql" value="true" /> <!--하이버네이트가 실행한 SQL을 출력-->
                  <property name="hibernate.format_sql" value="true" /> <!--하이버네이트가 실행한 SQL을 출력할 때 정렬-->
                  <property name="hibernate.use_sql_comments" value="true" /> <!--쿼리를 출력할 때 주석도 함께-->
                  <property name="hibernate.id.new_generator_mappings" value="true" /> <!--JPA표준에 맞춘 새로운 키 생성 전략 사용-->

                  <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
              </properties>
          </persistence-unit>
      </persistence>
    ```

## 2. 객체 매핑

- `Member.java`
    ```java
    package jpabook.start;

    import javax.persistence.Column;
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.Table;

    @Entity // 클래스를 테이블과 매핑한다고 JPA에게 알려준다.
    @Table(name = "MEMBER") // 엔티티 클래스에 매핑할 테이블 정보를 알려준다.
    public class Member {

        @Id // 엔티티 클래스의 필드를 테이블의 기본키에 매핑 , 식별자 필드라고도 한다.
        @Column(name = "ID") // 필드를 컬럼에 매핑
        private String id;

        @Column(name = "NAME") // 필드를 컬럼에 매핑
        private String username;

        // 매핑 정보가 없는 필드 : 어노테이션 생략하면 필드명을 사용해서 컬럼명으로 매핑해준다.
        // 데이터베이스가 대소문자를 구분하는 경우 명시적으로 매핑을 해줘야한다.
        private Integer age;

        public String getId() {
            return id;
        }

        public void setId(String id) {
            this.id = id;
        }

        public String getUsername() {
            return username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public Integer getAge() {
            return age;
        }

        public void setAge(Integer age) {
            this.age = age;
        }
    }
    ```

## 3. 애플리케이션 개발
- `JpaMain.java`
    ```java
    package jpabook.start;

    import javax.persistence.EntityManager;
    import javax.persistence.EntityManagerFactory;
    import javax.persistence.EntityTransaction;
    import javax.persistence.Persistence;
    import java.util.List;

    public class JpaMain {
        public static void main(String[] args) {

            // 1. 엔티티 매니저 설정
            // 엔티티 매니저 팩토리 생성
            EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
            // 엔티티 매너저 생성
            EntityManager em = emf.createEntityManager();
            // 트랜잭션 기능 획득
            EntityTransaction tx = em.getTransaction();

            // 2. 트랜잭션 관리
            try {
                tx.begin();     // 트랜잭션 시작
                logic(em);      // 비지니스 로직 실행
                tx.commit();    // 트랜잭션 commit
            } catch (Exception e) {
                tx.rollback();  // 트랜잭션 rollback
            } finally {
                em.close();     // 엔티티 매니저 종료
            }
            emf.close();        // 엔티티 매니저 팩토리 종료
        }

        // 3. 비지니스 로직
        public static void logic(EntityManager em) {
            String id = "id1";
            Member member = new Member();
            member.setId(id);
            member.setUsername("doubles");
            member.setAge(30);

            // 등록
            em.persist(member);

            // 수정
            member.setAge(31);

            // 한 건 조회
            Member findMember = em.find(Member.class, id);
            System.out.println("findMember=" + findMember.getUsername() + ", age=" + findMember.getAge());

            // 목록 조회
            List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();
            System.out.println("member.size=" + members.size());

            // 삭제
            em.remove(member);
        }
    }
    ```

- 엔티티 매니저의 생성과정

    1. 설정정보 조회
    2. 생성(EntityManagerFactory)
    3. 생성(EntityManager)

- 엔티티 매니저 팩토리 생성
    ```java
          EntityManagerFactory emf = Persistance.createEntityManagerFactory("jpabook");
    ```
    - JPA를 시작하려면 `persistance.xml`의 설정정보를 사용하여 엔티티 매니저 팩토리를 생성해야한다.
    - Persistance 클래스를 사용하는데 이 클래스는 엔티티 매니저 팩토리를 생성해서 JPA를 사용할 수 있게 준비!
    - 엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한번만 생성하고, 공유해서 사용해야함!

- 엔티티 매니저 생성
    ```java
      EntityManager em = emf.createEntityManager();
    ```
    - 엔티티 매니저 팩토리에서 엔티티 매니저 생성 (JPA기능의 대부분은 엔티티 매니저가 제공)
    - 엔티티 매니저를 사용하여 데이터베이스에 등록/수정/삭제/조회할 수 있음
    - 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있기 때문에 스레드간 공유 재사용 금지

- 종료
    - 사용이 끝난 엔티티 매니저는 반드시 종료
        ```java
          em.close();
        ```

    - 애플리케이션을 종료할 때 앤티티 매니저 팩토리도 다음처럼 종료
        ```java
          emf.close();
        ```    


- 수정
    ```java
      member.setAge(31);
    ```
    - JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있음
    - 엔티티 값만 변경하면 UPDATE SQL을 생성해서 데이터베이스에 값을 변경

- 삭제
    ```java
      em.remove(member);
    ```
    - 엔티티를 삭제하려면 위와 같이 작성하면된다.
    - JPA는 DELETE SQL을 생성해서 실행

- 한 건 조회     
    ```java
      Member findMember = em.find(Member.class, id);
    ```
    - `find()`메서드는 조회할 엔티티 타입과 `@Id`로 데이터베이스 테이블의 기본키와 매핑한 식별자 값으로 엔티티 하나를 조회하는 메서드

- JPQL(Java Persistence Query Language)
    ```java
      TypeQuery<Member> query = em.createQuery("select m from Member m", Member.class);
      List<Member> members = query.getResultList();
    ```
    - JPA를 사용하면 엔티티 객체를 중심으로 개발하고 데이터베이스는 JPA에게 맡겨야한다.
    - 검색쿼리의 경우 JPA는 엔티티 객체를 중심으로 개발하므로 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야한다.
    - 테이블이 아닌 엔티티 객체를 대상으로 검색하려면 데이터베이스의 모든 데이터를 애플리케이션으로 가져와 엔티티 객체로 변경한 뒤 검색해야하는데 이는 사실상 불가능하다.
    - 애플리케이션이 필요한 데이터만 데이터베이스에서 불러오려면 검색조건이 포함된 SQL를 사용해야 한다.
    - 이러한 문제를 JPQL이라는 쿼리 언어로 문제를 해결한다.
    - JPA는 SQL을 추상화한 JPQL이라는 객체지향 쿼리 언어를 제공한다.
    - JPQL은 SQL과 문법이 거의 유사해서 SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등을 사용할 수 있다.
        - SQL과 JPQL의 차이점
            1. SQL : 데이터베이스 테이블을 대상으로 쿼리한다.
            2. JPQL : 엔티티 객체를 대상으로 쿼리한다. 클래스와 필드를 대상으로 쿼리한다.
    - `select m from Member m`이 바로 JPQL이다. 여기서 `from Member`는 회원 엔티티 객체를 의미한다.
    - JPQL은 데이터베이스 테이블을 전혀 알지 못한다.


## ch02 정리

- JPA를 사용하여 객체 하나를 테이블에 등록/수정/삭제/조회하는 간단한 애플리케이션을 만들어보았는데 상당히 편리하고 좋았다.
- 반복적인 JDBC API 결과 값을 매핑처리해주기 때문에 코드량을 많이 줄여줬고, SQL도 작성할 필요가 없었다.
- 이후 책의 내용을 꾸준히 학습한다면 JPA를 다루는 것에 어려움이 없을 것 같다.
