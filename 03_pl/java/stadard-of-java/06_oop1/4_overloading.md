본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. 오버로딩이란?

메서드도 변수와 마찬가지로 같은 클래스 내에서 서로 구분할 수 있어야 하기 때문에 서로 다른 이름을 가져야만 한다. 하지만 한 클래스 내에서 사용하려는 이름과 동일한 메서드가 존재하더라도 매개변수의 타입과 갯수가 다르다면 같은 이름의 메서드를 정의할 수 있다.

즉, 한 문장으로 정리해보면 **한 클래스 내에 같은 이름의 메서드를 여러 개 정의하는 것** 을 **메서드 오버로딩(method overloading) 또는 오버로딩(overloading)** 이라고 한다.

## 2. 오버로딩의 조건

같은 이름의 메서드를 정의한다고 해서 무조건 오버로딩이 될 순 없다. 오버로딩이 성립되기 위해서는 아래와 같은 조건이 성립되어야 한다.

- **메서드 이름이 같아야 한다.**
- **매개변수의 갯수, 타입이 달라야 한다.**

그리고 주의해야 할 것은 **반환타입은 오버로딩을 구현하는데 아무런 영향을 미치지 못한다.**

## 3. 오버로딩의 예

오버로딩의 가장 대표적인 예는 `printStream`클래스의 `println()`이다. 아래의 코드에서처럼 어떤 종류의 매개변수를 지정하더라도 출력할 수 있게 10개의 오버로딩된 메서드를 정해놓고 있다. 그래서 매개변수에 어떤 인자를 지정하더라도 출력을 할 수 있는 것이다.

```java
void println();
void println(boolean x);
void println(char x);
void println(char[] x);
void println(double x);
void println(float x);
void println(int x);
void println(long x);
void println(Object x);
void println(String x);
```

## 4. 오버로딩의 장점

그렇다면 오버로딩을 구현함으로써 얻는 이득은 무엇일까?

만약 오버로딩이 되지 않는다면 위에서 본 `println()`들이 매개변수에 따라 메서드의 이름이 아래의 코드처럼 달라지게 될 것이다. 이 메서드들은 근본적으로 동일한 기능을 수행하지만 서로 다른 이름을 가져야만 하기 때문에 메서드를 작성할 때 이름을 일일히 작성해줘야하는 번거로움이 생긴다. 그리고 메서드를 사용할 때도 매개변수에 따라 일일히 이름을 기억해야만 하는 불편함이 생긴다.

```java
void println();
void printlnBoolean(boolean x);
void printlnChar(char x);
//...
void printlnLong(long x);
void printlnObject(Object x);
void printlnString(String x);
```

하지만 오버로딩을 함으로써 동일한 기능을 가진 여러 메서드들이 `println()`이라는 하나의 이름으로 정의될 수 있기때문에 메서드의 이름을 절약할 수 있다. 그리고 메서드를 사용하는 입장에서는 메서드의 이름이 동일하고, 매개변수가 다르기 때문에 같은 기능을 수행한다는 것을 유추할 수도 있다.



#### 오버로딩 예제1 : 오버로딩 구현
지금까지 오버로딩에 대해서 간단히 정리해보았는데 예제를 통해 오버로딩을 구현해보자.

```java
public class OverloadingTest {
    public static void main(String[] args) {

        MyMath3 myMath3 = new MyMath3();
        System.out.println("myMath3.add(3, 3) => result : " + myMath3.add(3, 3));
        System.out.println("myMath3.add(3L, 3) => result : " + myMath3.add(3L, 3));
        System.out.println("myMath3.add(3, 3L) => result : " + myMath3.add(3, 3L));
        System.out.println("myMath3.add(3L, 3L) => result : " + myMath3.add(3L, 3L));

        int[] a = {100, 200, 300};
        System.out.println("myMath3.add(a) => result : " + myMath3.add(a));
    }
}

class MyMath3 {

    int add(int a, int b) {
        System.out.print("int add(int a, int b) - ");
        return a + b;
    }

    long add(int a, long b) {
        System.out.print("long add(int a, long b) - ");
        return a + b;
    }

    long add(long a, int b) {
        System.out.print("long add(long a, int b) - ");
        return a + b;
    }

    long add(long a, long b) {
        System.out.print("long add(long a, long b) - ");
        return a + b;
    }

    int add(int[] a) {
        System.out.print("int add(int[] a) - ");
        int result = 0;
        for (int i = 0; i < a.length; i++) {
            result += a[i];
        }
        return result;
    }

}
```
```
int add(int a, int b) - myMath3.add(3, 3) => result : 6
long add(long a, int b) - myMath3.add(3L, 3) => result : 6
long add(int a, long b) - myMath3.add(3, 3L) => result : 6
long add(long a, long b) - myMath3.add(3L, 3L) => result : 6
int add(int[] a) - myMath3.add(a) => result : 600
```

