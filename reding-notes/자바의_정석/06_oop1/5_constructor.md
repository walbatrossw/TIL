# 5. 생성자(Constructor)

## 5.1 생성자란?

### 1) 생성자?
인스턴스가 생성될 때 호출되는 인스턴스의 초기화 메서드이다. 주로 인스턴스 변수의 초기화 작업에 주로 사용되며, 인스턴스 생성시 수행되어야할 작업을 위해서도 사용된다.

### 2) 생성자의 조건
- 생성자 이름은 클래스 이름과 같아야한다.
- 반환 값이 없다.

### 3) 생성자의 특징
- 생성자도 오버로딩이 가능하기 때문에 하나의 클래스에 여러 개의 생성자가 존재할 수 있다.
- 연산자 `new`가 인스턴스를 생성하지 생성자가 인스턴스를 생성하는 것이 아니기 때문에 용어때문에 혼동해서는 안된다.

## 5.2 기본생성자(default constructor)

### 1) 기본생성자?
만약 클래스에 생성자가 따로 정의되어 있지 않다면 컴파일러는 자동적으로 생성자를 추가해준다. 컴파일러는 매개변수도 없고, 아무런 내용이 없는 아주 기본적인 생성자를 추가해주는데 이를 기본 생성자(default constructor)라고 한다.

### 2) 기본생성자 예제 1
```java
public class ConstructorTest {
    public static void main(String[] args) {
        Data1 data1 = new Data1();
        Data2 data2 = new Data2(); // 컴파일 오류 기본생성자가 없기 때문에
    }
}

class Data1 {
    int value;
}

class Data2 {
  int value;

  // Data2() {}; 컴파일 오류를 수정하기 위해서는 기본 생성자를 추가해줘야 함
  Data2(int x) {
    value = x;
  }
}
```
```
Error:(6, 23) java: constructor Data2 in class com.doubles.standardofjava.ch06_oop1.part06_constructor.Data2 cannot be applied to given types;
  required: int
  found: no arguments
  reason: actual and formal argument lists differ in length
```
- `Data2`클래스의 인스턴스 생성시에 `Data2()`라는 기본생성자를 호출했지만 `Data2`클래스에 기본생성자가 정의되있지 않아 컴파일 오류가 발생한다.
- 컴파일러가 기본생성자를 대신 생성해줄 때에는 클래스에 생성자가 하나도 없을 때이다.
- `Data2`클래스는 이미 매개변수가 있는 생성자가 존재하기 때문에 컴파일러는 기본 생성자를 추가시키지 않는다.

## 5.3 매개변수가 있는 생성자

### 1) 매개변수
생성자도 메서드 처럼 매개변수를 선언하여 호출시 값을 넘겨받아 인스턴스 초기화 작업에 사용할 수 있다.

### 2) 매개변수가 있는 생성자 예제 1
```java
public class CarTest {
  public static void main(String[] args) {
    Car car1 = new Car();
    car1.color = "black";
    car1.gearType = "auto";
    car1.door = 4;

    Car car2 = new Car("white", "auto", 5);
    System.out.println("car1 :" + car1.color + "/" + car1.gearType + "/" + car1.door);
    System.out.println("car2 :" + car2.color + "/" + car2.gearType + "/" + car2.door);
  }
}

class Car {
  String color;
  String gearType;
  int door;

  // 기본생성자
  Car() {}

  // 매개변수가 있는 생성자
  Car(String color, String gearType, int door) {
      this.color = color;
      this.gearType = gearType;
      this.door = door;
  }
}
```
```
car1 :black/auto/4
car2 :white/auto/5
```

## 5.4 생성자에서 다른 생성자 호출하기 - `this()`, `this`

### 1) 생성자 간의 호출의 조건
같은 클래스의 멤버들 간에 서로 호출할 수 있는 것처럼 생성자 간에서도 서로 호출이 가능한데 아래와 같은 조건을 만족해야한다.
- 생성자의 이름으로 클래스 이름 대신 `this`를 사용한다.
- 한 생성자에서 다른 생성자를 호출할 때는 반드시 첫 줄에서만 호출이 가능하다.
- 아래의 코드를 보면 생성자의 호출이 잘못된 예이다.
  ```java
  Car(String color) {
    door = 5;
    Car(color, "auto", 4);
  }
  ```
  - 첫 번째 줄에서 다른 생성자를 호출해야한다.
  - `Car(color, "auto", 4)`는 `this(color, "auto", 4)`로 해야한다.

### 2) 생성자 간의 호출 예제 1
```java
public class CarTest2 {
  public static void main(String[] args) {
    Car2 carOne = new Car2();
    Car2 carTwo = new Car2("blue");

    System.out.println("carOne :" + carOne.color + "/" + carOne.gearType + "/" + carOne.door);
    System.out.println("carTwo :" + carTwo.color + "/" + carTwo.gearType + "/" + carTwo.door);
  }
}

class Car2 {
  String color;
  String gearType;
  int door;

  Car2() {
    this("white", "auto", 4);
  }

  Car2(String color) {
    this(color, "auto", 4);
  }

  Car2(String color, String gearType, int door) {
    this.color = color;
    this.gearType = gearType;
    this.door = door;
  }
}
```
```
carOne :white/auto/4
carTwo :blue/auto/4
```

### 3) `this`와 `this()`구분
- `this()` : 인스턴스 자신을 가리키는 참조변수, 인스턴스의 주소가 저장되어 있다.
- `this()`, `this(매개변수)` : 생성자, 같은 클래스의 다른 생성자를 호출할 때 사용한다.

## 5.5 생성자를 이용한 인스턴스의 복사

### 1) 생성자를 이용한 인스턴스 복사 예제 1
```java
public class CarTest3 {
  public static void main(String[] args) {
    Car3 carOne = new Car3();
    Car3 carTwo = new Car3(carOne);

    System.out.println("carOne :" + carOne.color + "/" + carOne.gearType + "/" + carOne.door);
    System.out.println("carTwo :" + carTwo.color + "/" + carTwo.gearType + "/" + carTwo.door);

    carOne.door = 6;
    System.out.println("carOne.door = 6; 수행후");

    System.out.println("carOne :" + carOne.color + "/" + carOne.gearType + "/" + carOne.door);
    System.out.println("carTwo :" + carTwo.color + "/" + carTwo.gearType + "/" + carTwo.door);
  }
}

class Car3 {
  String color;
  String gearType;
  int door;

  Car3() {
    this("white", "auto", 4);
  }

  Car3(Car3 car3) {
    color = car3.color;
    gearType = car3.gearType;
    door = car3.door;
  }

  Car3(String color, String gearType, int door) {
    this.color = color;
    this.gearType = gearType;
    this.door = door;
  }
}
```
```
carOne :white/auto/4
carTwo :white/auto/4
carOne.door = 6; 수행후
carOne :white/auto/6
carTwo :white/auto/4
```
