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

쓰레드 프로그래밍이 어려운 이유는 동기화(synchronization)와 스케쥴링(scheduling) 때문이다. 앞서 우선순위를 통해 쓰레드 간의 스케쥴링을 배웠지만 이것으로는 부족하다.
효율적인 멀티쓰레드 프로그램을 만들기 위해서는 보다 정교한 스케쥴링을 통해 프로세스에게 주어진 자원과 시간을 여러 쓰레드가 낭비없이 잘 사용하도록 프로그래밍해야한다.

쓰레드의 스케쥴링을 잘하기 위해서는 쓰레드 상태와 관련된 메서드에 대해 잘알아야한다.

- `static void sleep(long millis)`, `static void sleep(long millis, int nanos)` : 저정된 시간(1/1000초 단위)동안 쓰레드 일시정지, 일시정지가 끝나고 나면 실행대기 상태로 전환
- `void join()`, `void join(long millis)`, `void join(long millis, int nanos)` : 지정된 시간동안 쓰레드 실행되도록 함. 지정된 시간이 지나거나 작업이 종료되면 `join()`을 호출한 쓰레드로 다시 다시 돌아와 실행을 계속함
- `void interrupt()` : `sleep()`이나 `join()`에 의해 일시정지 상태인 쓰레드를 깨워서 실행 대기상태로 만듬. `interruptedException()`이 발생함으써 일시정지상태를 벗어난게 됨
- `void stop()` : 쓰레드를 즉시 종료
- `void suspend()` : 쓰레드를 일시정지 시킴.
- `void resume()` : `suspend()`에 의해 일시정지 상태에 있는 쓰레드를 실행대기 상태로 전환
- `static void yield()` : 실행 중에 자신에게 주어진 실행시간을 다른 쓰레드에게 양보하고 자신은 실행대기 상태로 전환

#### 8.1 쓰레드의 상태

쓰레드의 상태는 아래와 같다.

- NEW : 쓰레드가 생성되고 아직 `start()`가 호출되지 않은 상태
- RUNNABLE : 실행 중 또는 실행 가능한 상태
- BLOCKED : 동기화 블럭에 의해 일시정지된 상태(lock이 풀릴 때 까지 기다리는 상태)
- WAITING, TIMED_WAITING : 쓰레드의 작업이 종료되지는 않지만 실행가능하지 않은(unrunnable) 일시정지 상태, TIMED_WAITING은 일시정지시간이 지정된 경우를 의미
- TERMINATED : 쓰레드 작업이 종료된 상태

아래의 그림은 쓰레드의 생성부터 소멸까지의 모든 과정을 그린 것인데 앞서 소개한 메서드들에 의해서 쓰레드의 상태가 어떻게 변하는지 알 수 있다.

