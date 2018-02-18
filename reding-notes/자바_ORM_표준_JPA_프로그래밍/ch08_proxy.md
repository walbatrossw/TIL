# ch08. 프록시와 연관관계 관리

## 1) 프록시

- 프록시 기초 `EntityManager.find()` VS. `EntityManager.getReference()`
    * `EntityManager.find()`
        - JPA에서 식별자로 엔티티 하나를 조회할 때 사용
        - 영속성 컨텍스에 엔티티가 없으면 데이터베이스를 조회
        - 엔티티를 직접 조회하면 조회한 엔티티를 실제 사용하든 안하든 데이터베이슬 조회
    * `EntityManager.getReference()`
        - 엔티티를 사용하는 시점까지 데이터베이스 조회를 미루고 싶을 때 사용
        - 이 메서드를 호출할 때 JPA는 데이터베이스를 호출하지 않고 실제 엔티티 객체도 생성하지 않음
        - 대신 데이터베이스 접근을 위임한 프록시 객체를 반환
- 프록시 특징 1
    * 프록시 클래스는 실제 클래스를 상속 받아서 만들어지기 때문에 실제 클래스와 겉모양이 같음
    * 사용자 입장에서는 뭐가 진짜 객체인지 구분하지 않고 사용하면 됨
    * 프록시 객체는 실제 객체에 대한 참조를 보관
    * 프록시 객체의 메서드를 호출하면 실제 객체의 메서드를 호출
- 프록시 객체의 초기화
    * 프록시 객체는 실제 사용될 때 데이터베이스를 조회해서 실제 엔티티를 생성함(이것을 프록시 객체의 초기화라 부름)
    * 초기화 과정
        ```java
        // MemberProxy 반환
        Member member = em.getReference(Member.class, "id1");
        member.getName();  
        ```
        ```java
        // 프록시 클래스 예상 코드
        class MemberProxy extends Member {
              Member target = null;
              public String getName() {
                  if (target == null) {
                  // 2. 초기화 요청
                  // 3. DB 조회
                  // 4. 실제 엔티티 생성 및 참조 보관
                  this.target = ...;
                  }
              // 5. target.getName();
              return target.getName();    
              }
        }
        ```
        1. 프록시 객체에 `member,getName()`을 호출해서 실제 데이터를 조회
        2. 프록시 객체는 실제 엔티티가 생성되어 있지 않으면 영속성 컨텍스트에 실제 엔티티 생성을 요청(초기화)
        3. 영속성 컨텍스트는 데이터베이스를 조회해서 실제 엔티티 객체를 생성
        4. 생성된 실제 엔티티 객체의 참조를 `Member target` 멤버 변수에 보관
        5. 프록시 객체는 실제 엔티티 객체의 `getName()`를 호출해서 결과를 리턴
- 프록시의 특징 2
    * 처음 사용할 때 한번만 초기화
    * 프록시 객체가 초기화한다고 프록시 객체가 실제 엔티티로 바뀌는 것은 아님(프록시 객체를 통해 실제 엔티티에 접근이 가능해짐)
    * 프록시 객체는 원본 엔티티를 상속받은 객체이기 때문에 타입 체크시 주의
    * 영속성 컨텍스트에 찾는 엔티티가 이미 존재하면 데이터베이스를 조회할 필요가 없기 때문에 프록시가 아닌 실제 엔티티를 리턴
    * 초기화는 영속성 컨텍스트의 도움을 받아야 가능, 준영속 상태의 프록시를 초기화하면 문제가 발생
- 프록시와 식별자
    * 엔티티를 프록시로 조회할 때 식별자(PK) 값을 파라미터로 전달하는데 프록시 객체는 이 식별자 값을 보관
        ```java
        Team team = em.getReference(Team.class, "team1"); // 프록시 객체는 식별자 값을 보관
        team.getId(); // 초기화되지 않음(식별자값을 보관하고 있기 때문)
        ```
        1. 초기화 X : 엔티티 접근 방식을 프로퍼티`@Access(AccessType.PROPERTY)`로 설정한 경우
        2. 초기화 O : 필드 방식 `@Access(AccessType.FEILD)`로 설정한 경우
            - JPA는 `getId()`메서드가 id만 조회하는지 아니면 다른 필드까지 활용해서 다른 역할까지 수행하는지 알수 없기 때문에 프록시 객체를 초기화 함    
    * 프록시는 연관관계를 설정할 때 유용하게 사용할 수 있음(연관관계 설정시 식별자 값만을 사용하기 때문에 데이터베이스 접근 횟수를 줄임)
        ```java
        Member member = em.getReference(Member.class, "member1");
        Team team = em.getReference(Team.class, "team1"); // SQL 실행하지 않고 식별자 값을 보관
        member.setTeam(team);
        ```
        - 연관관계를 설정할 때는 엔티티 접근방식을 필드방식으로 설정하더라도 초기화하지 않음
