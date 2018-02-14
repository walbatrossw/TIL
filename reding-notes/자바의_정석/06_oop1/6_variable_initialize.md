본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.


## 1. 변수의 초기화

변수를 선언하고 처음으로 값을 저장하는 것을 **변수 초기화** 라고 한다. 변수의 초기화는 경우에 따라 필수적일수도 선택적일수도 있지만 가능하면 선언과 동시에 적절한 값으로 초기화하는 것이 바람직하다. **멤버변수는 초기화를 하지 않아도 변수의 타입에 맞는 기본값으로 초기화가 이루어지지만 지역변수는 사용하기 전에 반드시 초기화가 이루어져야 한다.**

```java
class InitTest {
  int x;      // 인스턴스 변수
  int y = x;  // 인스턴스 변수

  void method() {
    int i;      // 지역변수
    int j = i;  // 에러, 지역변수를 초기화하지 않고 바로 사용했기 때문
  }
}
```

##### 각 타입의 기본값(default value)

|자료형|기본값|
|---|---|
|`boolean`   |false   |
|`char`   |'\u0000'   |
|`byte`, `short`, `int`   |0   |
|`long`   |0   |
|`float`   |0.0f   |
|`double`   |0.0d or 0.0   |
|renference value   |null   |

멤버변수를 초기화하는 방법은 다음과 같다.
- **명시적 초기화(explicit initialization)**
- **생성자(constructor)**
- **초기화 블럭(initialization block)**
  - 인스턴스 초기화 블럭 : 인스턴스 변수를 초기화하는데 사용
  - 클래스 초기화 블럭 : 클래스를 초기화하는데 사용

## 2. 명시적 초기화

**변수 선언과 동시에 초기화** 하는 것을 명시적 초기화(explicit initialization), 변수의 초기화라고 한다. 가장 기본적이면서도 간단한 초기화 방법이므로 여러 초기화 방법 중에서도 가장 우선적으로 고려되어야 한다.

```java
class Car {
  int door = 4;                 // 기본형 변수의 초기화
  Engine engine = new Engine(); // 참조형 변수의 초기화
}
```

## 3. 초기화 블럭

초기화 블럭의 종류에 따라 정리하면 다음과 같다.

- **클래스 초기화 블럭**
  - 클래스를 초기화하는데 사용한다.
  - 클래스가 처음 메모리에 로딩될 때 한번만 수행된다.

- **인스턴스 초기화 블럭**
  - 인스턴스 변수를 초기화하는데 사용한다.
  - 생성자와 같이 인스턴스가 생성될 때 수행된다.
  - 인스턴스 초기화 블럭이 생성자보다 먼저 수행된다.

초기화 블럭 내에는 메서드 내에서와 같이 조건문, 반복문, 예외처리구문 등을 자유롭게 사용할 수 있으므로, 초기화 작업이 복잡하여 명시적 초기화만으로는 부족한 경우 초기화 블럭을 사용한다.
```java
class InitBlock {

  // 클래스 초기화 블럭
  static {

  }

  // 인스턴스 초기화 블럭
  {

  }
}
```

인스턴스 변수의 초기화는 주로 생성자를 사용하고, **인스턴스 초기화 블럭은 모든 생성자에서 공통으로 수행되야 하는 코드를 넣는데 사용** 한다. 아래와 같이 모든 생성자에서 공통으로 수행되어야 하는 문장들이 있을 때, 이 문장들을 각 생성자마다 써주기보다는 인스턴스 블럭에 넣어줌으로써 코드가 보다 간결하게 작성할 수 있다.

```java
Car() {
  count++;            // 코드 중복
  serialNo = count;   // 코드 중복
  color = "white";
  gearType = "auto";
}

Car(String color, String gearType) {
  count++;            // 코드 중복
  serialNo = count;   // 코드 중복
  this.color = color;
  this.gearType = gearType;
}
```
```java
// 인스턴스 초기화 블럭을 통해 중복 제거
{
  count++;            
  serialNo = count;   
}

Car() {
  color = "white";
  gearType = "auto";
}

Car(String color, String gearType) {
  this.color = color;
  this.gearType = gearType;
}
```

##### 초기화 블럭 예제 1 : 초기화 순서

예제를 통해 초기화 순서가 어떻게 진행되는지 보자.

