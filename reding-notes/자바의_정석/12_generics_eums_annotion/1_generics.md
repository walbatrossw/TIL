본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. `Generics`이란?
**`Generics`는 다양한 타입의 객체를 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입 체크(complie-time type check)를 해주는 기능** 이다. 객체의 타입을 컴파일 시에 체크하기 때문에 객체의 타입 안정성을 높이고 형변환의 번거로움이 줄어든다.

**타입의 안정성을 높이는 것이란?**
- 의도하지 않은 타입의 객체가 저장되는 것을 막고, 저장된 객체를 꺼내올 때 원래의 타입과 다른 타입으로 잘못 형변환되어 발생할 수 있는 오류를 줄여준다는 뜻이다.

**제네릭의 장점**
- 타입 안전성을 제공한다.
- 타입체크와 형변환을 생략할 수 있으므로 코드가 간결해진다.

## 2. `Generics` 클래스의 선언

**제네릭 클래스 선언방법**
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

**제네릭 클래스의 객체 생성**
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

**제네릭의 용어**

```java
class Box<T> {}
```
- `Box<T>` : 제네릭 클래스, T의 Box 또는 T Box라고 읽는다.
- `T` : 타입변수, 타입매개변수
- `Box` : 원시타입(raw type)

**제네릭의 제한**
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


## 3. `Generics`클래스의 객체 생성과 사용

**제네릭 객체 생성과 사용법**
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

##### 제네릭 클래스의 객체 생성과 사용 예제 1
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


## 4. 제한된 `Generics`클래스

**제한된 `Generics`클래스 사용법**

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

##### 제한된 `Generics`클래스 예제 1
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

## 5. 와일드카드

**와일드카드란?**
제네릭 타입을 매개변수나 반환 타입으로 사용할 때 구체적인 타입 대신 와일드 카드를 다음과 같이 세 가지 형태로 사용할 수 있다.
- `<? extends T>` : 와일드 카드의 상한제한, `T`와 그의 자손들만 가능
- `<? super T>` : 와일드 카드의 하한제한, `T`와 그의 조상들만 가능
- `<?>` : 제한 없음, 모든 타입이 가능, `<? extends Object>`와 동일
- 제네릭 클래스와 달리 와일드 카드에는 `&`를 사용할 수 없음

##### 와일드카드 예제1 : `<? extends T>` - 와일드카드 상한제한
```java
public class FruitBoxEx3 {
    public static void main(String[] args) {
        FruitBox3<Fruit3> fruitBox = new FruitBox3<Fruit3>();
        FruitBox3<Apple3> appleBox = new FruitBox3<Apple3>();

        fruitBox.add(new Apple3());
        fruitBox.add(new Grape3());
        appleBox.add(new Apple3());
        appleBox.add(new Apple3());

        // 와일드카드 적용 전
        System.out.println(Juicer.makeJuice1(fruitBox));   // OK
        //System.out.println(Juicer.makeJuice1(appleBox)); // 에러, fruitBox 외에 다른 매개변수는 들어갈 수가 없음

        // 와일드카드 적용후
        System.out.println(Juicer.makeJuice2(fruitBox));    // OK
        System.out.println(Juicer.makeJuice2(appleBox));    // OK
    }
}

class Juicer {
    // 와일드 카드 적용 전
    // 매개변수를 FruitBox3<Fruit3>로 고정시킬 경우, FruitBox3<Apple3>는 매개변수가 될 수 없음
    static Juice makeJuice1(FruitBox3<Fruit3> box3) {
        String temp = "";

        for (Fruit3 fruit3 : box3.getList())
            temp += fruit3 + " ";

        return new Juice(temp);
    }

    // 아래와 같이 오버로딩을 할 경우, 컴파일 에러가 발생함
    // 제네릭타입이 다른 것만으로 오버로딩이 성립되지 않기 때문
    // 제네릭타입은 컴파일러가 컴파일할 때만 사용하고 제거해버리기 때문에 오버로딩이 아닌 메서드 중복정의가 되버림
//    static Juice makeJuice1(FruitBox3<Apple3> box3) {
//        String temp = "";
//
//        for (Fruit3 fruit3 : box3.getList())
//            temp += fruit3 + " ";
//
//        return new Juice(temp);
//    }

    // 와일드 카드 적용 이후
    // 매개변수로 FruitBox3<Fruit3>, FruitBox3<Apple3>, FruitBox3<Grape3> 전부다 가능해짐
    static Juice makeJuice2(FruitBox3<? extends Fruit3> box3) {
        String temp = "";

        for (Fruit3 fruit3 : box3.getList())
            temp += fruit3 + " ";

        return new Juice(temp);
    }
}

class Juice {
    String name;

    Juice(String name) {
        this.name = name + "Juice";
    }

    public String toString() {
        return name;
    }
}

class Box3<T> {

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

class FruitBox3<T extends Fruit3> extends Box3<T> {

}

class Fruit3 {
    public String toString() {
        return "Fruit";
    }
}

class Apple3 extends Fruit3 {
    public String toString() {
        return "Apple";
    }
}

class Grape3 extends Fruit3 {
    public String toString() {
        return "Grape";
    }
}
```
```
Apple Grape Juice
Apple Grape Juice
Apple Apple Juice
```