- 프록시 확인
    - `PesistanceUnitUtil.isLoaded(Object entity)` : 프록시 인스턴스의 초기화 여부를 확인,
        1. false 리턴 : 초기화 되지 않은 경우
        2. true 리턴 : 이미 초기화 되거나 프록시 인스턴스가 아닌 경우

## 2) 지연 로딩과 즉시 로딩

- 즉시 로딩 : 엔티티를 조회할 때 연관된 엔티티도 즉시 함께 조회
    * `@ManyToOne(fetch = FetchType.EAGER)`
    * 즉시 로딩을 최적화하기 위해 가능하면 조인쿼리를 사용
- 지연 로딩 : 엔티티를 조회할 때 연관된 엔티티는 조회하지 않음, 연관된 엔티티를 실제 사용할 때 조회(프록시로 조회)    
    * `@ManyToOne(fetch = FetchType.LAZY)`

## 3) 지연 로딩 활용

- JPA 기본 페치 전략
    * 연관된 엔티티가 하나면 즉시 로딩!, 컬렉션이면 지연 로딩!
        1. `@ManyToOne`, `@OneToOne` - 즉시 로딩(`FetchType.EAGER`)
        2. `@OneToMany`, `@ManyToMany` - 지연 로딩(`FetchType.LAZY`)
    * 추천하는 방법은 모든 연관관계에 지연로딩을 사용하는 것
    * 그 이후 개발이 완료단계에 이르렀을 때 실제 사용하는 상황에 따라 즉시로딩을 사용하도록 최적화

- 컬렉션에 `FetchType.EAGER` 사용 시 주의!
    * 컬렉션을 하나 이상 즉시 로딩하는 것은 권장하지 않음
    * 컬렉션 즉시 로딩은 항상 외부조인(Outer join)을 사용
    * `FetchType.EAGER`  설정과 조인 전략
        1. `@ManyToOne`, `@OneToOne` : 연관 엔티티가 하나일 경우
            - `(optional = false)` : 내부조인
            - `(optional = true)`  : 외부조인
        2. `@OneToMany`, `@OneToMany` : 연관 엔티티가 컬렉션일 경우
            - `(optional = false)` : 외부조인
            - `(optional = true)`  : 외부조인

## 4) 영속성 전이 : CASCADE
특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶다면 영속성 전이 기능을 사용, 쉽게 말해 부모엔티티를 저장할 때 자식엔티티도 함께 저장하는 방법!

- 영속성 전이 적용 전 코드
    * 부모 엔티티
        ```java
        @Entity
        public class Parent {

            @Id
            @GeneratedValue
            private Long id;

            @OneToMany(mappedBy = "parent")
            private List<Child> children = new ArrayList<Child>();

            // ...
        }
        ```
    * 자식 엔티티
        ```java
        @Entity
        public class Child {
            @Id
            @GeneratedValue
            private Long id;

            @ManyToOne
            private Parent parent;

            // ...
        }
        ```
    * 부모, 자식 엔티티 저장
        ```java
        public class NoCascadeMain {
            public static void main(String[] args) {

                EntityManagerFactory emf = Persistence.createEntityManagerFactory("ch08");
                EntityManager em = emf.createEntityManager();
                EntityTransaction tx = em.getTransaction();

                try {
                    tx.begin();
                    // 비지니스 로직
                    saveNoCascade(em);
                    tx.commit();
                } catch (Exception e) {
                    e.printStackTrace();
                    tx.rollback();
                } finally {
                    em.close();
                }
                emf.close();
            }

            private static void saveNoCascade(EntityManager em) {
                // 부모 엔티티 저장
                Parent parent = new Parent();
                em.persist(parent);

                // 자식 엔티티 1 저장
                Child child1 = new Child();
                child1.setParent(parent);
                parent.getChildren().add(child1);
                em.persist(child1);

                // 자식 엔티티 2 저장
                Child child2 = new Child();
                child2.setParent(parent);
                parent.getChildren().add(child2);
                em.persist(child2);

                // JPA에서 엔티티를 저장할 때 연관된 모든 엔티티는 영속 상태이어야 함
                // 부모 엔티티를 영속상태로 만들고 나머지 두 자식 엔티티도 영속상태로 만듬
                // 영속성 전이를 사용하면 부모만 영속상태로 만들면 연관된 자식 엔티티도 함께 영속상태로 만들 수 있음
            }
        }      
        ```
