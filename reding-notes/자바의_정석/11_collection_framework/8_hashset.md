## 1.8 `HashSet`

### 1) `HashSet`?
`HashSet`은 **`Set`인터페이스를 구현한 가장 대표적인 컬렉션** 이며, `Set`인터페이스의 특징대로 **`HashSet`은 중복된 요소를 저장하지 않는다.** `HashSet`에 새로운 요소를 추가할 때는 `add()`나 `addAll()`를 사용하는데 만약 `HashSet`에 이미 저장되어 있는 요소와 중복된 요소를 추가하고자 한다면 이 메서드들은 `false`를 반환함으로써 중복된 요소이기 때문에 추가에 실패했다는 것을 알려준다.

`ArrayList`와 같이 `List`인터페이스를 구현한 컬렉션과 달리 `HashSet`은 저장순서를 유지하지 않으므로 **저장순서를 유지하고자 한다면 `LinkedHashSet`을 사용** 해야한다.

### 2) `HashSet` 예제 1 : `HashSet` 사용법과 이해
```java
public class HashSetEx1 {
    public static void main(String[] args) {
        Object[] objArr = {"1", new Integer(1), "2", "2", "3", "3", "4", "4", "4"};
        Set set = new HashSet();

        for (int i = 0; i < objArr.length; i++) {
            set.add(objArr[i]);
        }

        System.out.println(set);
    }
}
```
```
[1, 1, 2, 3, 4]
```
- 중복된 값은 저장되지 않는다.
- `add()`는 객체를 추가할 때 `HashSet`에 이미 같은 객체를 가지고 있으면 중복으로 간주하고, 저장하지 않고 `false`를 리턴한다.
- 그런데 1이 두번 출력된 것은 첫번째 1은 `String`인스턴스이고, 두번째 1은 `Integer`인스턴스이기 때문이다.

### 3) `HashSet` 예제 2 : 로또번호 생성기(중복된 숫자X)
```java
public class HashSetLotto {
    public static void main(String[] args) {
        Set set = new HashSet();

        for (int i = 0; set.size() < 6; i++) {
            int num = (int) (Math.random() * 45) + 1;
            set.add(new Integer(num));
        }

        List list = new LinkedList(set);
        Collections.sort(list);
        System.out.println(list);
    }
}
```
```
[3, 5, 14, 15, 32, 35]
```
- 중복된 값이 저장되지 않는 `HashSet`의 성질을 이용해서 로또번호를 생성했다.
- `Math.random()`를 이용했기 때문에 실행할 때마다 생성되는 번호가 달라진다.
- 번호를 크기 순으로 정렬하기 위해 `Collections.sort()`를 사용했는데 `sort()`의 인자로 `List`가 필요하기 때문에 `LinkedList`클래스의 생성자를 이용해 처리
### 4) `HashSet` 예제 3 : 빙고판 만들기(`HashSet`, `LinkedHashSet`)
```java
public class Bingo {
    public static void main(String[] args) {
        Set set = new HashSet();
        //Set set = new LinkedHashSet();
        int[][] board = new int[5][5];

        for (int i = 0; set.size() < 25; i++) {
            set.add((int)(Math.random() * 50) + 1 + "");
        }

        Iterator iterator = set.iterator();

        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[i].length; j++) {
                board[i][j] = Integer.parseInt((String) iterator.next());
                System.out.print((board[i][j] < 10 ? " " : " ")+ board[i][j]);
            }
            System.out.println();
        }
    }
}
```
```
22 44 23 46 25
47 26 49 29 50
31 10 33 13 35
14 15 16 17 19
1 5 9 40 41
```
- 1~50 사이의 숫자 중에서 25개를 골라 5X5크기의 빙고판을 만들었다.
- `next()`의 반환값이 `Object`이기때문에 형변환을 통해서 원래의 타입으로 바꿔놔야한다.
- 여러번 실행하다보면 같은 숫자가 비슷한 위치에 나온다는 사실을 발견할 수 있는데 `HashSet`은 저장된 순서를 보장하지 않고, 자체적인 방식에 따라 저장하게 된다.
- 이 경우에는 `HashSet`보다는 `LinkedHashSet`이 더 나은 선택일 수 있다.

### 5) `HashSet` 예제 4 : name, age가 같으면 같은 사람으로 인식하기 1
```java
public class HashSetEx2 {
    public static void main(String[] args) {
        HashSet hashSet = new HashSet();
        hashSet.add("abc");
        hashSet.add("abc");
        hashSet.add(new Person("Doubles", 10));
        hashSet.add(new Person("Doubles", 10));

        System.out.println(hashSet);
    }
}

class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String toString() {
        return name + ":" + age;
    }
}
```
```
[abc, Doubles:10, Doubles:10]
```
- `Person`의 `name`과 `age`가 같으면 같은 사람으로 간주하고 `HashSet`에 저장되지 않도록 하는 것이 위의 코드의 목적이다.
- 하지만 결과를 출력해보면 `name`과 `age`가 동일한데도 불구하고 2명이 출력되었다.
- 코드의 의도대로 두 인스턴스가 같은 것으로 인식하게 하려면 어떻게 해야할지 다음 예제를 통해 알아보자.