![thread_state](https://github.com/walbatrossw/TIL/blob/master/03_pl/java/stadard-of-java/13_thread/img/thread_state.png?raw=true)

1. 쓰레드를 생성하고, `start()`를 호출하면 바로 실행되는 것이 아니라 실행대기열에 저장되어 자신의 차례가 될 때가지 기다려야 한다. 실행대기열은 큐(queue)와 같은 구조로 먼저 실행대기얄에 들어온 쓰레드가 먼저 실행된다.
2. 실행대기상태에 있다가 자신의 차례가 되면 실행상태가 된다.
3. 주어진 실행시간이 다되거나 `yield()`를 만나면 다시 실행대기 상태가 되고, 다음 차례의 쓰레드가 실행상태가 된다.
4. 실행 중에 `suspend()`, `sleep()`, `wait()`, `join()`, I/O break에 의해 일시정지 상태가 될 수 있다.
5. 지정된 일시정지시간이 다되거나(time-out), `notify()`, `resume()`, `interrupt()`가 호출되면 일시정지 상태를 벗어나 다시 실행대기열에 저장되어 자신의 차례를 기다린다.
6. 실행을 모두 마치거나 `stop()`이 호출되면 쓰레드는 소멸된다.

#### 8.2 `sleep(long millis)` - 일정시간 동안 쓰레드 멈춤

**`sleep()`은 지정된 시간동안 쓰레드를 멈추게 한다.** 밀리세컨드(1/1000초)와 나노세컨드(10억분의 일초)의 시간단위로 세밀하게 값을 지정할 수 있지만 어느 정도 오차가 발생할 수 있다.

`sleep()`에 의해 일시정지가 된 쓰레드는 지정된 시간이 지나거나 `interrupt()` 메서드가 호출되면 `InterruptException`이 발생되어 잠에서 깨어나 실행대기 상태가 된다. 그래서 `sleep()` 메서드를 호출할 때는
항상 `try-catch`문으로 예외를 처리해줘야한다. 매번 예외처리를 해주는 것이 번거롭기 때문에 아래와 같이 `try-catch`문까지 포함하는 새로운 메서드를 만들어 사용하기도 한다.

```java
void delay(long millis) {
    try {
        Thread.sleep(millis);
    } catch(InterruptException e) {
        
    }
}
```

아래는 `sleep()` 메서드 예제이다.

```java
public class ThreadEx12 {
    public static void main(String[] args) {
        ThreadEx12_1 th1 = new ThreadEx12_1();
        ThreadEx12_2 th2 = new ThreadEx12_2();

        th1.start();
        th2.start();

        try {
            th2.sleep(2000);
        } catch (InterruptedException e) {

        }

        System.out.println("<<main 종료>>");
    }
}

class ThreadEx12_1 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 300; i++) {
            System.out.print("-");
        }
        System.out.println("<<th1 종료>>");
    }
}

class ThreadEx12_2 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 300; i++) {
            System.out.print("|");
        }
        System.out.println("<<th2 종료>>");
    }
}
```

```
... ---------<<th1 종료>>
... |||||||||<<th2 종료>>
<<main 종료>>
```

위의 결과를 보면 쓰레드 `th1`의 작업이 가장 먼저 종료되었고, 그 다음이 `th2`, `main`의 순인 것을 알 수 있다. 쓰레드 `th1`과 `th2`를 `start()` 메서드를 호출하자마자 `th1.sleep(2000)` 메서드를
호출하여 쓰레드 `th1`이 2초 동안 작업을 멈추고 일시정지 상태에 있도록 하였기 때문에 쓰레드 `th1`이 가장 늦게 종료되어야 하는데 결과에서는 제일 먼저 종료되었다.

이러한 이유는 `sleep()` 메서드가 항상 현재 실행 중인 쓰레드에 대해 작동하기 때문에 `th1.sleep(2000)`과 같이 호출하였어도 실제로 영향을 받는 것은 `main()`메서드를 실행하는 `main`쓰레드이다.
그래서 `sleep()` 메서드는 `static`으로 선언되어 있으며 참조변수를 이용해서 호출하기 보다는 `Thread.sleep(2000)`과 같이 해야한다.

#### 8.3 `interrupt()`, `interrupted()` - 쓰레드 작업 취소

진행 중인 쓰레드의 작업이 끝나기 전에 취소를 시켜야할 때가 있다. 예를 들면 큰 파일을 다운로드 받을 때 시간이 너무 오래 걸리면 중간에 다운로드를 포기하고 취소할 수 있어야 한다. **`interrupt()` 메서드는
쓰레드에게 작업을 멈추라고 요청한다. 하지만 쓰레드를 강제로 종료시키지는 못한다. `interrupt()` 메서드는 그저 쓰레드의 `interrupted`(인스턴스 변수)상태를 바꾸는 것일 뿐이다.**

**`interrupted()` 메서드는 쓰레드에 대해 `interrupt()`메서드가 호출되었는지 알려준다. `interrupt()`가 호출되었는지 여부에 따라 `true`나 `false`를 반환한다.**

- `void interrupt()` : 쓰레드의 `interrupted` 상태를 `false`에서 `true`로 변경
- `boolean isInterrupted()` : 쓰레드의 `interrupted` 상태를 반환
- `static boolean interrupted()` : 현재 쓰레드의 `interrupted` 상태를 알려주고, `false`로 초기화

쓰레드가 `sleep()`, `wait()`, `join()`에 의해 일시정지 상태에 있을 때, 해당 쓰레드에 대해 `interrupt()`를 호출하면 `sleep()`, `wait()`, `join()`에서 `InterruptedException`이 발생하고 쓰레드는
실행대기 상태로 바뀐다. 다시 말하자면 멈춰있던 쓰레드를 깨워서 실행가능 상태로 만드는 것이다.

아래의 예제는 이전의 예제를 `interrupt()`와 `interrupted()`를 사용하여 수정한 것으로 카운트 다운 도중 사용자 입력이 들어오면 카운트 다운을 종료하게 한다.

```java
public class ThreadEx13 {
    public static void main(String[] args) {
        ThreadEx13_1 th1 = new ThreadEx13_1();
        th1.start();

        String input = JOptionPane.showInputDialog("아무 값이나 입력하세요.");
        System.out.println("입력하신 값은" + input + "입니다.");
        th1.interrupt();    // interrupt()를 호출, 변수 interrupted는 true로 변경
        System.out.println("isInterrupted() : " + th1.isInterrupted());
    }
}

class ThreadEx13_1 extends Thread {
    @Override
    public void run() {
        int i = 10;
        while (i != 0 && !isInterrupted()) {
            System.out.println(i--);
            for (long x = 0; x < 2500000000L; x++); // 시간 지연
        }
        System.out.println("카운트가 종료되었습니다.");
    }
}
```
```
10
9
8
7
입력하신 값은123입니다.
isInterrupted() : true
카운트가 종료되었습니다.
```

아래의 예제는 위의 예제에서 for문 대신 `Thread.sleep(1000)`으로 1초 동안 지연되도록 변경하였는데 카운트가 종료되지 않는다. 게다가 `isInterrupt()` 메서드를 호출한 결과도 `false`이다.
이러한 이유는 `Thread.sleep(1000)`에서 `InterruptException`이 발생했기 때문이다. `sleep()`메서드에 의해서 쓰레드가 잠시 멈춰있을 때, `interrupt()` 메서드를 호출하면 `InterruptException`이
발생하고 쓰레드의 `interrupted`의 상태는 `false`로 초기화가 된다. 이럴 때에는 `catch`블럭에서 `interrupt()`메서드를 넣어줘서 쓰레드의 변수 `interrupted`의 상태를 다시 `true`로 바꿔줘야 한다.

```java
public class ThreadEx14 {
    public static void main(String[] args) {
        ThreadEx14_1 th1 = new ThreadEx14_1();
        th1.start();

        String input = JOptionPane.showInputDialog("아무 값이나 입력하세요.");
        System.out.println("입력하신 값은 " + input + " 입니다.");
        th1.interrupt();
        System.out.println("isInterrupted() : " + th1.isInterrupted());
    }
}

class ThreadEx14_1 extends Thread {

    @Override
    public void run() {
        int i = 10;

        while (i != 0 && !isInterrupted()) {
            System.out.println(i--);
            try {
                Thread.sleep(1000); // 1초 지연
            } catch (InterruptedException e) {
                //interrupt(); // 추가하면 의도되로 프로그램이 작동된다.
            }
        }
        System.out.println("카운트가 종료되었습니다.");
    }
}
```

```
10
9
8
입력하신 값은 123 입니다.
isInterrupted() : true
7
6
5
4
3
2
1
카운트가 종료되었습니다.
```

#### 8.4 `suspend()`, `resume()`, `stop()`

- `suspend()` : `sleep()`처럼 쓰레드를 멈추게 한다.
- `resume()` : `resume()` 메서드를 호출하면 `suspend()`에 의해 정지된 쓰레드를 실행대기 상태가 된다.
- `stop()` : 호출되는 즉시 쓰레드가 종료된다.

`suspend()`, `resume()`, `stop()은 쓰레드의 실행을 제어하는 가장 손쉬운 방법이지만, `suspend()`와 `stop`이 교착상태(deadlock)를 일으키기 쉽게 작성되기 때문에 권장되지 않는다.
이 메서드들은 모두 `deprecated`되었다. 전에는 사용했지만 앞으로 사용하지 않을 것을 권장한다는 의미로 하위 호환성을 위해 삭제하지 않을 뿐이므로 사용해서는 안된다.


아래의 예제는 `suspend()`, `resume()`, `stop()`의 사용법을 보여주는 예제로 `sleep(2000)`은 쓰레드를 2초간 멈추게 하지만, 2초 후에 바로 실행상태가 아닌 실행 대기 상태가 된다.
아래의 예제는 간단한 상태이기 때문에 교착상태가 일어날 일이 없으므로 `suspend()`, `stop()`를 사용해도 되지만 좀더 복잡한 경우에는 사용하지 않는 것이 좋다.

```java
public class ThreadEx15 {
    public static void main(String[] args) {
        RunImplEx15 r = new RunImplEx15();
        Thread th1 = new Thread(r, "*");
        Thread th2 = new Thread(r, "**");
        Thread th3 = new Thread(r, "***");

        th1.start();
        th2.start();
        th3.start();

        try {

            Thread.sleep(2000);
            th1.suspend();  // 쓰레드 th1을 잠시 중단

            Thread.sleep(2000);
            th2.suspend();  // 쓰레드 th2을 잠시 중단

            Thread.sleep(3000);
            th1.resume();   // 쓰레드 th1을 다시 동작

            Thread.sleep(3000);
            th1.stop();     // 쓰레드 th1 강제 종료
            th2.stop();     // 쓰레드 th2 강제 종료

            Thread.sleep(2000);
            th3.stop();     // 쓰레드 th3을 강제 종료

        } catch (InterruptedException e) {

        }
    }
}

class RunImplEx15 implements Runnable {

    @Override
    public void run() {
        while (true) {
            System.out.println(Thread.currentThread().getName());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {

            }
        }
    }
}
```

```
*
**
***
**
***
*
**
***
**
***
***
***
***
*
***
*
***
*
***
***
***
```

아래의 예제는 위에서 언급한 교착상태를 해결할 방법을 적용해본 것이다. `stopped`와 `suspended`라는 `boolean`타입의 두 변수를 인스턴스 변수로 선언하고, 이 변수를 사용해 반복문과 조건문의
조건식을 작성한다. 그리고 이 변수의 값을 변경함으로써 쓰레드의 작업이 중지되었다가 재개되거나 종료되도록 할 수 있다.

```java
public class ThreadEx16 {
    public static void main(String[] args) {
        RunImplEx16 r1 = new RunImplEx16();
        RunImplEx16 r2 = new RunImplEx16();
        RunImplEx16 r3 = new RunImplEx16();

        Thread th1 = new Thread(r1, "*");
        Thread th2 = new Thread(r2, "**");
        Thread th3 = new Thread(r3, "***");

        th1.start();
        th2.start();
        th3.start();

        try {
            Thread.sleep(2000);
            r1.suspend();
            Thread.sleep(2000);
            r2.suspend();
            Thread.sleep(3000);
            r1.resume();
            Thread.sleep(3000);
            r1.stop();
            r2.stop();
            Thread.sleep(2000);
            r3.stop();
        } catch (InterruptedException e) {

        }
    }
}

class RunImplEx16 implements Runnable {

    volatile boolean suspended = false;
    volatile boolean stopped = false;

    @Override
    public void run() {
        while (!stopped) {
            if (!suspended) {
                System.out.println(Thread.currentThread().getName());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {

                }
            }
        }
        System.out.println(Thread.currentThread().getName() + " - stopped ");
    }

    public void suspend() {
        suspended = true;
    }

    public void resume() {
        suspended = false;
    }

    public void stop() {
        stopped = true;
    }
}

```

```
**
*
***
*
***
**
***
**
***
**
***
***
***
*
***
*
***
*
***
** - stopped 
* - stopped 
***
***
*** - stopped 
```

아래의 예제는 위의 예제보다 더 객체지향적으로 개선하였다.

```
public class ThreadEx17 {
    public static void main(String[] args) {
        ThreadEx17_1 th1 = new ThreadEx17_1("*");
        ThreadEx17_1 th2 = new ThreadEx17_1("**");
        ThreadEx17_1 th3 = new ThreadEx17_1("***");
        th1.start();
        th2.start();
        th3.start();

        try {
            Thread.sleep(2000);
            th1.suspend();
            Thread.sleep(2000);
            th2.suspend();
            Thread.sleep(3000);
            th1.resume();
            Thread.sleep(3000);
            th1.stop();
            th2.stop();
            Thread.sleep(2000);
            th3.stop();
        } catch (InterruptedException e) {
            
        }
    }
}

class ThreadEx17_1 implements Runnable {
    boolean suspended = false;
    boolean stopped = false;
    Thread th;

    ThreadEx17_1(String name) {
        th = new Thread(this, name);
    }

    @Override
    public void run() {
        while (!stopped) {
            if (!suspended) {
                System.out.println(Thread.currentThread().getName());
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {

                }
            }
        }
        System.out.println(Thread.currentThread().getName() + " - stopped");
    }

    public void suspend() {
        suspended = true;
    }

    public void resume() {
        suspended = false;
    }

    public void stop() {
        stopped = true;
    }

    public void start() {
        th.start();
    }
}
```

```
*
**
***
*
**
***
*
**
***
**
***
**
***
***
***
*
***
*
***
*
***
** - stopped
* - stopped
***
***
*** - stopped
```


#### 8.5 `yield()` - 다른 쓰레드에게 양보
**`yield()`는 쓰레드 자신에게 주어진 실행시간을 다음 차례의 쓰레드에게 양보하도록 한다.** 예를 들어 스케쥴러에 의해 1초의 실행시간을 할당받은 쓰레드가 0.5초의 시간동안
작업한 상태에서 `yield()`가 호출되면, 나머지 0.5초는 포기하고 다시 실행대기상태가 된다. `yield()`가 호출되면, 나머지 0.5초는 포기하고 다시 실행대기상태가 된다.

`yield()`와 `interrupt()`를 적절히 사용하면, 프로그램의 응답성을 높이고 보다 효율적인 실행이 가능하게 할 수 있다. 이 메서드들이 실제로 어떻게 사용되는지 아래의 예제를
통해 알아보자.

```java
public class ThreadEx18 {
    public static void main(String[] args) {
        ThreadEx18_1 th1 = new ThreadEx18_1("*");
        ThreadEx18_1 th2 = new ThreadEx18_1("**");
        ThreadEx18_1 th3 = new ThreadEx18_1("***");
        th1.start();
        th2.start();
        th3.start();
        try {
            Thread.sleep(2000);
            th1.suspend();
            Thread.sleep(2000);
            th2.suspend();
            Thread.sleep(3000);
            th1.resume();
            Thread.sleep(3000);
            th1.stop();
            th2.stop();
            Thread.sleep(2000);
            th3.stop();
        } catch (InterruptedException e) {

        }
    }
}

class ThreadEx18_1 implements Runnable {
    boolean suspended = false;
    boolean stopped = false;
    Thread th;

    ThreadEx18_1(String name) {
        th = new Thread(this, name);
    }

    @Override
    public void run() {
        String name = th.getName();
        while (!stopped) {
            if (!suspended) {
                System.out.println(name);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    System.out.println(name + " - interrupted");
                }
            } else {    // yield() 추가
                Thread.yield();
            }
        }
        System.out.println(name + " - stopped");
    }

    public void suspend() {
        suspended = true;
        th.interrupt();
        System.out.println(th.getName() + " - interrupted() by suspend()");
    }

    public void resume() {
        suspended = false;
    }

    public void stop() {
        stopped = true;
        th.interrupt();
        System.out.println(th.getName() + " - interrupted() by stop()");
    }

    public void start() {
        th.start();
    }
    
}
```

```
*
**
***
*
**
***
*
**
* - interrupted
* - interrupted() by suspend()
***
**
***
**
** - interrupted() by suspend()
** - interrupted
***
***
***
*
***
*
***
*
***
* - interrupted() by stop()
** - stopped
* - interrupted
* - stopped
** - interrupted() by stop()
***
***
*** - interrupted() by stop()
*** - interrupted
*** - stopped
```

이전에 보았던 예제에 `yield()`와 `interrupt()`를 추가해서 예제의 효율과 응답성을 향상 시켰다. 먼저 `if`문에 `yield()`를 호출하는 `else`블럭이 추가되었다.

이전의 코드는 변수 `suspended`의 값이 `true`일 경우 잠시 실행을 멈추게 한 상태라면 쓰레드는 주어진 실행시간을 그거 `while`문을 의미없이 돌면서 낭비하게 될 것이다.
이런 상황을 **바쁜 대기 상태(busy-waiting)**이라고 한다. 하지만 위의 코드에서는 `suspended`의 값이 `true`일 경우 `yield()`를 호출해서 남은 실행시간을 `while`문에서
낭비하지 않고, 다른 쓰레드에게 양보를 하게 되므로 더 효율적이다.

또한 추가적으로 변경된 내용은 `suspend()`와 `stop()`인데 `interrupt()`를 호출하는 코드를 추가한 부분인데 이전 코드에서는 `stop()`이 호출되었을 때 `Thread.sleep(1000)`에
의해 쓰레드가 일시정지 상태에 머물러있는 상황이라면, `stopped`의 값이 `true`로 바뀌었어도 쓰레드가 정지될 때까지 최대 1초의 시간지연이 생길 것이다. 그러나 위의 코드에서
`interrupt()`를 호출하게 되면 `sleep()`에서 `InterruptedException`이 발생하여 즉시 일시정지상태를 벗어나게되므로 응답성이 좋아지게 된다.

#### 8.6 `join()` - 다른 쓰레드 작업을 기다림
**쓰레드 자신이 하던 작업을 잠시 멈추고 다른 쓰레드가 지정된 시간동안 작업을 수행하도록 할 때 사용한다**

```java
join()
join(long millis)
join(long millis, int nanos)
```

시간을 지정하지 않으면, 해당 쓰레드가 작업을 모두 마칠 때까지 기다리게 된다. 작업 중에 다른 쓰레드의 작업이 먼저 수행되어야할 필요가 있을 때 `join()`을 사용한다.

```java
try {
    th1.join(); // 현재 실행중인 쓰레드가 쓰레드 th1의 작업이 끝날 때까지 기다린다.
} catch(InterruptedException e) {
    
}
```

`join()`도 `sleep()`처럼 `interrupt()`에 의해 대기상태에서 벗어날 수 있으며, `join()`이 호출되는 부분을 `try-catch`부분으로 감싸야만한다.

**`join()`은 `sleep()`과 유사한 점이 많다. 하지만 `sleep()`과 다른점은 `join()`은 현재 쓰레드가 아닌 특정 쓰레드에 대해 동작하므로 `static`메서드가 아니라는 것이다.

```java
public class ThreadEx19 {

