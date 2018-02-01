## 1.3 LinkedList

### 1) `Linked List`, `Doubly Linked List`, `Doubly Circular Linked List`?
배열은 기장 기본적인 형태의 자료구조로 구조가 간단하며, 사용하기 쉽고 데이터를 읽어오는데 걸리는 시간이 가장 빠르다는 장점을 가지고 있지만, 아래와 같은 단점이 존재한다.

**배열의 단점**
- 크기를 변경할 수 없다.
  - 크기를 변경할 수 없으므로 새로운 배열을 생성해 데이터를 복사하는 작업이 필요하다.
  - 실행속도를 향상시키기 위해서는 충분히 큰 크기의 배열을 생성해야하므로 메모리가 낭비가 될 수 있다.
- 비순차적인 데이터의 추가, 삭제에 시간이 많이 걸린다.
  - 차례대로 데이터를 추가하고, 삭제하는 것은 빠르다.
  - 배열의 중간에 데이터를 추가하려면 빈자리를 만들기 위해 다른 데이터를 복해서 이동해야한다.

이러한 **배열의 단점을 보완하기 위해서 `LinkedList`라는 자료구조가 고안** 되었다. **배열은 모든데이터가 연속적으로 존재** 하지만 **링크드 리스트는 불연속적으로 존재하는 데이터를 서로 연결한 형태** 로 구성되 있다.

**`LinkedList` 형태**

```
Array :  0 1 2 3 4

LinkedList : 0 -> 1 -> 2 -> 3 -> 4
```

각 요소들은 자신과 연결된 다름 요소에 대한 참조(주소값)와 데이터로 구성되있다.
```java
class Node {
  Node next; // 다음 요소의 주소를 저장
  Object obj; // 데이터를 저장
}
```

**`LinkedList` 데이터 삭제**

```
0 -> 1 -> 2 -> 3 -> 4
0 -> 1 -> 2 -┐ 3 ┌-> 4
             └---┘
```

`LinkedList`에서의 데이터 삭제는 간단하다. 삭제하고자하는 요소의 이전요소가 삭제하고자하는 요소의 다음요소를 참조하도록 변경하기만 하면된다. 단하나의 참조만 변경하면 삭제가 이루어지기 때문에 배열처럼 데이터를 이동하기 위해 복사하는 과정없이 빠르게 처리가 가능하다.

**`LinkedList` 데이터 추가**
```
0 -> 1 -> 2 -> 3 -> 4
0 -> 1 -> 2 -┐      ┌-> 3 -> 4
             └-> 5 -┘
```
새로운 데이터를 추가할 때는 새로운 요소를 생성한 다음 추가하고자 하는 위치의 이전요소의 참조를 새로운 요소에 대한 참조로 변경해주고, 새로운 요소가 그 다음 요소를 참조하도록 변경하기만 하면되므로 처리속도가 매우 빠르다.

**`Doubly Linked List`(이중연결리스트)**
`LinkedList`는 이동방향이 단방향이기 때문에 다음요소에 대한 접근은 쉽지만 이전요소에 대한 접근이 어렵다. 이러한 단점을 보완한 것이 `Doubly Linked List`(이중연결 리스트)이다. 단순히 `LinkedList`리스트에 이전요소에 대한 참조가 가능하도록만 했을 뿐 다른점은 없다.
```java
class Node {
  Node next;  // 다음요소의 주소를 저장
  Node prev;  // 이전요소의 주소를 저장
  Object obj; // 데이터를 저장
}
```
**`Doubly Linked List` 형태**
```
LinkedList         : 0 --> 1 --> 2 --> 3 --> 4
Doubly Linked List : 0 <-> 1 <-> 2 <-> 3 <-> 4
```

**`Doubly Circular Linked List`(이중 원형 연결리스트)**
`Doubly Linked List`의 접근성을 향상시킨 것으로, 단순히 `Doubly Linked List`의 첫번째 요소와 마지막 요소를 서로 연결시킨 것이다. 마지막 요소의 다음 요소는 첫번째 요소가 되고, 첫번째 요소의 이전 요소는 마지막 요소가 되는 것인데 마치 TV채널의 마지막에서 채널을 증가시키면 첫번째 채널로 이동하는것과 같다고 생각하면 된다.

**`Doubly Circular Linked List`형태**
```
Doubly Linked List          : 0 <-> 1 <-> 2 <-> 3 <-> 4
Doubly Circular Linked List : ┌-> 0 <-> 1 <-> 2 <-> 3 <-> 4 <-┐
                              └-------------------------------┘
```

