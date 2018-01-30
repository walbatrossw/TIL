# 인터페이스(interface)

## 7.1 인터페이스(interface)

인터페이스는 **일종의 추상클래스** 로서 추상클래스처럼 추상메서드를 갖지만 추상클래스보다 **추상화 정도가 높아서 추상클래스와 달리 일반메서드, 멤버변수를 구성원으로 가질 수 없다.**

- 추상클래스 : 부분적으로 완성된 "미완성 설계도"
- 인터페이스 : 구현된 것이 하나도 없이 밑그림만 그려진 "기본 설계도"

---

## 7.2 인터페이스의 작성

**인터페이스 작성법**
```java
interface InterfaceName {

  public static final Type CONSTANT_NAME = value;
  public abstract methodName();

}
```

**인터페이스 멤버들의 제약사항**
- 모든 멤버변수는 `public static final`이어야 하며, 이를 생략할 수 있음.
- 모든 메서드는 `public abstract`이어야 하며, 이를 생략할 수 있음.
- `static`메서드, `default`메서드는 예외(JDK 1.8부터 변경)

---

## 7.3 인터페이스의 상속
인터페이스는 **인터페이스로부터만 상속** 이 가능하며, 클래스와 달리 **다중상속이 가능** 하다.
```java
interface Movable {
  void move(int x, int y);
}

interface Attackable {
  void attack(Unit u);
}

// 인터페이스만 상속가능, 다중상속
interface Fightable extends Movable, Attackable {}
```



---

## 7.4 인터페이스의 구현

**인터페이스의 구현법**
```java
class ClassName implements InterfaceName {
  // 인터페이스에 정의된 추상메서드 구현
}
```
- 인터페이스도 추상클래스처럼 그 자체로 인스턴스 생성이 불가능하다.
- 만약 구현하는 인터페이스의 메서드 중 일부만 구현한다면 `abstract`를 붙여 추상클래스를 선언해줘야한다.
- 상속과 구현을 동시에 할 수도 있다.

**인터페이스 구현 예제 1**
```java
public class FighterTest {
  public static void main(String[] args) {
    Fighter fighter = new Fighter();
    if (fighter instanceof Unit)
      System.out.println("fighter는 Unit클래스의 자손입니다.");
    if (fighter instanceof Fightable)
      System.out.println("fighter는 Fighter인터페이스를 구현했습니다.");
    if (fighter instanceof Moveable)
      System.out.println("fighter는 Moveable인터페이스를 구현했습니다.");
    if (fighter instanceof Attackable)
      System.out.println("fighter는 Attackable인터페이스를 구현했습니다.");
    if (fighter instanceof Object)
      System.out.println("fighter는 Object클래스의 자손입니다.");
  }
}

class Fighter extends Unit implements Fightable {
  public void move(int x, int y) {

  }

  public void attack(Unit unit) {

  }
}

class Unit {
  int currentHP;
  int x;
  int y;
}

interface Fightable extends Moveable, Attackable {

}

interface Moveable {
  void move(int x, int y);
}

interface Attackable {
  void attack(Unit unit);
}
```
```
fighter는 Unit클래스의 자손입니다.
fighter는 Fighter인터페이스를 구현했습니다.
fighter는 Moveable인터페이스를 구현했습니다.
fighter는 Attackable인터페이스를 구현했습니다.
fighter는 Object클래스의 자손입니다.
```

---

## 7.5 인터페이스를 이용한 다중상속

---

## 7.6 인터페이스를 이용한 다형성

**인터페이스 타입의 참조변수**

자손클래스의 인스턴스를 조상타입의 참조변수로 참조하는 것이 가능하다는 것을 다형성에서 배웠었다. 인터페이스도 **인터페이스 타입의 참조변수로 이를 구현한 클래스의 인스턴스를 참조할 수 있고, 인터페이스 타입으로 형변환도 가능하다.**
```java
Fightable fightable = (Fightable) new Fighter();
Fightable fightable = new Fighter();
```

**인터페이스 타입의 매개변수**

메서드가 인터페이스 타입의 매개변수를 갖는다는 것은 메서드 호출 시 **해당 인터페이스를 구현한 클래스의 인스턴스를 매개변수** 로 제공해야한다는 것이다.
```java
void attack(Fightable fightable) {
  // ...
}
```

**인터페이스 리턴타입**

메서드의 리턴타입을 인터페이스로 지정하는 것도 가능하다. 리턴타입이 인터페이스라는 것은 해당 **인터페이스를 구현한 클래스의 인스턴스를 반환해야한다는 것** 을 의미한다.
```java
Fightable method() {
  //...
  Figther fighter = new Fighter();
  return fighter;
}
```

**인터페이스 다형성 예제 1**
```java
public class ParserTest {
    public static void main(String[] args) {
        Parseable parseable = ParserManager.getParser("XML");
        parseable.parse("document.xml");
        parseable = ParserManager.getParser("HTML");
        parseable.parse("document2.html");
    }
}

interface Parseable {
    public abstract void parse(String fileName);
}

class ParserManager {
    public static Parseable getParser(String type) {
        if (type.equals("XML")) {
            return new XMLParser();
        } else {
            return new HTMLParser();
        }
    }
}

class XMLParser implements Parseable {

    public void parse(String fileName) {
        System.out.println(fileName + " - XML parsing completed");
    }
}

class HTMLParser implements Parseable {

    public void parse(String fileName) {
        System.out.println(fileName + " - HTML parsing completed");
    }
}
```
```
document.xml - XML parsing completed
document2.html - HTML parsing completed
```

---

## 7.7 인터페이스의 장점
### 1) 인터페이스의 장점들
* 개발시간을 단축시킬 수 있다.
* 표준화가 가능하다.
* 서로 관계없는 클래스들에게 관계를 맺어줄 수 있다.
* 독립적인 프로그래밍이 가능하다.

### 2) 인터페이스의 장점 예제 1



---

## 7.8 인터페이스의 이해


---

## 7.9 디폴트 메서드와 `static`메서드


---
