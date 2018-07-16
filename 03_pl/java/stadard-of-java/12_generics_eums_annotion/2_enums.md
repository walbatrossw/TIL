본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. `Enums`(열거형)이란?
이전까지 Java는 C와 달리 열거형이라는 것이 존재하지 않았으나 JDK1.5부터 새로 추가되었다. Java의 열거형은 C의 열거형보다 더 향상된 것으로 열거형이 갖는 값뿐만 아니라 타입까지 관리하기 때문에 보다 논리적인 오류를 줄일 수 있다.

```java
class Card {
  static final int CLOVER = 0;
  static final int HEART = 1;
  static final int DIAMOND = 2;
  static final int SPADE = 3;

  static final int TWO = 0;
  static final int THREE = 1;
  static final int FOUR = 2;

  final int kind;
  final int num;
}
```
```java
if (Card.CLOVER == Card.TWO) // 결과는 true, false이어야 의미상 맞음
```

```java
class Card {
  enum Kind { CLOVER, HEART, DIAMOND, SPADE } // 열거형 Kind 정의
  enum Value { TWO, THREE, FOUR } // 열거형 Value 정의

  final Kind kind; // 타입이 int가 아닌 Kind임에 유의
  final Value value;
}
```

```java
if (Card.Kind.CLOVER == Card.Value.TWO) // 결과는 false, 값은 같지만 타입이 다름
```

기존의 많은 언어들, 예를 들어 C에서는 타입이 달라도 값이 같으면 조건식 결과가 `true`이었으나, **Java의 타입에 안전한 열거형에서는 실제 값이 같아도 타입이 다르면 조건식의 결과가 `false`가 된다.** 값뿐만 아니라 타입까지 체크하기 때문에 타입에 안전하다고 하는 것이다.

그리고 더 중요한 것은 상수의 값이 바뀌면, 해당 상수를 참조하는 모든 소스를 다시 컴파일해야한다는 것인데 **열거형 상수를 사용하면, 기존의 소스를 다시 컴파일하지 않아도 된다.**

## 2. 열거형의 정의와 사용

#### 열거형의 정의
열거형을 정의하는 방법은 아래의 코드처럼 간단하다. 열거형이름을 선언하고, `{}`안에 상수의 이름을 나열하기만 하면된다.

```java
// 열거형의 정의 방법
enum EnumName { constantName1, constantName2, constantName3, }

// 열거형의 정의 예제 : 동서남북
enum Direction { EAST, SOUTH, WEST, NORTH }
```

#### 열거형의 사용법
열거형을 사용하는 방법은 아래의 코드와 같다. 위에서 정의한 열거형을 이용하여 `Unit`의 방향을 초기화해보자.

```java
class Unit {
  int x, y; // Unit의 위치

  Direction dir;  // 열거형을 인스턴스 변수로 선언

  void int() {
    dir = Direction.WEST; // Unit의 방향을 WEST로 초기화
  }
}
```

#### 열거형의 특징
**열거형 상수간의 비교에는 `==`를 사용** 할 수 있는데, `equals()`가 아닌 `==`로 비교할 수 있다는 것은 그만큼 빠른 성능을 제공한다고 할 수 있다. **`<,>`와 같은 비교연산자를 사용할 수 없고, `compareTo()`는 사용이 가능** 하다. 참고로 `compareTo`는 비교대상이 같을 경우 `0`, 왼쪽이 크면 양수, 오른쪽이 크면 음수를 반환한다.

```java
if (dir == Direction.WEST) {
  x++;
} else if (dir > Direction.WEST) { // 에러, 열거형 상수에 비교연산자 사용불가
  // ...
} else if (dir.compareTo(Direction.WEST) > 0) { // OK, compareTO는 사용가능
  // ...
}
```

그리고 **`switch`문의 조건식에도 열거형을 사용** 할 수 있는데 이 때 주의할 점은 `case`문에 열거형의 이름은 적지 않고, **상수의 이름만 적어야 한다는 제약이 있다.** 이렇게 하는 이유는 아마도 오타를 줄일 수 있고, 보기에도 간결하기 때문인듯하다.

