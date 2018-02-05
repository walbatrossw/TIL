# 1 `Generics`

## 1.1 `Generics`?
**`Generics`는 다양한 타입의 객체를 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입체크(complie-time type check)를 해주는 기능** 이다. 객체의 타입을 컴파일 시에 체크하기 때문에 객체의 타입 안정성을 높이고 형변환의 번거로움이 줄어든다.

- 타입의 안정성을 높이는 것?
  - 의도하지 않은 타입의 객체가 저장되는 것을 막고, 저장된 객체를 꺼내올 때 원래의 타입과 다른 타입으로 잘못 형변환되어 발생할 수 있는 오류를 줄여준다는 뜻이다.
- 장점
  - 타입 안전성을 제공한다.
  - 타입체크와 형변환을 생략할 수 있으므로 코드가 간결해진다.

---

## 1.2 `Generics` 클래스의 선언

### 1) 제네릭 클래스 선언방법
```java
class Box {

  Object item;

  void setItem(Object item) {
    this.item = item;
  }

  Object getItem() {
    return item;
  }

}
```
```java
// 제네릭 타입 T를 선언
// Object를 모두 T로 바꿈
class Box<T> {
  T item;

  void setItem(T item) {
    this.item = item;
  }

  T getItem() {
    return item;
  }
}
```
- `Box<T>`의 `T`를 **타입변수** 라고 한다.
- 여기서 `T`는 Type의 첫글자를 따온 것인데 `T`가 아닌 다른 것을 사용해도 무방하다. `ArrayList<E>`의 경우, Element의 첫글자를 따서 사용했고, Map<K, V>처럼 타입변수가 여러 개일 때는 `,`를 구분자로 나열하면된다. `K`는 Key, `V`는 Value를 의미한다.
- 기호의 종류만 다를뿐 임의의 참조형 타입을 의미한다는 것은 모두 같다.
- 기존에는 다양한 종류의 타입을 다루는 메서드의 매개변수나 리턴타입으로 `Object`타입의 참조변수를 많이 사용했고, 그로 인해 형변환이 불가피 했지만 이젠 `Object`타입 대신 원하는 타입을 지정하기만 하기만 하면된다.

### 2) 제네릭 클래스의 객체 생성
```java
Box<String> b = new Box<String>(); // 타입 T대신 실제 타입을 지정
//b.setItem(new Object()); // 에러 String타입 이외의 타입은 지정이 불가능
b.setItem("abc"); // OK, String타입이므로 가능
//String item = (String) b.getItem(); // 형변환이 이제 필요 없음
```
- 제네릭 클래스가 된 `Box`클래스의 객체를 생성할 때는 다음과 같이 참조변수와 생성자에 타입 `T`대신 사용될 실제타입을 지정해주어야 한다.
```java
class Box<String> {
  String item;

  void setItem(String item) {
    this.item = item;
  }

  String getItem() {
    return item;
  }
}
```
- 위의 코드에서 타입 `T`대신 `String`을 지정해주었기 때문에 제네릭 클래스 `Box<T>`는 바로 위의 코드와 같이 정의된 것과 같다.
- 만약 `Box`클래스에 `String`만 담을 것이라면 타입변수를 선언하지 않고 위에서 처럼 직접 타입을 적어주는 것이 가능한데 `Box<String>`클래스는 `String`만을 담을 수밖에 없다.
- 반면에 `Box<T>`클래스는 어떤타입이든 한가지 타입을 정해서 담을 수 있다.
- 제네릭이 도입되기 이전의 코드와의 호환을 위해 제네릭 클래스인데도 불구하고 예전의 방식으로 객체를 생성하는 것이 허용된다.
- 이렇게 하였을 경우 제네릭 타입을 지정하지 않아 안전하지 않다는 경고가 발생한다 `unchecked or unsafe operation`

### 3) 제네릭의 용어
```java
class Box<T> {}
```
- `Box<T>` : 제네릭 클래스, T의 Box 또는 T Box라고 읽는다.
- `T` : 타입변수, 타입매개변수
- `Box` : 원시타입(raw type)

### 4) 제네릭의 제한
```java
Box<Apple> appleBox = new Box<Apple>();
Box<Grape> grapeBox = new Box<Grape>();
```
- 제네릭 클래스 `Box`의 객체를 생성할 때, 객체별로 다른 타입을 지정하는 것은 적절하다. 인스턴스별로 다르게 동작하도록 만든 기능이기 때문이다.

```java
class Box<T> {
  static T item;  // 에러
  static int compare(T t1, T t2) { // 에러
    //...
  }
}
```
- 하지만 모든 객체에 대해 동일하게 작동해야하는 **`static`멤버에 타입변수 `T`를 사용할 수 없다.**
- `T`는 인스턴스 변수로 간주되기 때문인데 `static`멤버는 인스턴스변수를 참조할 수 없다는 것일 이전에 배웠었다.
- `static`멤버는 타입변수에 지정된 타입의 종류에 관계없이 동일한 것이어야하기 때문이다.