##### 와일드카드 예제2 : `<? super T>` - 와일드카드의 하한제한
```java
public class FruitBoxEx4 {
    public static void main(String[] args) {
        FruitBox4<Apple4> appleBox = new FruitBox4<Apple4>();
        FruitBox4<Grape4> grapeBox = new FruitBox4<Grape4>();

        appleBox.add(new Apple4("GreenApple", 300));
        appleBox.add(new Apple4("GreenApple", 100));
        appleBox.add(new Apple4("GreenApple", 200));

        grapeBox.add(new Grape4("GrapeApple", 400));
        grapeBox.add(new Grape4("GrapeApple", 300));
        grapeBox.add(new Grape4("GrapeApple", 200));

        Collections.sort(appleBox.getList(), new AppleComp());
        Collections.sort(grapeBox.getList(), new GrapeComp());
        System.out.println(appleBox);
        System.out.println(grapeBox);

        Collections.sort(appleBox.getList(), new FruitComp());
        Collections.sort(grapeBox.getList(), new FruitComp());
        System.out.println(appleBox);
        System.out.println(grapeBox);
    }
}


class Box4<T> {

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

class FruitBox4<T extends Fruit4> extends Box4<T> {

}

class Fruit4 {
    String name;
    int weight;

    Fruit4(String name, int weight) {
        this.name = name;
        this.weight = weight;
    }

    public String toString() {
        return name + "(" + weight + ")";
    }
}

class Apple4 extends Fruit4 {

    Apple4(String name, int weight) {
        super(name, weight);
    }

}

class Grape4 extends Fruit4 {

    Grape4(String name, int weight) {
        super(name, weight);
    }

}

class AppleComp implements Comparator<Apple4> {
    public int compare(Apple4 t1, Apple4 t2) {
        return t2.weight - t1.weight;
    }
}

class GrapeComp implements Comparator<Grape4> {
    public int compare(Grape4 t1, Grape4 t2) {
        return t2.weight - t1.weight;
    }
}

class FruitComp implements Comparator<Fruit4> {
    public int compare(Fruit4 t1, Fruit4 t2) {
        return t1.weight - t2.weight;
    }
}
```
```
[GreenApple(300), GreenApple(200), GreenApple(100)]
[GrapeApple(400), GrapeApple(300), GrapeApple(200)]
[GreenApple(100), GreenApple(200), GreenApple(300)]
[GrapeApple(200), GrapeApple(300), GrapeApple(400)]
```
위 예제는 `Collections.sort()`를 이용해서 `appleBox`와 `grapeBox`에 담긴 과일을 무게별로 정렬하는 예제를 통해서 와일드카드 하한제한을 알아보자.
- `Collections.sort()`의 선언부는 아래의 코드와 같다.
  ```java
  static <T> void sort(List<T> list, Comparator<? super T> c)
  ```
  - `static`옆에 있는 `<T>`는 메서드에 선언된 제네릭타입으로 이런 메서드를 제네릭 메서드라고 한다.
  - 첫번째 매개변수는 정렬할 대상이고, 두번째 매개변수는 정렬할 방법이 정의된 `Comparator`이다.
  - **`Comparator`의 제네릭 타입에 하한제한이 걸려있는 와일드카드가 사용** 되었다.
- 그렇다면 먼저 아래와 같이 와일드카드가 사용되지 않았다고 가정해보고, 매개변수 `T`에 `Apple4`가 대입되면 어떻게 바뀌는지 보자.
  ```java
  static <T> void sort(List<T> list, Comparator<T> c)
  ```
  ```java
  static void sort(List<Apple4> list, Comparator<Apple4> c)
  ```
