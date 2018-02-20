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

#### 메서드를 람다식으로 변환 과정 및 주의 사항
```java
// 메서드
int max(int a, int b) { return a > b ? a : b; }

// 리턴타입과 메서드명 제거, -> 추가
(int a, int b) -> { return a > b ? a : b; }

// 코드블럭({})을 제거하고 리턴문 대신 식으로 변경할 경우, 식의 끝에 세미콜론(;)을 반드시 제거
(int a, int b) -> a > b ? a : b // OK
(int a, int b) -> a > b ? a : b; // Error

// return문을 작성할 경우, 코드블럭을 제거할 수 없음
(int a, int b) -> { return a > b ? a : b; }// OK
(int a, int b) -> return a > b ? a : b; // Error

// 매개변수의 타입이 추론이 가능한 경우 생략할 수 있음. 단, 하나만 생략할 수 없고, 타입이 있으면 괄호를 생략할 수 없음
(a, b) -> a > b ? a : b // OK
(int a, b) -> a > b ? a : b // Error
int a, int b -> a > b ? a : b // Error

// 만약 매개변수가 하나일 경우, 괄호를 생략할 수 있음, 단 타입을 생략해야함
(int a) -> a * a
a -> a * a // OK
int a -> a * a // Error
```

#### 람다식 변환 예제 1
```java
int max(int a, int b) {
  return a > b ? a : b;
}

(int a, int b) -> { return  a > b ? a : b; }

(int a, int b) -> a > b ? a : b

(a, b) -> a > b ? a : b
```

#### 람다식 변환 예제 2
```java
int printVar(String name, int i) {
  System.out.println(name + "=" + i);
}

(String name, int i) -> { System.out.println(name + "=" + i); }

(String name, int i) -> System.out.println(name + "=" + i)

(name, i) -> System.out.println(name + "=" + i)
```

#### 람다식 변환 예제 3
```java
int squre(int x) {
  return x * x;
}

(int x) -> { return x * x; }

(int x) -> x * x

x -> x * x
```

#### 람다식 변환 예제 4
```java
int roll() {
  return (int) (Math.random()) * 6);
}

() -> { return (int) (Math.random() * 6); }

() -> (int) (Math.random() * 6)
```

#### 람다식 변환 예제 5
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
MyFunction f = new Function() {
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


## 4. `java.util.function`패키지

## 5. `Function`의 합성과 `Predicate`의 결합

## 6. 메서드 참조
