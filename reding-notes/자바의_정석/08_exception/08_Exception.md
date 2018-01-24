# 예외처리(Exception Handling)

## 1. 예외처리(Exception Handling)

### 1.1 예외처리
- 컴파일 에러 : 컴파일 시 발생하는 에러
- 런타임 에러 : 실행시 발생하는 에러
- 논리적 에러 : 실행은 되지만, 의도와 다르게 동작하는 것
- Error VS, Exception
  - Error : 프로그램 코드에 의해서 수습될 수 없는 심각한 오류
  - Exception : 프로그램 코드에 의해서 수습될 수 있는 다소 미약한 오류

### 1.2 예외 클래스의 계층구조
- 예외 클래스의 계층도

- Exception클래스와 RuntimeException클래스 중심의 상속 계층도
  - 예외 클래스를 크게 두가지 그룹으로 나눌 수 있다.
    - RuntimeException클래스와 그 자손클래스 : 프로그래머의 실수로 발생하는 예외
    - Exception클래스와 그 자손클래스 : 사용자의 실수와 같은 외적인 요인에 의해 발생하는 예외

### 1.3 예외처리하기 : `try-catch`문
- 예외처리(Exception Handling)이란?
  - 프로그램 실행시 발생할 수 있는 예외의 발생에 대비한 코드를 작성하는 것
  - 프로그램의 비정상적인 종료를 막고, 정상적인 실행상태를 유지하는 것
- `try-catch`구조
  ```java
  try {
    // 예외가 발생할 가능성이 있는 코드
  } catch (Exception1 e1) {
    // Exception1이 발생했을 경우, 처리를 위한 코드
  } catch (Exception2 e2) {
    // Exception2이 발생했을 경우, 처리를 위한 코드
  } catch (Exception3 e3) {
    // Exception3이 발생했을 경우, 처리를 위한 코드
  }
  ```
  ```java
  pubilc static void main(String[] agrs) {
    try {
      try {} catch (Exception e) {} // try블럭 안에 try-catch문 사용가능
    } catch (Exception1 e1) {
      try {} catch (Exception e) {} // catch블럭 안에 try-catch문 사용가능
    } catch (Exception2 e2) {
      try {} catch (Exception2 e2) {} // 에러, 변수 e2중복선언
    }
  }
  ```
- 예외처리 예제) : 100을 0~9사이의 임의의 정수로 나눈 값을 출력을 10번 반복수행
  ```java
  pulic static void main(String[] args) {
    int number = 100;
    int result = 0;

    for (int i = 0; i < 10; i++) {
      result = number / (int) (Math.random() * 10);
      System.out.println(result);
    }
  }
  ```
  ```
  Exception in thread "main" java.lang.ArithmeticException: / by zero
  at com.doubles.standardofjava.exceptions.ExceptionEx02.main(ExceptionEx02.java:10)
  ```

  > 정수를 0으로 나누려했기때문에 ArithmeticException이 발생하여 비정상적인 종료

  ```java
  public static void main(String[] args) {
    int number = 100;
    int result = 0;

    for (int i = 0; i < 10; i++) {
        try {
            result = number / (int) (Math.random() * 10);
            System.out.println(result);
        } catch (ArithmeticException e) {
            System.out.println("0 <- 예외처리");
        }
    }
  }
  ```
  ```
  12
  100
  16
  33
  0 <- 예외처리
  25
  0 <- 예외처리
  0 <- 예외처리
  14
  33
  ```
  > 예외가 발생할 수 있는 지점에 예외처리하여 비정상적인 종료가 되지 않고 끝까지 프로그램이 수행되도록 처리

### 1.4 `try-catch`문에서의 흐름
- try블럭 내에서 예외가 발생한 경우
  ```java
  public static void main(String[] args) {
    System.out.println(1);
    System.out.println(2);
    try {
        System.out.println(3);
        System.out.println(0/0);
        System.out.println(4);
    } catch (ArithmeticException e) {
        System.out.println(5);
    }
    System.out.println(6);
  }
  ```
  - 발생한 예외와 일치하는 catch블럭이 존재하는지 확인
  - 일치하는 catch블럭 내의 문장들을 수행하고, try-catch문을 빠져나와 그 다음 문장을 수행
  - 만약 일치하는 catch블럭을 찾지 못한 경우 예외처리는 되지 않음
- try블럭 내에서 예외가 발생하지 않은 경우
  ```java
  public static void main(String[] args) {
      System.out.println(1);
      System.out.println(2);
      try {
          System.out.println(3);
          System.out.println(4);
      } catch (ArithmeticException e) {
          System.out.println(5);
      }
      System.out.println(6);
  }
  ```
  - catch블럭을 거치지 않고 try-catch문을 빠져나와 수행을 계속

### 1.5 예외 발생과 catch블럭
