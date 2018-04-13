본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.


## 1. 람다식(`Lambda expression`)이란?

**람다식은 간다히 말해서 메서드를 하나의 식으로 표현한 것이다.** 메서드를 람다식으로 표현하면 메서드의 이름과 반환값이 없어지므로 람다식을 **익명함수(anonymous function)** 라고도 한다. 이전에 `Collection`의 `Arrays`를 정리하면서 일부 코드에서 람다식을 봤었는데 그 코드를 메서드로 표현하면 아래와 같다.

```java
int[] arr = new int[5];

// 람다식                    
Arrays.setAll(arr, () -> (int)(Math.random()*5)+1);
```

```java
// 위의 람다식을 메서드로 표현
int method() {
  return (int) (Math.ramdom()*5) + 1;
}
```

위의 코드를 보면 메서드보다 람다식이 간결하고 이해하기 쉽다는 것을 알 수 있다. 게다가 모든 메서드는 클래스에 포함되어야 하므로 클래스도 새로 만들어야 하고, 객체도 생성해야만 비로소 메서드를 호출할 수 있다. 그러나 람다식은 이 모든 과정없이 오직 **람다식 자체만으로도 이 메서드의 역할을 수행**할 수 있다. 람다식은 **메서드의 매개변수로 전달하는 것이 가능** 하고, **메서드의 결과로 반환** 될 수도 있다. 람다식으로 **메서드를 변수처럼 다루는 것이 가능** 해진 것이다.

## 2. 람다식 작성하기

```java
// 메서드
ReturnType methodName(Argument argument) {
  //...
}

// 람다식 : 반환타입과 메서드명을 없애고, 매개변수와 {} 사이에 ->를 추가
(Argument argument) -> {
  //...
}
```
람다식은 익명함수답게 **메서드에서 이름과 반환타입을 제거하고 매개변수 선언부와 몸통`{}` 사이에 `->`를 추가** 한다.

#### # 메서드를 람다식으로 변환 과정 및 주의 사항
```java
// 메서드
int max(int a, int b) { return a > b ? a : b; }
```
```java
// 리턴타입과 메서드명 제거, -> 추가
(int a, int b) -> { return a > b ? a : b; }
```
```java
// 코드블럭({})을 제거하고 리턴문 대신 식으로 변경할 경우, 식의 끝에 세미콜론(;)을 반드시 제거
(int a, int b) -> a > b ? a : b // OK
(int a, int b) -> a > b ? a : b; // Error
```
```java
// return문을 작성할 경우, 코드블럭을 제거할 수 없음
(int a, int b) -> { return a > b ? a : b; }// OK
(int a, int b) -> return a > b ? a : b; // Error
```
```java
// 매개변수의 타입이 추론이 가능한 경우 생략할 수 있음. 단, 하나만 생략할 수 없고, 타입이 있으면 괄호를 생략할 수 없음
(a, b) -> a > b ? a : b // OK
(int a, b) -> a > b ? a : b // Error
int a, int b -> a > b ? a : b // Error
```
```java
// 만약 매개변수가 하나일 경우, 괄호를 생략할 수 있음, 단 타입을 생략해야함
(int a) -> a * a // OK
a -> a * a // OK
int a -> a * a // Error
```

#### # 람다식 변환 예제 1
```java
int max(int a, int b) {
  return a > b ? a : b;
}

(int a, int b) -> { return  a > b ? a : b; }

(int a, int b) -> a > b ? a : b

(a, b) -> a > b ? a : b
```

#### # 람다식 변환 예제 2
```java
int printVar(String name, int i) {
  System.out.println(name + "=" + i);
}

(String name, int i) -> { System.out.println(name + "=" + i); }

(String name, int i) -> System.out.println(name + "=" + i)

(name, i) -> System.out.println(name + "=" + i)
```

#### # 람다식 변환 예제 3
```java
int squre(int x) {
  return x * x;
}

(int x) -> { return x * x; }

(int x) -> x * x

x -> x * x
```

#### # 람다식 변환 예제 4
```java
int roll() {
  return (int) (Math.random()) * 6);
}

() -> { return (int) (Math.random() * 6); }

() -> (int) (Math.random() * 6)
```

#### # 람다식 변환 예제 5
```java
int sumArr(int[] arr) {
  int sum = 0;
  for(int i : arr)
      sum += i;
  return sum;
}

(int[] arr) -> {
  int sum = 0;
  for(int i : arr)
      sum += i;
  return sum;
}
```

