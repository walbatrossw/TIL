# 다형성(polymorphism)

## 5.1 다형성?

### 1) 다형성?
객체지향개념에서 다형성이란 "여러가지 형태를 가질 수 있는 능력"을 의미하며, java에서는 한 타입의 참조변수로 여러 타입의 객체를 참조할 수 있도록 함으로써 다형성을 프로그램적으로 구현하였다. 이를 좀더 구체적으로 말하자면 **조상클래스 타입의 참조변수로 자손 클래스의 인스턴스를 참조할 수 있도록 함** 이다.

### 2) 다형성 예제 1
```java
class Tv {

  boolean power; // 전원
  int channerl;  // 채널

  void power() {
    power = !power;
  }

  void channelUp() {
    ++channel;
  }

  void channelDown() {
    --channel;
  }

}

class CaptionTv extends Tv {
  String text;
  void caption() {}
}
```

위의 예제를 보면 `Tv`클래스와 `CaptionTv`는 서로 상속관계에 있으며 이 두 클래스의 인스턴를 생성하고 사용하기 위해서는 아래와 같이 할 수 있다.

```java
Tv tv = new Tv();
CaptionTv = new CaptionTv();
```

지금까지는 생성된 인스턴스를 다루기 위해서는 인스턴스 타입과 일치하는 타입의 참조변수만을 사용했다. 위의 코드와 같이 `Tv`인스턴스를 다루기 위해서는 `Tv`타입의 참조변수를 `CaptionTv`인스턴스를 다루기 위해서는 `CaptionTv`타입의 참조변수를 사용했다. 이처럼 인스턴스의 타입과 참조변수의 타입이 일치하는 것이 보통이지만 `Tv`와 `CaptionTv` 클래스가 서로 상속관계 있을 때는 아래와 같이 조상클래스 타입의 참조변수로 자손 클래스타입의 인스턴스를 참조하도록 하는 것이 가능하다.

```java
Tv tv = new CaptionTv();
```

그렇다면 **인스턴스 타입과 같은 참조변수를 사용하는 것** 과 **인스턴스 타입과 다른 조상 클래스 타입의 참조변수를 사용하는 것** 이 어떻게 다른지 알아보자.

```java
CaptionTv ctv = new CaptionTv();
Tv tv = new CaptionTv();
```
`CaptionTv`타입의 인스턴스일지라도 `Tv`클래스 타입의 참조변수 `tv`로는 `CaptionTv`인스턴스의 멤버를 사용할 수 없다. 즉, 다시말해 `tv.text`, `tv.caption()`을 사용할 수가 없다. 둘다 같은 타입의 인스턴스이지만 참조변수의 타입에 따라 사용할 수 있는 멤버의 갯수가 달라진다.

반대로 아래와 같이 자손타입의 참조변수로 조상타입의 인스턴스를 참조하는 것은 가능할까?
```java
CaptionTv ctv = new Tv();
```
실제로 위의 코드를 컴파일하면 에러가 발생하는데 그 이유는 실제 인스턴스인 `Tv`의 멤버 개수보다 참조변수 `ctv`가 사용할 수 있는 멤버 개수가 많기 때문에 이를 허용하지 않는다. 참조변수 `ctv`가 참조하고 있는 `Tv`인스턴스에는 `text`와 `caption()`이 존재하지 않기 때문이다.

### 3) 다형성 간단정리
- 조상타입의 참조변수로 자손타입의 인스턴스를 참조할 수 있다.
- 자손타입의 참조변수로 조상타입의 인스턴스를 참조할 수 없다.

---

## 5.2 참조변수의 형변환

### 1) 형변환?
기본형 변수와 마찬가지로 참조 변수도 형변환이 가능하다. 단 조건은 서로 상속관계에 있는 클래스 사이에서만 가능하다.

- 자손타입 -> 조상타입(Up-casting) : 형변환 생략가능
- 조상타입 -> 자손타입(Down-casting) : 형변환 생략불가

상속관계에 있는 클래스 간의 형변환을 하는 이유는 참조하고 있는 인스턴스에서 사용할 수 있는 멤버의 범위를 조절하기 위해서이다. 그리고 형변환은 참조변수의 타입을 변환하는 것일뿐이지 인스턴스가 변환하는 것이 아니기 때문에 형변환은 인스턴스에 영향을 미치지 않는다.

