본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. 다형성이란?

**객체지향개념에서 다형성이란 "여러가지 형태를 가질 수 있는 능력"을 의미** 하며, java에서는 한 타입의 참조변수로 여러 타입의 객체를 참조할 수 있도록 함으로써 다형성을 프로그램적으로 구현하였다. 이를 좀더 구체적으로 말하자면 **조상클래스 타입의 참조변수로 자손 클래스의 인스턴스를 참조할 수 있도록 함** 이다.

##### 다형성 예제 1
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

지금까지는 생성된 인스턴스를 다루기 위해서는 인스턴스 타입과 일치하는 타입의 참조변수만을 사용했다. 위의 코드와 같이 `Tv`인스턴스를 다루기 위해서는 `Tv`타입의 참조변수를 `CaptionTv`인스턴스를 다루기 위해서는 `CaptionTv`타입의 참조변수를 사용했다. 이처럼 인스턴스의 타입과 참조변수의 타입이 일치하는 것이 보통이지만 **`Tv`와 `CaptionTv` 클래스가 서로 상속관계 있을 때는 아래와 같이 조상클래스 타입의 참조변수로 자손 클래스타입의 인스턴스를 참조하도록 하는 것이 가능하다.**

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

위의 내용을 바탕으로 다형성을 정리하면 다음과 같다.
- **조상타입의 참조변수로 자손타입의 인스턴스를 참조할 수 있다.**
- **자손타입의 참조변수로 조상타입의 인스턴스를 참조할 수 없다.**

## 2. 참조변수의 형변환

기본형 변수와 마찬가지로 참조 변수도 형변환이 가능하다. 단 조건은 서로 상속관계에 있는 클래스 사이에서만 가능하다.

- **자손타입 -> 조상타입(Up-casting) : 형변환 생략가능**
- **조상타입 -> 자손타입(Down-casting) : 형변환 생략불가**

상속관계에 있는 클래스 간의 형변환을 하는 이유는 참조하고 있는 인스턴스에서 사용할 수 있는 멤버의 범위를 조절하기 위해서이다. 그리고 형변환은 참조변수의 타입을 변환하는 것일뿐이지 인스턴스가 변환하는 것이 아니기 때문에 형변환은 인스턴스에 영향을 미치지 않는다.

##### 형변환 예제 1 : 조상타입의 참조변수로 자손타입의 인스턴스를 참조
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

##### 형변환 예제 2 : 자손타입의 참조변수로 조상타입의 인스턴스를 참조는 불가
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

형변환을 정리해보자면 다음과 같다.
- **서로 상속관계에 있는 클래스 타입의 참조변수간의 형변환 은 양방향으로 자유롭게 수행가능**
- **참조변수가 참조하고 있는 인스턴스의 자손타입으로 형변환은 불가**

---

## 3. `instanceof` 연산자

참조변수가 참조하고 있는 인스턴스의 실제 타입을 알아보기 위해 사용한다. 주로 조건문에 사용되며, `instanceof`의 왼쪽에는 참조변수를 오른쪽에 타입이 피연산자로 위치한다. 연산의 결과는 `boolean`값이 반환된다. **즉, `true`가 반환되었다는 것은 참조변수가 검사한 타입으로 형변환이 가능하다는 뜻** 이다.

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

##### `instanceof` 예제 1
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

## 4. 참조변수와 인스턴스의 연결

멤버변수가 조상클래스와 자손클래스에 중복으로 정의되었을 경우는 어떻게 되는지 정리해보면 다음과 같다.
- **조상타입의 참조변수를 사용한 경우 : 조상클래스에 선언된 멤버변수 사용**
- **자손타입의 참조변수를 사용한 경우 : 자손클래스에 선언된 멤버변수 사용**
- **메서드의 경우**
  - **조상클래스의 메서드를 자손클래스에서 오버라이딩한 경우에도 참조변수의 타입과 관계없이 항상 실제 인스턴스의 메서드가 호출**

##### 예제 1 : 조상클래스와 자손클래스에 멤버변수가 중복정의 되었을 경우
```java
public class BindingTest {
    public static void main(String[] args) {
        Parent parent = new Child();
        Child child = new Child();

        // 조상타입의 참조변수를 사용
        System.out.println("parent.x = " + parent.x);
        parent.method();

        // 자손타입의 참보변수 사용
        System.out.println("child.x = " + child.x);
        child.method();
    }
}

class Parent {
    int x = 100;
    void method() {
        System.out.println("parent method");
    }
}

class Child extends Parent {
    int x = 200;
    void method() {
        System.out.println("child method");
    }
}
```
```
parent.x = 100
child method
child.x = 200
child method
```

