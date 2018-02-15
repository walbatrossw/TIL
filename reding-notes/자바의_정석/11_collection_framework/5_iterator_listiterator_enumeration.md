본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. `Iterator`, `ListIterator`, `Enumeration` ?

`Iterator`, `ListIterator`, `Enumeration`은 모두 **컬렉션에 저장된 요소를 접근하는데 사용하는 인터페이스** 이다. `Enumeration`은 `Iterator`의 구버전이고, `ListIterator`는 `Iterator`의 기능을 향상시킨 것이다.

## 2. `Iterator`
컬렉션 프레임워크에서는 컬렉션에 저장된 요소들을 읽어오는 방법을 표준화하였다. 컬렉션에 저장된 각 요소에 접근하는 기능을 가진 `Iterator`인터페이스를 정의하고, `Collection`인터페이스에서는 `Iterator`(`Iterator`를 구현한 클래스의 인스턴스)를 반환하는 `iterator()`를 정의하고 있다.
```java
public interface Iterator {
  boolean hasNext();  // 읽어올 요소가 있는지 확인,
  Object next();      // 다음 요소를 읽어옴, next()를 호출하기전에 hasNext()를 호출해 읽어올 요소가 있는지 확인하는 것이 안전함.
  void remove();      // next()로 읽어온 요소를 삭제, next()를 호출한 다음에 remove()를 호출해야함
}

public interface Collection {
  public Iterator iterator();
}
```

`iterator()`는 `Collection`인터페이스에 정의된 메서드이므로 자손인 `List`, `Set`에도 포함되어 있다. 그래서 `List`나 `Set`을 구현하는 컬렉션은 `iterator()`가 각 컬렉션의 특징에 알맞게 작성되어 있다. 컬렉션 클래스에 대해 `iterator()`를 호출하여 `Iterator`를 얻은 다음 반복문, 주로 `while`문을 사용해서 컬렉션 클래스의 요소들을 얻어 올 수 있다.

아래의 코드는 `ArrayList`에 저장된 요소들을 출력하기 위한 내용으로 이렇게 작성할 수 있다.
```java
List list = new ArrayList();

Iterator iterator = list.iterator();

while(iterator.hasNext())
  System.out.println(iterator.next());
```

**`Map`인터페이스를 구현한 컬렉션 클래스도 `iterator()`를 사용할 수 있다.** 하지만 키(key)와 값(value)을 쌍으로 저장하고 있기 때문에 `iterator()`를 직접 호출할 수 없고, 그 대신 **`keySet()`, `entrySet()`과 같은 메서드를 통해서 키와 값을 각각 따로 Set의 형태로 얻어온 후 다시 `iterator()`를 호출해야만 `Iterator`를 얻을 수 있다.**
```java
Map map = new HashMap();
Iterator iterator = map.keySet().iterator();
```

###### `Iterator` 예제 1
```java
public class IteratorEx1 {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();
        list.add("1");
        list.add("2");
        list.add("3");
        list.add("4");
        list.add("5");

        Iterator iterator = list.iterator();
        while (iterator.hasNext()) {
            Object obj = iterator.next();
            System.out.println(obj);
        }
    }
}
```
```
1
2
3
4
5
```
- `List`클래스들은 저장순서를 유지하기 때문에 `Iterator`를 이용해서 읽어온 결과 역시 저장순서와 동일하지만 `Set`클래스의 경우는 각 요소간의 순서가 유지되지 않기 때문에 `Iterator`를 이용해서 저장된 요소들을 읽어오더라도 처음 저장한 순서와 동일하지 않다.

## 3. `ListIterator`, `Enumeration`
- `Enumeration` : `Iterator`의 구버전
- `ListIterator` : `Iterator`에 양방향 조회기능 추가(`List`인터페이스를 구현한 경우만 사용가능)

##### `ListIterator` 예제 1
```java
public class ListIteratorEx1 {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();
        list.add("1");
        list.add("2");
        list.add("3");
        list.add("4");
        list.add("5");

        ListIterator listIterator = list.listIterator();

        while (listIterator.hasNext()) {
            // 순반향으로 진행하면서 읽어옴
            System.out.println(listIterator.next());
        }
        System.out.println();

        while (listIterator.hasPrevious()) {
            // 역방향으로 진행하면서 읽어옴
            System.out.println(listIterator.previous());
        }
        System.out.println();
    }
}
```
```
1
2
3
4
5

5
4
3
2
1
```
- `Iterator`는 단방향으로만 이동하기 때문에 컬렉션의 마지막 요소에 다다르면 더이상 사용할 수 없다.
- `ListIterator`는 양방향으로 이동하기 때문에 각 요소간의 이동이 자유롭다.
- 이동하기 전에는 반드시 `hasNext()`나 `hasPrevious()`를 호출해서 이동할 수 있는지를 확인해야한다.

##### `Iterator` 예제 2
```java
public class IteratorEx2 {
    public static void main(String[] args) {
        ArrayList original = new ArrayList(10);
        ArrayList copy1 = new ArrayList(10);
        ArrayList copy2 = new ArrayList(10);

        for (int i = 0; i < 10; i++)
            original.add(i +"");

        Iterator iterator = original.iterator();

        while (iterator.hasNext())
            copy1.add(iterator.next());

        System.out.println("original에서 copy1으로 복사");
        System.out.println("original : " + original);
        System.out.println("copy1 : " + copy1);
        System.out.println();

        // Iterator는 재사용이 안되기 때문에 다시 얻어와야함
        iterator = original.iterator();

        while (iterator.hasNext()) {
            copy2.add(iterator.next());
            iterator.remove();
        }

        System.out.println("original에서 copy2으로 이동");
        System.out.println("original : " + original);
        System.out.println("copy2 : " + copy2);
    }
}
```
```
original에서 copy1으로 복사
original : [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
copy1 : [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

original에서 copy2으로 이동
original : []
copy2 : [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

- `Iterator`의 `remove()`는 단독으로 쓰일 수 없고, `next()`와 같이 써야한다.
- 특정위치의 요소를 삭제하는 것이 아니라 읽어온 것을 삭제한다.
- 만약 `next()`의 호출 없이 `remove()`를 호출하면 `IllegalStateException`이 발생한다.
