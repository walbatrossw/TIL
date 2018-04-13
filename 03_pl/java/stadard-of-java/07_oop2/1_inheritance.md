본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. 상속의 정의와 장점

상속이란 **기존의 클래스를 재사용하여 새로운 클래스를 작성하는 것** 으로 상속을 통해 클래스를 작성하면 보다 적은 양의 코드로 새로운 클래스를 작성할 수 있고, 코드를 공통적으로 관리할 수 있기 때문에 코드의 추가와 변경이 매우 용이하다.

```java
// 상속을 구현하는 방법
class Child extends Parent {
  // ...
}

class Parent {
  // ...
}
```

위의 두 클래스는 서로 상속관에 있다고하며, 상속해주는 클래스를 조상 클래스라 하고, 상속받는 클래스를 자손 클래스라고 한다.

- 조상 클래스 : 부모(parent) 클래스, 상위(super) 클래스, 기반(base) 클래스
- 자손 클래스 : 자식(child) 클래스, 하위(sub) 클래스, 파생(derived) 클래스

상속의 특성의 특성은 다음과 같다.

- **조상 클래스의 변경이 자손 클래스에 영향을 미치지만, 자손 클래스의 변경이 조상 클래스에 영향을 미치지 않는다.**
- **생성자와 초기화 블럭은 상속되지 않으며 멤버만 상속된다.**
- **자손 클래스의 멤버 갯수는 조상 클래스보다 많거나 같다.**

##### 상속 예제 1 : `Tv`클래스, `Tv`클래스를 상속 받은 `CaptionTv`클래스

```java
class Tv {
  boolean power;
  int channel;

  void power() {
    power = !power;
  }

  void channelUp() {
    ++ channel;
  }

  void channelDown() {
    -- channel;
  }
}
```

```java
class CaptionTv extends Tv {
  boolean caption;

  void displayCaption(String text) {
    if (caption)
      System.out.println(text);
  }
}
```

```java
public class CaptionTvTest {
  public static void main(String[] args) {
    CaptionTv captionTv = new CaptionTv();
    captionTv.channel = 10;
    captionTv.channelUp();
    System.out.println(captionTv.channel);
    captionTv.displayCaption("Hello World!");
    captionTv.caption = true;
    captionTv.displayCaption("Hello World!");
  }
}
```

```
11
Hello World!
```

자손 클래스인 `CaptionTv`의 인스턴스를 생성하면 조상 클래스인 `Tv`의 멤버, 자손 클래스의 멤버가 합쳐진 하나의 인스턴스가 생성된다.

## 2. 클래스간의 관계(포함관계)

상속 외에도 클래스를 재사용할 수 있는 방법은 포함관계를 맺어주는 것이다. 포함관계를 맺어주는 것은 클래스의 멤버변수로 다른 클래스 타입의 참조변수를 선언하는 것을 의미한다.
```java
class Circle {
  int x; // 원점의 x좌표
  int y; // 원점의 y좌표
  int r; // 원의 반지름
}
```
```java
class Circle {
  Point c = new Point(); // 원점의 x,y좌표를 가진 Point클래스의 참조변수를 선언, 인스턴스를 생성
  int r; // 원의 반지름
}
```
```java
class Point {
  int x; // 원점의 x좌표
  int y; // 원점의 y좌표
}
```


## 3. 클래스간의 관계 결정하기

클래스간의 관계를 상속관계로 맺을 것인지 아니면 포함관계로 맺을 것인지에 대한 명확한 구분을 하는 기준은 다음과 같다.
- 상속관계 : A는 B이다. 예) 노트북은 컴퓨터이다.
- 포함관계 : A는 B를 가지고 있다. 예) 컴퓨터는 CPU를 가지고 있다.

###### 클래스간의 포함관계 예제 : 카드뽑기
```java
public class DeckTest {
    public static void main(String[] args) {
        Deck deck = new Deck();
        Card card = deck.pick(0);
        System.out.println(card);

        deck.shuffle();
        card = deck.pick(0);
        System.out.println(card);
    }
}

class Deck {
    final int CARD_NUM = 52; // 전체 카드 수
    Card cardArr[] = new Card[CARD_NUM]; // 카드 갯수 만큼 배열 생성

    Deck() {
        int i = 0;
        for (int k = Card.KIND_MAX; k > 0; k--)
            for (int n = 0; n < Card.NUM_MAX; n++)
                cardArr[i++] = new Card(k, n+1);

    }

    Card pick(int index) {
        return cardArr[index];
    }

    Card pick() {
        int index = (int) (Math.random() * CARD_NUM);
        return pick(index);
    }

    void shuffle() {
        for (int i = 0; i < cardArr.length; i++) {
            int r = (int) (Math.random() * CARD_NUM);
            Card temp = cardArr[i];
            cardArr[i] = cardArr[r];
            cardArr[r] = temp;
        }
    }
}

class Card {
    static final int KIND_MAX = 4; // 카드 무늬수
    static final int NUM_MAX = 13; // 무늬별 카드 수

    static final int SPADE = 4;
    static final int DIAMOND = 3;
    static final int HEART = 2;
    static final int CLOVER = 1;

    int kind;
    int number;

    Card() {
        this(SPADE, 1);
    }

    Card(int kind, int number) {
        this.kind = kind;
        this.number = number;
    }

    public String toString() {
        String[] kinds = {"", "CLOVER", "HEART", "DIAMOND", "SPADE"};
        String numbers = "0123456789XJQK";

        return "kind : " + kinds[this.kind] + ", number : " + numbers.charAt(this.number);
    }
}
```
`Deck`클래스를 작성하는데 `Card`클래스를 재사용하여 포함관계로 작성하였고, 카드 한벌이 52장으로 구성되있으므로 `Card`클래스를 길이가 52인 배열로 처리하였다.

## 4. 단일 상속

**자바의 경우 다중상속을 허용하지 않는다.** 다중상속을 허용함으로써 상속받은 여러 클래스의 멤버간의 이름이 같을 경우 구별할 방법이 없기 때문에 다중상속을 허용하지 않는다. 물론 `static`메서드라면 이름 앞에 클래스 이름을 붙여 구별할 수 있지만 인스턴스 메서드의 경우 선언부가 같은 두 메서드를 구분할 수 있는 있는 방법이 없기 때문이기도 하다.

## 5. Object 클래스 - 모든 클래스의 조상
모든 클래스의 상속 계층도에서 가장 상단에 위치하는 것은 `Object`클래스이다. 그렇기 때문에 모든 클래스는 `Object`클래스에 정의된 멤버들을 사용할 수 있다. `toString()`이나 `equals()`와 같이 모든 인스턴스가 가져야할 기본적인 메서드들이 정의 되어있다.
