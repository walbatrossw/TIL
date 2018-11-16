# 스프링부트 프로젝트 생성 및 기본세팅

## 1. 서버포트 변경

`application.properties`에서 필요한 속성들을 지정하는 자동완성 기능을 이용할 수 있는데
서버 포트를 변경하려면 `sever.port=포트번호`으로 변경하면 된다.

## 2. 기본 패키지가 아닌 다른 패키지의 클래스들을 스프링에 인식 시키는 방법

`@ComponentScan`의 속성을 아래와 같이 작성해준다.

```java
@SpringBootApplication
@ComponentScan(basePackages = {"com.doubles.demo1", "org.doubles.demo2"})
public class Demo1Application {

    public static void main(String[] args) {
        SpringApplication.run(Myboot01Application.class, args);
    }
}
```

## 3. Lombok 라이브러리

### 3.1 Lombok 라이브리란?

- `getter` / `setter` / `toString()` 메서드, 생성자 메서드 등을 자동으로 생성
- 자바코드를 컴파일할 때 자동으로 추가 메서드를 만들어서 컴파일해주는 라이브러리

### 3.2 `@Data` 애너테이션을 사용할 때 주의

객체 간의 연관관계(부모, 자식)에서 `toString()`메서드가 문제를 발생 시킬 수 있다.

```java
public class Member {

    private String memberId;
    private String memberPw;

    private Address address;

    @Override
    public String toString() {
        return "Member{" +
                "memberId='" + memberId + '\'' +
                ", memberPw='" + memberPw + '\'' +
                ", address=" + address +
                '}';
    }
}                
```

```java
public class Address {
    private String zipcode;
    private String city;
    private Member member;

    @Override
    public String toString() {
        return "Address{" +
                "zipcode='" + zipcode + '\'' +
                ", city='" + city + '\'' +
                ", member=" + member +
                '}';
    }
}
```

아래의 코드를 보면 `Member`의 인스턴스의 `toString()`를 호출하게 되면 `Address`
인스턴스의 `toString()`을 호출하게되고, 다시 `Member`의 `toString()`를 호출하게되면서
무한루프에 빠지게 된다. 그러므로 `@Data` 애너테이션을 사용하는 것보다 `@Getter`,
`@Setter`, `@ToString`을 따로 써주는 것이 바람직하다.
