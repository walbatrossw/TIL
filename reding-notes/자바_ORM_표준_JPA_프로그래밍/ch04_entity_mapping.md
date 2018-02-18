# CH04. 엔티티 매핑

## 1. `@Entity`

- JPA를 사용해서 테이블과 매핑할 클래스에는 `@Entity` 어노테이션을 붙여야한다.
- 속성
    1. `name` : JPA와 사용할 엔티티 이름을 지정, 사용하지 않을 경우 클래스이름이 그대로 적용
- 주의사항
    1. 기본생성자 필수
    2. `final`, `enum`, `interface`, `inner`클래스는 사용할 수 없음
    3. 저장할 필드에 `final` 사용할 수 없음

## 2. `@Table`

- 엔티티와 매핑할 테이블을 지정, 생략시 매핑한 엔티티 이름을 테이블 이름으로 사용
- 속성
    1. `name` : 매핑할 테이블
    2. `catalog` : catalog 기능이 있는 데이터베이스에서 catalog를 매핑
    3. `schema` : schema 기능이 있는 데이터베이스에서 schema를 매핑
    4. `uniqueConstraint` : DDL 매핑 시에 유니크제약조건을 만듬, 스키마 자동생성 기능을 사용해서 DDL을 만들때만 사용

## 3. 데이터베이스 스키마 자동생성

- `persistence.xml` : 아래와 같은 속성을 추가하면 데이터베이스 테이블을 자동으로 생성
    ```xml
    <property name="hibernate.hbm2ddl.auto" value="create" />
    ```
- 프로젝트 실행 후 콘솔화면
    ```
    Hibernate:
    8월 16, 2017 10:26:10 오전 org.hibernate.tool.hbm2ddl.SchemaExport execute
        drop table MEMBER if exists
    INFO: HHH000227: Running hbm2ddl schema export
    Hibernate:
        create table MEMBER (
            ID varchar(255) not null,
            age integer,
            createdDate timestamp,
            description clob,
            lastModifiedDate timestamp,
            roleType varchar(255),
            NAME varchar(255),
            primary key (ID)
        )
    ```
- 스키마 자동생성 기능을 사용하면 애플리케이션 실행 시점에 테이터베이스 테이블이 자동으로 생성
- `hibernate.hbm2ddl.auto`의 속성
    1. `create` : 기존 테이블을 삭제하고 새로 생성, DROP + CREATE
    2. `create-drop` : 애플리케이션을 종료할 때 생성한 DDL을 제거, DROP + CREATE + DROP
    3. `update` : DB 테이블과 엔티티 매핑정보를 비교해서 변경사항만 수정
    4. `validate` : DB 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션 실행X, DDL수정 X
    5. `none` : 자동생성 기능을 사용하지않으려면 속성을 제거하거나, 유효하지않은 옵션값을 주면 됨
- 주의사항 : DDL을 수정하는 옵션(create, creat-drop, update)은 운영서버에서 절대 사용 금지, 오직 개발서버나 개발단계에서만 사용
    1. 개발 초기단계 : create, update
    2. 테스트 서버 : update, validate
    3. 스테이징, 운영서버 : validate, none
- 이름 매핑 전략 변경 : 자바(카멜표기법), DB(언더스코어)
    1. `persistence.xml` : 속성추가
        ```xml
        <property name="hibernate.ejb.naming_strategy" value="org.hibernate.cfg.ImprovedNamingStrategy" />  
        ```
    2. 프로젝트 실행 후 콘솔화면
        ```
        Hibernate:
            drop table MEMBER if exists
        Hibernate:
            create table MEMBER (
                ID varchar(255) not null,
                AGE integer,
                CREATE_DATE timestamp,  // 칼럼명이 언더스코어로 변경
                DESCRIPTION clob,
                LAST_MODIFIED_DATE timestamp, // 칼럼명이 언더스코어로 변경
                ROLE_TYPE varchar(255), // 칼럼명이 언더스코어로 변경
                NAME varchar(255),
                primary key (ID)
            )
        ```

## 4. DDL 생성기능
- 회원 이름이 필수로 입력되고, 10자를 초과하면 안된다는 제약조건을 추가해보기
    1. `Member.java`
        ```java
        @Column(name = "NAME", nullable = false, length = 10) // 필수입력(null 허용X), 길이는 10자
        private String username;
        ```
    2. 프로젝트 실행 후 콘솔화면
        ```
        Hibernate:
            drop table MEMBER if exists
        Hibernate:
            create table MEMBER (
                ID varchar(255) not null,
                AGE integer,
                CREATE_DATE timestamp,
                DESCRIPTION clob,
                LAST_MODIFIED_DATE timestamp,
                ROLE_TYPE varchar(255),
                NAME varchar(10) not null, // null 허용금지, 길이 10이 추가
                primary key (ID)
            )
        ```