    static long startTime = 0;

    public static void main(String[] args) {
        ThreadEx19_1 th1 = new ThreadEx19_1();
        ThreadEx19_2 th2 = new ThreadEx19_2();

        th1.start();
        th2.start();
        startTime = System.currentTimeMillis();

        try {
            th1.join(); // 메인쓰레드가 th1의 작업이 끝날때까지 기다림
            th2.join(); // 메인쓰레드가 th2의 작업이 끝날때까지 기다림
        } catch (InterruptedException e) {

        }
        System.out.println("소요시간 : " + (System.currentTimeMillis() - ThreadEx19.startTime));
    }

}

class ThreadEx19_1 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 300; i++) {
            System.out.print(new String("-"));
        }
    }
}

class ThreadEx19_2 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 300; i++) {
            System.out.print(new String("|"));
        }
    }
}
```
```
// 출력내용 일부 생략
-------------|||||||||-----------------------|||||||||||||--------------||||-------------||||||||||||||||||||||||||||||||||||소요시간 : 6
```

`join()`을 사용하지 않았으면 `main`쓰레드는 바로 종료되었겠지만 `join()`으로 쓰레드 `th1`과 `th2`의 작업을 마칠 때까지 `main`쓰레드가 기다리도록 했다. 그래서 `main`쓰레드가 두 쓰레드의 작업에
소요된 시간을 출력할 수 있다.

위의 예제만으로는 `join()`을 언제 사용해야하는지 감이 오지 않기 때문에 아래의 예제를 통해 `join()`이 어떻게 사용되는지 보자.

```java
public class ThreadEx20 {
    public static void main(String[] args) {
        ThreadEx20_1 gc = new ThreadEx20_1();
        gc.setDaemon(true); // 데몬쓰레드로 변경
        gc.start();

        int requiredMemory = 0;

        for (int i = 0; i < 20; i++) {고
            requiredMemory = (int) (Math.random() * 10) * 20;
            
            // 필요한 메모리가 사용할 수 있는 양보다 적거나 전체 메모리의 60%이상을 사용했을 경우 gc를 깨움
            if (gc.freeMemory() < requiredMemory || gc.freeMemory() < gc.totalMemory() * 0.4) {
                gc.interrupt(); // 잠자고 있는 쓰레드 gc를 깨움
            }

            gc.usedMemory += requiredMemory;
            System.out.println("usedMemory : " + gc.usedMemory);
        }
    }
}