참조변수 `parent`와 `child`는 타입이 다르지만 `Child`인스턴스를 참조하고 있고, `Parent`와 `Child`클래스는 같이 이름의 멤버변수를 가지고 있다. 참조변수를 어떤 타입으로 하는지에 따라 결과가 다르게 나온다는 것을 알 수 있다. 메서드의 경우 참조변수에 관계없이 항상 실제 인스턴스의 `Child`에 정의된 메서드가 호출된다.

##### 예제 2 : 중복정의가 되지 않은 경우
```java
public class BindingTest2 {
  public static void main(String[] args) {
    Parent2 parent2 = new Child2();
    Child2 child2 = new Child2();

    System.out.println("parent2.x = " + parent2.x);
    parent2.method();

    System.out.println("child2.x = " + child2.x);
    child2.method();
  }
}

class Parent2 {
  int x = 100;
  void method() {
    System.out.println("parent2 method");
  }
}

class Child2 extends Parent2 {

}
```
```
parent2.x = 100
parent2 method
child2.x = 100
parent2 method
```
이전 예제와 다르게 `Child2`클래스의 경우 멤버변수가 정의되어 있지 않고 단순히 조상클래스로부터 멤버를 상속받게 된다. 그래서 참조변수의 타입과 관계없이 조상클래스의 멤버들을 사용하게 된다.

##### 예제 3 : `super`, `this`를 통해 조상, 자손클래스의 멤버 구분
```java
public class BindingTest3 {
  public static void main(String[] args) {
    Parent3 parent3 = new Child3();
    Child3 child3 = new Child3();

    System.out.println("parent3.x = " + parent3.x);
    parent3.method();

    System.out.println();

    System.out.println("child3 = " + child3.x);
    child3.method();
  }
}

class Parent3 {
  int x = 100;
  void method() {
    System.out.println("parent3 method");
  }
}

class Child3 extends Parent3 {
  int x = 200;
  void method() {
    System.out.println("child3 method");
    System.out.println("x = " + x);
    System.out.println("super.x = " + super.x);
    System.out.println("this.x = " + this.x);
  }
}
```
```
parent3.x = 100
child3 method
x = 200
super.x = 100
this.x = 200

child3 = 200
child3 method
x = 200
super.x = 100
this.x = 200
```

자손클래스 `Child3`에 선언된 인스턴스변수 `x`와 조상클래스 `Parent3`로부터 상속받은 인스턴스변수 `x`를 구분하는데 `this`와 `super`가 사용된다.


## 5. 매개변수의 다형성

##### 매개변수의 다형성 예제 1

```java
public class PolyArgumentTest {
  public static void main(String[] args) {
    Buyer buyer = new Buyer();
    buyer.buy(new Tv());
    buyer.buy(new Computer());

    System.out.println("현재 잔액은 " + buyer.money + "만원 입니다.");
    System.out.println("현재 보너스 포인트는 " + buyer.bonusPoint + "점 입니다.");
  }
}

class Product {
  int price;
  int bonusPoint;

  Product(int price) {
    this.price = price;
    bonusPoint = (int) (price/10.0);
  }
}

class Tv extends Product {
  Tv() {
    super(100);
  }

  public String toString() {
    return "Tv";
  }
}

class Computer extends Product {
  Computer() {
    super(200);
  }

  public String toString() {
    return "Computer";
  }
}

class Buyer {
  int money = 1000;
  int bonusPoint = 0;

  // 상품이 새로 추가될 때마다 중복되는 로직의 메서드를 작성하지 않기 위해
  // 매개변수의 다형성을 적용, Product을 상속받은 자손클래스의 경우 매개변수로 사용가능
  void buy(Product product) {
    if (money < product.price) {
        System.out.println("잔액이 부족합니다.");
        return;
    }

    money -= product.price; // 구매한 상품의 가격만큼 보유금액에서 차감
    bonusPoint += product.bonusPoint; // 구매한 상품의 가격의 10%를 보너스포인트로 적립
    System.out.println(product + "을/를 구입하셨습니다.");
  }
}
```
```
Tv을/를 구입하셨습니다.
Computer을/를 구입하셨습니다.
현재 잔액은 700만원 입니다.
현재 보너스 포인트는 30점 입니다.
```

## 6. 여러 종류의 객체를 배열로 다루기

