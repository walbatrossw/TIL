> 본 글은 [이펙티브 자바 2nd](https://book.naver.com/bookdb/book_detail.nhn?bid=8064518)를
읽고 개인적으로 학습한 내용을 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다.
오류가 있다면 지적 부탁드리겠습니다.

# rule 5. 불필요한 객체는 만들지 말것.

## 5.1 변경 불가능한 객체는 언제나 재사용 가능

기능적으로 동일한 객체는 필요할 때마다 생성하지말고 재사용하는 편이 좋다. 변경 불가능한
객체는 언제나 재사용이 가능하다.

### 5.1.2 절대로 피해야할 코드 작성법

```java
String s = new String("stringette");
```

위의 코드의 문장이 실행될 때 마다 `String` 객체를 만드는 쓸데없는 짓을 하게 된다. 만약
위의 문장이 반복문이나 자주 호출되는 메서드 안에 있다면 수백만 갱의 `String` 객체가
쓸데없이 만들어질 것이다.

### 5.1.3 수정된 코드

```java
String s = "stringette";
```

위의 코드는 실행할 때마다 객체를 만드는 대신, 동일한 `String` 객체를 사용한다.

생성자와 정적 팩터리 메서드를 함께 제공하는 변경 불가능 클래스의 경우, 생성자 대신 정적
팩터리 메서드를 이용하면 불필요한 객체 생성을 피할 수 있다. 예를 들어 `Boolean(String)`
보다는 `Boolean.valueOf(String)`이 바람직하다. 생성자는 호출될 때마다 새 객체를 생성하지만
정적 팩터리 메서드는 그렇지 않다.

## 5.2 변경 가능한 객체도 재사용이 가능

### 5.2.1 비효율적인 코드

```java
public class Person {

  private final Date birthDate; // 출생일자

  // 베이비붐 세대 여부 확인 메서드
  public boolean isBabyBoomer() {

    Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));  // Calendar, TimeZone 객체 생성

    gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
    Date boomStart = gmtCal.getTime();  // Date 객체 생성

    gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
    Date boomEnd = gmtCal.getTime();    // Date 객체 생성

    return birthDate.compareTo(boomStart) >= 0 && birthDate.compareTo(boomEnd) < 0;
  }

}
```

위의 코드의 클래스는 베이비 붐 세대에 속하는 사람인지 아닌지 여부를 알려주는 메서드를
가지고 있다. `isBabyBoomer`메서드는 호출될 때마다 `Calendar`, `TimeZone` 객체 각각
하나씩 그리고 `Date`객체 두 개를 쓸데 없이 만들어낸다.


### 5.2.2 수정된 코드

```java
public class Person {

  private final Date birthDate; // 출생일자

  private static final Date BOOM_START; // 베이비부머 시작 연도
  private static final Date BOOM_END;   // 베이비부머 종료 연도

  // 초기화 블럭
  static {
    Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));

    gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
    Date boomStart = gmtCal.getTime();

    gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
    Date boomEnd = gmtCal.getTime();
  }

  // 베이비부머 세대 여부 확인
  public boolean isBabyBoomer() {
    return birthDate.compareTo(boomStart) >= 0 && birthDate.compareTo(boomEnd) < 0;
  }

}
```

위의 코드는 이전에 보았던 코드의 비효율을 개선한 코드인데 정적 초기화 블럭을 통해 개선하는 것이
좋다. 이전의 코드는 `isBabyBoomer`가 호출될 때마다 객체를 생성했지만 위의 코드는 `Person`
클래스가 초기화 할 때에만 객체를 한번 생성한다.

## 5.3 자동객체화(Autoboxing)

JDK 1.5부터는 쓸데 없이 객체를 만들 새로운 방법이 더 생겼다. 자바의 기본 자료형(primitive)과
그 객체의 표현형을 섞어 사용헐 수 있도록 해준다. 둘 간의 변환은 자동으로 이루어진다.
의미적인 차이는 미미하지만, 성능에서 차이가 발생한다.

```java
Long sum = 0L;
for (long i = 0; i < Integer.MAX_VALUE; i++) {
  sum += i;
}
System.out.println(sum);
```

위의 코드는 모든 양의 정수의 합을 계산하는 코드다. 계산 결과는 정확하게 나오지만 성능은
낮게 나온다. 그 이유는 `sum`이 `long`이 아나리 `Long`으로 선언되어야 한다. 그 이유로
2<sup>31</sup>개의 쓸데없는 객체가 만들어지게 된다.

**그렇기 때문에 객체 표현형 대신 기본 자료형을 사용하고, 생각치도 못한 자동 자동객체가
발생하지 않도록 유의해야한다.**

## 5.4 객체풀(Object pool)

객체풀을 만들어 객체 생성을 피하는 기법은 객체 생성 비용이 극단적으로 높지 않다면 사용하지
않는 것이 좋다.

- 객체 생성 비용이 극단적으로 높은 대표적인 예로는 데이터베이스 접속이다.
- 데이터베이스의 라이선스 정책에 따라 데이터베이스 연결 수가 제한될 수 있다.
- 코드의 복잡성, 메모리 요구량이 중가하고, 성능도 저하된다.

최신 JVM은 고도로 최적화된 가비지 컬렉터를 가지고 있어 가벼운 객체라면 월등한 성능을
보여준다.

## 5.5 방어적 복사(Defensive Copy)

재사용이 가능하다면 새로운 객체를 만들지 않는 것이 좋지만, 객체를 새롭게 만들어야 한다면
기존 객체를 재사용하지 않는 것도 좋다. 하지만 방어적 복사가 요구되는 상황에서 객체를
재사용하는데 드는 비용은 쓸데없이 같은 객체를 여러개 만드는 비용보다 훨씬 높기 때문이다.

즉 필요할 때 방어적 복사본을 만들지 못하면 골치아픈 버그나 보안 결함이 생길 수 있다.
