## 1.2 `ArrayList`

### 1) `ArrayList`?
`ArrayList`는 컬렉션 프레임워크에서 가장 많이 사용되는 컬렉션 클래스이다. `List`인터페이스를 구현하기 때문에 **데이터의 저장순서는 유지되고, 중복을 허용** 한다. `Object`배열을 이용하여 **데이터를 순차적으로 저장** 한다.

예를 들면, 첫 번째로 저장한 객체는 `Object`배열의 0번째에 저장되고 그 다음에 저장하는 객체는 1번째에 저장된다. 이런 식으로 계속 배열에 순서대로 저장되며, **배열에 더이상 저장할 공간이 없으면 보다 큰 새로운 배열을 생성하여 기존의 배열에 저장된 내용을 새로운 배열로 복사한 다음에 저장된다.**

---

### 2) `ArrayList` 예제 1
```java
public class ArrayListEx1 {
    public static void main(String[] args) {

        ArrayList list1 = new ArrayList(10);
        list1.add(new Integer(5));
        list1.add(new Integer(4));
        list1.add(new Integer(3));
        list1.add(new Integer(2));
        list1.add(new Integer(0));
        list1.add(new Integer(1));
        list1.add(new Integer(3));

        // list2 에 list1에 저장된 객체 중에서 1번째부터 4번째까지 저장
        ArrayList list2 = new ArrayList(list1.subList(1, 4));
        print(list1, list2);

        // list1, list2 정렬
        Collections.sort(list1);
        Collections.sort(list2);
        print(list1, list2);

        // list1이 list2의 모든요소를 포함하고 있는지 확인
        System.out.println("list1.containsAll(list2) : " + list1.containsAll(list2));

        // 새로운 객체를 추가
        list2.add("B");
        list2.add("C");
        list2.add(3, "A");  // 3번째에 새로운 객체 추가하고, 3번째부터 저장된 객체들은 4번째 요소부터 저장됨
        print(list1, list2);

        // 3번째 요소에 새로운 객체로 교체
        list2.set(3, "AA");
        print(list1, list2);

        // list1, list2에 저장된 객체중에서 주어진 컬렉션과 공통된 것만 남기고 나머지는 삭제
        System.out.println("list1.retainAll(list2) : " + list1.retainAll(list2));
        print(list1, list2);


        for (int i = list2.size() - 1; i >= 0; i--) {
            if (list1.contains(list2.get(i)))
                list2.remove(i);
        }
        print(list1, list2);
    }

    static void print(ArrayList list1, ArrayList list2) {
        System.out.println("list1 : " + list1);
        System.out.println("list2 : " + list2);
        System.out.println();
    }
}
```
```
list1 : [5, 4, 3, 2, 0, 1, 3]
list2 : [4, 3, 2]

list1 : [0, 1, 2, 3, 3, 4, 5]
list2 : [2, 3, 4]

list1.containsAll(list2) : true
list1 : [0, 1, 2, 3, 3, 4, 5]
list2 : [2, 3, 4, A, B, C]

list1 : [0, 1, 2, 3, 3, 4, 5]
list2 : [2, 3, 4, AA, B, C]

list1.retainAll(list2) : true
list1 : [2, 3, 3, 4]
list2 : [2, 3, 4, AA, B, C]

list1 : [2, 3, 3, 4]
list2 : [AA, B, C]
```

---

### 3) `ArrayList` 예제 2
```java
public class ArrayListEx2 {
  public static void main(String[] args) {
    final int LIMIT = 10;
    String source = "0123456789abcdefghiABCDEFGHIJ!@#$%^&*()ZZZ";
    int length = source.length();

    List list = new ArrayList(length/LIMIT + 10);

    for (int i = 0; i < length; i+=LIMIT) {
      if (i + LIMIT < length)
        list.add(source.substring(i, i+LIMIT));
      else
        list.add(source.substring(i));
    }

    for (int i = 0; i < list.size(); i++) {
      System.out.println(list.get(i));
    }
  }
}
```
```
0123456789
abcdefghiA
BCDEFGHIJ!
@#$%^&*()Z
ZZ
```

### 4) `Vector` 예제
```java
public class VectorEx1 {
    public static void main(String[] args) {
        Vector vector = new Vector(5);
        vector.add("1");
        vector.add("2");
        vector.add("3");
        print(vector);

        System.out.println();

        vector.trimToSize();
        System.out.println("after trimToSize()");
        print(vector);

        System.out.println();

        vector.ensureCapacity(6);
        System.out.println("after ensureCapacity()");
        print(vector);

        System.out.println();

        vector.setSize(7);
        System.out.println("after setSize()");
        print(vector);

        System.out.println();

        vector.clear();
        System.out.println("after clear()");
        print(vector);

    }

    public static void print(Vector vector) {
        System.out.println(vector);
        System.out.println("size : " + vector.size());
        System.out.println("capacity : " + vector.capacity());
    }
}
```
```
[1, 2, 3]
size : 3
capacity : 5

after trimToSize()
[1, 2, 3]
size : 3
capacity : 3

after ensureCapacity()
[1, 2, 3]
size : 3
capacity : 6

after setSize()
[1, 2, 3, null, null, null, null]
size : 7
capacity : 12

after clear()
[]
size : 0
capacity : 12
```