```java
public class BlockTest {
  public static void main(String[] args) {

    System.out.println("BlockTest blockTest = new BlockTest()");
    BlockTest blockTest = new BlockTest();

    System.out.println();

    System.out.println("BlockTest blockTest2 = new BlockTest()");
    BlockTest blockTest2 = new BlockTest();

  }

  // 클래스 초기화 블럭
  static {
    System.out.println("static{} class init block");
  }

  // 인스턴스 초기화 블럭
  {
    System.out.println("{} : instance init block");
  }

  // 생성자
  public BlockTest() {
    System.out.println("BlockTest() : constructor");
  }
}
```
```
static{} class init block
BlockTest blockTest = new BlockTest()
{} : instance init block
BlockTest() : constructor

BlockTest blockTest2 = new BlockTest()
{} : instance init block
BlockTest() : constructor
```

콘솔화면에서와 같이 클래스 초기화 블럭, 인스턴스 블럭, 생성자 순으로 초기화가 진행된다. 초기화 블럭의 경우 메모리에 로딩될 때 한번만 초기화가 이루어지고, 인스턴스 블럭과 생성자는 인스턴스가 생성될 때마다 초기화가 수행된다.


##### 초기화 블럭 예제 2 : `static` 초기화 블럭(클래스 초기화 블럭)

```java
public class StaticBlockTest {
  public static void main(String[] args) {
    for (int i = 0; i < arr.length; i++) {
      System.out.println("arr["+i+"] : " + arr[i]);
    }
  }

  // 명시적 배열 초기화
  static int[] arr = new int[10];

  // 클래스 초기화 블럭 : 배열의 각요소들을 random()로 임의의 값을 채움
  static {
    for (int i = 0; i < arr.length; i++) {
      arr[i] = (int) (Math.random() * 10) + 1;
    }
  }
}
```

```
arr[0] : 7
arr[1] : 1
arr[2] : 2
arr[3] : 10
arr[4] : 3
arr[5] : 10
arr[6] : 3
arr[7] : 2
arr[8] : 2
arr[9] : 1
```

위의 예제에서는 명시적 초기화를 통해 배열 `arr`을 생성하고, 클래스 초기화 블럭을 이용해서 배열의 각 요소들을 `random()`을 사용해 임의의 값을 채우도록 했다. 이처럼 배열이나 예외처리가 필요한 초기화에는 명시적 초기화만으로는 복잡한 초기화 작업을 할 수 없기 때문에 클래스 초기화 작업을 사용하도록 하는 것이 좋다.


## 4. 멤버변수의 초기화 시기와 순서

**클래스 변수**
- 초기화 시점 : 클래스가 처음 로딩될 때 딱 한번
- 초기화 순서 : 기본값 -> 명시적 초기화 -> 클래스 초기화 블럭

**인스턴스 변수**
- 초기화 시점 : 인스턴스가 생성될 때마다 각 인스턴스별로 초기화
- 초기화 순서 : 기본값 -> 명시적 초기화 -> 인스턴스 초기화 블럭 -> 생성자

```java
class InitTest {

  // 명시적 초기화
  static int cv = 1;
  int iv = 1;

  // 클래스 초기화 블럭
  static {
    cv = 2;
  }

  // 인스턴스 초기화 블럭
  {
    iv = 2;
  }

  // 생성자
  InitTest() {
    iv = 3;
  }

}
```

##### 인스턴스 블럭을 이용한 초기화 예제 1
```java
public class ProductTest {
  public static void main(String[] args) {
    Product p1 = new Product();
    Product p2 = new Product();
    Product p3 = new Product();

    System.out.println("p1 serialNo : " + p1.serialNo);
    System.out.println("p2 serialNo : " + p2.serialNo);
    System.out.println("p3 serialNo : " + p3.serialNo);

    System.out.println("generated product's count : " + Product.count);
  }
}

class Product {
  static int count = 0; // 클래스 변수
  int serialNo; // 인스턴스 변수

  {
      ++count;
      serialNo = count;
  }

  public Product() {

  }
}
```
```
p1 serialNo : 1
p2 serialNo : 2
p3 serialNo : 3
generated product's count : 3
```

##### 생성자를 이용한 초기화 예제 2
```java
public class DocumentTest {
  public static void main(String[] args) {
    Document d1 = new Document();
    Document d2 = new Document();
    Document d3 = new Document("자바.txt");
    Document d4 = new Document();
    Document d5 = new Document("자바스크립트.txt");
    Document d6 = new Document("스프링.txt");
    Document d7 = new Document();
  }
}

class Document {
  static int count = 0;
  String name;

  Document() {
    this("제목없음" + ++count);
  }

  Document(String name) {
    this.name = name;
    System.out.println("문서 " + this.name + "가 생성됨.");
  }
}
```
```
문서 제목없음1가 생성됨.
문서 제목없음2가 생성됨.
문서 자바.txt가 생성됨.
문서 제목없음3가 생성됨.
문서 자바스크립트.txt가 생성됨.
문서 스프링.txt가 생성됨.
문서 제목없음4가 생성됨.
```
