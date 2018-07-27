# 쓰레드(Thread)

## 1. 프로세스와 쓰레드

#### 1.1 프로세스(process), 쓰레드(thread)

**프로세스를 간단히 말해서 실행 중인 프로그램이라고 할 수 있다.** 프로그램을 실행하면 OS로부터 실해엥 필요한 자원(메모리)를 할당받아
프로세스가 된다. 프로세스는 프로그램을 수행하는데 필요한 데이터와 메모리 등의 자원 그리고 쓰레드로 구성되어 있으며 **프로세스의 자원을
이용해서 실제로 작업을 수행하는 것이 바로 쓰레드이다.**

- 프로세스 : 실행 중인 프로그램
- 쓰레드 : 프로세스의 자원을 이용해 실제로 작업을 수행하는 것

#### 1.2 멀티태스킹과 멀티쓰레딩

**멀티태스킹(Multi-tasking)이란 여러 개의 프로세스가 동시에 실행되는 것을 말한다.** 윈도우나 유닉스와 같은 대부분의 OS에서 멀티태스킹을
지원한다. 이와 마찬가지로 **멀티쓰레딩(Multi-threading)은 하나의 프로세스 내에서 여러 쓰레드가 동시에 작업을 수행하는 것을 말한다.**

- 멀티태스킹 : 여러 개의 프로세스가 동시에 실행되는 것
- 멀티쓰레딩 : 하나의 프로세스 내에서 여러 쓰레드가 동시에 작업을 수행하는 것

#### 1.3 멀티쓰레딩의 장단점

- 장점
    - CPU의 사용률을 향상시킨다.
    - 자원을 보다 효율적으로 사용할 수 있다.
    - 사용자에 대한 응답성이 향상된다.
    - 작업이 분리되어 코드가 간결해진다.

- 단점
    - 동기화(synchronization) 문제 발생
    - 교착상태(deadrock) 문제 발생

## 2. 쓰레드의 구현과 실행방법

#### 2.1 쓰레드 구현방법 1 : `Thread`클래스 상속

**`Thread`을 상속받으면 다른 클래스를 상속받을 수 없다.** 그래서 보통 `Runnable`인터페이스를 구현하는 방법이 일반적이다.

```java
// Thread클래스 외에 다른 클래스 상속을 할 수 없음.
class MyThread extends Thread {
  public void run() { // Thread클래스의 run()을 오버라이딩
    // ...
  }
}
```

#### 2.2 쓰레드 구현방법 2 : `Runnable`인터페이스 구현

**`Runnable`인터페이스는 오로지 `run()`만 정의되어 있는 간단한 인터페이스이다.** `Runnable`인터페이스를 구현하기 위해서 해야할 일을
`run()`의 `{}`을 만들어 주는 것뿐이다.

```java
class MyThread implements Runnable {
  public void run() { // Runnable인터페잇의 추상메서드 run() 구현
    // ...
  }
}
```

#### 2.3 `Thread` 예제 1 : `Thread` 상속과 `Runnable`의 구현의 차이점 구분하기

아래의 예제를 통해 쓰레드를 구현하는 2가지 방법의 차이를 확인해보자.

```java
public class ThreadEx1 {

    public static void main(String[] args) {

        // Thread를 상속받은 클래스의 인스턴스를 생성
        ThreadEx1_1 t1 = new ThreadEx1_1();

        // Runnable을 구현한 클래스의 인스턴스 생성
        Runnable r = new ThreadEx1_2();

        // 생성자 Thread(Runnable target)
        Thread t2 = new Thread(r);

        // 쓰레드 실행
        t1.start();
        t2.start();

        //t1.start(); //에러, 실행이 종료된 쓰레드는 다시 실행할 수 없음
        t1 = new ThreadEx1_1(); // 쓰레드 작업을 위해서는 쓰레드를 다시 생성해야 함
        t1.start();
    }

}

// 쓰레드 상속
class ThreadEx1_1 extends Thread {

    // run() 메서드 오버라이딩
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(getName());  // Thread의 getName호출
        }
    }

}

// Runnable 구현
class ThreadEx1_2 implements Runnable {

    // run() 추상메서드 구현
    public void run() {
        for (int i = 0; i < 5; i++) {
            // Thread.currentThread() : 현재 실행중인 Tread반환
            System.out.println(Thread.currentThread().getName());
        }
    }
}
```

