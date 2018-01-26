# 2. 오버라이딩(Overriding)

## 2.1 오버라이딩?

### 1) 오버라이딩의 정의
조상 클래스로부터 상속받은 메서드의 내용을 변경하는 것을 말한다. 조상클래스로부터 상속받은 메서드를 그대로 사용하지 않고 자신의 클래스에 맞게 변경하여 사용하기 위해서 오버라이딩을 사용한다. 새로운 메서드를 작성하는 것보다 기존의 메서드를 변경하여 사용하는 것이 좀더 효율적이기 때문이다.

### 2) 오버라이딩의 예
```java
class Ponit {
  int x;
  int y;

  String getLocation() {
    return "x :" + x + ", y :" + y;
  };
}

class Point3D extends Point {
  int z;

  String getLocation() { // 메서드 오버라이딩
    return "x :" + x + ", y :" + y + ", z :" + z;
  };
}
```

## 2.2 오버라이딩의 조건

### 1) 조건 : 선언부가 같아야 한다.
- 이름이 같아야 한다.
- 매개변수가 같아야 한다.
- 반환타입이 같아야 한다.

### 2) 제약사항
- 접근 제어자는 조상클래스보다 좁은 범위로 변경할 수 없다.
- 조상 클래스의 메서드보다 많은 수의 예외를 선언할 수 없다.
```java
class Parent {
  void parentMethod() throws IOException, SQLException {
    //...
  };
}

class Child extends Parent {
  void parentMethod() throws Exception { // 조상 클래스보다 예외의 갯수가 작지만 예외 클래스의 최상위 클래스이기 허용되지 않는다.
    //...
  };
}
```

## 2.3 오버로딩 VS. 오버로딩
- 오버로딩(overloading) : 기존에 없는 새로운 메서드를 정의하는 것(new)
- 오버라이딩(Overriding) : 상속받은 메서드의 내용을 변경하는 것(change, modify)
```java
class Parent {
  void parentMethod() {};
}

class Child extends Parent {
  void parentMethod() {};     // 오버라이딩
  void parentMethod(int i) {}; // 오버로딩
  void childMethod() {};
  void childMethod(int i) {}; // 오버로딩
}
```

## 2.4 `super`

### 1) `super`?
`super`는 자손클래스에서 조상클래스로부터 상속받은 멤버를 참조하는데 사용되는 참조변수이다. 상속받은 멤버와 자신의 클래스의 멤버의 이름이 같을 때 `super`를 사용해서 구별할 수 있다.

### 2) `super` 예제 1
```java
public class SuperTest {
  public static void main(String[] args) {
    Child child = new Child();
    child.method();
  }
}

class Parent {
  int x = 10;
}

class Child extends Parent {
  void method() {
    System.out.println("x = " + x);
    System.out.println("this.x = " + this.x);
    System.out.println("super.x = " + super.x);
  }
}
```
```
x = 10
this.x = 10
super.x = 10
```
- `x`, `this.x`, `super.x` 모두가 같은 변수를 의미하기 때문에 모두 같은 값이 출력
### 2) `super` 예제 2
```java
public class SuperTest2 {
  public static void main(String[] args) {
    Child2 child2 = new Child2();
    child2.method();
  }

}

class Parent2 {
  int x = 10;
}

class Child2 extends Parent2 {
  int x = 20;

  void method() {
    System.out.println("x = " + x);
    System.out.println("this.x = " + this.x);
    System.out.println("super.x = " + super.x);
  }
}
```
```
x = 20
this.x = 20
super.x = 10
```
- 이전 예제와 달리 조상클래스와 자손클래스에 동일한 이름의 멤버가 존재하기 때문에 `super.x`, `this.x`는 다른 값을 참조하게 된다.
- 출력결과는 당연히 다르게 나온다.

## 2.5 `super()` - 조상 클래스의 생성자

### 1) `super()`?
`this()`와 마찬가지로 `super()`역시 생성자이다. `this()`는 같은 클래스의 다른 생성자를 호출할 때 사용하지만 `super()`는 조상클래스의 생성자를 호출할 때 사용한다.

