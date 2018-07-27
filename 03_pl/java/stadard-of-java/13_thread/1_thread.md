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

#### 4.1 싱글쓰레드와 멀티쓰레드의 차이 예제 1

```java
```

## 5. 쓰레드의 우선순위

## 6. 쓰레드 그룹

## 7. 데몬 쓰레드

## 8. 쓰레드의 실행제어

## 9. 쓰레드의 동기화

#### `synchronized`를 이용한 동기화

#### `wait()`와 `nofify()`

#### `Lock`과 `Condition`을 이용한 동기화

#### `volatile`

#### `fork`, `join` 프레임워크