```
Thread-1
Thread-1
Thread-1
Thread-1
Thread-1
Thread-0
Thread-0
Thread-0
Thread-0
Thread-0
Thread-2
Thread-2
Thread-2
Thread-2
Thread-2
```

`Thread`클래스를 상속받은 경우와 `Runnable`인터페이스를 구현한 경우 인스턴스 생성방법이 다르다. **`Runnable`인터페이스를 구현한 경우,
`Runnable`인터페이스를 구현한 클래스의 인스턴스를 생성한 다음, 이 인스턴스를 `Thread`클래스의 생성자의 매개변수로 제공해줘야 한다.**

아래의 코드는 실제 `Thread`클래스의 소스코드를 알기 쉽게 일부를 수정한 것인데 인스턴스 변수로 `Runnable`타입의 변수 `r`을 선언해 놓고,
생성자를 통해 `Runnable`인터페이스를 구현한 인스턴스를 참조하도록 되어 있는 것을 볼 수 있다. 그리고 `run()`을 호출하면 참조변수 `r`을
통해 `Runnable`인터페이스를 구현한 인스턴스의 `run()`이 호출된다. 이렇게 함으로써 상속을 통해 `run()`을 오버라이딩하지 않고도 외부로부터
`run()`을 제공받을 수 있게 된다.

```java
public class Thread {

  private Runnable r; // Runnable을 구현한 클래스의 인스턴스를 참조하기 위한 변수 선언

  public Thread(Runnable r) {
    this.r = r;
  }

  public void run() {
    if (r != null)
      r.run();  // Runnable인터페이스를 구현한 인스턴스의 run()을 호출
  }

}
```

`Thread`클래스를 상속받으면, 자손 클래스에서 조상인 `Thread`클래스의 메서드를 직접 호출할 수 있지만, **`Runnable`을 구현하면 `Thread`클래스의
`static`메서드인 `currentThread()`를 호출하며 쓰레드에 대한 참조를 얻어와야만 호출이 가능하다.**

```java
static Thread currentThread() // 현재 실행중인 쓰레드의 참조를 반환
String getName() // 쓰레드의 이름을 반환
```

**쓰레드를 생성했다고 해서 자동으로 쓰레드가 실행되는 것이 아니라 `start()`를 호출해야만 쓰레드가 실행된다.** `start()`가 호출 되었다고 해서
바로 실행되는 것이 아니라, 일단 실행대기 상태에서 있다가 자신의 차례가 되어야 비로소 실행이 된다. **주의해야할 점은 한번 실행이 종료된 쓰레드는
다시 실행할 수 없다는 점이다.** 다시 말하자면 하나의 쓰레드에 대해 `start()`가 한번만 호출될 수 있다는 뜻이다. 만약 쓰레드 작업을 한번 더 호출
해야한다면 아래의 코드처럼 새로운 쓰레드를 생성한 후에 `start()`를 호출해야 한다.

```java
ThreadEx1_1 t1 = new ThreadEx1_1();
t1.start();

// 다시 생성
t1 = new ThreadEx1_1();
t1.start();
```

## 3. `start()`와 `run()`의 차이

앞서 쓰레스를 실행시킬 때 `run()`이 아닌 `start()`를 호출하는 것에 대해 다소 의문 들었을 것이다. 이제 `start()`와 `run()`의 차이와 쓰레드가
실행되는 과정에 대해서 알아보자.

**`main()`에서 `run()`을 호출하는 것은 생성된 쓰레드를 실행시키는 것이 아니라, 단순히 클래스에 선언된 메서드를 호출하는 것일 뿐이다.**
아래의 그림은 `main()`에서 `run()`을 호출했을 때의 호출 스택이다.