### 2) 형변환 예제 1 : 조상타입의 참조변수로 자손타입의 인스턴스를 참조
```java
public class CastingTest1 {
  public static void main(String[] args) {
    Car car = null; // 조상타입 참조변수 선언
    FireEngine fe = new FireEngine();   // 자손클래스 타입의 참조변수 선언, 자손클래스의 인스턴스 생성
    FireEngine fe2 = null;  // 자손타입 참조변수 선언

    fe.water(); // 정상적으로 사용가능
    car = fe;   // 형변환 생략 : 자손 -> 조상
    // car.water(); // 컴파일에러, Car 타입의 참조변수로는 FireEngine 의 멤버를 사용할 수 없기때문

    fe2 = (FireEngine) car; // 형변환 생략불가 : 조상 -> 자손
    fe2.water();
  }
}

class Car {
  String color;
  int door;

  void drive() {
    System.out.println("drive");
  }

  void stop() {
    System.out.println("stop");
  }
}

class FireEngine extends Car {
  void water() {
    System.out.println("water");
  }
}
```
```
water
water
```

### 3) 형변환 예제 2 : 자손타입의 참조변수로 조상타입의 인스턴스를 참조는 불가
```java
public class CastingTest2 {
  public static void main(String[] args) {
    Car car = new Car();    // 조상타입의 참조변수 선언, 조상타입의 인스턴스 생성
    Car car2 = null;        // 조상타입의 참조변수 선언
    FireEngine2 fe2 = null; // 자손타입의 참조변수 선언

    car.drive();                // 정상적으로 사용
    fe2 = (FireEngine2) car;   // 에러, 조상타입의 인스턴스는 자손타입의 참조변수로 참조하는 것을 허용하지 않기때문
    fe2.drive();
    car2 = fe2;
    car2.drive();
  }
}

class Car2 {
  String color;
  int door;

  void drive() {
    System.out.println("drive");
  }
  void stop() {
    System.out.println("stop");
  }
}

class FireEngine2 extends Car2 {
  void water() {
    System.out.println("water");
  }
}
```

### 4) 형변환 간단정리
- 서로 상속관계에 있는 클래스 타입의 **참조변수간의 형변환** 은 양방향으로 자유롭게 수행가능
- 참조변수가 참조하고 있는 **인스턴스의 자손타입으로 형변환은 불가**

---

## 5.3 `instanceof` 연산자

### 1) `instanceof`?
참조변수가 참조하고 있는 인스턴스의 실제 타입을 알아보기 위해 사용한다. 주로 조건문에 사용되며, `instanceof`의 왼쪽에는 참조변수를 오른쪽에 타입이 피연산자로 위치한다.
연산의 결과는 `boolean`값이 반환된다. 즉, **`true`가 반환되었다는 것은 참조변수가 검사한 타입으로 형변환이 가능하다는 뜻** 이다.

```java
void doWork(Car car) {
  if (car instanceof FireEngine) {
    FireEngine fe = (FireEngine) car;
    fe.water();
    // ...
  } else if (car instanceof Ambulance)  {
    Ambulance a - (Ambulance) car;
    a.siren();
    // ...
  }
  // ...
}
```

위 코드는 `Car`클래스 타입의 참조변수 `car`를 매겨변수로 하는 메서드이다. 이 메서드가 호출될 때 매개변수로 `Car`클래스 또는 자손클래스 타입의 인스턴스를 넘겨받지만 메서드에서는 어떤 인스턴스인지 알길이 없다. 그래서 `instanceof`연산자를 이용해서 참조변수 `c`가 가리키고 있는 인스턴스의 타입을 체크하고, 적절한 형변환을 한 다음 작업을 수행해야한다.

### 2) `instanceof` 예제 1
```java
public class InstanceofTest {
  public static void main(String[] args) {
    FireEngine3 fe = new FireEngine3();

    if (fe instanceof FireEngine3)
      System.out.println("this is a FireEngine3's instance");

    if (fe instanceof Car3)
      System.out.println("this is a Car3's instance");

    if (fe instanceof Object)
      System.out.println("this is a Object's instance");

    System.out.println(fe.getClass().getName());
  }
}

class Car3{

}

class FireEngine3 extends Car3 {

}
```
```
this is a FireEngine3's instance
this is a Car3's instance
this is a Object's instance
com.doubles.standardofjava.ch07_oop2.part05_polymorphism.FireEngine3
```

---

## 5.4 참조변수와 인스턴스의 연결

---

## 5.5 매개변수의 다형성

---

## 5.6 여러 종류의 객체를 배열로 다루기