- 유니크 제약조건 `@Table`의 `uniqueConstraints`속성
    1. `Member.java`
        ```java
        @Entity
        // 유니크 제약조건 추가
        @Table(name = "MEMBER", uniqueConstraints = {@UniqueConstraint(
                name = "NAME_AGE_UNIQUE",
                columnNames = {"NAME", "AGE"} )})
        public class Member {
           // ...
        }
        ```
    2. 프로젝트 실행 후 콘솔화면
        ```
        Hibernate:
        8월 16, 2017 12:00:45 오후 org.hibernate.tool.hbm2ddl.SchemaExport execute
            drop table MEMBER if exists
        INFO: HHH000227: Running hbm2ddl schema export
        Hibernate:
            create table MEMBER (
                ID varchar(255) not null,
                AGE integer,
                CREATE_DATE timestamp,
                DESCRIPTION clob,
                LAST_MODIFIED_DATE timestamp,
                ROLE_TYPE varchar(255),
                NAME varchar(10) not null,
                primary key (ID)
            )
        Hibernate:
            alter table MEMBER // 유니크 제약조건 추가됨
                add constraint NAME_AGE_UNIQUE  unique (NAME, AGE)
        ```

## 5. 기본키 매핑

- 기본키를 애플리케이션에서 직접 할당하는 대신 데이터베이스가 생성해주는 값을 사용하려면? ex) 오라클의 sequence, MySql의 auto_increment
- JPA의 데이터베이스 기본키 생성 전략 종류
    1. 직접할당 : 기본키를 애플리케이션에서 직접할당
    2. 자동생성 : 대리키 사용방식
        - IDENTITY : 기본키 생성을 데이터베이스에 위임, 데이터베이스에 의존
        - SEQUENCE : 데이터베이스 시퀀스를 사용하여 기본키를 할당, 데이터베이스에 의존
        - TABLE : 키 생성 테이블을 사용, 데이터베이스에 의존X
- 키 생성 전략을 사용하기 위한 설정
    1. `pesistence.xml` : 속성추가
    ```xml
    <property name="hibernate.id.new_generator_mappings" value="true" />
    ```
- 기본키 직접할당 전략 : 엔티티를 저장하기전 애플리케이션에서 기본키를 직접 할당하는 방법
    1. `Member.java`
        ```java
        @Id   // 적용가능 자바타입 : 자바 기본형, 래퍼형, String, java,util.Date, java.sql.Date, java.math.BigDecimal, java.math.BigInteger
        @Column(name = "id")
        private String id;
        ```
    2. `JpaMain.java`
        ```java
        String id = "id001";
        Member member = new Member();
        member.setId(id);
        ```
- IDENTITY 전략 : 기본키 생성을 데이터베이스에 위임, 주로 MySql, PostgreSQL, SQL Server, DB2에서 사용
    1. `Member.java`
        ```java
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY) // 식별자 생성 전략 IDENTITY로 설정
        @Column(name = "ID")
        private Long id;
        ```
    2. `JpaMain.java`
        ```java
        // Mysql : 기본키 IDENTITY 전략
        MemberForIdentity member = new MemberForIdentity();
        member.setUsername("더블에스");
        em.persist(member);
        System.out.println("mysql member.id = " + member.getId());
        ```
    3. 프로젝트 실행 후 콘솔화면
        ```
        mysql member.id = 1
        ```
    4. 주의사항
        - 엔티티가 영속상태가 되려면 식별자가 반드시 필요한데 IDENTITY전략의 경우 엔티티를 데이터베이스에 저장해야먼 식별자를 구할수 있음
        - `em.persitence()`를 호출하는 즉시 INSERT SQL이 데이터베이스에 전달
        - 그래서 이 전략은 트랜잭션을 지원하는 쓰기 지연이 동작하지 않음
