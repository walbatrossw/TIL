본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. 예외처리(Exception Handling)

##### 에러의 종류
- 컴파일 에러 : 컴파일 시 발생하는 에러
- 런타임 에러 : 실행시 발생하는 에러
- 논리적 에러 : 실행은 되지만, 의도와 다르게 동작하는 것

##### Error와 Exception 차이점
- Error : 프로그램 코드에 의해서 수습될 수 없는 심각한 오류
- Exception : 프로그램 코드에 의해서 수습될 수 있는 다소 미약한 오류


## 2. 예외 클래스의 계층구조

##### 예외 클래스의 계층도
![](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%9E%90%EB%B0%94%EC%9D%98_%EC%A0%95%EC%84%9D/08_exception/pic/exception/exception_layer_diagram_1.png?raw=true)

##### `Exception`클래스와 `RuntimeException`클래스 중심의 상속 계층도
![](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%9E%90%EB%B0%94%EC%9D%98_%EC%A0%95%EC%84%9D/08_exception/pic/exception/exception_layer_diagram_2.png?raw=true)
- `RuntimeException`클래스와 그 자손클래스 : 프로그래머의 실수로 발생하는 예외
- `Exception`클래스와 그 자손클래스 : 사용자의 실수와 같은 외적인 요인에 의해 발생하는 예외


## 3. 예외처리하기 : `try-catch`문

##### 예외처리(Exception Handling)이란?
- 프로그램 실행시 발생할 수 있는 예외의 발생에 대비한 코드를 작성하는 것
- 프로그램의 비정상적인 종료를 막고, 정상적인 실행상태를 유지하는 것

##### `try-catch`구조
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

##### 예외처리 예제 : 100을 0~9사이의 임의의 정수로 나눈 값을 출력을 10번 반복수행
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

정수를 0으로 나누려했기때문에 ArithmeticException이 발생하여 비정상적으로 종료된다.

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

예외가 발생할 수 있는 지점에 예외처리하여 비정상적인 종료가 되지 않고 끝까지 프로그램이 수행되도록 코드를 작성해주었다.


## 4. `try-catch`문에서의 흐름

##### `try`블럭 내에서 예외가 발생한 경우
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
```
1
2
3
5
6
```
- 발생한 예외와 일치하는 `catch`블럭이 존재하는지 확인한다
- 일치하는 `catch`블럭 내의 문장들을 수행하고, `try-catch`문을 빠져나와 그 다음 문장을 수행된다.
- 만약 일치하는 `catch`블럭을 찾지 못한 경우 예외처리는 되지 않는다.

##### `try`블럭 내에서 예외가 발생하지 않은 경우
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
```
1
2
3
4
6
```
- `catch`블럭을 거치지 않고 `try-catch`문을 빠져나와 수행을 계속한다.

## 5. 예외 발생과 catch블럭

##### catch블럭의 참조변수
- `catch`블럭에서 `()`괄호내에 처리하려는 예외와 같은 타입의 참조변수를 선언해야함.
- 예외가 발생하면 해당 예외에 해당하는 클래스의 인스턴스 생성
- 여러 개의 `catch`블럭이 있을 경우 각 `catch`블럭의 `()`에 선언된 참조변수와 생성된 인스턴스의 `instanceof`연산자를 이용하여 검사
- 검사 결과가 `true`이면 `{}`내의 문장을 수행, `false`이면 문장을 실행하지 않음
- 모든 예외 클래스는 `Exception`클래스의 자손이기때문에 `Exception`으로 선언해 놓으면 어떤 종류의 예외가 발생하더라도 이 곳에서 처리

```java
public static void main(String[] args) {
    System.out.println(1);
    System.out.println(2);
    try {
        System.out.println(3);
        System.out.println(0/0);
        System.out.println(4);
    } catch (Exception e) { // ArithmeticException 대신 Exception
        System.out.println(5);
    }
    System.out.println(6);
}
```
```
1
2
3
5
6
```
```java
public static void main(String[] args) {
    System.out.println(1);
    System.out.println(2);
    try {
        System.out.println(3);
        System.out.println(0/0);
        System.out.println(4);
    } catch (ArithmeticException ae) {  // ArithmeticException 처리
        if (ae instanceof ArithmeticException)
            System.out.println("true");
        System.out.println("ArithmeticException");
    } catch (Exception e) { // ArithmeticException을 제외한 나머지 예외 처리
        System.out.println("Exception");
    }
    System.out.println(6);
}
```
```
1
2
3
true
ArithmeticException
6
```