![thread1](https://github.com/walbatrossw/TIL/blob/master/03_pl/java/stadard-of-java/13_thread/img/thread1.png?raw=true)

반면에 **`start()`는 새로운 쓰레드가 작업을 실행하는데 필요한 호출스택을 생성한 다음에 `run()`을 호출해서 생성된 호출스택에 `run()`이 첫번째로
올라가게 한다.** 모든 쓰레드는 독립적인 작업을 수행하기 위해 자신만의 호출 스택을 필요로 하기 때문에 새로운 쓰레드를 생성하고 실행시킬 때마다
새로운 호출스택이 생성되고 쓰레드가 종료되면 작업에 사용된 호출스택은 소멸된다.

아래의 그림은 새로운 쓰레드를 생성하고, `start()`를 호출한 후 호출스택의 변화를 순서대로 나타낸 것이다.

![thread2](https://github.com/walbatrossw/TIL/blob/master/03_pl/java/stadard-of-java/13_thread/img/thread2.png?raw=true)

1. `main()`에서 쓰레드의 `start()`를 호출한다.

![thread3](https://github.com/walbatrossw/TIL/blob/master/03_pl/java/stadard-of-java/13_thread/img/thread3.png?raw=true)

2. `start()`는 새로운 쓰레드를 생성하고, 쓰레드가 작업하는데 사용될 호출스택을 생성한다.

![thread4](https://github.com/walbatrossw/TIL/blob/master/03_pl/java/stadard-of-java/13_thread/img/thread4.png?raw=true)

3. 새로 생성된 호출스택에 `run()`이 호출되어, 쓰레드가 독립된 공간에서 작업을 수행한다.

![thread5](https://github.com/walbatrossw/TIL/blob/master/03_pl/java/stadard-of-java/13_thread/img/thread5.png?raw=true)

4. 이제는 호출스택이 2개이므로 스케쥴러가 정한 순서에 의해 번갈아가면서 실행된다.

보통 일반적으로 쓰레드가 하나일 경우 호출스택에서는 가장 위에 있는 메서드가 현재 실행중인 메서드이고, 나머지 메서드들은 대기상태에 있다.
그러나 위의 그림과 같이 쓰레드가 둘 이상일 때는 호출스택의 최상위에 있는 메서드일지라도 대기상태에 있을 수 있다. 스케쥴러는 실행대기 중인
쓰레드들의 우선순위를 고려하여 실행순서와 실행시간을 결정하고, 각 쓰레드들은 작성된 스케쥴에 따라 자신의 순서가 되면 지정된 시간동안
작업을 수행한다.

스케쥴러는 실행 대기 중인 쓰레드들의 우선순위를 고려하여 실행순서와 실행시간을 결정하고, 각 쓰레드들은 작성된 스케쥴에 따라 자신의 순서가
되면 지정된 시간동안 작업을 수행한다.

이 때 주어진 시간동안 작업을 마치지 못한 쓰레드는 다시 자신의 차례가 돌아올 때까지 대기상태로 있게되며, 작업을 마친 쓰레드, 즉 `run()`의
수행이 종료된 쓰레드는 호출스택에 모두 비워지면서 쓰레드가 사용하던 호출스택은 사라지게된다.

#### 3.1 쓰레드 예제 1 : `start()`호출하기
새로 생성한 쓰레드에서 고의로 예외를 발생시키고, `printStackTrace()`을 이용해서 예외가 발생한 당시의 호출스택을 출력하는 예제이다.

```java
public class ThreadEx2 {

    public static void main(String[] args) {
        ThreadEx2_1 t1 = new ThreadEx2_1();
        t1.start();
    }

}


class ThreadEx2_1 extends Thread {

    public void run() {
        throwException();
    }
    
    // 예외 발생
    public void throwException() {
        try {
            throw new Exception();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```
```
java.lang.Exception
	at com.doubles.standardofjava.ch13_thread.ThreadEx2_1.throwException(ThreadEx2.java:19)
	at com.doubles.standardofjava.ch13_thread.ThreadEx2_1.run(ThreadEx2.java:14)
```

- 호출스택의 첫번째 메서드가 `main()`이 아니라 `run()`인 것을 알 수 있다.
- 한 쓰레드가 예외가 발생해서 종료되어도 다른 쓰레드의 실행에는 영향을 미치지 않는다.
- `main`쓰레드의 호출스택이 없는 이유는 `main`쓰레드가 종료되었기 때문이다.

#### 3.2 쓰레드 예제 2 : `run()`호출하기

이 예제도 역시 새로 생성한 쓰레드에서 고의로 예외를 발생시키고, `printStackTrace()`을 이용해서 예외가 발생한 당시의 호출스택을 출력하는 예제이다.

```java
public class ThreadEx3 {

    public static void main(String[] args) {
        ThreadEx3_1 t1 = new ThreadEx3_1();
        t1.run();
    }

}

class ThreadEx3_1 extends Thread {

    public void run() {
        throwException();
    }

    // 예외 발생
    public void throwException() {
        try {
            throw new Exception();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```
```
java.lang.Exception
	at com.doubles.standardofjava.ch13_thread.ThreadEx3_1.throwException(ThreadEx3.java:20)
	at com.doubles.standardofjava.ch13_thread.ThreadEx3_1.run(ThreadEx3.java:15)
	at com.doubles.standardofjava.ch13_thread.ThreadEx3.main(ThreadEx3.java:7)
```
- 이전 예제와 다르게 쓰레드가 새로 생성되지 않았는데, `ThreadEx3_1`클래스의 `run()`이 호출되었을 뿐이기 때문이다.

## 4. 싱글 쓰레드와 멀티 쓰레드

위에서 멀티 쓰레드의 장단점에 대해 정리했었는데 이번에는 예제를 통해 싱글 쓰레드 프로세스와 멀티 쓰레드 프로세스의 차이를 보다 깊이 있게 이해할
수 있도록 해보자.

두 개의 작업을 하나의 쓰레드(`th1`)로 처리하는 경우와 두 개의 쓰레드(`th1`, `th2`)로 처리하는 경우를 가정하고 차이점을 알아보자.

![single_multi_thread](https://github.com/walbatrossw/TIL/blob/master/03_pl/java/stadard-of-java/13_thread/img/single_multi_thread.png?raw=true)

- 하나의 쓰레드로 두 작업을 처리하는 경우 : 한 작업을 마친 후에 다른 작업을 시작하여 처리
- 두 개의 쓰레드로 작업을 처리하는 경우 : 짧은 시간동안 2개의 쓰레드가 번걸아 가면서 작업을 수행해서 동시에 두 작업이 처리되는 것 같이 느껴짐

위의 그래프에서 볼 수 있듯이 하나의 쓰레드로 두 작업을 처리한 결과와 두 개의 쓰레드로 작업을 수행한 시간은 거의 같다. 오히려 두 개의 쓰레드로 작업을
처리한 시간이 싱글 쓰레드로 작업한 시간보다 더 걸리게 되는데 그 이유는 쓰레드간의 작업 전환에 시간이 걸리기 때문이다.

그래서 싱글 코어에서 단순히 CPU만을 사용하는 계산 작업일 경우 오히려 멀티쓰레드보다 싱글쓰레드로 프로그래밍하는 것이 효율적이다.

#### 4.1 싱글쓰레드와 멀티쓰레드의 차이 예제 1 : 하나의 쓰레드로 두 개의 작업을 처리할 경우

```java
public class ThreadEx6 {
    public static void main(String[] args) {
        String input = JOptionPane.showInputDialog("아무 값이나 입력하세요.");
        System.out.println("입력하신 값은 " + input + "입니다. ");

        for (int i = 10; i > 0; i--) {
            System.out.println(i);
            try {
                Thread.sleep(1000);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```
```
입력하신 값은 1234입니다. 
10
9
8
7
6
5
4
3
2
1
```

위 코드를 실행해보면 하나의 쓰레드로 사용자의 입력을 받는 작업과 숫자를 출력하는 작업을 처리하기 때문에 사용자가 입력을 마칠 때까지 화면에 숫자가 출력되지
않다가 사용자가 입력을 마치면 콘솔 화면에 숫자가 출력된다.

#### 4.2 싱글쓰레드와 멀티쓰레드의 차이 예제 2 : 두 개의 쓰레드로 두 개의 작업을 처리할 경우

```java
public class ThreadEx7 {
    public static void main(String[] args) {
        ThreadEx7_1 t1 = new ThreadEx7_1();
        t1.start();

        String input = JOptionPane.showInputDialog("아무 값이나 입력해주세요.");
        System.out.println("입력하신 값은 " + input + " 입니다.");
    }
}

class ThreadEx7_1 extends Thread {
    @Override
    public void run() {
        for (int i = 10; i > 0; i--) {
            System.out.println(i);
            try {
                sleep(1000);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

```
10
9
8
입력하신 값은 1234 입니다.
7
6
5
4
3
2
1
```

첫번째 예제와 다르게 사용자로부터 입력받는 부분과 화면에 숫자를 출력하는 부분을 두 개의 쓰레드로 나누어 처리했기 때문에 사용자가 입력을 마치지 않았지만
콘솔화면에 숫자가 출력되는 것을 알 수 있다.

## 5. 쓰레드의 우선순위

쓰레드는 우선순위(Priority)라는 속성을 가지고 있는데, 이 우선순위의 값에 따라 쓰레다가 얻는 실행시간이 달라진다. 쓰레드가 수행하는 작업의 중요도에 따라
쓰레드의 우선순위를 다르게 지정하여 특정 쓰레드가 더 많은 작업시간을 갖도록 할 수 있다.

예를 들어 설명해보면 파일전송 기능이 있는 메신저의 경우 파일 다운로드를 처리하는 쓰레드보다 채팅 내용을 전송하는 쓰레드의 우선순위가 높아야 사용자가 채팅을
하는데 불편함이 없을 것이다. 하지만 파일을 전송하는데 걸리는 시간은 더 길어지게 된다.

위와 같이 시각적인 부분이나 사용자에게 빠르게 반응해야하는 작업을 하는 쓰레드의 우선 순위는 다른 작업을 수행하는 쓰레드에 비해 높아야 한다.

#### 5.1 쓰레드 우선순위 지정방법
쓰레드의 우선순위와 관련된 메서드와 상수는 다음과 같다.

```java
void setPriority(int newPriority); // 쓰레드의 우선순위를 지정한 값으로 변경
int getPriority(); // 쓰레드의 우선순위를 반환

public static final int MAX_PRIORITY = 10;  // 최대 우선순위
public static final int MIN_PRIORITY = 1;   // 최소 우선순위
public static final int NORM_PRIORITY = 5;  // 보통 우선순위
```

쓰레드가 가질 수 있는 우선순위의 범위는 1~10이며 숫자가 높을 수록 우선순위가 높다. 쓰레드의 우선순위는 쓰레드를 생성한 쓰레드로부터 상속받는다. `main()`메서드를
수행하는 쓰레드는 우선순위가 5이므로 `main()`메서드 내에서 생성하는 쓰레드 우선순위는 자동적으로 5가 된다.

#### 5.2 쓰레드 우선순위 예제

```java
public class ThreadEx8 {

    public static void main(String[] args) {
        ThreadEx8_1 th1 = new ThreadEx8_1();
        ThreadEx8_2 th2 = new ThreadEx8_2();

        th2.setPriority(7);

        System.out.println("Priority of th1(-) : " + th1.getPriority());
        System.out.println("Priority of th2(|) : " + th2.getPriority());

        th1.start();
        th2.start();
    }

}

class ThreadEx8_1 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 300; i++) {
            System.out.print("-");
            for (int x = 0; x < 1000000000; x++);
        }
    }
}

class ThreadEx8_2 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 300; i++) {
            System.out.print("|");
            for (int x = 0; x < 1000000000; x++);
        }
    }
}
```

```
Priority of th1(-) : 5
Priority of th2(|) : 7
// 생략 ...

