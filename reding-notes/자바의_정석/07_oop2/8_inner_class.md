# 내부 클래스(inner class)

## 8.1 내부 클래스?
내부클래스는 클래스 내에 선언된 클래스다. 클래스에 다른 클래스를 선언하는 이유는 두 클래스가 서로 긴밀한 관계에 있기 때문이다.

**내부 클래스의 장점**
- 내부 클래스에서 외부 클래스의 멤버들을 쉽게 접근할 수 있다.
- 코드의 복잡성을 줄일 수 있다.

```java
class A {
  // ...
}

class B {
  // ...
}
```

```java
// 외부클래스(outer class)
class A {

  // ...

  // 내부클래스(inner class)
  class B {
    // ...
  }
}
```

---

## 8.2 내부 클래스의 종류와 특징

내부 클래스의 종류는 변수의 선언위치에 따른 종류와 같다. 내부 클래스는 마치 변수를 선언한 것과 같은 위치에 선언할 수 있으며, 변수의 선언위치에 따라 인스턴수 변수, 클래스 변수, 지역변수로 구분되는 것과 같이 내부 클래스도 선언위치에 따라 아래의 표와 같이 구분된다.
|내부클래스|특징|
|---|---|
|instance class|외부 클래스의 멤버변수 선언위치에 선언, 외부 클래스의 인스턴스 멤버처럼 다루어짐, 주로 외부 클래스의 인스턴스멤버들과 관련되 작업에 사용될 목적으로 쓰임   |
|static class|외부 클래스의 멤버변수 선언위치에 선언, 외부 클래스의 `static`멤버처럼 다루어짐, 주로 외부 클래스의 `static`멤버, `static`메서드에 사용될 목적으로 쓰임|
|local class|외부 클래스의 메서드나 초기화블럭 안에서 선언, 선언된 내부에서만 사용|
|anonymous class|클래스의 선언과 객체의 생성을 동시에 하는 이름없는 클래스(일회용)|

---

## 8.3 내부 클래스의 선언
```java
class Outer {
  int iv = 0; // 인스턴스 변수
  static int cv = 0; // 클래스 변수
  void myMethod() {
    int lv = 0; // 지역변수
  }
}
```
```java
class Outer {
  class InstanceInnerClass {}
  static class StaticInnerClass {}
  void myMethod() {
    class LocalInnerClass {}
  }
}
```
위의 코드 중 2번째를 보면 외부클래스에 서로 다른 3개의 클래스가 선언되있다. 1번째 코드와 2번째 코드를 비교해보면 내부 클래스의 선언 위치가 변수의 선언위치가 동일하다는 것을 알 수 있다.변수가 선언된 위치에 따라 변수의 종류가 나뉘듯이 내부 클래스도 선언된 위치에 따라 종류가 나뉜다. 각 내부 클래스의 선언 위치에 따라 변수와 동일한 유효범위와 접근성을 갖게된다.

---

## 8.4 내부 클래스의 제어자와 접근성

내부 클래스도 클래스이기 때문에 `abstract`나 `final`과 같은 제어자를 사용할 수 있을 뿐만아니라, 멤버변수들처럼 `private`, `protected`과 접근제어자도 사용이 가능하다.

**내부 클래스 예제 1**
```java
public class InnerClassEx1 {

  public static void main(String[] args) {
    System.out.println(InstanceInner.CONST);
    System.out.println(StaticInner.cv);
  }

  class InstanceInner {
    int iv = 100;
    //static int cv = 100; // static 변수를 선언 불가
    final static int CONST = 100;   // 상수이기에 선언 가능
  }

  static class StaticInner {
    int iv = 100;
    static int cv = 200; // static클래스만 static변수 선언이 가능
  }

  void myMethod() {
    class LocalInner {
      int iv = 300;
      // static int cv = 300; // static 변수선언 불가
      final static int CONST = 300; // 상수이기에 가능
    }
  }

}
```
```
100
200
```

**내부 클래스 예제 2**
```java
public class InnerClassEx2 {

  class InstanceInner {

  }

  static class StaticInner {

  }

  // 인스턴스 멤버 간에는 서로 직접 접근가능
  InstanceInner iv = new InstanceInner();

  // static 멤버 간에는 서로 직접 접근가능
  static StaticInner cv = new StaticInner();

  static void staticMethod() {

    // static 멤버는 인스턴스멤버에 직접 접근불가
    //InstanceInner obj1 = new InstanceInner();
    StaticInner obj2 = new StaticInner();

    // 굳이 접근하려한다면 객체를 먼저 생성해야 함,
    // 인스턴스클래스는 외부클래스를 먼저 생성해야만 생성이 가능
    InnerClassEx2 outer = new InnerClassEx2();
    InstanceInner obj = outer.new InstanceInner();

  }

  void instanceMethod() {

    // 인스턴스메서드에서는 인스턴스 멤버와 static 멤버 모두 접근가능
    InstanceInner obj1 = new InstanceInner();
    StaticInner obj2 = new StaticInner();

    // 메서드 내에 지역적으로 선언된 내부 클래스는 외부에서 접근불가
    //LocalInner lv = new LocalInner();

  }

  void myMethod() {
    class LocalInner {

    }
    LocalInner lv = new LocalInner();
  }
}
```