##### `printStackTrace()`, `getMessage()` 메서드

예외가 발생했을 때 생성되는 예외 클래스의 인스턴스에는 발생한 **예외에 대한 정보** 가 담겨져있다.

- `printStackTrace()` - 예외발생 당시의 호출스택에 있었던 메서드의 정보와 예외메시지를 화면 출력
- `getMessage()` - 발생한 예외클래스의 인스턴스에 저장된 메시지를 얻을 수 있음

```java
} catch (ArithmeticException ae) {
  ae.printStackTrace();
}
```
```
1
java.lang.ArithmeticException: / by zero
2
	at com.doubles.standardofjava.exceptions.ExceptionEx08.main(ExceptionEx08.java:9)
3
6
```

##### 멀티 `catch`블럭

jdk1.7부터 여러 `catch`블럭을 `|`기호를 통해 하나의 `catch`블럭으로 합칠 수 있게 되었다.

```java
try {
  //...
} catch (Exception1 e1) {
  e1.printStackTrace();
} catch (Exception2 e2) {
  e2.printStackTrace();
}
```

```java
try {
  //...
} catch (Exception1 | Exception2 e) {
  e.printStackTrace();
}
```

- 멀티 `catch`블럭의 `|`기호로 연결된 클래스가 조상(`ParentException`)과 자손(`ChildException`)의 관계에 있다면 컴파일 에러발생(굳이 그렇게 할 필요가 없기 때문)
- 여러 예외를 하나의 참조변수로 선언한 것이기때문에 실제로 어떤 예외가 발생했는지 알 수 없다. 그래서 공통 분모인 조상 예외클래스에 선언된 멤버만을 사용할 수 있다.


## 6. 예외 발생시키기

##### 예외를 발생시키는 법
```java
Exception e = new Exception("예외발생"); // 예외 객체 생성
throw e;  // 예외발생
```
```java
public static void main(String[] args) {
  try {
    Exception e = new Exception("예외 발생!");
    throw e;
  } catch (Exception e) {
    System.out.println("에러메시지 : "+e.getMessage());
    e.printStackTrace();
  }
  System.out.println("정상종료...");
}
```
```
에러메시지 : 예외 발생!
java.lang.Exception: 예외 발생!
	at com.doubles.standardofjava.exceptions.ExceptionEx09.main(ExceptionEx09.java:6)
정상종료...
```

##### `checked`예외 VS `unchecked`예외
**`checked` : 예외처리를 확인하는 `Exception`클래스(`RuntimeException`클래스들을 제외한 클래스들)**
```java
public static void main(String[] args) {
  throw new Exception();
}
```
```
Error:(5, 9) java: unreported exception java.lang.Exception; must be caught or declared to be thrown
```
- 컴파일이 완료되지 않음.
- 예외처리가 되어야할 부분에 예외처리가 되지 않았기 때문에

**`unchecked` : 예외처리를 확인하지 않는 `Exception`클래스(`RuntimeException`클래스들)**
```java
public static void main(String[] args) {
  throw new RuntimeException();
}
```
```
Exception in thread "main" java.lang.RuntimeException
	at com.doubles.standardofjava.exceptions.ExceptionEx11.main(ExceptionEx11.java:5)
```
- 예외처리가 되지 않았음에도 불구하고 성공적으로 컴파일은 진행이 됨.


## 7 메서드에 예외 선언하기

##### 메서드 예외 선언하는 법

메서드에 예외를 선언하려면, 메서드 선언부에 `throws`키워드를 사용해 메서드 내에 발생할 수 있는 예외를 작성, 예외가 여러개일 경우 `,`로 구분한다.
```java
void method() throws Exception1, Exception2, ..., ExceptionN {
  // 메서드 내용
}
```