```

`th1`과 `th2` 모두 `main()` 메서드에서 생성하였기 때문에 `main()` 메서드를 실행하는 쓰레드의 우선순위인 5를 상속받았지만 `th2`는 `setPriority()`메서드를 통해
우선순위를 7로 변경하였다. 쓰레드를 실행하기 전에만 우선 순위를 변경할 수 있다.

실제로 위의 코드를 실행해보면 멀티코어에서는 쓰레드의 우선순위에 따른 차이가 거의 없다. 우선순위를 다르게하여 여러번 테스트를 하더라도 결과는 같다. 결국 우선순위에
차등을 두어 쓰레드를 실행시키는 것이 별 효과가 없다는 것을 알 수 있다. 쓰레드에 높은 우선순위를 주면 더 많은 실행시간과 실행기회를 갖게 될 것이라고 기대할 수 없다.

쓰레드에 우선순위를 부여하는 대신 작업에 우선순위를 두어 PriorityQueue에 저장해 놓고, 우선순위가 높은 작업이 먼저 처리되도록 하는 것이 나을 수 있다.

## 6. 쓰레드 그룹

**쓰레드 그룹은 서로 관련된 쓰레드를 그룹으로 다루기 위한 것으로 폴더를 생성해서 관련된 파일들을 함께 넣어서 관리하는 것처럼 쓰레드 그룹을 생성해서 쓰레드를 그룹으로
묶어서 관리할 수 있다.**

폴더 안에 폴더를 생성할 수 있듯이 쓰레드 그룹에 다른 쓰레드 그룹을 포함시킬 수 있다. 쓰레드 그룹은 보안상의 이유로 도입된 개념으로 자신이 속한 쓰레드 그룹이나 하위 쓰레드
그룹은 변경할 수 있지만 다른 쓰레드 그룹의 쓰레드를 변경할 수는 없다.

쓰레드 그룹의 주요 생성자와 메서드는 아래와 같다.

- `TreadGroup(String name)` : 지정된 이름의 새로운 쓰레드 그룹 생성
- `TreadGroup(TreadGroup parent, String name)` : 지정된 쓰레드 그룹에 포함되는 새로운 쓰레드 그룹 생성
- `int activeCount()` : 쓰레드 그룹에 포함된 활성상태에 있는 쓰레드의 수를 반환
- `int activeGroupCount()` : 쓰레드 그룹에 포함된 활성상태에 있는 쓰레드 그룹의 수를 반환
- `void checkAccess()` : 현재 실행중인 쓰레드가 쓰레드 그룹을 변경할 권한이 있는지 체크, 권한이 없을 경우, `SecurityException`발생
- `void destroy()` : 쓰레드 그룹과 하위 쓰레드 그룹까지 삭제, 쓰레드 그룹이나 하위 쓰레드 그룹이 비어있어야함|
- `int enumerate(Thread[] list)`, `int enumerate(Thread[] list, boolean rescue)`, `int enumerate(ThreadGroup[] list)`, `int enumerate(ThreadGroup[] list, boolean rescue)`
    - 쓰레드 그룹에 속한 쓰레드 또는 하위 쓰레드 그룹의 목록을 지정된 배열에 담고 그 개수를 반환
    - 두번째 매개변수인 `rescue`의 값을 `true`로 하면 쓰레드 그룹에 속한 하위 쓰레드 그룹에 쓰레드 또는 쓰레드 그룹까지 배열에 담는다.
- `int getMaxPriority()` : 쓰레드 그룹의 최대우선순위 반환
- `String getName()` : 쓰레드 그룹의 이름 반환
- `ThreadGroup getParent()` : 쓰레드 그룹의 상위 쓰레드 그룹을 반환
- `void interrupt()` : 쓰레드 그룹에 속한 모든 쓰레드를 interrupt
- `boolean isDaemon()` : 쓰레드 그룹이 데몬 쓰레드인지 확인
- `boolean isDestroyed()` : 쓰레드 그룹이 삭제되었는지 확인
- `void list()` : 쓰레드 그룹에 속한 쓰레드와 하위 쓰레드그룹에 대한 정보 출력
- `boolean parentOf(ThreadGroup g)` : 지정된 쓰레드 그룹의 상위 쓰레드그룹인지 확인
- `void setDaemon(boolean daemon)` : 쓰레드 그룹을 데몬 쓰레드 그룹으로 설정/해제
- `void setMaxPriority(int pri)` : 쓰레드 그룹의 최대우선순위를 설정

쓰레드를 쓰레드 그룹에 포함시키려면 `Thread`의 생성자를 이용해야한다.

```java
Thread(ThreadGroup group, String name)
Thread(ThreadGroup group, Runnable target)
Thread(ThreadGroup group, Runnable target, String name)
Thread(ThreadGroup group, Runnable target, String name, long stackSize)
```

모든 쓰레드는 반드시 쓰레드 그룹에 포함되어 있어야하기 때문에 위와 같이 쓰레드 그룹을 지정하는 생성자를 사용하지 않은 쓰레드는 기본적으로 자신을 생성한 쓰레드와 같은
쓰레드 그룹에 속하게 된다.

자바 어플리케이션이 실행되면 JVM은 main과 system이라는 쓰레드 그룹을 만들고 JVM운영에 필요한 쓰레드들을 생성해서 이 쓰레드 그룹에 포함시킨다. 예를 들어 `main()`메서드를
수행하는 main이라는 이름의 쓰레드는 이름의 쓰레드는 main쓰레드 그룹에 속하고, 가비지 컬렉션을 수행하는 Finalizer쓰레드는 system쓰레드 그룹에 속한다.

우리가 생성하는 모든 쓰레드 그룹은 main쓰레드 그룹의 하위 쓰레드 그룹이 되며, 쓰레드 그룹을 지정하지 않고 생성한 쓰레드는 자동적으로 main쓰레드 그룹에 속하게 된다.

#### 6.1 쓰레드 그룹 예제

```java
public class ThreadEx9 {
    public static void main(String[] args) {
        ThreadGroup main = Thread.currentThread().getThreadGroup();
        ThreadGroup grp1 = new ThreadGroup("Group1");
        ThreadGroup grp2 = new ThreadGroup("Group2");

        ThreadGroup subGrp1 = new ThreadGroup(grp1, "SubGroup1");

        grp1.setMaxPriority(3);

        Runnable r = new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        new Thread(grp1, r, "th1").start();
        new Thread(subGrp1, r, "th2").start();
        new Thread(grp2, r, "th3").start();

        System.out.println(">> List of ThreadGroup : " + main.getName()
                            + ", Active ThreadGroup : " + main.activeGroupCount()
                            + ", Active Thread : " + main.activeCount());

        main.list();

    }
}
```

```
>> List of ThreadGroup : main, Active ThreadGroup : 3, Active Thread : 5
java.lang.ThreadGroup[name=main,maxpri=10]
    Thread[main,5,main]
    Thread[Monitor Ctrl-Break,5,main]
    java.lang.ThreadGroup[name=Group1,maxpri=3]
        Thread[th1,3,Group1]
        java.lang.ThreadGroup[name=SubGroup1,maxpri=3]
            Thread[th2,3,SubGroup1]
    java.lang.ThreadGroup[name=Group2,maxpri=10]
        Thread[th3,5,Group2]
