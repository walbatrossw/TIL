본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## `Comparator`과 `Comparable`이란?
이전에 본 `Arrays.sort()`는 호출만 하면 알아서 배열을 정렬하는 것처럼 보였지만 사실은 `Character`클래스의 `Comparable`의 구현에 의해 정렬된 것이다. **`Comparator`, `Comparable`는 모두 인터페이스로 컬렉션을 정렬하는데 필요한 메서드를 정의** 하고 있으며, **`Comparable`을 구현하고 있는 클래스** 들은 같은 타입의 **인스턴스끼리 비교할 수 있는 클래스들, 주로 `Integer`와 같은 `Wrapper`클래스와 `String`, `Date`, `File`과 같은 것들** 이며 기본적으로 **오름차순으로 정렬** 되도록 구현되어 있다.

```java
// java.lang 패키지에
public interface Comparable {
  // 객체를 비교하여 같으면 0, 비교한 값보다 작으면 음수, 크면 양수
  int compareTo(Object o);
}
```
- `Comparable` : 기본 정렬기준을 구현하는데 사용
```java
// java.util 패키지에
public interface Comparator {
  // 객체를 비교하여 같으면 0, 비교한 값보다 작으면 음수, 크면 양수
  int compare(Object o1, Object o2);
  boolean equals(Object obj);
}
```
- `Comparator` : 기본 정렬기준 외에 다른 기준으로 정렬하고자 할 때 사용

##### `Comparator`, `Comparable` 예제
```java
public class ComparatorEx {
    public static void main(String[] args) {
        String[] strArr = {"cat", "Dog", "lion", "tiger"};

        Arrays.sort(strArr); // String의 Comparable 구현에 의한 정렬
        System.out.println("strArr = " + Arrays.toString(strArr));

        Arrays.sort(strArr, String.CASE_INSENSITIVE_ORDER); // 대문자 구분X
        System.out.println("strArr = " + Arrays.toString(strArr));

        Arrays.sort(strArr, new Descending());  // 역순정렬
        System.out.println("strArr = " + Arrays.toString(strArr));
    }
}

class Descending implements Comparator {
    public int compare(Object o1, Object o2) {
        if (o1 instanceof Comparable && o2 instanceof Comparable) {
            Comparable c1 = (Comparable) o1;
            Comparable c2 = (Comparable) o2;
            // return c2.compareTo(c1);     // 역순 정렬
            return c1.compareTo(c2) * -1;   // 역순 정렬
        }
        return -1;
    }
}
```
```
strArr = [Dog, cat, lion, tiger]
strArr = [cat, Dog, lion, tiger]
strArr = [tiger, lion, cat, Dog]
```
- `Arrays.sort()`는 배열을 정렬할 때, `Comparator`를 지정해주지 않으면 저장하는 객체에 구현된 내용에 따라 정렬된다.
  - `static void sort(Object[] o)` : 객체 배열에 저장된 객체가 구현한 `Comparable`에 의한 정렬
  - `static void sort(Object[] o, Comparator c)` : 지정한 `Comparator`에 의한 정렬
- `String`의 `Comparable`구현은 문자열이 사전 순으로 정렬되도록 작성되어 있다.
  - 위의 예제와 같이 대소문자를 구분하지 않고 비교하는 `Comparator`를 상수의 형태로 제공한다.
- `String`의 기본정렬형태를 반대로하는 것, 즉 내림차순으로 구현하는 방법은 간단하다.
  - `String`에 구현된 `compareTo()`의 결과에 -1을 곱하기만 하면된다.
  - 또는 비교하는 객체의 위치를 바꿔서 `c2.compareTo(c1)`과 같이 해도 된다.
  - `compare()`의 매개변수 타입이 `Object`이기때문에 `compareTo()`를 바로 호출할 수 없으므로 먼저 `Comparable`로 형변환을 꼭 해주어야한다.