##### 여러 종류의 객체를 배열로 다루기 예제 1 : 배열사용
```java
public class PolyArgumentTest2 {
    public static void main(String[] args) {
        Buyer2 buyer2 = new Buyer2();
        buyer2.buy(new Computer2());
        buyer2.buy(new Tv2());
        buyer2.buy(new Audio2());
        buyer2.summary();
    }
}

class Product2 {
    int price;
    int bonusPoint;

    Product2(int price) {
        this.price = price;
        bonusPoint = (int) (price/10.0);
    }
}

class Tv2 extends Product2 {

    Tv2() {
        super(100);
    }

    public String toString() {
        return "Tv";
    }
}

class Computer2 extends Product2 {

    Computer2() {
        super(200);
    }

    public String toString() {
        return "Computer";
    }

}

class Audio2 extends Product2 {

    Audio2() {
        super(50);
    }

    public String toString() {
        return "Audio";
    }
}

class Buyer2 {

    int money = 1000;   // 잔액
    int bonusPoint = 0; // 보너스 포인트
    Product2[] item = new Product2[10]; // 구매한 상품을 저장하기 위한 배열
    int i = 0;  // Product 배열에 사용될 카운터

    void buy(Product2 product) {
        if (money < product.price) {
            System.out.println("잔액이 부족합니다.");
            return;
        }

        money -= product.price;
        bonusPoint += product.bonusPoint;
        item[i++] = product;
        System.out.println(product + "을/를 구입하셨습니다.");
    }

    void summary() {
        int sum = 0;    // 구매한 물품의 가격합계
        String itemList = ""; // 구매한 물품목록

        for (int i = 0; i < item.length; i++) {
            if (item[i] == null)
                break;
            sum += item[i].price;
            itemList += (i == 0) ? "" + item[i] : "," + item[i];
        }
        System.out.println("구입하신 물품의 총금액은 " + sum + "만원입니다.");
        System.out.println("구입하신 제품은 " + itemList + "입니다.");
    }
}
```
```
Computer을/를 구입하셨습니다.
Tv을/를 구입하셨습니다.
Audio을/를 구입하셨습니다.
구입하신 물품의 총금액은 350만원입니다.
구입하신 제품은 Computer,Tv,Audio입니다.
```
##### 여러 종류의 객체를 배열로 다루기 예제 2 : `Vector`사용
```java
public class PolyArgumentTest3 {
    public static void main(String[] args) {
        Buyer3 buyer3 = new Buyer3();
        buyer3.buy(new Computer3());
        buyer3.buy(new Tv3());
        buyer3.buy(new Audio3());
        buyer3.summary();
    }
}

class Product3 {
    int price;
    int bonusPoint;

    Product3(int price) {
        this.price = price;
        bonusPoint = (int) (price/10.0);
    }

    Product3() {
        price = 0;
        bonusPoint = 0;
    }
}

class Tv3 extends Product3 {

    Tv3() {
        super(100);
    }

    public String toString() {
        return "Tv";
    }
}

class Computer3 extends Product3 {

    Computer3() {
        super(200);
    }

    public String toString() {
        return "Computer";
    }

}

class Audio3 extends Product3 {

    Audio3() {
        super(50);
    }

    public String toString() {
        return "Audio";
    }
}

class Buyer3 {

    int money = 1000;   // 잔액
    int bonusPoint = 0; // 보너스 포인트
    Vector item = new Vector(); // 구입한 제품을 저장하기 위한 Vector 객체
    int i = 0;  // Product 배열에 사용될 카운터

    void buy(Product3 product) {
        if (money < product.price) {
            System.out.println("잔액이 부족합니다.");
            return;
        }

        money -= product.price;
        bonusPoint += product.bonusPoint;
        item.add(product);
        System.out.println(product + "을/를 구입하셨습니다.");
    }

    void refund(Product3 product3) {
        if (item.remove(product3)) {
            money += product3.price;
            bonusPoint -= product3.bonusPoint;
            System.out.println(product3 + "을/를 반품하셨습니다.");
        } else {
            System.out.println("구입하신 제품이 없습니다.");
        }
    }

    void summary() {
        int sum = 0;    // 구매한 물품의 가격합계
        String itemList = ""; // 구매한 물품목록

        if (item.isEmpty()) {
            System.out.println("구입하신 제품이 없습니다.");
            return;
        }

        for (int i = 0; i < item.size(); i++) {
            Product3 product = (Product3) item.get(i);
            sum += product.price;
            itemList += (i == 0) ? "" + product : "," + product;
        }
        System.out.println("구입하신 물품의 총금액은 " + sum + "만원입니다.");
        System.out.println("구입하신 제품은 " + itemList + "입니다.");
    }
}
```
```
Computer을/를 구입하셨습니다.
Tv을/를 구입하셨습니다.
Audio을/를 구입하셨습니다.
구입하신 물품의 총금액은 350만원입니다.
구입하신 제품은 Computer,Tv,Audio입니다.
```