class ThreadEx20_1 extends Thread {
    final static int MAX_MEMORY = 1000;
    int usedMemory = 0;

    @Override
    public void run() {
        // 10초마다 가비지 컬렉션 수행
        while (true) {
            try {
                Thread.sleep(10 * 1000);
            } catch (InterruptedException e) {
                System.out.println("Awaken by interrupt().");
            }
            gc(); // 가비지 컬렉션을 수행
            System.out.println("Garbage Collected, Free Memory : " + freeMemory());
        }

    }

    public void gc() {
        usedMemory -= 300;
        if (usedMemory < 0) usedMemory = 0;
    }

    public int totalMemory() {
        return MAX_MEMORY;
    }

    public int freeMemory() {
        return MAX_MEMORY - usedMemory;
    }
}
```

```
usedMemory : 80
usedMemory : 120
usedMemory : 300
usedMemory : 420
usedMemory : 540
usedMemory : 580
usedMemory : 580
usedMemory : 740
usedMemory : 920
usedMemory : 940
Awayken by interrupt().
usedMemory : 1040
Garbage Collected, Free Memory : 260
usedMemory : 860
Awayken by interrupt().
usedMemory : 880
usedMemory : 620
usedMemory : 760
usedMemory : 780
usedMemory : 960
usedMemory : 1020
usedMemory : 1040
usedMemory : 1080
```

위의 예제는 JVM의 가비지 컬렉터를 흉내내어 간단히 구현해본 것으로 `random()`을 사용했기 때문에 실행결과가 다를 수 있다. 먼저 `sleep()`을 이용해서 10초마다 한번씩 가비지 컬렉션을 수행하는 쓰레드를 만든 다음,
쓰레드를 생성해 데몬쓰레드로 설정하였다. 반복문을 사용해서 메모리의 양을 계속 감소 시키도록 했고, 매 만복마다 if문을 메모리를 확인해서 남은 메모리가 전체 메모리의 40%미만일 경우 `interrupt()`를 호출해서 즉시
가비지 컬렉터 쓰레드를 깨워서 `gc()`를 수행하도록 하였다.

하지만 예제의 실행결과를 보면 `MAX_MEMORY`가 1000임에도 불구하고 `usedMemory`의 값이 1000을 넘는 것을 알 수 있다. 이것은 쓰레드 `gc`가 `interrupt()`에 의해서 깨어났음에도 불구하고 `gc()`가 수행되기 이전에
`main`쓰레드의 작업이 수행되어 메모리를 사용하기 때문이다. 그래서 쓰레드 `gc`를 깨우는 것 뿐만아니라 `join()`을 이용해서 쓰레드 `gc`가 작업할 시간을 어느 정도 주고 `main`쓰레드가 기다리도록 해서, 사용할 수 있는
메모리가 확보된 다음에 작업을 계속하는 것이 필요하다.

```java
if (gc.freeMemory() < requiredMemory || gc.freeMemory() < gc.totalMemory() * 0.4) {
    gc.interrupt();
    try {
        gc.join(100);
    } catch (InterruptedException e) {
        
    }
}    
```

그래서 예제를 위의 코드와 같이 `join()`을 호출해서 쓰레드 `gc`가 0.1초 동안 수행될 수 있도록 변경해야한다.

```
usedMemory : 20
usedMemory : 160
usedMemory : 280
usedMemory : 420
usedMemory : 440
usedMemory : 540
usedMemory : 600
usedMemory : 780
Awaken by interrupt().
Garbage Collected, Free Memory : 520
usedMemory : 660
Awaken by interrupt().
Garbage Collected, Free Memory : 640
usedMemory : 520
usedMemory : 700
Awaken by interrupt().
Garbage Collected, Free Memory : 600
usedMemory : 480
usedMemory : 660
Awaken by interrupt().
Garbage Collected, Free Memory : 640
usedMemory : 520
usedMemory : 660
Awaken by interrupt().
Garbage Collected, Free Memory : 640
usedMemory : 440
usedMemory : 620
Awaken by interrupt().
Garbage Collected, Free Memory : 680
usedMemory : 480
usedMemory : 580
usedMemory : 760
```

코드를 변경 하고 예제를 다시 실행시켜보면 위와 같이 메모리가 1000을 넘지 않는 것을 확인할 수 있다.

가비지 컬렉터와 같은 데몬 쓰레드의 우선순위를 낮추기보다는 `sleep()`을 이용해서 주기적으로 실행되도록 하다가 필요할 때마다 `interrupt()`를 호출해서 즉시 가비지 컬렉션이 이루어지도록 하는 것이 좋다.
**그리고 필요하다면 `join()`도 함께 사용해야한다는 것을 기억하자**

## 9. 쓰레드의 동기화

싱글쓰레드 프로세스의 경우 프로세스 내에서 단 하나의 쓰레드만을 작업하기 때문에 프로세스 자원을 가지고 작업하는데 별 문제가 없지만, 멀티쓰레드 프로세스의 경우 여러 쓰레드가 같은 프로세스 내의 자원을 공유해서
작업하기 때문에 서로의 작업에 영향을 주게된다.

만약 쓰레드A가 작업하던 도중에 다른 쓰레드B에게 제어권이 넘어갔을 때 쓰레드A가 작업하던 공유데이터를 쓰레드B가 임의로 변경했다면 다시 쓰레드A가 제어권을 받아서 나머지 작업을 마쳤을 때 원래 의도했던 것과는 다른
결과를 얻게 될 수 있다.

이러한 문제가 발생하는 것을 방지하기 위해 한 쓰레드가 특정 작업을 마치기 전까지 다른 쓰레드에 의해 방해받지 않도록 하는 것이 필요하다. 그래서 도입된 개념이 **임계영역(critical section)과 잠금(lock)** 이다.

공유데이터를 사용하는 코드 영역을 임계영역으로 지정해놓고, 공유데이터(객체)가 가지고 있는 lock을 획득한 단 하나의 쓰레드만 이 영역 내의 코드를 수행할 수 있도록 한다. 그리고 해당 쓰레드가 임계 영역내의 모든 코드를
수행하고 벗어나서 lock을 반납해야 비로소 다른 쓰레드가 반납된 lock을 획득하여 임계 영역의 코드를 수행할 수 있게된다.

**위에서 설명한 것처럼 한 쓰레드가 진행 중인 작업을 다른 쓰레드가 간섭하지 못하도록 막는 것을 쓰레드의 동기화(synchronization)라고 한다.** 자바에서는 `synchronized`블럭을 이용해서 쓰레드 동기화를 지원했지만,
JDK1.5부터는 `java.util.concurrent.locks`와 `java.util.concurrent.atomic`패키지를 통해서 다양한 방식으로 동기화를 구현할 수 있도록 지원하고 있다.

#### 9.1 `synchronized`를 이용한 동기화

**`synchronized`라는 키워드는 임계영역을 설정하는데 사용하며 아래와 같이 2가지 방식이 있다.**

1. 메서드 전체를 임계영역으로 지정
    ```java
    public synchronized void calcSum() {
    // ...
    }
    ```
    - 쓰레드는 `synchronized` 메서드가 호출된 시점부터 해당 메서드가 포함된 객체의 lock을 얻어 작업을 수행하다가 메서드가 종료되면 lock을 반환

2. 특정한 영역을 임계 영역으로 지정
    ```java
    synchronized(객체 참조변수) {
    // ...   
    }
    ```
    - 메서드 내의 코드 일부를 블럭으로 감싸고 블럭 앞에서 `synchronized(참조변수)`를 붙이는데 이때 참조변수는 lock을 걸고자하는 객체를 참조하는 것이어야함
    - 이 블럭을 `synchronized`블럭이라고 함
    - 이 블럭의 영역 안에 들어가면서부터 쓰레드는 지정된 객체의 lock을 얻게되고, 이 블럭을 벗어나면 lock을 반납함

두 방법 모두 lock의 획득과 반납이 자동적으로 이루어지므로 해야할 일은 그저 임계영역만 설정해주는 것뿐이다. 모든 객체는 lock을 하나씩 가지고 있으며, 해당 객체의 lock을 가지고 있는 쓰레드만 임계 영역의 코드를 수행할 수 있다.
그리고 나머지 쓰레드들은 loack을 얻을 때까지 기다리게 된다.

임계 영역은 멀티 쓰레드 프로그램의 성늘을 좌우하기 때문에 가능하면 메서드 전체에 락을 거는 것보다 `synchronized`블럭으로 임계 영역을 최소화하는 것이 바람직하다.

아래는 동기화가 왜 필요한지를 보여주는 예제이다.

```java
public class ThreadEx21 {
    public static void main(String[] args) {
        Runnable r = new RunnableEx21();
        new Thread(r).start();
        new Thread(r).start();
    }
}