모든 예외의 최고 조상인 `Exception`클래스를 메서드에 선언하면, 모든 종류의 예외가 발생할 가능성이 존재한다.
```java
void method() throws Exception {
  // 메서드 내용
}
```
- 이렇게 예외를 선언하면, 이 예외 뿐만아니라 그 자손타입의 예외까지도 발생할 수 있음
- 오버라이딩 할 때는 단순히 선언된 예외의 개수가 아니라 상속관계까지도 고려해야함

##### 메서드 예외 선언하기 예제 1 : 예외만 전달

예외를 메서드의 `throws`에 명시하는 것은 예외를 처리하는 것이 아니라 자신을 호출한 메서드에게 예외를 전달하여 예외처리를 떠맡기는 것으로 예외를 전달받은 메서드는 또다시 메서드를 호출한 메서드에게 예외를 전달할 수 있다.

```java
public static void main(String[] args) throws Exception {
  method1();
}

static void method1() throws Exception {
  method2();
}

static void method2() throws Exception {
  throw new Exception();
}
```
```
Exception in thread "main" java.lang.Exception
	at com.doubles.standardofjava.exceptions.ExceptionEx12.method2(ExceptionEx12.java:11)
	at com.doubles.standardofjava.exceptions.ExceptionEx12.method1(ExceptionEx12.java:8)
	at com.doubles.standardofjava.exceptions.ExceptionEx12.main(ExceptionEx12.java:5)
```
- 예외가 발생했을 때, 3개의 모든 메서드가 호출스택에 있었음.
- 예외가 발생한 곳은 제일 윗줄에 있는 `method2()`라는 것
- `main()`가 `method1()`을, `method1()`은 `method2()`를 호출했다는 것을 알 수 있음.
- 예외처리를 해주지 않았기 때문에 호출한 메서드에 계속 예외를 전달하고 `main()`이 종료되면서 프로그램이 비정상적으로 종료됨

##### 메서드 예외 선언하기 예제 2 : 예외처리(`method1()`에서)

```java
public static void main(String[] args) {
  method1();
}

static void method1() {
  try {
      throw new Exception();
  } catch (Exception e) {
      System.out.println("method1()에서 예외처리");
      e.printStackTrace();
  }
}
```

```
method1()에서 예외처리
java.lang.Exception
	at com.doubles.standardofjava.exceptions.ExceptionEx13.method1(ExceptionEx13.java:10)
	at com.doubles.standardofjava.exceptions.ExceptionEx13.main(ExceptionEx13.java:5)
```

- `method1()`에서 예외가 처리되면 `main()`에서는 예외가 발생했다는 것을 알지 못함.

##### 메서드 예외 선언하기 예제 3 : 예외처리(`main()`에서)

```java
public static void main(String[] args) {
  try {
      method1();
  } catch (Exception e) {
      System.out.println("main()에서 예외처리");
      e.printStackTrace();
  }
}

static void method1() throws Exception {
  throw new  Exception();
}
```

```
java.lang.Exception
	at com.doubles.standardofjava.exceptions.ExceptionEx14.method1(ExceptionEx14.java:14)
	at com.doubles.standardofjava.exceptions.ExceptionEx14.main(ExceptionEx14.java:6)
main()에서 예외처리
```

- `method1()`에서 예외를 선언하고, 호출한 `main()`에게 예외를 전달하여 `try-catch`문으로 예외를 처리

##### 메서드 예외 선언하기 예제 3 : 예외가 발생한 메서드에서 직접 예외를 처리

```java
public static void main(String[] args) {
  String fileName = "";
  File file = createFile(fileName);
  System.out.println(file.getName() + "파일이 성공적으로 생성됨.");
}

static File createFile(String fileName) {
  try {
      if (fileName == null || fileName.equals(""))
        throw new Exception("파일이름이 유효하지 않음.");
  } catch (Exception e) {
      fileName = "제목없음.txt";
  } finally {
      File file = new File(fileName);
      createNewFile(file);
      return file;
  }
}

static void createNewFile(File file) {
  try {
    file.createNewFile();
  } catch (Exception e) {

  }
}
```
```
제목없음.txt파일이 성공적으로 생성됨.
```
- 예외가 발생한 메서드에 직접 예외를 처리
- `createFile()`를 호출한 `main()`은 예외가 발생한 사실을 모름
- 파일명이 없거나 공백일 경우, 파일명을 `제목없음.txt`로 설정하여 파일을 생성
- `finally`블럭에서는 예외의 발생 여부와 관계 없이 파일을 생성하도록 처리