```

위의 코드는 쓰레드 그룹과 쓰레드를 생성하고 `main.list()`를 호출해서 main쓰레드 그룹의 정보를 출력한 예제이다. 콘솔화면에 출력된 결과를 보면 쓰레드 그룹에
포함된 하위 쓰레드 그룹이나 쓰레드는 들여쓰기를 이용해서 구별되는 것을 알 수 있다.

## 7. 데몬 쓰레드

**데몬쓰레드는 일반 쓰레드의 작업을 돕는 보조적인 역할을 수행하는 쓰레드이다.** 일반 쓰레드가 모두 종료되면 데몬쓰레드는 강제적으로 자동 종료되는데 그 이유는 간단하다.
데몬쓰레드는 일반 쓰레드를 도와주는 보조적인 역할이므로 존재의 이유가 없어지기 때문이다. 이 점을 제외하면 일반 쓰레드와 동일하다.

데몬쓰레드는 무한루프와 조건문을 이용해서 실행 후 대기하고 있다가 특정 조건이 만족되면 작업을 수행하고 다시 대기하도록 작성한다.

데몬쓰레드는 일반 쓰레드와 작성방법, 실행방법이 동일하며 쓰레드를 생성한 다음 실행하기 전에 `setDaemon(true)`를 호출하기만 하면된다. 그리고 데몬쓰레드가 생성한 쓰레드는
자동적으로 데몬 쓰레드가 된다.

#### 7.1 데몬 쓰레드 예제1

```java
public class ThreadEx10 implements Runnable {