class Account {
    private int balance = 1000;

    public int getBalance() {
        return balance;
    }

    public void withdraw(int money) {
        if (balance >= money) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {

            }
            balance -= money;
        }
    }
}

class RunnableEx21 implements Runnable {

    Account account = new Account();

    @Override
    public void run() {
        while (account.getBalance() > 0) {
            int money = (int) (Math.random() * 3 + 1) * 100;
            account.withdraw(money);
            System.out.println("money : " + money);
            System.out.println("balance : " + account.getBalance());
        }
    }
}
```
```
money : 300
balance : 700
money : 200
balance : 500
money : 300
balance : 200
money : 300
balance : 200
money : 100
balance : 100
money : 100
balance : 0
money : 100
balance : -100
```

위의 코드는 은행계좌에서 잔고를 확인하고 임의로 금액을 출금하도록 작성되었다. `Account`클래스의 `withdraw()`메서드를 보면 잔고가 출금하려는 금액보다 클 경우에만 출금이 가능하도록 조건문을 작성했다.
하지만 위의 콘솔출력결과를 보면 잔고가 음수인 것을 확인할 수 있다. 그 이유는 한 쓰레드가 if문의 조건식을 통과하고 출금하기 바로 직전에 다른 쓰레드가 끼어들어서 출금을 먼저 했기 때문이다. 그래서 잔고를
확인하는 if문과 출금하는 문장은 하나의 임계 영역으로 묶여져야 한다. 이처럼 **한 쓰레드의 작업이 다른 쓰레드에 의해서 영향을 받는 일이 발생할 수 있기 때문에 동기화가 반드시 필요하다.**

```java
// 메서드 동기화
public synchronized void withdraw(int money) {
    if (balance >= money) {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {

        }
        balance -= money;
    }
}
```

그래서 `withdraw()`메서드에 `synchronized`키워드를 위와 같이 붙이기만 하면 간단히 동기화가 된다. 한 쓰레드에 의해 먼저 `withdraw()`가 호출되면, 이 메서드가 종료되어 lock이 반납될 때까지 다른 쓰레드는 `withdraw()`를
호출하더라도 대기상태에 머물게 된다.

메서드 앞에 `synchronized`를 붙이는 대신, `synchronized`블럭을 사용하면 다음과 같이 작성할 수도 있다.

```java
public void withdraw(int money) {
    synchronized(this) {
        if (balance >= money) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
    
            }
            balance -= money;
        }
    }
}
```

위와 같이 코드를 수정하고 다시 실행하면 아래와 같이 의도한대로 결과가 출력된다.

```
money : 100
balance : 900
money : 100
balance : 800
money : 200
balance : 600
money : 200
balance : 400
money : 300
money : 200
balance : 100
balance : 100
money : 200
balance : 100
money : 200
balance : 100
money : 200
balance : 100
money : 300
balance : 100
money : 300
balance : 100
money : 200
balance : 100
money : 100
balance : 0
money : 300
balance : 0
```

#### 9.2 `wait()`와 `nofify()`

#### 9.3 `Lock`과 `Condition`을 이용한 동기화

#### 9.4 `volatile`

#### 9.5 `fork`, `join` 프레임워크