- 이것은 `List<Apple4>`를 정렬하기 위해서는 `Comparator<Apple4>`이 필요하다는 의미인데, 그래서 `Comparator<Apple4>`를 구현한 `AppleComp`클래스를 아래와 같이 정의하였다.
  ```java
  class AppleComp implements Comparator<Apple4> {
    public int compare(Apple4 t1, Apple4 t2) {
      return t2.weight - t1.weight;
    }
  }
  ```
- 지금까지는 별문제가 없어보이지만 만약 `Apple4`대신 `Grape4`가 대입된다면 `List<Grape4>`를 정렬하기 위해서는 `Comparator<Grape>`가 아래의 코드와 같이 또 하나의 클래스가 필요하게 된다.
  ```java
  class GrapeComp implements Comparator<Grape4> {
    public int compare(Grape4 t1, Grape4 t2) {
      return t2.weight - t1.weight;
    }
  }
  ```
- 그런데 `AppleComp`와 `GrapeComp`는 타입만 다를뿐 완전히 같은 코드이다. 코드의 중복도 문제지만, 새로운 `Fruit`자손이 생길 때마다 위와 같은 코드를 반복해서 만들어야한다는 문제도 발생한다.
- 이러한 문제를 해결하기 위해서는 타입 매개변수에 하한 제한의 와일드 카드를 적용해야만한다.
- 그래서 `Collections.sort()`가 처음에 본 것처럼 정의가 되어 있었던 것이다.
  ```java
  static <T> void sort(List<T> list, Comparator<? super T> c)
  ```
- 위의 코드의 타입 매개변수에 `Apple4`가 대입되면 아래와 같이 된다.
  ```java
  static void sort(List<Apple4> list, Comparator<? super Apple4> c)
  ```
- 매개변수 타입이 `Comparator<? super Apple4>`이라는 의미는 `Comparator`의 타입 매개변수로 `Apple4`와 그 조상이 가능하다는 것이다.
- 즉, `Comparator<Apple4>`, `Comparator<Fruit4>`, `Comparator<Object>` 중 하나가 매개변수로 올 수 있다는 것을 의미한다.
- 그래서 아래와 같이 `FruitComp`를 만들면 `List<Apple4>`와 `List<Grape4>`를 모두 정렬할 수 있게 된다.
  ```java
  class FruitComp implements Comparator<Fruit4> {
      public int compare(Fruit4 t1, Fruit4 t2) {
          return t1.weight - t2.weight;
      }
  }
  ```

## 6. 제네릭 메서드

**제네릭 메서드란?**

**매서드의 선언부에 제네릭 타입이 선언된 메서드** 를 제네릭 메서드라고 한다. 이전 예제에서 본 것처럼 `Collections.sort()`가 바로 제네릭 메서드이며, **제네릭 타입의 선언 위치는 반환 타입 앞** 이다.

```java
static <T> void sort(List<T> list, Comparator<? super T> c)
```

**제네릭 클래스에 정의된 타입 매개변수와 제네릭 메서드에 정의된 타입 매개변수는 별개의 것** 이다. 같은 타입 문자 `T`를 사용하더라도 같은 것이 아니라는 것에 주의해야한다.

```java
// 클래스의 타입 매개변수 T와
class FruitBox<T> {
  // 메서드 타입 매개변수 T는 타입 문자만 같을 뿐 다르다
  static <T> void sort(List<T> list, Comparator<? super T> c) {

  }
}
```
- 제네릭 클래스 `FruitBox`에 선언된 타입 매개변수 `T`와 제네릭 메서드 `sort()`에 선언된 타입 매개변수 `T`는 타입 문자만 같을 뿐 다르다.
- `static`멤버에는 타입 매개변수를 사용할 수 없지만, 이처럼 메서드에 제네릭 타입을 선언하고 사용하는 것은 가능하다.
- 메서드에 선언된 제네릭 타입은 마치 지역 변수를 선언한 것과 같다고 생각하면 이해하기가 쉬운데 이 타입 매개변수는 메서드 내에서만 지역적으로 사용되기 때문에 메서드가 `static`이든 아니든 상관없기 때문이다.

**제네릭 메서드 선언, 호출**