    static boolean autoSave = false;

    public static void main(String[] args) {

        Thread t = new Thread(new ThreadEx10());
        t.setDaemon(true);  // 이부분이 중요! 없을 경우 종료되지 않음
        t.start();

        for (int i = 1; i <= 10; i++) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(i);

            if (i == 5)
                autoSave = true;
        }

        System.out.println("프로그램을 종료합니다.");

    }

    @Override
    public void run() {
        while (true) {
            try {
                Thread.sleep(3*1000); // 3초마다
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            if (autoSave) {
                autoSave();
            }
        }
    }

    public void autoSave() {
        System.out.println("작업파일이 자동저장되었습니다.");
    }
}
```

```
1
2
3
4
5
작업파일이 자동저장되었습니다.
6
7
8
작업파일이 자동저장되었습니다.
9
10
프로그램을 종료합니다.
```

위의 코드는 3초마다 변수 `autoSave`의 값을 확인해서 그 값이 `true`이면, `autoSave()` 메서드를 호출하는 일을 무한히 반복하도록 쓰레드를 작성하였다.
만약 이 쓰레드를 데몬 쓰레드로 설정하지 않았다면, 이 프로그램은 강제종료하지 않으면 계속 실행될 것이다.

`setDaemon()`메서드는 반드시 `start()`메서드를 호출하기 전에 실행되어야한다. 그렇지 않으면 `IllegalThreadStateException`이 발생하게 된다.

#### 7.2 데몬 쓰레드 예제2

```java
public class ThreadEx11 {
    public static void main(String[] args) {
        ThreadEx11_1 t1 = new ThreadEx11_1("Thread1");
        ThreadEx11_2 t2 = new ThreadEx11_2("Thread2");

        t1.start();
        t2.start();

    }
}