##### 메서드 예외 선언하기 예제 4 : 호출한 메서드에서 예외를 전달받아 처리
```java
public static void main(String[] args) {
  String fileName = "";
  try {
      File file = createFile(fileName);
      System.out.println(file.getName() + "파일이 성공적으로 생성됨.");
  } catch (Exception e) {
      System.out.println(e.getMessage() + "파일명 재확인 요망.");
  }
}

static File createFile(String fileName) throws Exception {
    if (fileName == null || fileName.equals(""))
        throw new Exception("파일이름이 유효하지 않음. ");
    File file = new File(fileName);
    file.createNewFile();
    return file;
}
```
- 예외가 발생한 `createFile()`메서드에서 예외를 처리하지 않고, 호출한 `main()`에 예외발생을 전달하여 파일명을 확인하도록 처리

##### 메서드 예외의 처리 방법
- 예외가 발생할 경우 메서드 내에서 자체적으로 처리해도 되는 것은 메서드 내에서 `try-catch`문을 사용해 처리
- 메서드에 호출한 값을 다시 넘겨받아야 할 경우(메서드 내에서 자체적으로 해결이 안될 경우) 예외를 메서드에 선언해서 호출한 메서드에서 처리



## 8 `finally`블럭

##### `finally`블럭의 사용법
`finally`블럭은 `try-catch`문과 함께 예외의 발생여부와 관계없이 실행되어야할 코드를 포함시킬 목적으로 사용되며, `try-catch`문의 끝에 덧붙여 사용할 수 있다.

```java
try {
  // 예외 가능성이 있는 코드
} catch (Exception e) {
  // 예외 처리 코드
} finally {
  // 예외 발생여부와 관계없이 실행되는 코드
}
```

##### `finally` 블럭 예제 1, 2 : 동일로직의 중복제거
```java
public static void main(String[] args) {
  try {
      startInstall(); // 설치
      copyFiles(); // 파일복사
      deleteTempFiles(); // 삭제
  } catch (Exception e) {
      e.printStackTrace();
      deleteTempFiles(); // 삭제
  }
}

static void startInstall() {};
static void copyFiles() {};
static void deleteTempFiles() {};
```
```java
public static void main(String[] args) {
  try {
      startInstall(); // 설치
      copyFiles(); // 파일복사
  } catch (Exception e) {
      e.printStackTrace();
  } finally {
      deleteTempFiles(); // 삭제
  }
}

static void startInstall() {};
static void copyFiles() {};
static void deleteTempFiles() {};
```
- 위, 아래 예제는 동일하게 설치를 시작하고, 파일을 복사, 임시파일을 삭제하는 일을 수행
- 만약 예외가 발생하면 임시파일을 삭제
- 설치가 완료되면 임시파일을 삭제
- 예외의 발생여부와 관계없이 삭제처리를 수행
- 그렇다면 `finally`블럭에 파일을 삭제하는 코드를 작성함으로써 예외의 발생여부와 관계없이 중복된 코드를 줄일 수 있음

##### `finally` 블럭 예제 3
```java
public static void main(String[] args) {
  method1();
  System.out.println("method1() ended and returned main() from method1()");
}

static void method1() {
  try {
      System.out.println("method1() called...");
      return;
  } catch (Exception e) {
      e.printStackTrace();
  } finally {
      System.out.println("method1() finally block executed...");
  }
}
```
```
method1() called...
method1() finally block executed...
method1() ended and returned main() from method1()
```
- 위 예제의 실행결과를 보면 `try`블럭에서 `return`문이 실행되더라도 `finally`블럭의 문장이 먼저 완료되고 메서드를 종료


## 9 자동 자원 반환 - `try-with-resources`문

