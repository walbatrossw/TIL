> 본 글은 [이펙티브 자바 2nd](https://book.naver.com/bookdb/book_detail.nhn?bid=8064518)를
읽고 개인적으로 학습한 내용을 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다.
오류가 있다면 지적 부탁드리겠습니다.

# Rule 3. `private` 생성자나 `enum` 자료형은 싱글턴 패턴을 사용

## 3.1 싱글턴 패턴이란?

객체를 하나만 만들 수 있는 클래스다. 클래스를 싱글턴으로 만들면 클라이언트를 테스트하기가
어려워질 수 있다. 싱글턴이 어떤 인터페이스를 구현한 것이 아니면 가짜 구현으로 대체할 수
없기 때문이다.

## 3.2 JDK 1.5 이전의 싱글턴 구현

생성자를 `private`으로 선언하고, 싱글턴 객체는 정적 멤버를 통해 이용하는 방법 2가지는
아래와 같다.

### 3.2.1 `public final` 필드를 이용한 싱글턴 구현

```java

public class Elvis {

  // 정적 멤버 final 선언
  public static final Elvis Instance = new Elvis();

  // 생성자
  private Elvis() {
    //...
  }

  public void leavingTheBuilding() {
    //...
  }

}
```

리플렉션 기능을 통해 `private` 생성자를 호출할 수 있기 때문에 2번째 객체를 생성하라는
요청을 받으면 예외를 던지도록 생성자를 수정해야한다.

### 3.2.2 정적 팩터리를 이용할 싱글턴 구현

```java
public class Elvis {

  // 정적 멤버 final 선언
  private static final Elvis INSTANCE = new Elvis();

  // 생성자
  private Elvis() {
    //...
  }

  // 정적 팩터리 메서드
  public static Elvis getInstance() {
    return INSTANCE;
  }

  public void leavingTheBuilding() {
    //...
  }

}
```

`Elvis.getInstance`는 항상 같은 객체에 대한 참조를 반환한다. 위에서 말한 것과 같이
리플렉션 기능을 통해 `private` 생성자 호출이 가능하다. 그래서 예외 설정을 해야한다.

### 3.2.3 직렬화 가능 클래스

`implement Serializable`을 추가하는 것으로는 부족하기 때문에 모든 객체 필드에 `transient`로
선언하고 `readResolve`메서드를 추가 해야한다. 그렇지 않으면 역직렬화될 때마다 새로운
객체가 생기게 된다.

```java
private Object readResolve() {
  // 동일한 Elvis 객체가 반환되도록 하는 동시에,
  // 가짜 Elvis 객체는 가비지 컬렉터가 처리하도록 만든다.
  return INSTANCE;
}
```

## 3.3 JDK 1.5 부터의 싱글턴 구현

**원소가 하나뿐인 `enum` 자료형을 정의하는 방법이다.**

```java
public enum Elvis {
  INSTANCE;

  public void leavingTheBuilding() {
    // ...
  }
}
```

기능적으로는 `public` 필드를 사용하여 구현하는 방법과 동등하다. 하지만 이 구현 방법은
좀더 간결하고, 직렬화가 자동으로 처리되는 것이 차이점이다. 그리고 리플렉션에도 안전하다.

## 3.4 요약

원소가 하나뿐인 `enum` 자료형이야말로 싱글턴을 구현하는 가장 좋은 방법이다.