class ThreadEx11_1 extends Thread {

    // 생성자
    public ThreadEx11_1(String name) {
        super(name);
    }

    @Override
    public void run() {
        try {
            sleep(5 * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class ThreadEx11_2 extends Thread {

    public ThreadEx11_2(String name) {
        super(name);
    }

    @Override
    public void run() {
        Map map = getAllStackTraces();
        Iterator it = map.keySet().iterator();

        int x = 0;
        while (it.hasNext()) {
            Object obj = it.next();
            Thread t = (Thread) obj;
            StackTraceElement[] ste = (StackTraceElement[]) (map.get(obj));

            System.out.println("[" + ++x + "]" + t.getName()
                                + ", group : " + t.getThreadGroup().getName()
                                + ", daemon : " + t.isDaemon());

            for (int i = 0; i < ste.length; i++) {
                System.out.println(ste[i]);
            }

            System.out.println();
        }
    }
}
```

```
[1]DestroyJavaVM, group : main, daemon : false

[2]Signal Dispatcher, group : system, daemon : true

[3]Attach Listener, group : system, daemon : true

[4]Thread1, group : main, daemon : false
java.lang.Thread.sleep(Native Method)
com.doubles.standardofjava.ch13_thread.ThreadEx11_1.run(ThreadEx11.java:27)

[5]Finalizer, group : system, daemon : true
java.lang.Object.wait(Native Method)
java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:212)

[6]Monitor Ctrl-Break, group : main, daemon : true
java.net.SocketInputStream.socketRead0(Native Method)
java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
java.net.SocketInputStream.read(SocketInputStream.java:171)
java.net.SocketInputStream.read(SocketInputStream.java:141)
sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
java.io.InputStreamReader.read(InputStreamReader.java:184)
java.io.BufferedReader.fill(BufferedReader.java:161)
java.io.BufferedReader.readLine(BufferedReader.java:324)
java.io.BufferedReader.readLine(BufferedReader.java:389)
com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:64)

[7]Reference Handler, group : system, daemon : true
java.lang.Object.wait(Native Method)
java.lang.Object.wait(Object.java:502)
java.lang.ref.Reference.tryHandlePending(Reference.java:191)
java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

[8]Thread2, group : main, daemon : false
java.lang.Thread.dumpThreads(Native Method)
java.lang.Thread.getAllStackTraces(Thread.java:1610)
com.doubles.standardofjava.ch13_thread.ThreadEx11_2.run(ThreadEx11.java:42)
```

`getAllStackTraces()`를 이용하면 실행 중 또는 대기상태, 즉 작업이 완료되지 않은 모든 쓰레드의 호출스택을 출력할 수 있다. 결과를 보면 `getAllStackTraces()` 메서드가 호출되었을 때
새로 생성한 `Thread1`, `Thread2`를 포함해서 8개의 쓰레드가 실행 중 또는 대기 상태에 있다는 것을 알 수 있다.

프로그램을 실행하면 JVM은 가비지컬렉션, 이벤트처리, 그래픽처리와 같이 프로그램이 실행되는데 필요한 보조작업을 수행하는 데몬쓰레드들을 자동으로 생성해서 실행시킨다. 그리고 이들은 system쓰레드
그룹 또는 main쓰레드 그룹에 속한다.


## 8. 쓰레드의 실행제어

## 9. 쓰레드의 동기화

#### `synchronized`를 이용한 동기화

#### `wait()`와 `nofify()`

#### `Lock`과 `Condition`을 이용한 동기화

#### `volatile`

#### `fork`, `join` 프레임워크