## 5. 가변인자(varargas)와 오버로딩

기존에는 메서드의 매개변수의 갯수가 고정적이었지만, JDK1.5부터는 동적으로 매개변수를 지정해 줄 수 있게 가변인자라는 것이 생겼다.

```java
Type methodName(Type... variableName) {
  //...
}
```

가변인자는 위와 같이 **`Type... variableName`과 같은 형식으로 선언** 하며, `PrintStream`클래스의 `printf()`가 대표적인 예이다. 만약 아래의 코드와 같이 가변인자 외에도 매개변수가 있다면, 가변인자는 가장 마지막에 선언해야한다. 그렇지 않으면 컴파일 에러가 발생하는데 가변인자인지 아닌지를 구분할 방법이 없기 때문이다.

```java
// 가변인자는 항상 마지막에 선언, 그렇지 않으면 컴파일 에러 발생
public PrintStream printf(String format, Object... args) {
  // ...
}
```

#### 가변인자 예제 1 : 여러문자 합치기

만약 여러 문자열을 하나로 결합하여 반환하는 기능을 가진 메서드 `concatenate()`를 작성한다면, 아래와 같이 작성을 해줘야 할 것이다.

```java
String concatenate(String s1, String s2) {}
String concatenate(String s1, String s2, String s3) {}
String concatenate(String s1, String s2, String s3, String s4) {}
```

하지만 가변인자를 사용하면 위의 코드를 한줄로 줄일 수 있다.

```java
String concatenate(String... str) {}
```

이 메서드를 호출할 때는 아래와 같이 인자의 갯수를 가변적으로 처리할 수 있다.

```java
System.out.println(concatenate());  // 인자 없음
System.out.println(concatenate("a")); // 인자 1개
System.out.println(concatenate("a", "b")); // 인자 2개
System.out.println(concatenate(new String[]{"a", "b"})); // 인자 배열
```

그렇다면 가변인자와 매개변수의 타입을 배열로 선언하는 것과 차이는 무엇일까? 아래의 코드를 보면 배열타입의 매개변수의 경우 반드시 인자를 지정해줘야하기 때문에 위의 코드처럼 **인자를 생략할 수가 없다.** 그래서 `null`이나 `0`인 배열을 인자로 지정해줘야하는 불편함이 있다.

```java
String concatenate(String[] str) {}
String result = concatenate(new String[0]); // 인자를 배열로 지정
String result = concatenate(null); // 인자를 null로 지정
String result = concatenate(); // 에러, 인자가 반드시 필요하다.
```

가변인자를 오버로딩할 때 주의해야할 점이 있는데, 아래의 예제를 통해 알아보자.
```java
public class VarArgsEx {
    public static void main(String[] args) {
        String[] strArr = {"100", "200", "300"};
        System.out.println(concatenate("", "100", "200", "300"));
        System.out.println(concatenate("-", strArr));
        System.out.println(concatenate(",", new String[]{"1", "2", "3"}));
        System.out.println("["+concatenate(",", new String[0])+ "]");
        System.out.println("["+concatenate(",")+ "]");
    }

    static String concatenate(String delim, String... args) {
        String result = "";
        for (String str : args) {
            result += str + delim;
        }

        return result;
    }

//
//    static String concatenate(String... args) {
//        return concatenate("", args);
//    }
//
}
```
```
100200300
100-200-300-
1,2,3,
[]
[]
```

`concatenate()`는 매개변수로 입력된 문자열에 구분자를 결합하여 반환한다. 가변인자를 매개변수로 선언했기 때문에 문자열의 갯수의 제한없이 매개변수로 지정할 수 있다. 그리고 `concatenate()`의 또다른 메서드, 오버로딩된 메서드가 주석처리가 되어있는데 이 주석을 풀고 컴파일하면 아래와 같은 에러가 발생한다.

```
Error:(6, 28) java: reference to concatenate is ambiguous both method concatenate(java.lang.String,java.lang.String...)
in com.doubles.standardofjava.ch06_oop1.part04_overloading.VarArgsEx and method concatenate(java.lang.String...)
in com.doubles.standardofjava.ch06_oop1.part04_overloading.VarArgsEx match
```

에러의 내용을 살펴보면 두 오버로딩된 메서드가 구분되지 않아서 발생한 에러이다. 가변인자를 선언한 메서드를 오버로딩하면 메서드를 호출했을 때 위와 같이 구별되지 못하는 경우가 발생하기 쉽기때문에 주의를 해야한다. 가능하다면 가변인자를 사용한 메서드는 오버로딩을 하지 않는 것이 바람직하다.
