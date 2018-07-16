본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. `TreeSet`이란?
`TreeSet`은 **이진검색트리(binary search tree)라는 자료구조의 형태로 데이터를 저장하는 컬렉션 클래스** 이다. 이진 검색 트리는 정렬, 검색, 범위검색(range search)에 높은 성능을 보이는 구조로 `TreeSet`은 이진 검색 트리의 성능을 향상시킨 레드-블랙-트리(red-black-tree)로 구현되어 있다. 그리고 `Set`인터페이스를 구현했기 때문에 **중복된 데이터의 저장을 허용하지 않으며, 정렬된 유치에 저장하므로 저장 순서를 유지하지도 않는다.**

## 2. 이진트리(binary-tree)

![이진트리](https://raw.githubusercontent.com/walbatrossw/TIL/master/03_pl/java/stadard-of-java/11_collection_framework/img/tree_set_1.png)

`LinkedList`처럼 여러 개의 노드(node)가 서로 연결된 구조로 각 노드에 최대 2개의 노드를 연결할 수 있으며 루트(root)라고 불리는 하나의 노드에서부터 시작해서 계속 확장해나갈 수 있다. 위아래로 연결된 두 노드를 부모-자식관계에 있다고 하며 위의 노드를 부모노드, 아래의 노드를 자식노드라고 한다. 부모-자식관계는 상대적인 것이며 하나의 부모노드는 두 개의 자식노드와 연결될 수 있다.

```java
// 이진트리의 노드를 코드로 표현
class TreeNode {
  TreeNode left;  // 왼쪽 자식 노드
  Object element; // 객체를 저장하기 위한 참조변수
  TreeNode light; // 오른쪽 자식 노드
}
```

**이진트리의 저장 순서**

![이진트리2](https://raw.githubusercontent.com/walbatrossw/TIL/master/03_pl/java/stadard-of-java/11_collection_framework/img/tree_set_2.png)

- 첫번째로 저장되는 값은 루트가 됨
- 두번째 값은 트리의 루트부터 시작해서 값의 크기를 비교하면서 트리를 따라 내려감
- 작은 값은 왼쪽, 큰 값은 오른쪽에 저장
- 왼쪽 마지막 레벨이 가장 작은 값, 오른쪽 마지막 레벨이 가장 큰 값이 됨
- `TreeSet`에 저장되는 객체가 `Comparable`을 구현하던지 `Comparator`를 제공해서 두 객체를 비교할 방법을 알려줘야함
- 그렇지 않으면 `TreeSet`에 두번째 객체를 저장할 때 예외가 발생한다.

**이진트리의 특성 정리**
- 모든 노드는 최대 두개의 자식노드를 가질 수 있다.
- 왼쪽 자식노드는 부모노드 값보다 작고, 오른쪽 자식노드는 부모노드 값보다 크다.
- 노드의 추가 삭제에 시간이 걸린다.
- 검색(범위검색)과 정렬에 유리하다
- 중복된 값을 저장하지 못한다.

##### `TreeSet` 예제 1 : 로또 번호 생성기(정렬하면서 값을 저장)
```java
public class TreeSetLotto {
    public static void main(String[] args) {
        Set set = new TreeSet();
        for (int i = 0; set.size() < 6; i++) {
            int num = (int) (Math.random() * 45) + 1;
            set.add(num);
        }
        System.out.println(set);
    }
}
```
```
[1, 4, 13, 24, 27, 29]
```
- `HashSetLotto`예제를 `TreeSet`을 사용해 로또 번호 생성기를 작성했다.
- 이전예제와 다르게 정렬하는 코드가 빠져 있는데 `TreeSet`은 저장할 때 이미 정렬을 하기 때문에 읽어올 때 따로 정렬할 필요가 없다.

##### `TreeSet` 예제 2 : 범위 검색 (`subSet()`)
```java
public class TreeSetEx1 {
    public static void main(String[] args) {
        TreeSet set = new TreeSet();

        String from = "b";
        String to = "d";

        set.add("abc");
        set.add("alien");
        set.add("bat");
        set.add("boy");
        set.add("ball");
        set.add("Car");
        set.add("car");
        set.add("Document");
        set.add("delivery");
        set.add("disc");
        set.add("dance");
        set.add("dZZZZ");
        set.add("dzzzz");
        set.add("elephant");
        set.add("elevator");
        set.add("fan");
        set.add("flower");

        System.out.println(set);
        System.out.println("range search from " + from + " to " + to);
        System.out.println("result1 : " + set.subSet(from, to));
        System.out.println("result1 : " + set.subSet(from, to + "zzz"));
    }
}
```
```
[Car, Document, abc, alien, ball, bat, boy, car, dZZZZ, dance, delivery, disc, dzzzz, elephant, elevator, fan, flower]
range search from b to d
result1 : [ball, bat, boy, car]
result1 : [ball, bat, boy, car, dZZZZ, dance, delivery, disc]
```
- `subSet()`을 이용해 범위검색을 할 때 시작범위는 포함되지만 끝범위는 포함되지 않는다.

##### `TreeSet` 예제 3 : 기준 검색 (`hedSet()`, `tailSet()`)
```java
public class TreeSetEx2 {
    public static void main(String[] args) {
        TreeSet set = new TreeSet();
        int[] score = {80, 95, 50, 35, 45, 65, 10, 100};

        for (int i = 0; i < score.length; i++)
            set.add(new Integer(score[i]));

        System.out.println("50보다 작은 값 : " + set.headSet(new Integer(50)));
        System.out.println("50보다 큰 값 : " + set.tailSet(new Integer(50)));
    }
}
```
```
50보다 작은 값 : [10, 35, 45]
50보다 큰 값 : [50, 65, 80, 95, 100]
```
- `headSet()`을 사용하면 저장된 객체 중에서 지정된 기준 값보다 큰 객체를 얻을 수 있다.
- `tailSet()`을 사용하면 저장된 객체 중에서 지정된 기준 값보다 작은 객체를 얻을 수 있다.