그렇다면 이전 예제의 `makeJuice2()`를 제네릭 메서드로 변경하고, 호출해보자
```java
static Juice makeJuice2(FruitBox3<? extends Fruit3> box3) {
    //...
}
```
```java
// 제네릭 메서드로 변환하기
// 제네릭 타입을 반환타입 앞에 선언
static <T extends Fruit3> Juice makeJuice2(FruitBox3<T> box3) {
    //...
}
```
```java
// 제네릭 메서드 호출하기
FruitBox3<Fruit3> fruitBox = new FruitBox3<Fruit3>();
FruitBox3<Apple3> appleBox = new FruitBox3<Apple3>();

//...

// 메서드를 호출할 때는 타입변수에 타입을 대입해야함
System.out.println(Juicer.<Fruit3>makeJuice2(fruitBox));
System.out.println(Juicer.<Apple3>makeJuice2(appleBox));

// 그러나 대부분 컴파일러가 타입을 추정할 수 있기 때문에 생략이 가능
System.out.println(Juicer.makeJuice2(fruitBox));
System.out.println(Juicer.makeJuice2(appleBox));

// 주의사항
// 제네릭 메서드를 호출할 때는 대입된 타입을 생략할 수 없는 경우, 참조변수나 클래스 이름을 생략할 수 없음
System.out.println(<Fruit3>makeJuice2(fruitBox)); //에러, 클래스 이름 생략 불가
System.out.println(this.makeJuice2(fruitBox));  // OK
System.out.println(Juicer.makeJuice2(fruitBox)); // OK
```

**제네릭 메서드 매개변수 타입의 간소화**
```java
public static void printAll(ArrayList<? extends Products> list1,
                            ArrayList<? extends Products> list2) {
  //...
}
```
제네릭 메서드는 메개변수의 타입이 복잡할 때도 유용한데, 만일 위와 같은 코드가 있다면 타입을 별도로 선언함으로써 코드를 간략하게 만들 수 있다.
```java
// 타입을 별도로 선언
public static <T extends Products> void printAll(ArrayList<T> list1,
                                                  ArrayList<T> list2) {
  //...
}
```

**제네릭 메서드 타입이 복잡하게 선언된 경우?**

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
  //...
}
```
위의 코드는 `Collections`클래스의 `sort()` 중에 하나인데 매개변수가 하나짜리이다. 매개변수로 지정된 `List<T>`를 정렬한다는 것은 알겠지만, 메서드에 선언된 제네릭 타입이 다소 좀 복잡하다. 이럴 경우 일단 와일드 카드를 걷어내보자

```java
// 와일드 카드를 걷어낸 뒤
public static <T extends Comparable<T>> void sort(List<T> list) {
  //...
}
```
일단 와일드 카드를 걷어내고 보니 `List<T>`의 요소가 `Comparable`인터페이스를 구현한 것이어야 한다는 뜻인데 이제 다시 와일드 카드를 넣고 자세히 살펴보자.

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
  //...
}
```
- `List<T> list` : 타입 `T`를 요소로 하는 `List`를 매개변수로 허용
- `<T extends Comparable>` : `T`는 `Comparable`인터페이스를 구현한 클래스이어야 한다는 것을 의미
- `Comparable<? super T>` : `T`또는 그 조상의 타입을 비교하는 `Comparable`이어야 한다는 것을 의미
- 만약 `T`가 `Student`이고, `Person`의 자손이라면 `<? super T>`는 `Student`, `Person`, `Object` 모두 가능하다.

## 7. 제네릭 타입의 형변환

**제네릭 타입과 넌제네릭(non-generic) 타입 간의 형변환 : 가능**
```java
Box box = null;
Box<Object> objBox = null;

box = (Box) objBox; // OK, 형변환 가능 : 제네릭 -> 원시타입, 경고발생
objBox = (Box<Object>) box; // OK, 형변환 가능 : 원시타입 -> 제네릭, 경고발생
```


**대입된 타입이 다른 제네릭 타입 간의 형변환 : 불가능**
```java
Box<Object> objBox = null;
Box<String> strBox = null;

objBox = (Box<Object>) strBox; // 에러, 형변환 불가
strbox = (Box<String>) objBox; // 에러, 형변환 불가
```

**제네릭 타입과 와일드 카드 타입 간의 형변환**
```java
Box<? extends Object> wBox = new Box<String>();
```