**내부 클래스 예제 3**
```java
public class InnerClassEx3 {

  private int outerIv = 0;
  static int outerCv = 0;

  class InstanceInner {
    int iiv = outerIv;  // 외부 클래스의 private 멤버도 접근가능
    int iiv2 = outerCv;
  }

  static class StaticInner {
    // static 클래스는 외부 클래스의 인스턴스 멤버에 접근불가
    //int siv = outerIv;
    static int scv = outerCv;
  }


  void myMethod() {
    int lv = 0;
    final int LV = 0;   // JDK 1.8 부터 final 생략가능

    class LocalInner {
      int liv = outerIv;
      int liv2 = outerCv;
      int liv3 = lv;  // JDK 1.8 부터는 에러가 아님
      int liv4 = LV;
    }
  }

}
```

**내부 클래스 예제 4**
```java
public class InnerClassEx4 {

  public static void main(String[] args) {
    Outer outer = new Outer();
    Outer.InstanceInner instanceInner = outer.new InstanceInner();
    System.out.println("instanceInner.iv : " + instanceInner.iv);
    System.out.println("Outer.StaticInner.cv : " + Outer.StaticInner.cv);

    Outer.StaticInner staticInner = new Outer.StaticInner();
    System.out.println("staticInner.iv : " + staticInner.iv);
  }

}

class Outer {

  class InstanceInner {
    int iv = 100;
  }

  static class StaticInner {
    int iv = 200;
    static int cv = 300;
  }

  void myMethod() {
    class LocalInner {
      int iv = 400;
    }
  }

}

```
```
instanceInner.iv : 100
Outer.StaticInner.cv : 300
staticInner.iv : 200
```

**내부 클래스 예제 5**
```java
public class InnerClassEx5 {

  public static void main(String[] args) {
    Outer2 outer2 = new Outer2();
    Outer2.Inner inner = outer2.new Inner();
    inner.method1();
  }

}

class Outer2 {

  int value = 10;

  class Inner {
    int value = 20;
    void method1() {
      int value = 30;
      System.out.println("value : " + value);
      System.out.println("this.value : " + this.value);
      System.out.println("Outer2.this.value : " + Outer2.this.value);
    }
  }

}
```
```
value : 30
this.value : 20
Outer2.this.value : 10
```

---

## 8.5 익명 클래스(anonymous class)

익명 클래스는 다른 내부 클래스와 달리 **이름이 없다.** 클래스의 선언과 객체의 생성을 동시에 하기 때문에 단 한번만 사용할 수 있고 **오직 하나의 객체만 생성할 수 있는 일회용 클래스** 이다. 이름이 없기 때문에 **생성자도 가질 수 없고**, 조상클래스의 이름이나 구현하고자하는 인터페이스의 이름을 사용하여 정의하기 때문에 하나의 클래스를 상속받는 동시에 인터페이스를 구현하거나 둘 이상의 인터페이스를 구현할 수 없다. 오로지 **단 하나의 클래스를 상속받거나 단 하나의 인터페이스를 구현** 해야만 한다.

```java
new SuperClassName() {
  // 멤버 선언
};

new InterfaceName() {
  // 멤버 선언
};
```

**익명 클래스 예제 1**

```java
public class InnerClassEx6 {

  Object iv = new Object() { void method() {} };  // 익명 클래스

  static Object cv = new Object() { void method() {} };   // 익명 클래스

  void myMethod() {
    Object lv = new Object() { void method() {} };  // 익명 클래스
  }
}
```

**익명 클래스 예제 2, 3**

```java
public class InnerClassEx7 {
  public static void main(String[] args) {
    Button button = new Button("Start");
    button.addActionListener(new EventHandler());
  }
}

class EventHandler implements ActionListener {
  public void actionPerformed(ActionEvent e) {
    System.out.println("ActionEvent occurred!");
  }
}
```

```java
public class InnerClassEx8 {
  public static void main(String[] args) {
    Button button = new Button("start");
    button.addActionListener(new ActionListener() {
      public void actionPerformed(ActionEvent e) {
        System.out.println("ActionEvent occurred!");
      }
    });
  }
}
```
