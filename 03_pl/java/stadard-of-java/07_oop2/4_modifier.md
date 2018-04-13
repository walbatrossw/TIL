본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.


## 1. 제어자?

**제어자는 클래스, 변수 또는 메서드의 선언부에 함께 사용되어 부가적인 의미를 부여** 한다. 제어자는 클래스나 멤버변수와 메서드에 주로 쓰이며 하나의 대상에 대해 여러 제어자를 조합하여 사용하는 것이 가능하다. 단, 접근 제어자의 경우 하나만 선택해서 사용해야 한다.
- 접근 제어자 : `public`, `protected`, `default`, `private`
- 그 외의 제어자 : `static`, `final`, `abstract`, `native`, `transient`, `synchronized`, `volatile`, `strictfp`


## 2. `static` - 클래스의, 공통적인

**`static`은 "클래스의", "공통적인"의 의미를 가지고 있다.** 인스턴스 변수는 하나의 클래스로부터 생성되었더라도 각기 다른 값을 유지하지만, 클래스 변수는 인스턴스 변수에 관계 없이 같은 값을 가진다. 그 이유는 **하나의 변수를 모든 인스턴스가 공유** 하기 때문이다. **`static`이 붙은 멤버변수와 메서드, 초기화 블럭은 인스턴스가 아닌 클래스에 관계된 것이기 때문에 인스턴스를 사용하지 않고도 사용이 가능** 하다.

```java
class StaticTest {

  // 클래스 변수
  static int width = 200;
  static int height = 100;

  // 클래스 초기화 블럭
  static {

  }

  // 클래스 메서드
  static int max(int a, int b) {
    return a > b ? a : b;
  }

}
```

- `static` 변수
  - 모든 인스턴스에 공통적으로 사용되는 클래스 변수가 된다.
  - 클래스 변수는 인스턴스를 생성하지 않고 사용이 가능하다.
  - 클래스가 메모리에 로딩될 때 생성된다.
- `static` 메서드
  - 인스턴스를 생성하지 않고도 호출이 가능한 메서드가 된다.
  - `static`메서드 내에서는 인스턴스 멤버들을 직접 사용할 수 없다.

---

## 3. `final` - 마지막의, 변경될 수 없는

**`final`은 "마지막의", "변경될 수 없는"의 의미** 를 가지고 있다. 변수에 사용하게 되면 **값을 변경할 수 없는 상수** 가 되고, **메서드에 사용하면 오버라이딩을 할 수 없게 되고, 클래스에 사용하게 되면 자신의 클래스를 확장하는 자손클래스를 정의 할 수 없다.** 대표적인 `final`클래스로는 `String`, `Math`클래스가 있다.

```java
final class FinalClass {   // 조상이 될 수 없는 클래스
  final int MAX_SIZE = 10; // 값을 변경할 수 없는 멤버변수(상수)

  final void getMaxSize()  {  // 오버라이딩을 할 수 없는 메서드(변경불가)
    final int LV = MAX_SIZE; // 값을 변경할 수 없는 지역변수(상수)
  }
}
```
- `final` 클래스
  - 변경될 수 없는 클래스, 확장될 수 없는 클래스가 된다.
  - 그래서 `final`로 지정된 클래스는 다른 클래스의 조상이 될 수 없다.
- `final` 메서드
  - 변경될 수 없는 메서드
  - `final`로 지정된 메서드는 오버라이딩을 통해 재정의 될 수 없다.
- `final` 멤버변수 / `final` 지역변수
  - 변수 앞에 `final`이 붙으면 값을 변경할 수 없는 상수가 된다.

##### 생성자를 이용한 `final`멤버 변수의 초기화 / 초기화 예제 1

`final`이 붙은 변수는 상수이므로 보통 선언과 초기화를 동시에 하지만, 인스턴스 변수의 경우 생성자에서 초기화 되도록 할 수 있다. 클래스 내에 매개변수를 갖는 생성자를 선언하여, **인스턴스를 생성할 때 `final`이 붙은 멤버변수를 초기화하는데 필요한 값을 생성자의 매개변수로부터 제공받는 것이다.** 이러한 기능을 사용하면 각 인스턴스마다 `final`이 붙은 멤버변수가 다른 값을 갖도록 하는 것이 가능하다. 이것이 불가능하다면 클래스에 선언된 `final`이 붙은 인스턴스변수는 모든 인스턴스에서 같은 값을 가져야만 할 것이기 때문이다.


