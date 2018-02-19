## 1. 프로세스와 쓰레드

#### 프로세스(process), 쓰레드(thread)
**프로세스를 간단히 말해서 실행 중인 프로그램이라고 할 수 있다.** 프로그램을 실행하면 OS로부터 실해엥 필요한 자원(메모리)를 할당받아 프로세스가 된다. 프로세스는 프로그램을 수행하는데 필요한 데이터와 메모리 등의 자원 그리고 쓰레드로 구성되어 있으며 **프로세스의 자원을 이용해서 실제로 작업을 수행하는 것이 바로 쓰레드이다.**

#### 멀티태스킹과 멀티쓰레딩
**멀티태스킹(Multi-tasking)이란 여러 개의 프로세스가 동시에 실행되는 것을 말한다.** 윈도우나 유닉스와 같은 대부분의 OS에서 멀티태스킹을 지원한다. 이와 마찬가지로 **멀티쓰레딩(Multi-threading)은 하나의 프로세스 내에서 여러 쓰레드가 동시에 작업을 수행하는 것을 말한다.**

#### 멀티쓰레딩의 장단점
##### 장점
- CPU의 사용률을 향상시킨다.
- 자원을 보다 효율적으로 사용할 수 있다.
- 사용자에 대한 응답성이 향상된다.
- 작업이 분리되어 코드가 간결해진다.

##### 단점
멀티쓰레드 프로세스는 여러 쓰레드가 같은 프로세스 내에서 자원을 공유하면서 작업을 하기 때문에 발생할 수 있는 동기화(synchronization), 교착상태(deadrock)와 같은 문제들을 고려해야한다.

## 2. 쓰레드의 구현과 실행방법

#### 쓰레드 구현방법 1 : `Thread`클래스 상속
**`Thread`을 상속받으면 다른 클래스를 상속받을 수 없다.** 그래서 보통 `Runnable`인터페이스를 구현하는 방법이 일반적이다.
```java
// Thread클래스 외에 다른 클래스 상속을 할 수 없음.
class MyThread extends Thread {
  public void run() { // Thread클래스의 run()을 오버라이딩
    // ...
  }
}
```

#### 쓰레드 구현방법 2 : `Runnable`인터페이스 구현
**`Runnable`인터페이스는 오로지 `run()`만 정의되어 있는 간단한 인터페이스이다.** `Runnable`인터페이스를 구현하기 위해서 해야할 일을 `run()`의 `{}`을 만들어 주는 것뿐이다.
```java
class MyThread implements Runnable {
  public void run() { // Runnable인터페잇의 추상메서드 run() 구현
    // ...
  }
}
```

#### `Thread` 예제 1 : `Thread`와 `Runnable`의 구현의 차이점
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


class ThreadEx1_1 extends Thread {

    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(getName());  // Thread의 getName호출
        }
    }

}

class ThreadEx1_2 implements Runnable {

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

`Thread`클래스를 상속받은 경우와 `Runnable`인터페이스를 구현한 경우 인스턴스 생성방법이 다르다. **`Runnable`인터페이스를 구현한 경우, `Runnable`인터페이스를 구현한 클래스의 인스턴스를 생성한 다음, 이 인스턴스를 `Thread`클래스의 생성자의 매개변수로 제공해줘야 한다.** 아래의 코드는 실제 `Thread`클래스의 소스크도를 알기 쉽게 일부를 수정한 것인데 인스턴스 변수로 `Runnable`타입의 변수 `r`을 선언해 놓고, 생성자를 통해 `Runnable`인터페이스를 구현한 인스턴스를 참조하도록 되어 있는 것을 볼 수 있다. 그리고 `run()`을 호출하면 참조변수 `r`을 통해 `Runnable`인터페이스를 구현한 인스턴스의 `run()`이 호출된다. 이렇게 함으로써 상속을 통해 `run()`을 오버라이딩하지 않고도 외부로부터 `run()`을 제공받을 수 있게된다.

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

`Thread`클래스를 상속받으면, 자손 클래스에서 조상인 `Thread`클래스의 메서드를 직접 호출할 수 있지만, **`Runnable`을 구현하면 `Thread`클래스의 `static`메서드인 `currentThread()`를 호출하며 쓰레드에 대한 참조를 얻어와야만 호출이 가능하다.**
```java
static Thread currentThread() // 현재 실행중인 쓰레드의 참조를 반환
String getName() // 쓰레드의 이름을 반환
```

**쓰레드를 생성했다고 해서 자동으로 쓰레드가 실행되는 것이 아니라 `start()`를 호출해야만 쓰레드가 실행된다.** `start()`가 호출 되었다고 해서 바로 실행되는 것이 아니라, 일단 실행대기 상태에서 있다가 자신의 차례가 되어야 비로소 실행이 된다. **주의해야할 점은 한번 실행이 종료된 쓰레드는 다시 실행할 수 없다는 점이다.** 다시 말하자면 하나의 쓰레드에 대해 `start()`가 한번만 호출될 수 있다는 뜻이다. 만약 쓰레드 작업을 한번 더 해야한다면 아래의 코드처럼 새로운 쓰레드를 생성한 후에 `start()`를 호출해야 한다.
```java
ThreadEx1_1 t1 = new ThreadEx1_1();
t1.start();
// 다시 생성
t1 = new ThreadEx1_1();
t1.start();
```

## 3. `start()`와 `run()`의 차이

앞서 쓰레스를 실행시킬 때 `run()`이 아닌 `start()`를 호출하는 것에 대해 다소 의문 들었을 것이다. 이제 `start()`와 `run()`의 차이와 쓰레드가 실행되는 과정에 대해서 알아보자.

`main()`에서 `run()`을 호출하는 것은 생성된 쓰레드를 실행시키는 것이 아니라, 단순히 클래스에 선언된 메서드를 호출하는 것일 뿐이다. 반면에 `start()`는 새로운 쓰레드가 작업을 실행하는데 필요한 호출스택을 생성한 다음에 `run()`을 호출해서 생성된 호출스택에 `run()`이 첫번째로 올라가게 한다. 모든 쓰레드는 독립적인 작업을 수행하기 위해 자신만의 호출 스택을 필요로 하기 때문에 새로운 쓰레드를 생성하고 실행시킬 때마다 새로운 호출스택이 생성되고 쓰레드가 종료되면 작업에 사용된 호출스택은 소멸된다.

## 4. 싱글 쓰레드와 멀티 쓰레드
## 5. 쓰레드의 우선순위
## 6. 쓰레드 그룹
## 7. 데몬 쓰레드
## 8. 쓰레드의 실행제어
## 9. 쓰레드의 동기화