```java
class Box<T> {
  T[] itemArr; // OK T타입의 배열을 위한 참조변수
  T[] toArray() {
    T[] tempArr = new T[itemArr.length]; // 에러, 제네릭 배열 생성불가
    return tempArr;
  }
}
```
- 제네릭 배열 타입의 참조변수를 선언은 가능하지만 **제네릭 타입의 배열의 생성을 허용하지 않는다.**
- 제네릭 배열을 생성할 수 없는 것은 `new`연산자 때문인데, 이 연산자는 컴파일 시점에 타입 `T`가 뭔지 정확히 알아야한다. 그런데 위의 코드에 정의된 `Box<T>`클래스를 컴파일하는 시점에서는 `T`가 어떤 타입이 될지 전혀 알 수 없다.
- **`instanceof`연산자도 `new`연산자와 같은 이유로 `T`를 피연산자로 사용할 수 없다.**

---

## 1.3 `Generics`클래스의 객체 생성과 사용

### 1) 제네릭 객체 생성과 사용법
```java
class Box<T> {
  ArrayList<T> list = new ArrayList<T>();

  void add(T item) {
    list.add(item);
  }

  T get(int i) {
    return list.get(i);
  }

  ArrayList<T> getList() {
    return list;
  }

  int size() {
    return list.size();
  }

  public String toString() {
    return list.toString();
  }
}
```
- `ArrayList`를 이용해 여러 객체를 저장할 수 있도록 하였다.

```java
Box<Apple> appleBox = new Box<Apple>(); // OK, 타입 일치
Box<Apple> appleBox = new Box<Grape>(); // 에러, 타입 불일치
```
- `Box<T>`의 객체를 생성할 때는 위와 같이 작성하면 된다.
- **참조변수와 생성자에 대입된 타입이 일치** 해야한다.
- 만약 일치하지 않으면 에러가 발생한다.

```java
Box<Fruit> appleBox = new Box<Apple>(); // 에러, 대입된 타입이 다르다
```
- **상속관계에 있어도 타입이 일치** 해야만 한다.
- `Apple`이 `Fruit`에 자손임에도 에러가 발생한다.

```java
Box<Apple> appleBox = new FruitBox<Apple>(); // OK, 타입일치, 두 제네릭 클래스의 타입이 상속관계
```
- **두 제네릭 클래스의 타입이 상속관계에 있고, 대입된 타입이 같을 경우는 허용** 된다.
- `FruitBox`는 `Box`의 자손이고, 참조변수의 타입과 생성자에 대입된 타입이 `Apple`과 동일하다.

```java
Box<Apple> appleBox = new Box<Apple>();
Box<Apple> appleBox = new Box<>(); // OK, JDK1.7부터 생략 가능
```
- **JDK1.7부터 추정이 가능한 경우 타입을 생략** 할 수 있게 되었다.
- 참조변수의 타입으로부터 `Box`가 `Apple`타입의 객체만 저장한다는 것을 알 수 있기 때문에, 생성자에 반복해서 타입을 지정해주지 않아도 된다.

```java
Box<Apple> appleBox = new Box<Apple>();
appleBox.add(new Apple());  // OK
appleBox.add(new Greape()); // 에러, Box<Apple>에는 Apple객체만 추가 가능
```
- **메서드의 매개변수에 대입된 타입과 다른타입의 객체를 추가할 수 없다.**

```java
Box<Fruit> FruitBox = new Box<Fruit>();
fruitBox.add(new Fruit());  // OK
fruitBox.add(new Apple());  // OK, void add(Fruit item);
```
- **메서드의 매개변수에 대입된 타입의 자손의 객체는 추가할 수 있다.**

### 2) 제네릭 클래스의 객체 생성과 사용 예제1
```java
public class FruitBoxEx1 {
    public static void main(String[] args) {
        Box<Fruit> fruitBox = new Box<Fruit>();
        Box<Apple> appleBox = new Box<Apple>();
        Box<Toy> toyBox = new Box<Toy>();
        //Box<Grape> grapeBox = new Box<Fruit>(); // 에러, 타입불일치

        fruitBox.add(new Fruit());
        fruitBox.add(new Apple());

        appleBox.add(new Apple());
        appleBox.add(new Apple());
        //appleBox.add(new Toy()); // 에러 Box<Apple>에는 Apple만 담을 수 있음

        toyBox.add(new Toy());
        //toyBox.add(new Apple()); // 에러 Box<Toy>에는 Apple을 담을 수 없음

        System.out.println(fruitBox);
        System.out.println(appleBox);
        System.out.println(toyBox);
    }
}

class Box<T> {
    // ArrayList를 이용하여 여러 객체를 저장
    ArrayList<T> list = new ArrayList<T>();

    void add(T item) {
        list.add(item);
    }

    T get(int i) {
        return list.get(i);
    }

    ArrayList<T> getList() {
        return list;
    }

    int size() {
        return list.size();
    }

    public String toString() {
        return list.toString();
    }
}

class Fruit {
    public String toString() {
        return "Fruit";
    }
}

class Apple extends Fruit {
    public String toString() {
        return "Apple";
    }
}

class Grape extends Fruit {
    public String toString() {
        return "Grape";
    }
}

class Toy {
    public String toString() {
        return "Toy";
    }
}
```
```
[Fruit, Apple]
[Apple, Apple]
[Toy]
```

