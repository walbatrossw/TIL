본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.


## 1. 생성자란?
생성자라는 이름만 언듯 보면 무엇인가를 새로 생성하는 작업을 하는 역할을 하는 것처럼 보이지만 실제로는 인스턴스가 생성될 때 호출되는 **인스턴스의 초기화 메서드** 이다. 주로 인스턴스 변수의 초기화 작업에 주로 사용되며, 인스턴스 생성시 수행되어야할 작업을 위해서도 사용된다.

```java
ClassName(Type variableName) {
  // Initialization code
}
```

생성자를 작성하는데 아래와 같은 **2가지 조건** 을 반드시 만족해야한다.

- **생성자 이름은 클래스 이름과 같아야한다.**
- **반환 값이 없다.**

생성자가 가지는 **특징** 은 다음과 같다.

- **생성자도 오버로딩이 가능하기 때문에 하나의 클래스에 여러 개의 생성자가 존재할 수 있다.**
- **연산자 `new`가 인스턴스를 생성하지 생성자가 인스턴스를 생성하는 것이 아니기 때문에 용어 때문에 혼동해서는 안된다.**

## 2. 기본 생성자(default constructor)

만약 클래스에 생성자가 따로 정의되어 있지 않다면, 컴파일러는 자동적으로 생성자를 추가해준다. 컴파일러는 **매개변수도 없고, 아무런 내용이 없는 아주 기본적인 생성자를 추가해주는데 이를 기본 생성자(default constructor)** 라고 한다.

예제를 통해 기본 생성자에 대해 좀더 알아보자.

##### 기본생성자 예제 1 : 기본 생성자의 유무

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

`Data2`클래스의 인스턴스 생성시에 `Data2()`라는 기본생성자를 호출했지만 `Data2`클래스에 기본생성자가 정의되있지 않아 컴파일 오류가 발생한다.
컴파일러가 기본생성자를 대신 생성해줄 때에는 클래스에 생성자가 하나도 없을 때이다. `Data2`클래스는 이미 매개변수가 있는 생성자가 존재하기 때문에 컴파일러는 기본 생성자를 추가시키지 않는다.

## 3. 매개변수가 있는 생성자

**생성자도 메서드 처럼 매개변수를 선언하여 호출시 값을 넘겨받아 인스턴스 초기화 작업에 사용할 수 있다.** 인스턴스마다 각기 다른 값으로 초기화 되어야할 경우가 많기 때문에 매개변수를 이용한 초기화는 매우 유용하다.

예제를 통해 매개변수가 있는 생성자가 어떻게 유용한지 구체적으로 알아보자.

##### 매개변수가 있는 생성자 예제 1 : 매개변수를 통한 초기화
```java
public class CarTest {
    public static void main(String[] args) {

        Car car1 = new Car();
        System.out.println("car1 :" + car1.color + "/" + car1.gearType + "/" + car1.door);

        // 인스턴스 초기화 작업
        car1.color = "black";
        car1.gearType = "auto";
        car1.door = 4;

        // 인스턴스 생성, 초기화 작업 동시 수행
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
car1 :null/null/0
car1 :black/auto/4
car2 :white/auto/5
```

위의 예제에는 2개의 생성자가 존재한다. 기본생성자의 경우 인스턴스를 생성한 뒤 인스턴스 변수들의 초기화 작업을 따로 해줘야하지만, 매개변수가 있는 생성자의 경우 인스턴스를 생성함과 동시에 인스턴스 변수들의 초기화 작업까지 수행함으로써 코드를 보다 간결하고, 직관적으로 만들 수 있는 장점이 있다. 클래스를 작성할 때 다양한 생성자를 제공함으로써 인스턴스 생성후 별도로 초기화 작업을 다시하지않도록 하는 것이 바람직하다.


## 4. 생성자에서 다른 생성자 호출하기 - `this()`, `this`

같은 클래스의 멤버들 간에 서로 호출할 수 있는 것처럼 생성자 간에서도 서로 호출이 가능한데 아래와 같은 조건을 만족해야한다.

- **생성자의 이름으로 클래스 이름 대신 `this`를 사용한다.**
- **한 생성자에서 다른 생성자를 호출할 때는 반드시 첫 줄에서만 호출이 가능하다.**

아래의 코드를 보면 생성자의 호출이 잘못된 예이다.

```java
Car(String color) {
  // 이 라인에서 다른 생성자를 호출 해야함
  door = 5;
  Car(color, "auto", 4); // this(color, "auto", 4); 로 호출 해야함
}
```

생성자에서 다른 생성자를 호출할 때, 첫번째 라인에 작성해야만 하는 이유는 생성자 내에서 초기화 작업 도중에 다른 생성자를 호출하면 호출된 다른 생성자 내에서도 멤버변수의 값을 초기화 할 것이다. 그렇게 되면 **이전에 수행한 멤버 변수의 초기화 작업이 무의미** 해지기 때문이다.

예제를 통해 생성자에서 다른 생성자를 호출해보자.

##### 생성자 호출 예제 1

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
    this("white", "auto", 4); // 생성자 호출시에는 반드시 생성자의 첫번째 라인에 작성
  }

  Car2(String color) {
    this(color, "auto", 4); // 생성자 호출시에는 반드시 생성자의 첫번째 라인에 작성
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
위의 예제에 `this`와 `this()`가 등장하는데 차이점을 구분해 정리해보면 아래와 같다.
- **`this` : 인스턴스 자신을 가리키는 참조변수, 인스턴스의 주소가 저장되어 있다.**
- **`this()`, `this(매개변수)` : 생성자, 같은 클래스의 다른 생성자를 호출할 때 사용한다.**

## 5. 생성자를 이용한 인스턴스의 복사
현재 사용하고 있는 인스턴스와 같은 상태를 갖는 인스턴스를 하나 더 만들고자 할 때 생성자를 이용할 수 있다. 두 인스턴스가 같은 상태를 갖는다는 것은 인스턴스 변수가 동일한 값을 가지고 있다는 것을 의미한다. 하나의 클래스로부터 생성된 인스턴스의 메서드, 클래스 변수는 서로 동일하기 때문에 인스턴스의 차이는 인스턴스마다 각기 다른 값을 가질 수 있는 것은 인스턴스 변수 뿐이다.

```java
Car(Car car2) {
  color = car2.color;
  gearType = car2.gearType;
  door = car2.door;
}
```

위의 코드는 `Car`클래스의 참조변수를 매개변수로 한 생성자이다. 매개변수로 넘거진 참조변수가 가리키는 `Car`인스턴스의 인스턴스 변수인 `color`, `gearType`, `door`의 값을 인스턴스 자신으로 복사한다.

##### 생성자를 이용한 인스턴스 복사 예제 1
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