##### `try-with-resources`를 사용하는 이유?
JDK1.7부터 `try-with-resources`문이 추가되었는데 이 구문은 주로 입출력(I/O)와 관련된 클래스를 사용할 때 유용, 주로 입출력에 사용되는 클래스의 경우 사용했던 자원을 반환하기 위해서 사용한 뒤 반드시 받아줘야함
```java
try {
  fis = new FileInputStream("score.dat");
  dis = new DateInputStream(fis);
  // ...
} catch(IOException ie) {
  ie.printStackTrace();
} finally {
  dis.close();
}
```
- `DateInputStream`스트림을 통해 파일을 읽는 코드
- 데이터를 읽는 도중 예외가 발생하더라도 `DateInputStream`이 닫히도록 `finally`블럭에서 `close()`를 작성
- 하지만 `close()`에서 예외가 발생할 수 있기 때문에 아래 코드와 같이 작성해야함.

```java
try {
  fis = new FileInputStream("score.dat");
  dis = new DateInputStream(fis);
  // ...
} catch(IOException ie) {
  ie.printStackTrace();
} finally {
  try {
      if(dis != null)
        dis.close();
  } catch (IOException ie) {
    ie.printStackTrace();
  }

}
```
- `finally`블럭 안에 `try-catch`문을 작성해 `close()`에서 발생할 수 있는 예외를 처리하도록 코드를 변경
- 코드가 복잡해져서 보기에도 안 좋을뿐더 `try`블럭과 `finally`블럭에서 모두 예외가 발생하게된다면 `try`블럭의 예외가 무시된다는 문제가 발생함
- 이러한 문제를 개선하기 위해 `try-with-resources`문이 추가됨

```java
try (fis = new FileInputStream("score.dat"); dis = new DateInputStream(fis)) {
  while (true) {
    score = dis.readInt();
    System.out.println(score);
    sum += score;
  }
} catch (EOFException e) {
  System.out.println("점수의 총합 :" + sum +);
} catch (IOException ie) {
  ie.printStackTrace();
}
```
- `try-with-resources`문의 `()`에 객체를 생성하는 문장을 넣으면 이 객체는 `try`블럭을 벗어나는 순간 자동적으로 `close()`가 호출
- `try-with-resources`문에 의해 자동적으로 객체가 `close()`되려면 클래스가 `AutoCloseable`이라는 인터페이스를 구현한 것이어야만 함
  ```java
  public interface AutoCloseable {
    void close() throws Exception;
  }
  ```

##### `try-with-resources` 예제 1
```java
public static void main(String[] args) {
  // close()만 예외 발생
  try (CloseableResource cr = new CloseableResource()) {
      cr.exceptionWork(false);
  } catch (WorkException e) {
      e.printStackTrace();
  } catch (CloseException e) {
      e.printStackTrace();
  }

  System.out.println();

  // close()와 exceptionWork() 예외 발생
  try (CloseableResource cr = new CloseableResource()) {
      cr.exceptionWork(true);
  } catch (WorkException e) {
      e.printStackTrace();
  } catch (CloseException e) {
      e.printStackTrace();
  }

}

class CloseableResource implements AutoCloseable {
  public void exceptionWork(boolean exception) throws WorkException {
    System.out.println("exceptionWork("+exception+") called...");
    if (exception)
      throw new WorkException("WorkException!!!");
  }

  public void close() throws CloseException {
    System.out.println("close() called...");
    throw new CloseException("CloseException!!!");
  }
}

class WorkException extends Exception {
  WorkException(String msg) {
    super(msg);
  }
}

class CloseException extends Exception {
  CloseException(String msg) {
    super(msg);
  }
}
```

```
exceptionWork(false) called...
close() called...

exceptionWork(true) called...
close() called...
com.doubles.standardofjava.exceptions.CloseException: CloseException!!!
	at com.doubles.standardofjava.exceptions.CloseableResource.close(TryWithResourceEx.java:37)
	at com.doubles.standardofjava.exceptions.TryWithResourceEx.main(TryWithResourceEx.java:8)
com.doubles.standardofjava.exceptions.WorkException: WorkException!!!
	at com.doubles.standardofjava.exceptions.CloseableResource.exceptionWork(TryWithResourceEx.java:32)
	at com.doubles.standardofjava.exceptions.TryWithResourceEx.main(TryWithResourceEx.java:17)
	Suppressed: com.doubles.standardofjava.exceptions.CloseException: CloseException!!!
		at com.doubles.standardofjava.exceptions.CloseableResource.close(TryWithResourceEx.java:37)
		at com.doubles.standardofjava.exceptions.TryWithResourceEx.main(TryWithResourceEx.java:18)
```