---

## 1.4 제한된 `Generics`클래스

### 1) 제한된 `Generics`클래스 사용법

```java
FruitBox<Toy> fruitBox = new FruitBox<Toy>();
fruitBox.add(new Toy());  // OK, 과일상자에 장난감을 담을 수 있다.. 그러나 뭔가 잘못된 것 같지 않은가?
```
- 타입문자로 사용할 타입을 명시하면 한 종류의 타입만 저장할 수 있도록 제한할 수 있지만, 그래도 위의 코드처럼 여전히 모든 종류의 타입을 지정할 수 있다는 것에는 변함이 없다.
- 그렇다면 타입 매개변수 `T`에 지정할 수 있는 타입의 종류를 제한할 수 있는 방법에 대해 알아보자.

```java
// Fruit의 자손만 타입으로 지정 가능
class FruitBox<T extends Fruit> {
  ArrayList<T> list = new ArrayList<T>();
}
```
- 위와 같이 **제네릭 타입에 `extends`를 사용** 하면 특정 타입의 자손들만 대입할 수 있게 제한할 수 있다.
```java
FruitBox<Apple> appleBox = new FruitBox<Apple>(); // OK, Apple은 Fruit의 자손이기 때문에 가능
FruitBox<Toy> toyBox = new FruitBox<Toy>(); // 에러, Toy는 Fruit의 자손이 아니기 때문 불가능
```
- 여전히 한 종류의 타입만 담을 수 있지만, `Fruit`클래스의 자손들만 담을 수 있다는 제한이 더 추가됬을 뿐이다.

```java
FruitBox<Fruit> appleBox = new FruitBox<Fruit>();
fruitBox.add(new Apple());  // OK, Apple은 Fruit의 자손
fruitBox.add(new Grape());  // OK, Grape은 Fruit의 자손
```
- `add()`의 매개변수 타입 `T`도 `Fruit`와 자손 타입이 될 수 있으므로, 위와 같이 여러과일을 담을 수 있는 상자가 될 수 있다.

```java
class FruitBox<T extends Eatable> {
  //...
}

interface Eatable {}
```
- 만약 클래스가 아닌 **인터페이스를 구현해야할 때는 implement가 아닌 extends를 사용** 한다.

```java
class FruitBox<T extends Fruit & Eatable> {
  //...
}
```
- 타입 `T`를 Fruit의 자손이면서 `Eatable`인터페이스도 구현해야한다면 위와 같이 `&`기호로 연결해준다.

### 2) 제한된 `Generics`클래스 예제 1
```java
public class FruitBoxEx2 {
    public static void main(String[] args) {
        FruitBox2<Fruit2> fruitBox = new FruitBox2<Fruit2>();
        FruitBox2<Apple2> appleBox = new FruitBox2<Apple2>();
        FruitBox2<Grape2> grapeBox = new FruitBox2<Grape2>();
        //FruitBox2<Grape2> grapeBox = new FruitBox2<Apple2>(); // 에러, 타입 불일치
        //FruitBox2<Toy2> toyBox = new FruitBox2<Toy2>(); // 에러

        fruitBox.add(new Fruit2());
        fruitBox.add(new Apple2());
        fruitBox.add(new Grape2());
        appleBox.add(new Apple2());
        //appleBox.add(new Grape2()); // 에러, Grape는 Apple의 자손이 아님
        grapeBox.add(new Grape2());

        System.out.println("fruitBox = " + fruitBox);
        System.out.println("appleBox = " + appleBox);
        System.out.println("grapeBox = " + grapeBox);
    }
}

interface Eatable {}

class Fruit2 implements Eatable {
    public String toString() {
        return "Fruit";
    }
}

class Apple2 extends Fruit2 {
    public String toString() {
        return "Apple";
    }
}

class Grape2 extends Fruit2 {
    public String toString() {
        return "Grape";
    }
}

class Toy2 {
    public String toString() {
        return "Toy";
    }
}

class Box2<T> {
    // ArrayList를 이용하여 여러 객체를 저장
    ArrayList<T> list = new ArrayList<T>();

    void add(T item) {
        list.add(item);
    }

    T get(int i) {
        return list.get(i);
    }

    ArrayList<T> getList() {
        return list;
    }

    int size() {
        return list.size();
    }

    public String toString() {
        return list.toString();
    }
}

// 타입 : Fruit2 상속, Eatable 구현 / 클래스 : Box2 상속
class FruitBox2<T extends Fruit2 & Eatable> extends Box2<T> {}
```
```
fruitBox = [Fruit, Apple, Grape]
appleBox = [Apple]
grapeBox = [Grape]v
```

---

## 1.5 와일드카드



## 1.6 제네릭 메서드

## 1.7 제네릭 타입의 형변환

## 1.8 제네릭 타입의 제거
