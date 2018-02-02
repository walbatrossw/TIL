## 1.6 `Arrays`

### 1) `Arrays`?
`Arrays` 클래스에는 **배열을 다루는데 유용한 메서드들이 정의** 되어 있다. 같은 기능의 메서드가 배열의 타입만 다르게 오버로딩되어 있을 뿐, 실제로는 그리 많지 않다. 정의된 메서드는 모두 **`static`메서드** 이다. 주요 메서드들에 대해 알아보자.

### 2) `copyOf()`, `copyOfRange()` : 배열의 복사
```java
int[] arr = {0, 1, 2, 3, 4};
int[] arr2 = Arrays.copyOf(arr, arr.length); // arr2 = {0, 1, 2, 3, 4}
int[] arr3 = Arrays.copyOf(arr, 3); // arr3 = {0, 1, 2}
int[] arr4 = Arrays.copyOf(arr, 7); // arr4 = {0, 1, 2, 3, 4, 0, 0}

int[] arr5 = Arrays.copyOfRange(arr, 2, 4); // arr5 = {2, 3}
int[] arr6 = Arrays.copyOfRange(arr, 0, 7); // arr6 = {0, 1, 2, 3, 4, 0, 0}
```
- `copyOf()` : 배열전체를 복사해서 새로운 배열에 반환
- `copyOfRange()` : 배열의 일부를 복사해서 새로운 배열에 반환, 지정된 범위의 끝은 포함되지 않음

### 3) `fill()`, `setAll()` : 배열 채우기
```java
int[] arr = new int[5];
Arrays.fill(arr, 9);  // arr = {9, 9, 9, 9, 9}
Arrays.setAll(arr, () -> (int) (Math.random() * 5 ) + 1); // arr = {1, 5, 2, 1, 1}
```
- `fill()` : 배열의 모든 요소를 지정된 값으로 채움
- `setAll()` : 배열을 채우는데 사용할 함수형 인터페이스를 매개변수로 받는다. 함수형객체나 람다식을 매개변수로 지정해야함

### 4) `sort()`, `binarySearch()` : 배열의 정렬과 검색
```java
int[] arr = {3, 2, 0, 1, 4};
int index = Arrays.binarySearch(arr, 2); // index = -5 로 잘못된 결과가 나온다.

Arrays.sort(arr); // 배열 arr을 정렬
System.out.println(Arrays.toString(arr)); // [0, 1, 2, 3, 4]
int index = Arrays.binarySearch(arr, 2); // index = 2 올바른 결과가 나온다.
```
- `sort()` : 배열을 정렬할 때 사용
- `binarySearch()` : 배열에 저장된 요소를 검색할 때 사용, 배열에서 지정된 값이 저장된 위치를 찾아서 반환하는데 반드시 배열이 정렬되어 있어야함

### 5) `equals()`, `toString()` : 문자열 비교와 출력
```java
String[][] str2D = new String[][]{{"aaa", "bbb"}, {"AAA", "BBB"}};
String[][] str2D2 = new String[][]{{"aaa", "bbb"}, {"AAA", "BBB"}};

System.out.println(Arrays.equals(str2D, str2D2)); // false
System.out.println(Arrays.deepEquals(str2D, str2D2)); // true
```
```java
int[] arr = {0, 1, 2, 3, 4};
int[][] arr2D = {{11, 12}, {21, 22}};

System.out.println(Arrays.toString(arr)); // [0, 1, 2, 3, 4]
System.out.println(Arrays.deepToString(arr2D)); // [[11, 12], [21, 22]]
```
- `equals()` : 일차원 문자열만 비교, 다차원 배열을 비교하게되면 문자열을 비교하는게 아니라 배열에 저장된 주소를 비교하게 되서 `false`가 리턴됨
- `deepEquals()` : 다차원 문자열 비교
- `toString()` : 일차원 배열만 출력
- `deepToString()` : 다차원 배열 출력