아래의 예제와 같이 카드의 경우, 각 카드마다 다른 종류와 카드를 가져야하는데 일단 카드가 생성되고 나면 값이 변경되어서는 안된다. 만약 52장의 카드의 값이 1장이라도 변경된다면 동일한 카드가 되어버리기 때문이다.
```java
public class FinalCardTest {
  public static void main(String[] args) {
    Card card = new Card("HEART", 10);
    System.out.println(card.KIND);
    System.out.println(card.NUMBER);
    System.out.println(card);
  }
}

class Card {
  final int NUMBER;       // 상수지만 선언과 함께 초기화하지 않고,
  final String KIND;      // 생성자에서 단 한번만 초기화 할 수 있다.
  static int width = 100;
  static int height = 250;

  public Card(String kind, int number) {
    KIND = kind;
    NUMBER = number;
  }

  public Card() {
    this("HEART", 1);
  }

  public String toString() {
    return KIND + " " + NUMBER;
  }
}
```
```
HEART
10
HEART 10
```

## 4. `abstract` - 추상의, 미완성의

**`abstract`는 "미완성"의 의미** 를 가지고 있다. **메서드 선언부만 작성하고, 실체 수행내용은 구현하지 않은 추상 메서드를 선언하는데 사용** 된다. 또한 클래스에서도 사용되며 클래스 내에 추상메서드가 존재한다는 것을 쉽게 알 수 있게 한다. 추상 클래스는 아직 완성되지 않은 메서드가 존재하는 미완성 설계도이기 때문에 인스턴스를 생성할 수 없다.

```java
abstract class AbstractTest {   // 추상 클래스(추상메서드를 포함한 클래스)
  abstract void move();         // 추상 메서드(구현부가 없는 메서드)
}
```
- `abstract` 클래스 : 클래스 내에 추상메서드가 선언되어 있음을 알린다.
- `abstract` 메서드 : 선언부만 작성하고 구현부는 작성하지 않은 추상 메서드임을 알린다.


## 5. 접근제어자(access modifier)

접근 제어자는 **멤버 또는 클래스에 사용되어, 해당하는 멤버, 클래스를 외부에서 접근하지 못하도록 제한하는 역할** 을 한다. 접근 제어자가 `default`라면 실제로 `default`를 붙이지 않는다. 클래스, 멤버변수, 메서드, 생성자에 접근제어자가 지정되어 있지 않다면 `default`가 접근제어자이다.

- 접근 범위 : `public` > `protected` > `default` > `private`
  |제어자|같은 클래스|같은 패키지|자손 클래스|전체 클래스|
  |---|:---:|:---:|:---:|:---:|
  |`public`|O|O|O|O|
  |`protected`|O|O|O|X|
  |`default`|O|O|X|X|
  |`private`|O|X|X|X|
  - `public` : 접근제한이 전혀 없음
  - `protected` : 패키지에 관계 없이 상속관계에 있는 자손 클래스에서 접근할 수 있도록하는 것이 목적, 같은 패키지도 가능
  - `default` : 같은 패키지 내에서만 접근 가능
  - `private` : 같은 클래스 내에서만 접근 가능

- 대상에 따라 사용할 수 있는 접근 제어자
  |제어자|같은 클래스|
  |---|---|
  |클래스|`public`, `default`|
  |메서드|`public`, `default`, `protected`, `private`|
  |멤버변수|`public`, `default`, `protected`, `private`|
  |지역변수|없음|


##### 접근 제어자를 이용한 캡슐화
접근제어자를 사용하는 이유는 **클래스 내부의 데이터를 외부로부터 보호하기 위해서이다.** 그리고 **외부에는 불필요하고, 내부적으로만 사용되는 부분을 감추기 위해서이기도 하다.** 만일 메서드 하나를 변경한다고 가정해보자. 이 메서드의 접근제어자가 `public`이라면 변경한 뒤 오류가 없는지 테스트를 해야할 범위가 넓어지게 된다. 하지만 제어자가 `default`라면 패키지 내부만 확인해보면되고, `private`이라면 클래스 내부만 확인해보면 된다. 이처럼 접근제어자 하나가 상당한 차이를 만들어낼 수 있기 때문에 접근제어자를 적절히 선택해서 접근 범위를 최소화 할 수 있도록 하는 것이 바람직하다.

```java
public class Time {
  public int hour;
  public int minute;
  public int second;
}
```
```java
Time time = new Time();
time.hour = 25;
```
구체적인 예를 통해 접근제어자를 이용한 캡슐화에 대해 알아보자. 위와 같이 시간을 나타내는 `Time`클래스가 정의되어 있다면 인스턴스를 생성한 다음, 멤버변수에 직접 접근하여 값을 변경할 수 있을 것이다. 하지만 시를 나타내는 멤버변수 `hour`는 0보다는 크거나 같고, 24보다는 작은 범위의 값을 가져야 하지만 위의 코드처럼 25를 지정한다면 막을 방법이 없다. 이럴 경우는 멤버변수를 `private`나 `protected`로 제한하고 멤버변수의 값을 읽고 변경할 수 있는 `public`메서드를 제공함으로써 간접적으로 멤버변수를 다룰 수 있게 하는 것이 바람직하다.

