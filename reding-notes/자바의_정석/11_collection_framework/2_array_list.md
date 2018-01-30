# 1. 컬렉션 프레임워크(collection framework)

## 1.2 `ArrayList`

### 1) `ArrayList`?
`ArrayList`는 컬렉션 프레임워크에서 가장 많이 사용되는 컬렉션 클래스이다. `List`인터페이스를 구현하기 때문에 데이터의 저장순서는 유지되고, 중복을 허용한다. `Object`배열을 이용하여 데이터를 순차적으로 저장한다. 예를 들면, 첫 번째로 저장한 객체는 `Object`배열의 0번째에 저장되고 그 다음에 저장하는 객체는 1번째에 저장된다. 이런 식으로 계속 배열에 순서대로 저장되며, 배열에 더이상 저장할 공간이 없으면 보다 큰 새로운 배열을 생성하여 기존의 배열에 저장된 내용을 새로운 배열로 복사한 다음에 저장된다.

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

---

### 3) `ArrayList` 예제 2