## 3. 함수형 인터페이스(functional interface)

#### # 익명클래스와 함수형 인터페이스

자바에서는 모든 메서드는 클래스 내에 포함되어 있어야 하는데 람다식은 어떤 클래스에 포함되는 것일까? 람다식이 메서드와 동등한 것처럼 설명했지만, 사실 **람다식은 익명클래스의 객체와 동등하다.**
```java
// 람다식
(int a, int b) -> a > b ? a : b

// 익명클래스 객체
new Object() {
  int max(int a, int b) {
    return  a > b ? a : b;
  }
}
```

그렇다면 람다식으로 정의된 익명 객체의 메서드를 어떻게 호출할 수 있을까? 이미 알고 있듯이 참조변수가 있어야 객체의 메서드를 호출할 수 있으므로 **익명객체의 주소를 `f`라는 참조변수에 저장한다.**
```java
Type f = (int a, int b) -> a > b ? a : b;
```

참조변수 `f`의 타입은 어떤 것이야할까? 참조형이기때문에 **클래스 또는 인터페이스가 가능하다**. 그리고 **람다식과 동등한 메서드가 정의되어 있는 것** 이어야 한다.
```java
interface MyFunction {
  public abstract max(int a, int b);
}
```

이제 이 인터페이스를 구현한 익명 클래스의 객체는 아래의 코드와 같이 생성을 할 수 있다.
```java
MyFunction f = new MyFunction() {
  public int max(int a, int b) {
    return a > b ? a : b;
  }
};

int big = f.max(5, 3); // 익명 객체의 메서드 호출
```

`MyFunction`인터페이스에 정의된 메서드 `max()`는 람다식 `(int a, int b) -> a > b ? a : b`과 메서드 선언부가 일치하기 때문에 위 코드의 익명 객체를 람다식으로 대체할 수 있다.
```java
MyFunction f = (int a, int b) -> a > b ? a : b // 익명 객체를 람다식으로 교체

int big = f.max(5, 3); // 익명 객체의 메서드를 호출
```

`MyFunction`인터페이스를 구현한 익명 객체를 람다식으로 대체가 가능한 이유는, 람다식도 실제로 익명객체이고, `MyFunction`인터페이스를 구현한 익명 객체의 메서드 `max()`와 람다식의 매개변수 타입과 갯수, 반환값이 일치하기 때문이다.


```java
@FunctionalInterface
interface MyFunction { // 함수형 인터페이스 MyFunction을 정의
  public abstract max(int a, int b);
}
```
지금까지 살펴본 것처럼 인터페이스를 통해 람다식을 다룰수 있고, 이러한 인터페이스를 **함수형 인터페이스(Functional interface)** 라고 부른다. **함수형 인터페이스는 오직 하나의 추상메서드만 정의되어 있어야 한다는 제약이 있다.** 그 이유는 람다식과 인터페이스의 메서드가 1:1로 연결되어야 하기 때문이다. 반면에 `static`메서드와 `default`메서드의 갯수에는 제약이 없다.

기존에는 아래의 코드 첫번째와 같이 인터페이스의 메서드 하나를 구현하는데도 복잡하게 해야 했다. 하지만 아래의 2번째 코드와 같이 람다식으로 처리하면 간단하게 처리를 할 수 있게 된다.
```java
List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");

Collections.sort(list, new Comparator<String>()) {
  public int compare(String s1, String s2) {
    return s2.compareTo(s1);
  }
}
```
```java
List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");

// 람다식으로 처리
Collections.sort(list, (s1, s2) -> s2.compareTo(s1));
```

#### # 함수형 인터페이스 타입의 매개변수
함수형 인터페이스 `MyFunction`이 아래의 첫번째 코드와 같이 정의가 되있고, 두번째 코드와 같이 매서드의 매개변수가 `MyFunction`타입이면, 이 메서드를 호출할 때 람다식을 참조하는 참조변수를 매개변수로 지정해야 한다.
```java
@FunctionalInterface
interface MyFunction {  // 함수형 인터페이스
  void myMethod(); // 추상메서드
}
```
```java
void aMethod(MyFunction f) { // 매개변수 타입이 함수형 인터페이스인 메서드
  f.myMethod(); // 함수형 인터페이스 MyFunction에 정의된 메서드 호출
}

// 람다식을 참조하는 참조변수 f
MyFunction f = () -> System.out.println("myMethod()");

// 람다식을 참조하는 참조변수를 인자로 넣어주고, 메서드 호출
aMethod(f);

// 또는 참조변수 없이 직접 람다식을 매개변수로 지정
aMethod(() -> System.out.println("myMethod()"));
```