```java
class Time {

  private int hour;
  private int minute;
  private int second;

  public int getHour() {
    return hour;
  }

  public void setHour(int hour) {
    if (hour < 0 || hour > 25)
      return;
    this.hour = hour;
  }

  public int getMinute() {
    return minute;
  }

  public void setMinute(int minute) {
    if (minute < 0 || minute > 59)
      return;
    this.minute = minute;
  }

  public int getSecond() {
    return second;
  }

  public void setSecond(int second) {
    if (second < 0 || second > 59)
      return;
    this.second = second;
  }
}
```
위의 예제는 이전에 살펴본 코드의 문제점을 수정한 코드이다. `Time`클래스의 멤버변수는 `private` 접근제어자를 사용하여 외부로부터 직접 접근을 하지 못하도록 제한하고, get메서드, set메서드를 통해 값을 반환하거나, 값을 변경할 수 있도록 수정하였다. 특히 set메서드의 경우 각 시, 분, 초의 범위에 맞는 값일 때만 멤버변수의 값을 변경하도록 작성하였다.

##### 접근제어자를 이용한 캡슐화 예제
```java
public class TimeTest {
  public static void main(String[] args) {

    Time time = new Time(12, 30, 45);
    System.out.println(time);

    time.setHour(time.getHour()+1);
    System.out.println(time);

  }
}

class Time {

  private int hour;
  private int minute;
  private int second;

  Time(int hour, int minute, int second) {
    setHour(hour);
    setMinute(minute);
    setSecond(second);
  }

  public int getHour() {
    return hour;
  }

  public void setHour(int hour) {
    if (hour < 0 || hour > 25)
        return;
    this.hour = hour;
  }

  public int getMinute() {
    return minute;
  }

  public void setMinute(int minute) {
    if (minute < 0 || minute > 59)
        return;
    this.minute = minute;
  }

  public int getSecond() {
    return second;
  }

  public void setSecond(int second) {
    if (second < 0 || second > 59)
    this.second = second;
  }

  @Override
  public String toString() {
    return "Time{" +
            "hour=" + hour +
            ", minute=" + minute +
            ", second=" + second +
            '}';
  }
}
```
```
Time{hour=12, minute=30, second=0}
Time{hour=13, minute=30, second=0}
```


##### 생성자의 접근제어자
**생성자에 접근제어자를 사용함으로써 인스턴스의 생성을 제한할 수도 있다.** 보통 생성자의 접근제어자는 클래스의 접근제어자와 같지만, 다르게 지정할 수도 있다. 생성자의 접근제어자를 `private`으로 설정하면 외부에서 인스턴스를 생성할 수 없게 되지만 내부에서의 인스턴스 생성이 가능하다. 대신 인스턴스를 생성해서 반환해주는 `public`메서드를 제공함으로써 외부에서 이 클래스의 인스턴스를 사용할 수 있도록 할 수 있다. 대신 이 메서드는 `public`인 동시에 `static`이어야 한다.
```java
class SingleTon {
  // ...

  // getInstance()에서 사용될 수 있도록 인스턴스가 미리 생성되어야 하기 때문에 static
  private static SingleTon st = new SingleTon();
  private SingleTon() {
    // ...
  }

  // 인스턴스를 생성하지 않고도 호출할 수 있어야하기 때문에 static
  public static SingleTon getInstance() {
    return st;
  }

}
```

##### 생성자의 접근제어자 예제
```java
public class SingleTonTest {
  public static void main(String[] args) {
    // SingleTon st = new SingleTon(); // 에러발생, 생성자에 접근할 수 없기 때문
    SingleTon st = SingleTon.getInstance();
  }
}

final class SingleTon {
  private static SingleTon st = new SingleTon();

  private SingleTon() {

  }

  public static SingleTon getInstance() {
    if (st == null)
        st = new SingleTon();
    return st;
  }
}
```



## 6 제어자(modifier)의 조합
**제어자를 조합해서 사용할 때의 주의사항**
- 메서드에 `static`와 `abstract`를 함께 사용할 수 없다.
  - `static`메서는 몸통이 있는 메서드만 사용할 수 있기 때문
- 클래스에 `abstract`와 `final`을 함께 사용할 수 없다.
  - 클래스에 사용되는 `final`은 확장할 수 없음을 의미하고, `abstract`는 상속을 통해 완성되어야한다는 의미로 서로 모순되기 때문
- `abstract`메서드의 접근제어자가 `private`일 수 없다.
  - `abstract`메서드는 자손클래스에서 구현해주어야 하는데 접근제어자가 `private`이면, 자손 클래스에서 접근할 수 없기 때문
- 메서드에 `private`와 `final`을 같이 사용할 필요는 없다.
  - `private`인 메서드는 오버라이딩 될 수 없기 때문에 굳이 `final`을 써주지 않아도 되기 때문