- 첫번째의 경우 일반적인 예외가 발생했을 때와 같은 형태로 출력
- 두번째의 경우 `exceptionWork()`에서 발생한 예외에 대한 내용 출력, `close()`에서 발생한 예외는 `Suppressed`(억제된)이라는 머리말과 함께 출력
- 두 예외가 동시에 발생할 수 없기 때문에 실제로 발생한 예외를 `WorkException`으로 하고, `CloseException`은 억제된 예외로 처리됨
- 만일 기존의 `try-catch`문을 사용했다면 먼저 발생한 `WorkException`은 무시되고, 마지막의 `CloseException`의 내용만 출력됨



## 10 사용자정의 예외 만들기

##### 사용자정의 예외란?
기존의 정의된 예외 클래스 외에 필요에 따라 프로그래머가 새로운 예외 클래스를 정의하여 사용할 수 있는데 보통 `Exception`클래스로부터 상속받는 클래스를 만들지만 필요에 따라 알맞은 예외 클래스를 선택할수도 있다.

```java
class MyException extends Exception {
  MyException(String msg) {
    super(msg);
  }
}
```

```java
class MyException extends Exception {
  // 에러코드 값을 저장하기 위한 필드추가
  private final int ERR_CODE;

  MyException(String msg, int errCode) { // 생성자
    super(msg);
    ERR_CODE = errCode;
  }

  MyException(String msg) { // 생성자
    this(msg, 100); // ERR_CODE를 기본값(100)으로 설정
  }

  public int getErrCode() { // 에러 코드를 얻을수 있는 메서드
    return ERR_CODE;
  }
}
```
- 기존의 예외 클래스는 보통 `Exception`클래스를 상속받아 `checked`예외로 작성하는 경우가 많았지만 요즘은 예외처리를 선택적으로 처리할 수 있도록 `RuntimeException`을 상속받아 작성하는 쪽으로 바뀌고 있음
- `checked`예외는 반드시 예외처리를 해줘야하기 때문에 예외처리가 필요하지 않은 경우에도 `try-catch`문을 넣어 코드가 복잡해지기 때문
- 필요에 따라 예외처리를 선택할 수 있는 `unchecked`예외가 선호되고 있다.

##### 사용자정의 예외 예제 : 파일 설치 프로그램(설치공간, 메모리 부족 예외발생)

```java
public class NewExceptionTest {
    public static void main(String[] args) {
        try {
            startInstall();
            copyFiles();
        } catch (SpaceException se) {
            System.out.println("에러메시지 : " + se.getMessage());
            se.printStackTrace();
            System.out.println("공간을 확보해주세요.");
        } catch (MemoryException me) {
            System.out.println("에러메시지 : " + me.getMessage());
            me.printStackTrace();
            System.out.println("메모리가 부족합니다.");
        } finally {
            deleteTempFiles();
        }
    }

    static void startInstall() throws SpaceException, MemoryException {
        if (!enoughSpace())
            throw new SpaceException("설치공간부족");
        if (!enoughMemory())
            throw new MemoryException("메모리부족");
    };

    static void copyFiles() {};

    static void deleteTempFiles() {};

    static boolean enoughSpace() {
        // 설치 공간이 충분한지 판단하는 코드 작성
        return false;
    };
    static boolean enoughMemory() {
        // 설치 메모리가 충분한지 판단하는 코드 작성
        return true;
    };
}
```

```java
class SpaceException extends Exception {
    SpaceException(String msg) {
        super(msg);
    }
}
```

```java
class MemoryException extends Exception {
    MemoryException(String msg) {
        super(msg);
    }
}
```


- 사용자 정의 예외 클래스(`SpaceException`, `MemoryException`)를 새로 만들어 사용

### 11. 예외 되던지기(exception re-throwing)