### 2) `Linked List` 예제 1 : `ArrayList` VS, `LinkedList` 데이터 추가/삭제(순차, 중간)
```java
public class ArrayListLinkedListTest {
    public static void main(String[] args) {
        ArrayList arrayList = new ArrayList(20000000);
        LinkedList linkedList = new LinkedList();

        System.out.println("순차적으로 추가");
        System.out.println("ArrayList : " + add1(arrayList));
        System.out.println("linkedList : " + add1(linkedList));

        System.out.println();

        System.out.println("중간에 추가");
        System.out.println("ArrayList : " + add2(arrayList));
        System.out.println("linkedList : " + add2(linkedList));

        System.out.println();

        System.out.println("순차적으로 삭제");
        System.out.println("ArrayList : " + remove2(arrayList));
        System.out.println("linkedList : " + remove2(linkedList));

        System.out.println();

        System.out.println("중간에 삭제");
        System.out.println("ArrayList : " + remove1(arrayList));
        System.out.println("linkedList : " + remove1(linkedList));


    }

    public static long add1(List list) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++)
            list.add(i+"");
        long end = System.currentTimeMillis();
        return end - start;
    }

    public static long add2(List list) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++)
            list.add(500, "X");
        long end = System.currentTimeMillis();
        return end - start;
    }

    public static long remove1(List list) {
        long start = System.currentTimeMillis();
        for (int i = list.size() - 1; i >= 0; i--)
            list.remove(i);
        long end = System.currentTimeMillis();
        return end - start;
    }

    public static long remove2(List list) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++)
            list.remove(i);
        long end = System.currentTimeMillis();
        return end - start;
    }
}
```
```
순차적으로 추가
ArrayList : 186
linkedList : 844

중간에 추가
ArrayList : 2108
linkedList : 10

순차적으로 삭제
ArrayList : 2033
linkedList : 118

중간에 삭제
ArrayList : 6
linkedList : 16
```
- 결론1 : 순차적으로 추가/삭제하는 경우 `Array List`가 `Linked List`보다 빠르다.
  - 단순히 데이터를 저장하는 시간만 비교했지만, 저장할 데이터의 갯수만큼 저장공간이 확보되있지 않을 경우, `ArrayList`를 새로 생성해서 데이터를 복사하게 되면 `LinkedList`가 더 빠르게 될 수도 있다.
- 결론2 : 중간에 데이터를 추가/삭제하는 경우 `Linked List`가 `Array List`보다 빠르다.
  - `ArrayList`와 `LinkedList`의 차이를 비교하기 위해서 데이터의 갯수를 크게 잡았지만 만약 데이터의 갯수가 크지 않다면 어느 것을 사용해도 차이는 그리 크지 않다.

### 3) `Linked List` 예제 2 : `ArrayList` VS, `LinkedList` 데이터 접근시간 테스트
```java
public class ArrayListLinkedListTest2 {
    public static void main(String[] args) {
        ArrayList arrayList = new ArrayList(1000000);
        LinkedList linkedList = new LinkedList();
        add(arrayList);
        add(linkedList);

        System.out.println("access time test");
        System.out.println("Array List : " + access(arrayList));
        System.out.println("Linked List : " + access(linkedList));
    }

    public static void add(List list) {
        for (int i = 0; i < 100000; i++)
            list.add(i + "");
    }

    public static long access(List list) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++)
            list.get(i);
        long end = System.currentTimeMillis();
        return end - start;
    }
}
```
```
access time test
Array List : 5
Linked List : 9813
```
- 결론 : 데이터를 접근하는 시간은 `Array List`가 `Linked List`보다 빠르다.
  - 배열은 각 요소들이 연속적으로 메모리상에 존재하여 데이터의 주소를 얻어 저장된 데이터를 곧바로 읽어올 수 있지만, `LinkedList`는 불연속적으로 위치한 각 요소들이 연결된 형태라서 처음부터 n번째 데이터까지 차례대로 따라가야만 원하는 값을 얻을 수 있기 때문에 속도면에서 `Array List`보다 느릴수 밖에 없다.

### 4) `Array List`와 `Linked List` 차이점 정리
|컬렉션|읽기(접근시간)|추가/삭제|비고|
|---|---|---|---|
|`ArrayList`|빠르다|느리다|순차적인 추가삭제는 더 빠르나, 비효율적인 메모리를 사용|
|`LinkedList`|느리다|빠르다|데이터가 많을수록 접근성이 떨어짐|