### 6) `asList(Object... a)` : 배열을 `List`로 변환
```java
List list = Arrays.asList(new Integer[]{1, 2, 3, 4, 5}); // list = [1, 2, 3, 4, 5]
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5); // list = [1, 2, 3, 4, 5]
list.add(6); // UnsupportedOperationException 예외 발생
```
- 배열을 `List`에 담아서 반환
- 매개변수 타입이 가변인수라서 배열 생성 없이 저장할 요소들만 나열하는 것도 가능
- 주의점은 `asList()`가 반환한 `List`의 크기를 변경불가 즉, 추가나 삭제가 불가능만 저장된 내용은 변경이 가능
- 만약 크기를 변경할 수 있는 `List`가 필요하다면 아래와 같이 하면된다.
  ```java
  List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
  ```

### 7) `parallelXXX()`, `spliterator()`, `stream()`
- `parallelXXX()` : 보다 빠른 결과를 얻기 위해 여러 쓰레드가 작업을 나누어 처리
- `spliterator()` : 여러 쓰레드가 처리할 수 있게 하나의 작업을 여러 작업으로 나누는 `Spliterator`를 반환
- `stream()` : 컬렉션을 스트림으로 반환

### 8) `Arrays` 예제 1
```java
public class ArraysEx {
    public static void main(String[] args) {
        int[] arr = {0, 1, 2, 3, 4};
        int[][] arr2D = {{11, 12, 13}, {21, 22, 23}};

        System.out.println("arr = " + Arrays.toString(arr));
        System.out.println("arr2D = " + Arrays.toString(arr2D));

        int[] arr2 = Arrays.copyOf(arr, arr.length);
        int[] arr3 = Arrays.copyOf(arr, 3);
        int[] arr4 = Arrays.copyOf(arr, 7);
        int[] arr5 = Arrays.copyOfRange(arr, 2, 4);
        int[] arr6 = Arrays.copyOfRange(arr, 0, 7);

        System.out.println("arr2 = " + Arrays.toString(arr2));
        System.out.println("arr3 = " + Arrays.toString(arr3));
        System.out.println("arr4 = " + Arrays.toString(arr4));
        System.out.println("arr5 = " + Arrays.toString(arr5));
        System.out.println("arr6 = " + Arrays.toString(arr6));

        int[] arr7 = new int[5];

        Arrays.fill(arr7, 9);
        System.out.println("arr7 = " + Arrays.toString(arr7));

        Arrays.setAll(arr7, i -> (int) (Math.random() * 6) + 1);
        System.out.println("arr7 = " + Arrays.toString(arr7));

        for (int i : arr7) {
            char[] graph = new char[i];
            Arrays.fill(graph, '*');
            System.out.println(new String(graph)+i);
        }

        String[][] str2D = new String[][]{{"aaa", "bbb"}, {"AAA", "BBB"}};
        String[][] str2D2 = new String[][]{{"aaa", "bbb"}, {"AAA", "BBB"}};

        System.out.println(Arrays.equals(str2D, str2D2)); // false
        System.out.println(Arrays.deepEquals(str2D, str2D2)); // true

        char[] chArr = {'A', 'D', 'E', 'C', 'B'};

        //int idx = Arrays.binarySearch(chArr, 'B');
        System.out.println("chArr = " + Arrays.toString(chArr));
        System.out.println("index of B = " + Arrays.binarySearch(chArr, 'B'));
        System.out.println("After sorting...");
        Arrays.sort(chArr);
        System.out.println("chArr = " + Arrays.toString(chArr));
        System.out.println("index of B = " + Arrays.binarySearch(chArr, 'B'));
    }
}
```
```
arr = [0, 1, 2, 3, 4]
arr2D = [[I@4554617c, [I@74a14482]
arr2 = [0, 1, 2, 3, 4]
arr3 = [0, 1, 2]
arr4 = [0, 1, 2, 3, 4, 0, 0]
arr5 = [2, 3]
arr6 = [0, 1, 2, 3, 4, 0, 0]
arr7 = [9, 9, 9, 9, 9]
arr7 = [6, 6, 1, 3, 6]
******6
******6
*1
***3
******6
false
true
chArr = [A, D, E, C, B]
index of B = -2
After sorting...
chArr = [A, B, C, D, E]
index of B = 1
```