- 영속성 전이 적용 후 코드
    * 부모 엔티티 : 영속성 전이를 활성화하는 CASCADE 옵션 설정
        ```java
        @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST) // 영속성 전이 활성화
        private List<Child> children = new ArrayList<Child>();
        ```
    * 부모, 자식 엔티티 저장 : 이전의 코드에서 처럼 부모, 자식 각각 따로 엔티티를 저장할 필요가 없음
        ```java
        public class CascadeMain {
            public static void main(String[] args) {

                EntityManagerFactory emf = Persistence.createEntityManagerFactory("ch08");
                EntityManager em = emf.createEntityManager();
                EntityTransaction tx = em.getTransaction();

                try {
                    tx.begin();
                    // 비지니스 로직
                    saveWithCascade(em);
                    tx.commit();
                } catch (Exception e) {
                    e.printStackTrace();
                    tx.rollback();
                } finally {
                    em.close();
                }
                emf.close();
            }

            private static void saveWithCascade(EntityManager em) {

                Child child1 = new Child();
                Child child2 = new Child();

                Parent parent = new Parent();
                child1.setParent(parent);   // 연관관계 추가
                child2.setParent(parent);   // 연관관계 추가
                parent.getChildren().add(child1);
                parent.getChildren().add(child2);

                // 부모, 연관된 엔티티(자식엔티티) 저장
                em.persist(parent);

            }
        }
        ```
- 영속성 전이는 연관관계 매핑과는 관련이 없고, 엔티티를 영속화할 때 연관된 엔티티도 같이 영속화하는 편리함을 제공할 뿐
- 영속성 전이 삭제
    1. 영속성 전이 미적용
        ```java
       private static void removeNoCascade(EntityManager em) {
            // 엔티티 제거
            Parent findParent = em.find(Parent.class, 1L);
            Child findChild1 = em.find(Child.class, 1L);
            Child findChild2 = em.find(Child.class, 2L);

            em.remove(findChild1);
            em.remove(findChild2);
            em.remove(findParent);
       }
        ```
    2. 영속성 전이 적용
        ```java
        @OneToMany(mappedBy = "parent", cascade = CascadeType.REMOVE); // 영속성 전이 삭제 옵션
        ```
        ```java
        private static void removeWithCascade(EntityManager em) {
            Parent findParent = em.find(Parent.class, 1L);
            em.remove(findParent);
        }
        ```
- 영속성 전이 종류
    * `CascadeType.ALL` - 모두 적용
    * `CascadeType.PERSIST` - 영속
    * `CascadeType.MERGE` - 병합
    * `CascadeType.REMOVE` - 삭제
    * `CascadeType.REFRESH` - REFRESH
    * `CascadeType.DETACH` - DETACH
    * 여러 속성을 같이 사용할 경우
        ```java
        cascade = {CascadeType.PERSIST, CascadeType.REMOVE}
        ```
    * `CascadeType.PERSIST`, `CascadeType.REMOVE`는 `persist()`, `remove()`를 실행할 때 전이가 발생하지 않고 플러시를 호출할 때 전이가 발생

## 5) 고아객체
JPA는 부모 엔티티와 연관관계 끊어진 자식 엔티티를 자동으로 삭제하는 기능을 제공하는데 이것을 고아 객체(ORPHAN) 제거라고 함

- 고아 객체 제거 기능 설정
    ```java
    @Entity
    public class Parent {

        @Id
        @GeneratedValue
        private Long id;

        @OneToMany(mappedBy = "parent", orphanRemoval = true) // 고아객체 제거
        private List<Child> children = new ArrayList<Child>();

        // ...
    }
    ```
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능

## 6) 정리

- JPA 구현체들은 객체 그래프를 탐색할 수 있도록 지원하는데 프록시 기술을 사용
- 객체를 조회할 때 연관된 객체를 즉시 로딩하는 방법을 즉시로딩, 연관된 객체를 지연해서 로딩하는 방법을 지연로딩
- 객체를 저장하거나 삭제할 때 연관된 객체도 함께 저장하거나 삭제할 수 있는데 이것을 영속성 전이라고 함
- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제하려면 고아 객체 제거 기능을 사용