##### 예외 되던지기란?
한 메서드에서 발생할 수 있는 예외가 여러 개인 경우, 몇 개는 `try-catch`문을 통해 메서드 내에서 자체적으로 처리하도록 하고 그 나머지는 메서드 선언부에서 지정하여 호출한 메서드에서 예외를 처리하도록 할 수 있다. 심지어 하나의 예외에 대해서도 예외가 발생한 메서드와 호출한 메서드 양쪽에서 예외를 처리할 수 있도록 할 수도 있다. 이것을 예외를 처리한 후 인위적으로 예외를 발생시키는 방법을 통해 구현이 가능한데 이것을 예외 되던지기라고 한다.

이러한 방법을 쓰는 이유는 예외가 발생한 메서드와 호출한 메서드 양쪽 모두에서 처리해줘야할 작업이 있을 경우 사용한다.

### 2) 예외 되던지기 예제
```java
public static void main(String[] args) {
  try {
      method1();
  } catch (Exception e) {
      System.out.println("main()에서 예외처리");
  }
}

static void method1() throws Exception {
  try {
      throw new Exception();
  } catch (Exception e) {
      System.out.println("method1()에서 예외처리");
      throw e;
  }
}
```
```
method1()에서 예외처리
main()에서 예외처리
```
- 결과에서와 같이 `method1()`과 `main()` 양쪽의 `catch`블럭이 수행되었음
- `method1()`의 `catch`블럭에서 예외를 처리하고 `throw`문을 통해 다시 예외를 발생시키고, 그 예외를 `main()`에서 한번 더 처리
- 만약 반환값이 있는 `return`문일 경우, `catch`블럭에도 `return`문이 반드시 있어야 함

## 12 연결된 예외(chained exception)
##### 연결된 예외란?
한 예외가 다른 예외를 발생시킬 수 있다. 예를 들어 예외 A가 예외 B를 발생시켰다면, A를 B의 원인예외(cuase exception)라고 한다. 발생한 예외를 그냥 처리하면 될텐데 굳이 원인 예외로 등록하여 다시 예외를 발생시키는 이유는 여러가지 예외를 하나의 큰 분류의 예외로 묶어서 다루기 위해서이다.

##### 연결된 예외 예제
```java
public class ChainedExceptionEx {
    public static void main(String[] args) {
        try {
            install();
        } catch (InstallException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    static void install() throws InstallException {
        try {
            startIinstall();
            copyFiles();
        } catch (SpaceException2 se) {
            InstallException ie = new InstallException("설치 중 예외발생");
            ie.initCause(se);
            throw ie;
        } catch (MemoryException2 me) {
            InstallException ie = new InstallException("설치 중 예외발생");
            ie.initCause(me);
            throw ie;
        } finally {
            deleteTempFile();
        }
    };

    static void startIinstall() throws SpaceException2, MemoryException2 {
        if (!enoughSpace())
            throw new SpaceException2("설치 공간 부족");
        if (!enoughMemory())
            throw new MemoryException2("메모리 부족");
    };

    static void copyFiles() {};

    static void deleteTempFile() {};

    static boolean enoughSpace() {
        return false;
    };

    static boolean enoughMemory() {
        return true;
    };

}
```
```java
class InstallException extends Exception {
    InstallException(String msg) {
        super(msg);
    }
}
```
```java
class SpaceException2 extends Exception {
    SpaceException2(String msg) {
        super(msg);
    }
}
```
```java
class MemoryException2 extends Exception {
    MemoryException2(String msg) {
        super(msg);
    }
}
```

```
com.doubles.standardofjava.exceptions.InstallException: 설치 중 예외발생
	at com.doubles.standardofjava.exceptions.ChainedExceptionEx.install(ChainedExceptionEx.java:19)
	at com.doubles.standardofjava.exceptions.ChainedExceptionEx.main(ChainedExceptionEx.java:6)
Caused by: com.doubles.standardofjava.exceptions.SpaceException2: 설치 공간 부족
	at com.doubles.standardofjava.exceptions.ChainedExceptionEx.startIinstall(ChainedExceptionEx.java:33)
	at com.doubles.standardofjava.exceptions.ChainedExceptionEx.install(ChainedExceptionEx.java:16)
	... 1 more
```