### 6) `HashSet` 예제 5 : name, age가 같으면 같은 사람으로 인식하기 2 (`equals()`와 `hashCode()`)
```java
public class HashSetEx3 {
    public static void main(String[] args) {
        HashSet hashSet = new HashSet();
        hashSet.add("abc");
        hashSet.add("abc");
        hashSet.add(new Person2("Doubles", 10));
        hashSet.add(new Person2("Doubles", 10));

        System.out.println(hashSet);
    }
}

class Person2 {
    String name;
    int age;

    Person2(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // name과 age가 같으면 true 리턴
    public boolean equals(Object obj) {
        if (obj instanceof Person2) {
            Person2 temp = (Person2) obj;
            return name.equals(temp.name) && age == temp.age;
        }
        return false;
    }

    public int hashCode() {
        return Objects.hash(name, age);
    }

    public String toString() {
        return name + ":" + age;
    }
}
```
```
[abc, Doubles:10]
```
- 일단 결과부터 살펴보면 `name`과 `age`가 같기 때문에 동일한 인스턴스로 간주하여 더이상 저장을 하지 않아 1개만 출력되었다.
- `HashSet`의 `add()`는 새로운 요소를 추가하기 전에 기존에 저장된 요소와 같은 것인지 판별하기 위해 추가하려는 요소의 `equals()`와 `hashCode()`를 호출하기 때문에 `equals()`와 `hashCode()`를 목적에 맞게 오버라이딩 해야만한다.
- `String`클래스에서 같은 내용의 문자열에 대한 `equals()`의 호출결과가 `true`인 것과 같이, `Person2`클래스에서도 두 인스턴스의 `name`과 `age`가 같으면 `true`를 반환하도록 `equals()`를 오버라이딩했다.
- 오버라이딩을 통해 작성된 `HashCode()`는 아래와 같은 세가지 조건을 만족해야한다.
  - 실행 중인 애플리케이션 내의 동일한 객체에 대해서 여러번 `hashCode()`를 호출해도 동일한 `int`값을 반환해야한다.
  - `equals()`를 이용한 비교에 의해서 `true`를 얻은 두 개체에 대해 각각 `hashCode()`를 호출해서 얻은 결과는 반드시 같아야한다.
  - `equals()`를 호출했을 때 `false`를 반환하는 두 객체는 `hashCode()` 호출에 대해 같은 `int`값을 반환하는 경우가 있어도 괜찮지만, 해싱(hashing)을 사용하는 컬렉션의 성능을 향상시키기 위해 다른 `int`값을 반환하는 것이 좋다.

### 7) `HashSet` 예제 6 : 교집합, 합집합, 차집합
```java
public class HashSetEx4 {
    public static void main(String[] args) {
        HashSet setA = new HashSet();
        HashSet setB = new HashSet();
        HashSet setUnion = new HashSet();
        HashSet setInter = new HashSet();
        HashSet setDiff1 = new HashSet();
        HashSet setDiff2 = new HashSet();

        setA.add("1");
        setA.add("2");
        setA.add("3");
        setA.add("4");
        setA.add("5");
        System.out.println("A = " + setA);

        setB.add("4");
        setB.add("5");
        setB.add("6");
        setB.add("7");
        setB.add("8");
        System.out.println("B = " + setB);

        Iterator iterator = setB.iterator();
        while (iterator.hasNext()) {
            Object temp = iterator.next();
            if (setA.contains(temp))
                setInter.add(temp);
        }

        iterator = setA.iterator();
        while (iterator.hasNext()) {
            Object temp = iterator.next();
            if (!setB.contains(temp))
                setDiff1.add(temp);
        }

        iterator = setB.iterator();
        while (iterator.hasNext()) {
            Object temp = iterator.next();
            if (!setA.contains(temp))
                setDiff2.add(temp);
        }

        iterator = setA.iterator();
        while (iterator.hasNext())
            setUnion.add(iterator.next());

        iterator = setB.iterator();
        while (iterator.hasNext())
            setUnion.add(iterator.next());

        System.out.println("A ∩ B = " + setInter);
        System.out.println("A ∪ B = " + setUnion);
        System.out.println("A - B = " + setDiff1);
        System.out.println("B - A = " + setDiff2);
    }
}
```
```
A = [1, 2, 3, 4, 5]
B = [4, 5, 6, 7, 8]
A ∩ B = [4, 5]
A ∪ B = [1, 2, 3, 4, 5, 6, 7, 8]
A - B = [1, 2, 3]
B - A = [6, 7, 8]
```