- SEQUENCE 전략 : 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트, 시퀀스를 지원하는 Oracle, PostgreSQL, DB2, H2에서 사용가능
    1. `MemberForSequence.java`
        ```java
        @Entity
        @SequenceGenerator( name = "MEMBER_SEQUENCE_GENERATOR", // 식별자 생성기 이름
                sequenceName = "MEMBER_SEQUENCE", // 매핑할 데이터베이스 시퀀스 이름
                initialValue = 1,   // DDL생성시에만 사용
                allocationSize = 1) // 시퀀스 한번 호출에 증가하는 수
        public class MemberForSequence {
            @Id
            @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "MEMBER_SEQUENCE_GENERATOR") // 식별자 생성 전략 SEQUENCE로 설정
            @Column(name = "ID")
            private Long id;    // Long 타입
            // ...
        }      
        ```
    2. `JpaMain.java`
        ```java
        // H2 DB : 기본키 SEQUENCE 전략
        MemberForSequence member = new MemberForSequence();
        member.setUsername("더블에스");
        em.persist(member);
        System.out.println("SEQUENCE member.id = " + member.getId());
        ```
    3. 프로젝트 실행 후 콘솔화면
        ```
        ibernate:
            drop table member_for_sequence if exists
        Hibernate:
            drop sequence if exists MEMBER_SEQUENCE
        Hibernate:
            create table member_for_sequence (
                id bigint not null,
                age integer,
                create_date timestamp,
                description clob,
                last_modified_date timestamp,
                role_type varchar(255),
                name varchar(10) not null,
                primary key (id)
            )
        Hibernate:
            create sequence MEMBER_SEQUENCE start with 1 increment by 1 // 시퀀스 생성
        8월 16, 2017 2:51:19 오후 org.hibernate.tool.hbm2ddl.SchemaExport execute
        INFO: HHH000230: Schema export complete
        Hibernate:
            call next value for MEMBER_SEQUENCE
        SEQUENCE member.id = 1
        ```
    4. SEQUENCE 전략 설명
        - SEQUENCE 사용코드는 IDENTITY와 같지만 동작방식은 다르다.
        - SEQUENCE 전략의 경우 `em.persit()`를 호출할 때 먼저 데이터베이스 시퀀스를 사용해 식별자를 조회
        - 조회한 식별자를 엔티티에 할당한 뒤 영속성 컨텍스트에 저장
        - 트랜젝션 커밋해서 플러시가 일어나면 엔티티를 저장           
- TABLE 전략 : 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 칼럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략, 모든 DB에서 사용가능
    1. `MemberForTable.java`
        ```java
        @Entity
        @TableGenerator(name = "MEMBER_SEQ_GENERATOR",
                table = "MY_SEQUENCE",
                pkColumnName = "MEMBER_SEQ",
                allocationSize = 1)
        public class MemberForTable {
            @Id
            @GeneratedValue(strategy = GenerationType.TABLE, generator = "MEMBER_SEQ_GENERATOR") // 식별자 생성 전략 SEQUENCE로 설정
            @Column(name = "ID")
            private Long id;    // Long 타입
            // ...
        }
        ```
    2. `JpaMain.java`
        ```java
        // H2 DB : 기본키 Table 전략
        MemberForSequence member = new MemberForSequence();
        member.setUsername("더블에스");
        em.persist(member);
        System.out.println("TABLE member.id = " + member.getId());
        ```
    3. 프로젝트 실행 후 콘솔화면
        ```
        Hibernate:
            drop table member_for_table if exists
        Hibernate:
            drop table MY_SEQUENCE if exists
        Hibernate:
            create table member_for_table (
                id bigint not null,
                age integer,
                create_date timestamp,
                description clob,
                last_modified_date timestamp,
                role_type varchar(255),
                name varchar(10) not null,
                primary key (id)
            )
        Hibernate:
            create table MY_SEQUENCE (
                 MEMBER_SEQ varchar(255) not null ,
                next_val bigint,
                primary key ( MEMBER_SEQ )
            )
        8월 16, 2017 3:23:51 오후 org.hibernate.tool.hbm2ddl.SchemaExport execute
        INFO: HHH000230: Schema export complete
        Hibernate:
            select
                tbl.next_val
            from
                MY_SEQUENCE tbl
            where
                tbl.MEMBER_SEQ=? for update

        Hibernate:
            insert
            into
                MY_SEQUENCE
                (MEMBER_SEQ, next_val)  
            values
                (?,?)
        Hibernate:
            update
                MY_SEQUENCE
            set
                next_val=?  
            where
                next_val=?
                and MEMBER_SEQ=?
        TABLE member.id = 1
        ```
    4. TABLE 전략 설명
        - TABLE 전략은 값을 조회하면서 SELECT 쿼리를 사용하고, 다음 값을 증가 시키기 위해 UPDATE 쿼리를 사용한다.

