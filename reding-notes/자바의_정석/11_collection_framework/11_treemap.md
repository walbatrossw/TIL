## 1.11 `TreeMap`

### 1) `TreeMap`?
이름에서 알 수 있듯이 **이진검색트리의 형태로 키와 값의 쌍으로 이루어진 데이터를 저장하는데 검색과 정렬에 적합한 컬렉션 클래스** 이다. 기본적인 검색성능은 `HashMap`이 더 뛰어나지만 **범위검색이나 정렬이 필요한 경우는 `TreeMap`을 사용하는 것** 이 좋다.

### 2) `TreeMap`예제 1 : `TreeMap`을 이용한 그래프 표현하기
```java
public class TreeMapEx1 {
    public static void main(String[] args) {
        String[] data = {"A", "K", "A", "K", "D", "K", "A", "K", "K", "K", "Z", "D"};

        TreeMap map = new TreeMap();

        for (int i = 0; i < data.length; i++) {
            if (map.containsKey(data[i])) {
                Integer value = (Integer) map.get(data[i]);
                map.put(data[i], new Integer(value.intValue() + 1));
            } else {
                map.put(data[i], new Integer(1));
            }
        }

        Iterator iterator = map.entrySet().iterator();

        while (iterator.hasNext()) {
            Map.Entry entry = (Map.Entry) iterator.next();
            int value = ((Integer)entry.getValue()).intValue();
            System.out.println(entry.getKey() + " : " + printBar('#', value) + " " + value);
        }

        System.out.println();

        // map을 ArrayList로 변환한 다음에 Collections.sort()로 정렬
        Set set = map.entrySet();
        List list = new ArrayList(set);

        Collections.sort(list, new ValueComparator());

        iterator = list.iterator();

        System.out.println("After Descending Sort");
        while (iterator.hasNext()) {
            Map.Entry entry = (Map.Entry) iterator.next();
            int value = ((Integer)entry.getValue()).intValue();
            System.out.println(entry.getKey() + " : " + printBar('#', value) + " " + value);
        }
    }

    static class ValueComparator implements Comparator {
        public int compare(Object o1, Object o2) {
            if (o1 instanceof Map.Entry && o2 instanceof Map.Entry) {
                Map.Entry entry1 = (Map.Entry) o1;
                Map.Entry entry2 = (Map.Entry) o2;

                int value1 = ((Integer)entry1.getValue()).intValue();
                int value2 = ((Integer)entry2.getValue()).intValue();

                return value2 - value1;
            }
            return -1;
        }
    }

    public static String printBar(char ch, int value) {
        char[] bar = new char[value];

        for (int i = 0; i < bar.length; i++) {
            bar[i] = ch;
        }

        return new String(bar);
    }
}
```
```
A : ### 3
D : ## 2
K : ###### 6
Z : # 1

After Descending Sort
K : ###### 6
A : ### 3
D : ## 2
Z : # 1
```
- `HashMap`예제들 중에서 하나를 `TreeMap`을 이용해서 변형한 예제이다.
- `HashMap`예제와 달리 결과가 정렬된 것을 볼 수 있는데 키(key)가 `String`인스턴스이기 때문에 `String`클래스에 정의된 정렬기준에 의해 정렬된 것이다.
