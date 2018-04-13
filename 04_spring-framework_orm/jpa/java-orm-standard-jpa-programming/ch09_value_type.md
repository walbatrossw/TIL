# Ch09. 값타입

## 1) 기본값 타입
- 기본값 타입
    ```java
    @Entity
    public class Member {

        @Id
        @GeneratedValue
        private Long id;

        // 값 타입
        private String name;

        // 값 타입
        private int age;

        // name, age 속성은 식별자 값X, 생명주기 회원엔티티에 의존
        // 값 타입은 공유X
    }
    ```

## 2) 임베디드 타입(복합 값 타입)

JPA에서는 새로운 값 타입을 직접 정의해서 사용하는 것을 임베디드 타입이라고 한다.

- 임베디드 타입을 사용하기 전

    * 일반적인 회원 엔티티

        ```java
        @Entity
        public class GeneralMember {

            @Id
            @GeneratedValue
            private Long id;              // 아이디

            private String username;      // 회원이름

            // 근무 기간
            @Temporal(TemporalType.DATE) java.util.Date startDate;
            @Temporal(TemporalType.DATE) java.util.Date endDate;

            // 집주소
            private String city;
            private String street;
            private String zipcode;

        }
        ```
- 임베디드 타입을 사용한 후

    * 값 타입 적용 회원 엔티티

        ```java
        @Entity
        public class EmbeddedMember {

            @Id
            @GeneratedValue
            private Long id;                // 아이디
            private String username;        // 회원이름
            // 값 타입을 사용하는 곳에 표시
            @Embedded Period workPeriod;    // 근무 기간
            @Embedded Address homeAdress;   // 집주소

        }
        ```
    * 기간 임베디드 타입

        ```java
        @Embeddable
        public class Period {

            @Temporal(TemporalType.DATE) java.util.Date startDate;
            @Temporal(TemporalType.DATE) java.util.Date endDate;

            public boolean isWork(Date date) {
                // 값타입을 위한 메서드 정의 가능...
                return false;
            }
        }
        ```

    * 주소 임베디드 타입

        ```java
        @Embeddable // 값타입을 정의하는 곳을 표시
        public class Address {

            @Column(name = "city")  // 매핑할 컬럼 정의 가능
            private String city;
            private String street;
            private String zipcode;
        }
        ```

- 임베디드 타입과 테이블 매핑

    * 임베디드 타입은 엔티티 값일뿐, 값이 속한 엔티티의 테이블에 매핑

    * 임베디드 타입을 사용하기 전과 후에 매핑한 테이블은 동일

- 임베디드 타입과 연관관계

    * 임베디드 타입은 값 타입을 포함하거나 엔티티를 참조할 수 있음

    * 임베디드 타입과 연관관계


- `@AttributeOverride` : 속성 재정의

    * 임베디드 타입에 정의한 매핑정보를 재정의하려면 엔티티에 `@AttributeOverride`를 사용

    * 예를 들어 회원에게 집주소 외에 직장주소가 필요하면 어떻게 해야할까?

- 임베디드 타입과 null

    * 임베디드 타입이 null이면 매핑한 칼럼 값은 모두 null이 됨         


## 3) 값 타입과 불변객체
## 4) 값 타입 비교
## 5) 값 타입 컬렉션
## 6) 정리