```java
void move() {
  switch (dir) {
    case EAST: x++; // Direction.EAST 라고 쓰지 않음
      break;
    case WEST: x--;
      break;
    case SOUTH: y++;
      break;
    case NORTH: y--;
      break;

  }
}
```

#### 모든 열거형의 조상 - `java.lang.Enum`
열거형 `Direction`에 정의된 모든 상수를 출력하려면, 다음과 같이 하면된다.

```java
Direction[] dArr = Direction.values();

for (Direction d : dArr)
  System.out.printf("%s=%d%n", d.name(), d.ordinal());
```

**`Enum` 클래스에 정의된 메서드**
- `values()` : 열거형의 모든 상수를 배열에 담아 반환
- `Class<E> getDeclaringClass()` : 열거형의 `Class`객체를 반환
- `String name()` : 열거형 상수의 이름을 문자열로 반환
- `int ordinal()` : 열거형 상수가 정의된 순서를 반환(0부터 시작)
- `T valueOf(Class<T> enumType, String name)` : 지정된 열거형에서 `name`과 일치하는 열거형 상수를 반환

#### 열거형 예제 1
지금까지 살펴본 열거형의 내용을 예제를 통해 다시 복습해보자

```java
public class EnumEx1 {
    public static void main(String[] args) {
        Direction d1 = Direction.EAST;
        Direction d2 = Direction.valueOf("WEST");
        Direction d3 = Enum.valueOf(Direction.class, "EAST");

        System.out.println("d1 = " + d1);
        System.out.println("d2 = " + d2);
        System.out.println("d3 = " + d3);

        System.out.println("d1 == d2 ? " + (d1 == d2));
        System.out.println("d1 == d3 ? " + (d1 == d3));
        System.out.println("d1.equals(d3) ? " + d1.equals(d3));
        // System.out.println("(d2 > d3) ? " + (d2 > d3)); // 에러
        System.out.println("d1.compareTo(d2) ? " + d1.compareTo(d2));
        System.out.println("d1.compareTo(d3) ? " + d1.compareTo(d3));

        switch (d1) {
            case EAST:
                System.out.println("The direction is EAST.");
                break;
            case SOUTH:
                System.out.println("The direction is SOUTH.");
                break;
            case WEST:
                System.out.println("The direction is WEST.");
                break;
            case NORTH:
                System.out.println("The direction is NORTH.");
                break;
            default:
                System.out.println("Invalid direction.");
                break;
        }

        Direction[] dArr = Direction.values();
        for (Direction d : dArr)
            System.out.printf("%s=%d%n", d.name(), d.ordinal());
    }
}

enum Direction {
    EAST, SOUTH, WEST, NORTH
}
```

```
d1 = EAST
d2 = WEST
d3 = EAST
d1 == d2 ? false
d1 == d3 ? true
d1.equals(d3) ? true
d1.compareTo(d2) ? -2
d1.compareTo(d3) ? 0
The direction is EAST.
EAST=0
SOUTH=1
WEST=2
NORTH=3
```

## 3. 열거형에 멤버 추가하기

#### 열거형 값 지정
`Enum`클래스에 정의된 **`ordinal()`이 열거형 상수가 정의된 순서** 를 반환하지만, 이 값을 **열거형의 상수 값으로 사용하는 것은 바람직하지 않다.** 그 이유는 이 값은 내부적인 용도로만 사용되기 위한 것이기 때문이다. 그래서 열거형 상수의 값이 불규칙한 경우에는 아래의 코드와 같이 **열거형 상수의 이름 옆에 원하는 값을 괄호`()`와 함께 적어주면 된다.**

```java
// 원하는 값을 설정
enum Direction { EAST(1), SOUTH(5), WEST(-1), NORTH(10) }
```

