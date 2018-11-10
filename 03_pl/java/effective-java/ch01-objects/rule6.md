> 본 글은 [이펙티브 자바 2nd](https://book.naver.com/bookdb/book_detail.nhn?bid=8064518)를
읽고 개인적으로 학습한 내용을 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다.
오류가 있다면 지적 부탁드리겠습니다.

# rule 6. 유효기간이 지난 객체 참조는 폐기할 것

자바의 경우 가비지 컬렉터로 인해 쓰지않는 객체는 자동으로 반환된다. 하지만 메모리 관리는
필요하다.

## 6.1 메모리 누수의 예

```java
public class Stack {

  private Object[] elements;
  private int size;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }

  public Object pop() {
    if (size == 0) {
      throw new EmptyStackException();
    }
    return elements[--size];
  }

  public void ensureCapacity() {
    if (elements.length == size) {
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }
  }

}
```

위의 코드는 문제 없이 잘 작동하지만 하나의 문제는 메모리 누수가 발생한다는 것이다.
그 결과로 가비지 컬렉터가 할 일이 많아져 성능이 저하되거나, 메모리 요구량이 증가한다.

그렇다면 어디에서 메모리 누수가 발생할까? 스택이 커졌다 줄어들면서 제거한 객체들을 가비지
컬렉터에서 처리하지 못해 발생한다. 스택이 객체에 대한 만기 참조를 제거하지 않기 때문이다.
*만기 참조란 다시 이용되지 않을 참조를 말한다.*

자동적으로 쓰레기 객체를 수집하는 언어에서 발생하는 메모리 누수 문제는 찾아내기 어렵기
때문에 성능에 끼칠 잠재적 악영향을 무시할 수 없다.

```java
public Object pop() {
  if (size == 0) {
    throw new EmptyStackException();
  }
  Object results = elements[--size];
  elements[size] = null; // 만기 참조 제거
  return results;
}
```

위의 코드와 같이 쓸 일 없는 객체 참조는 무조건 `null`로 만들어 문제를 해결할 수 있다.
만기 참조를 `null`로 만들면 나중에 실수로 그 참조를 사용하더라도 `NullPointerException`이
발생하기 때문에, 프로그램은 오작동하는 대신 바로 종료되는 장점이 있다.

하지만 너무나 만기참조를 `null`로 만드는 것에 집착하기보다는 예외적인 조치가 되어야한다.
그렇지 않으면 프로그램만 난잡해지기 때문이다. 만기참조를 제거하는 가장 좋은 방법은 유효범위를
벗어나게 두는 것이다. 변수를 정의할 때 그 유효범위를 최대한 좁게 만들면 자연스레 해결된다.

## 6.2 메모리 누수에 주의해야 할 곳과 해결책

### 6.2.1 자체적으로 관리하는 메모리가 있는 클래스

객체참조는 `null`로 반드시 바꾼다.

### 6.2.2 캐시(Cache)

객체 참조를 캐시 안에 넣고 잊어버리는 경우가 많다. `WekHashMap`을 가지고 캐시를 구현한다.
캐시 바깥에서 키를 참조하고 있을 때만 값을 보관하면 될 때 쓸 수 있는 전략으로 키에 대한
참조가 만기 참조가 되는 순간 캐시안에 보관된 키-값 쌍은 자동으로 삭제된다.

### 6.2.3 리스너(Listner)등의 역호출자(Callback)

콜백을 명시적으로 제거하지 않은 경우, 적절한 조치를 취하기 전까지 메모리는 점유된 상태로
남아있는다. 그렇기 때문에 역호출자에 대한 약한 참조만 저장하도록 하는 것이 좋다.


## 6.3 요약

메모리 누수는 보통 뚜렷한 오류로 이어지지 않기 때문에 주의 깊게 코드를 검토하다가 발견되거나
힙 프로파일러 같은 도구를 통해 검증하다가 발견된다. 따라서 이런 문제가 생길 수 있다는 것을
사전에 인지하고 방지 대첵을 세우는 것이 좋다.
