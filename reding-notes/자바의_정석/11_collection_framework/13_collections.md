본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## `Collections`란?
`Arrays`가 배열과 관련된 메서드를 제공하는 것처럼, `Collections`는 컬렉션과 관련된 메서드를 제공한다. `fill()`, `copy()`, `sort()`, `binarySearch()` 등의 메서드는 두 클래스에서 모두 포함되어 있으며 같은 기능을 수행한다.

##### `Collections`예제 1
```java
public class CollectionEx1 {
    public static void main(String[] args) {
        List list = new ArrayList();
        System.out.println(list);

        addAll(list, 1,2,3,4,5);
        System.out.println(list);

        // 오른쪽으로 두 칸씩 이동
        rotate(list, 2);
        System.out.println(list);

        // 첫번째와 세번째를 교환
        swap(list, 0, 2);
        System.out.println(list);

        // 저장된 요소의 위치를 임의로 변경
        shuffle(list);
        System.out.println(list);

        // 정렬
        sort(list);
        System.out.println(list);

        // 오름차순 정렬
        sort(list, reverseOrder());
        System.out.println(list);

        System.out.println("max = " + max(list));
        System.out.println("min = " + min(list));
        System.out.println("min = " + max(list, reverseOrder()));

        // list를 9로 채움
        fill(list, 9);
        System.out.println("list = " + list);

        // list와 같은 크기의 새로운 list를 생성하고 2로 채움
        List newList = nCopies(list.size(), 2);
        System.out.println("newList = " + newList);

        // 공통요소가 없으면 true
        System.out.println(disjoint(list, newList));

        copy(list, newList);
        System.out.println("newList = " + newList);
        System.out.println("list = " + list);

        replaceAll(list, 2, 1);
        System.out.println("list = " + list);

        Enumeration enumeration = enumeration(list);
        ArrayList list2 = list(enumeration);
        System.out.println("list2 = " + list2);
    }
}
```
```
[]
[1, 2, 3, 4, 5]
[4, 5, 1, 2, 3]
[1, 5, 4, 2, 3]
[5, 4, 1, 2, 3]
[1, 2, 3, 4, 5]
[5, 4, 3, 2, 1]
max = 5
min = 1
min = 1
list = [9, 9, 9, 9, 9]
newList = [2, 2, 2, 2, 2]
true
newList = [2, 2, 2, 2, 2]
list = [2, 2, 2, 2, 2]
list = [1, 1, 1, 1, 1]
list2 = [1, 1, 1, 1, 1]
```