#### # 함수형 인터페이스 타입의 반환타입
메서드의 반환타입이 함수형 인터페이스라면, 이 함수형 인터페이스의 추상메서드와 동등한 람다식을 가리키는 참조변수를 반환하거나, 람다식을 직접 반환할 수 있다.
```java
MyFunction myMethod() {
  MyFunction f = () -> {};
  return f;
  //return () -> {};
}
```


**람다식을 참조변수로 다룰수 있다는 것은 메서드를 통해 람다식을 주고 받을 수 있다는 것, 즉 변수처럼 메서드를 주고 받을 수 있다는 것을 의미한다.**

#### # 함수형 인터페이스 예제 1
```java
public class LambdaEx1 {

    // 매개변수 타입이 함수형 인터페이스 MyFunction인 메서드
    static void execute(MyFunction f) {
        f.run();
    }

    // 매개변수 타입이 함수형 인터페이스 MyFunction인 메서드
    static MyFunction getMyFunction() {
        MyFunction f = () -> System.out.println("f3.run()");
        return f;
    }

    public static void main(String[] args) {

        // 람다식으로 MyFunction의 run() 구현
        MyFunction f1 = () -> System.out.println("f1.run()");

        // 익명클래스로 run() 구현
        MyFunction f2 = new MyFunction() {

            public void run() {
                System.out.println("f2.run()");
            }

        };

        MyFunction f3 = getMyFunction();

        f1.run();
        f2.run();
        f3.run();

        execute(f1);

        execute( () -> System.out.println("run()"));

    }
}

@FunctionalInterface
interface MyFunction {
    void run();
}
```
```
f1.run()
f2.run()
f3.run()
f1.run()
run()
```

#### # 람다식의 타입과 형변환
함수형 인터페이스로 람다식을 참조할 수 있는 것일 뿐, 람다식의 타입이 함수형 인터페이스의 타입과 일치하는 것은 아니다. 람다식은 익명 객체이고, 익명 객체는 타입이 없다. 정확히는 타입이 있지만 컴파일러가 임의의 이름을 정하기 때문에 알 수 없는 것이다. 그래서 대입 연산자의 양변의 타입을 일치시키기 위해 아래와 같이 형변환이 필요하다.
```java
MyFunction f = (MyFunction) (() -> {}); // 양변의 타입이 다르기때문에 형변환이 필요, 형변환 생략가능
```

**람다식은 `Object`타입으로 형변환은 할 수없다.** 그럼에도 `Object`타입으로 형변환을 하려면 함수형 인터페이스로 형변환을 먼저 해야한다.
```java
Object obj = (Object) (() -> {}); // Error, 함수형 인터페이스만 형변환 가능
Object obj = (Object) (MyFunction) (() -> {}); // 함수형 인터페이스로 먼저 형변환, Object타입으로 다시 형변환
String str = ((Object) (MyFunction) (() -> {})).toString();
```

#### # 람다식의 형변환 예제 1
```java
public class LambdaEx2 {
    public static void main(String[] args) {
        MyFunction2 f = () -> {};
        Object obj = (MyFunction2) (() -> {});
        String str = ((Object) (MyFunction2) (() -> {})).toString();

        System.out.println(f);
        System.out.println(obj);
        System.out.println(str);

        //System.out.println(()->{}); // Error, 람다식은 Object타입으로 형변환 불가
        System.out.println((MyFunction2) (()->{}));
        //System.out.println((MyFunction2) (()->{}).toString()); // Error
        System.out.println((Object) ((MyFunction2) (() -> {})).toString());

    }
}

@FunctionalInterface
interface MyFunction2 {
    void myMethod();
}
```
```
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$1/1078694789@7cca494b
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$2/1023892928@7ba4f24f
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$3/558638686@448139f0
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$4/999966131@7699a589
com.doubles.standardofjava.ch14_lambda_stream.part01_lambda.LambdaEx2$$Lambda$5/1480010240@4dd8dc3
```