- AUTO 전략 : 선택한 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택
    1. `MemberAuto.java`
        ```java
        @Entity
        public class MemberAuto {
            @Id
            @Column(name = "ID")
            @GeneratedValue(strategy = GenerationType.AUTO)
            private Long id;
            // ...
        }
        ```
    2. `JpaMain.java`
        ```java
        // H2 DB : 기본키 Auto 전략
        MemberAuto member = new MemberAuto();
        member.setUsername("더블에스");
        em.persist(member);
        System.out.println("AUTO member.id = " + member.getId());
        ```
    3. 프로젝트 실행 후 콘솔 화면
        ```
        Hibernate:
            drop table member_auto if exists
        Hibernate:
            drop sequence if exists hibernate_sequence
        Hibernate:
            create table member_auto (
                id bigint not null,
                age integer,
                create_date timestamp,
                description clob,
                last_modified_date timestamp,
                role_type varchar(255),
                name varchar(10) not null,
                primary key (id)
            )
        Hibernate:
            create sequence hibernate_sequence start with 1 increment by 1
        8월 16, 2017 4:01:13 오후 org.hibernate.tool.hbm2ddl.SchemaExport execute
        INFO: HHH000230: Schema export complete
        Hibernate:
            call next value for hibernate_sequence
        AUTO member.id = 1
        Hibernate:
            /* insert com.doubles.jpastudy.MemberAuto
                */ insert
                into
                    member_auto
                    (age, create_date, description, last_modified_date, role_type, name, id)
                values
                    (?, ?, ?, ?, ?, ?, ?)
        ```
     4. AUTO 전략 설명
        - AUTO 전략의 장점은 데이터베이스를 변경해도 코드를 수정할 필요가 없다
        - 특히 키 생성 전략이 확정되지 않은 개발 초기단계나 프로토 타입 개발시 편리하게 사용할 수 있음

- 기본키 매핑 정리
    1. 직접할당 : `em.persist()`를 호출하기전에 애플리케이션에서 직접 식별자 값을 할당해야한다. 값이 없으면 예외발생
    2. SEQUENCE : 데이터베이스 시퀀스에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장
    3. TABLE : 데이터베이스 시퀀스 생성요 테이블에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장
    4. IDENTITY : 테이터베이스에 엔티티를 저장해서 식별자 값을 획득한 후 영속성 컨텍스트에 저장(테이블에 데이터를 저장해야 식별자 값 획득 가능)

## 6. 필드와 칼럼 매핑 : 레퍼런스

- 필드와 칼럼 매핑 분류
    1. `@Column` : 칼럼을 매핑
        - `name` 속성 - 필드와 매핑할 테이블의 컬럼 이름
        - `nullable` 속성 - null 값의 허용여부를 설정, 기본값 true
        - `unique` 속성 - 한 컬럼에 간단한 유니크 제약조건을 걸 때 사용, 만약 두 컬럼이상일 경우 클래스 레벨에서 사용해야함
        - `length` 속성 - 문자 길이 제약조건, `String` 타입일 경우만 사용, 기본값 255
        - 주의 : 자바의 기본타입의 경우 `@Column`을 사용하면 `nullable = false`로 지정하는 것이 안전하다.
    2. `@Enumerated` : 자바의 enum 타입을 매핑
        - `value` 속성의 기능들
            - `EnumType.ORDINAL` - enum 순서를 데이터베이스에 저장, DB에 저장되는 크기가 작음, 저장된 enum의 순서변경 불가
            - `EnumType.STRING` - enum 이름을 데이터베이스에 저장, DB에 저장되는 크기가 비교적 큼, 저장된 enum의 순서가 바뀌거나 추가되도 안전
    3. `@Temporal` : 날짜 타입을 매핑
        - `value` 속성의 기능들
            - `TemporalType.DATE` - 날짜, 데이터베이스 date 타입과 매핑, ex) 2017-01-01
            - `TemporalType.TIME` - 시간, 데이터베이스 time 타입과 매핑, ex) 01:00:00
            - `TemporalType.TIMESTAMP` - 날짜와 시간 데이터베이스 timestamp 타입과 매핑, ex) 2017-01-01 01:00:00
    4. `@Lob` : BLOB, CLOB 타입을 매핑
        - 지정할 수 있는 속성이 없음
        - 매핑하는 필드 타입이 문자면 CLOB으로 매핑, 나머지는 BLOB으로 매핑
            - `CLOB` - String, char[], java.sql.CLOB
            - `BLOB` - byte[], java.sql.BLOB
    5. `@Transient` : 특정 필드를 DB에 매핑하지 않음
        - 데이터베이스에 저장하지 않고, 조회도 하지않음
        - 객체에 임시로 어떤 값을 보관하고 싶을 경우 사용
    6. `@Access` : JPA가 엔티티에 접근하는 방식을 지정
        - 필드 접근 - `AccessType.FIELD`, 필드에 직접 접근, 접근권한이 `private`이어도 가능
        - 프로퍼티 접근 - `AccessType.PROPERTY`  접근자(`getter`)를 사용