#### 열거형에 값 지정 후 추가해야할 사항
지정된 값을 저장할 수 있는 인스턴스 변수와 생성자를 새로 추가해줘야 한다. 이 때 주의해야할 것은 먼저 열거형 상수를 모두 정의한 다음에 다른 멤버들을 추가해야한다. 그리고 열거형 상수의 마지막에 세미콜론(`;`)도 잊지 않아야한다.

```java
enum Direction {

  EAST(1), SOUTH(5), WEST(-1), NORTH(10); // 상수 마지막에는 항상 세미콜론을 꼭 붙여야한다.

  private final int value; // 정수를 저장할 필드를 추가, 열거형 상수를 저장하기 위해 final을 붙임

  // 생성자 추가
  Direction(int value) {
    this.value = value;
  }

  // 외부에서 value 값을 얻을 수 있도록 메서드 추가
  public int getValue() {
    return value;
  }

}
```

열거형 `Direction`에 새로운 생성자가 추가되었지만, **열거형의 객체를 생성할 수 없다. 열거형의 생성자는 묵시적으로 `private`이기 때문이다.**
```java
Direction d = new Direction(1); // 에러, 열거형의 생성자는 외부에서 호출불가
```

#### 열거형 상수의 여러 값 지정
열거형 상수에 여러 값을 지정할 수 있는데, 그에 맞게 인스턴수 변수와 생성자 등을 새로 추가해주어야 한다.
```java
enum Direction {

  // 열거형 상수에 숫자와 방향을 나타내는 기호 두가지를 지정
  EAST(1, ">"), SOUTH(5, "V"), WEST(-1, "<"), NORTH(10, "^");

  private final int value; // 정수를 저장할 필드
  private finla String symbol; // 기호를 저장할 필드

  Direction(int value, String symbol) {
    this.value = value;
    this.symbol = symbol;
  }

  public int getValue() {
    return value;
  }

  public String getSymbol() {
    return symbol;
  }

}
```

#### 열거형 멤버 추가하기 예제 1

```java
public class EnumEx2 {
    public static void main(String[] args) {
        for (Direction2 d : Direction2.values())
            System.out.printf("%s=%d%n", d.name(), d.getValue());

        Direction2 d1 = Direction2.EAST;
        Direction2 d2 = Direction2.of(1);

        System.out.printf("d1=%s, %d%n", d1.name(), d1.getValue());
        System.out.printf("d2=%s, %d%n", d2.name(), d2.getValue());

        System.out.println(Direction2.EAST.rotate(1));
        System.out.println(Direction2.EAST.rotate(2));
        System.out.println(Direction2.EAST.rotate(-1));
        System.out.println(Direction2.EAST.rotate(-2));
    }
}

enum Direction2 {

    EAST(1, ">"), SOUTH(2, "V"), WEST(3, "<"), NORTH(4, "^");

    private static final Direction2[] DIR_ARR = Direction2.values();
    private final int value;        // 정수를 저장할 필드
    private final String symbol;    // 기호를 저장할 필드

    Direction2(int value, String symbol) {
        this.value = value;
        this.symbol = symbol;
    }

    public int getValue() {
        return value;
    }

    public String getSymbol() {
        return symbol;
    }

    public static Direction2 of(int dir) {
        if (dir < 1 || dir > 4) {
            throw new IllegalArgumentException(("Invalid value : " + dir));
        }
        return DIR_ARR[dir - 1];
    }

    // 방향을 회전시키는 메서드, num의 값만큼 90도씩 시계방향으로 회전
    public Direction2 rotate(int num) {
        num = num % 4;

        if (num < 0)    // num이 음수일 경우, 시계반대방향으로 회전
            num += 4;

        return DIR_ARR[(value - 1 + num) % 4];
    }

    public String toString() {
        return name() + " : " +getSymbol();
    }
}
```
```
EAST=1
SOUTH=2
WEST=3
NORTH=4
d1=EAST, 1
d2=EAST, 1
SOUTH : V
WEST : <
NORTH : ^
WEST : <
```