### 2) `super()` 생성과정
- 자손클래스의 인스턴스를 생성하면 자손의 멤버, 조상의 멤버 모두가 합쳐진 하나의 인스턴스를 생성한다.
- 조상 클래스의 멤버 초기화 작업이 가장 먼저 수행되어야 하기 때문에 자손 클래스의 생성자의 가장 첫 줄에서 조상 클래스의 생성자가 호출된다.
- 생성자의 첫 줄에서 조상클래스의 생성자가 호출되는 이유는 자손 클래스의 멤버가 조상 클래스의 멤버를 사용할 수도 있기 때문에 조상 클래스의 멤버들이 가장 먼저 초기화 되어 있어야 하기 때문이다.
- 조상클래스의 생성자 호출은 클래스 상속관계에서 최상위에 위치한 `Object`클래스의 `Object()`까지 거슬러 올라가서야 끝이 난다.
- 만약 생성자를 따로 추가하지 않았다면 컴파일되는 시점에서 컴파일러가 자동적으로 추가시켜주게 된다.

### 3) `super()` 예제 1
```java
public class PointTest {
  public static void main(String[] args) {
    Point3D p3 = new Point3D(1, 2, 3);
  }
}

class Point {
  int x;
  int y;

  Point(int x, int y) {
    this.x = x;
    this.y = y;
  }

  String getLocation() {
    return "x : " + x + ", y : " + y;
  }
}

class Point3D extends Point {
  int z;

  Point3D(int x, int y, int z) {
    // super();  // 컴파일러가 자동으로 추가
    this.x = x;
    this.y = y;
    this.z = z;
  }

  String getLocation() {
    return "x : " + x + ", y : " + y + ", z : " + z;
  }
}
```
```
Error:(26, 34) java: constructor Point in class com.doubles.standardofjava.ch07_oop2.part02_overriding.Point cannot be applied to given types;
  required: int,int
  found: no arguments
  reason: actual and formal argument lists differ in length
```
- 위 코드를 실행시키면 컴파일러가 자동적으로 `Point3D()`의 첫 줄에 `super()`를 추가시키고 작업을 수행한다.
- 하지만 `super()`인 `Point()`가 `Point`클래스에 정의되있지 않기 때문에 컴파일 오류가 발생한다.
- 컴파일 오류를 수정하기 위해서는 `Point3D()`의 첫줄에 `super(x, y)`를 추가하거나, `Ponit`클래스에 `Point()`를 추가해주어야한다.

### 4) `super()` 예제 2
```java
public class PointTest2 {
  public static void main(String[] args) {
      // 인스턴스 생성
      Point3D2 p3 = new Point3D2();
      System.out.println("p3.x=" + p3.x);
      System.out.println("p3.y=" + p3.y);
      System.out.println("p3.z=" + p3.z);

  }
}

class Point2 {
  int x = 10;
  int y = 10;

  // 3. 호출
  Point2(int x, int y) {
      // 4. 호출
      // super(); ==> Object();
      this.x = x;
      this.y = y;
  }

  String getLocation() {
    return "x : " + x + ", y : " + y;
  }
}

class Point3D2 extends Point2 {
  int z = 30;
  // 1. 호출
  Point3D2() {
    this(100, 200, 300);
  }

  // 2. 호출
  Point3D2(int x, int y, int z) {
    super(x, y);
    this.z = z;
  }

  String getLocation() {
    return "x : " + x + ", y : " + y + ", z : " + z;
  }
}
```
- `Point3D2`클래스의 인스턴스를 생성하면 `Point3D2()`가 호출되고, `Point3D2()`는 `Point3D2(int x, int y, int z)`를 호출한다.
- `Point3D2(int x, int y, int z)`는 조상 클래스인 `Point2`의 `Point2(int x, int y)`를 호출하여 초기화 작업을 수행한다.
- 그리고 `Point2(int x, int y)`는 조상 클래스인 `Object`의 `Object()`를 호출하게 된다.
