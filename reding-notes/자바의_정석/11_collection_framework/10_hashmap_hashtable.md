본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## `HashMap`이란?
`HashMap`은 `Map`인터페이스를 구현했기 때문에 `Map`과 같이 **키(key)와 값(value)을 묶어서 하나의 데이터(entry)로 저장** 한다. 그리고 **해싱(Hashing)을 사용하기 때문에 많은 양의 데이터를 검색하는데 있어서 뛰어난 성능** 을 보인다.

```java
public class HashMap extends AbstractMap implements Map, Cloneable, Serializable {
  transient Entry[] table;
  //...
  static class Entry implements Map, Entry {
    final Object key;
    Object value;
    //...
  }
}
```

위의 코드는 `HashMap`이 데이터를 어떻게 저장하는지 확인하기 위해 실제소스의 일부를 발췌한 것이다. `HashMap`은 `Entry`라는 내부클래스를 정의하고, 다시 `Entry`타입의 배열을 선언하고 있다. 키와 값은 별개의 값이 아니라 서로 관련된 값이기 때문에 각가의 배열로 선언하기보다는 하나의 클래스로 정의해서 배열로 다루는 것이 데이터의 무결성적인 측면에서 바람직하기 때문이다.

```java
// 비객체지향적인 코드
Object[] key;
Object[] value;
```

```java
// 객체지향적인 코드
Entry[] table;
class Entry {
  Object key;
  Object value;
}
```

`HashMap`은 **키와 값을 각각 `Object`타입으로 저장** 한다, 즉 `(Object, Object)`의 형태로 저장하기 때문에 어떠한 객체도 저장할 수 있지만 **키는 주로 `String`을 대문자로 또는 소문자로 통일해서 사용** 하곤 한다.

**키(key)와 값(value)**
- 키(key) : 컬렉션 내의 키(key)중에서 유일해야한다.
- 값(value) : 키(key)와 달리 데이터의 중복을 허용한다.

##### `HashMap`예제 1 : `HashMap`을 이용해 id, pw 인증 해보기 - `containKey()`
```java
public class HashMapEx1 {
    public static void main(String[] args) {
        HashMap map = new HashMap();
        map.put("doubles", "1234");
        map.put("doublek", "1111");
        map.put("doublek", "1234");

        Scanner scanner = new Scanner(System.in);

        while (true) {
            System.out.println("id와 pw를 입력해주세요");
            System.out.println("id : ");
            String id = scanner.nextLine().trim();

            System.out.println("pw : ");
            String pw = scanner.nextLine().trim();
            System.out.println();

            // 아이디 불일치
            if (!map.containsKey(id)) {
                System.out.println("입력한 id는 존재하지 않습니다. 다시 입력해주세요.");
                continue;
            } else {
                if (!(map.get(id)).equals(pw)) {
                    System.out.println("비밀번호가 일치하지 않습니다. 다시 입력해주세요.");
                } else {
                    System.out.println("id와 pw가 일치합니다.");
                    break;
                }
            }
        }
    }
}
```
```
id와 pw를 입력해주세요
id : doublea
pw : 1234

입력한 id는 존재하지 않습니다. 다시 입력해주세요.
id와 pw를 입력해주세요
id : doubles
pw : 1234

id와 pw가 일치합니다.
```
```
id와 pw를 입력해주세요
id : doublek
pw : 1111

비밀번호가 일치하지 않습니다. 다시 입력해주세요.
id와 pw를 입력해주세요
id : doublek
pw : 1234

id와 pw가 일치합니다.
```
- `HashMap`을 생성하고 id와 pw을 키와 값의 쌍으로 저장한 다음, 입력받은 id를 key로 `HashMap`에서 검색해서 얻은 비밀번호을 입력받은 비밀번호와 비교하는 예제이다.
- 3개의 데이터 쌍을 저장했지만 실제로는 2개 밖에 저장되지 않은 이유는 중복된 키가 있기 때문이다.
- 세번째로 저장한 데이터의 키인 `doublek`는 이미 존재하기 떄문에 새로 추가되는 대신 기존의 값을 덮어씌우게 된다.
- `Map`의 값은 중복을 허용하지만 키는 중복을 허용하지 않는다.

##### `HashMap`예제 2 : 데이터 저장, 읽기(`etrySet()`, `keySet()`, `values()`)
```java
public class HashMapEx2 {
    public static void main(String[] args) {
        HashMap map = new HashMap();
        map.put("doubleS", new Integer(95));
        map.put("doubleA", new Integer(88));
        map.put("doubleB", new Integer(72));
        map.put("doubleC", new Integer(66));
        map.put("doubleD", new Integer(80));
        map.put("doubleE", new Integer(80));

        Set set = map.entrySet();
        Iterator iterator = set.iterator();

        while (iterator.hasNext()) {
            Map.Entry entry = (Map.Entry) iterator.next();
            System.out.println("name : " + entry.getKey() + ", score : " + entry.getValue());
        }

        set = map.keySet();
        System.out.println("name list : " + set);

        Collection values = map.values();
        iterator = values.iterator();

        int total = 0;

        while (iterator.hasNext()) {
            Integer i = (Integer) iterator.next();
            total += i.intValue();
        }

        System.out.println("sum :" + total);
        System.out.println("average : " + (float) total / set.size());
        System.out.println("max : " + Collections.max(values));
        System.out.println("min : " + Collections.min(values));
    }
}
```
```
name : doubleE, score : 80
name : doubleD, score : 80
name : doubleS, score : 95
name : doubleC, score : 66
name : doubleB, score : 72
name : doubleA, score : 88
name list : [doubleE, doubleD, doubleS, doubleC, doubleB, doubleA]
sum :481
average : 80.166664
max : 95
min : 66
```
- `entrySet()` : `HashMap`에 저장된 값을 엔트리(키와 값의 결합)의 형태로 `Set`에 저장해서 반환
- `keySet()` : `HashMap`에 저장된 모든 키가 저장된 `Set`을 반환
- `values()` : `HashMap`에 저장된 모든 값을 컬렉션의 형태로 반환

##### `HashMap`예제 3 : 전화번호부 만들기 : `HashMap`안에 또다른 `HashMap`
```java
public class HashMapEx3 {

    static HashMap phoneBook = new HashMap();

    public static void main(String[] args) {
        addPhoneNo("friends", "kim", "010-1111-1234");
        addPhoneNo("friends", "yoon", "010-1234-1234");
        addPhoneNo("friends", "seo", "010-4321-4321");
        addPhoneNo("friends", "lee", "010-1100-7890");
        addPhoneNo("company", "choi", "010-0000-1010");
        addPhoneNo("company", "jung", "010-3333-4444");
        addPhoneNo("company", "park", "010-1111-1234");
        addPhoneNo("market", "070-1121-5432");

        printList();
    }

    static void addGroup(String groupName) {
        if (!phoneBook.containsKey(groupName))
            phoneBook.put(groupName, new HashMap());
    }

    static void addPhoneNo(String groupName, String name, String tel) {
        addGroup(groupName);
        HashMap group = (HashMap) phoneBook.get(groupName);
        group.put(tel, name);
    }

    static void addPhoneNo(String name, String tel) {
        addPhoneNo("etc", name, tel);
    }

    static void printList() {
        Set set = phoneBook.entrySet();
        Iterator iterator = set.iterator();

        while (iterator.hasNext()) {
            Map.Entry entry = (Map.Entry) iterator.next();

            Set subSet = ((HashMap) entry.getValue()).entrySet();
            Iterator subIterator = subSet.iterator();

            System.out.println(" * " + entry.getKey() + "[" + subSet.size() + "]");

            while (subIterator.hasNext()) {
                Map.Entry subEntry = (Map.Entry) subIterator.next();
                String telNo = (String) subEntry.getKey();
                String name = (String) subEntry.getValue();
                System.out.println(name + " " + telNo);
            }
            System.out.println();
        }
    }
}
```
```
* etc[1]
market 070-1121-5432

* company[3]
choi 010-0000-1010
jung 010-3333-4444
park 010-1111-1234

* friends[4]
lee 010-1100-7890
yoon 010-1234-1234
seo 010-4321-4321
kim 010-1111-1234
```
- `HashMap`은 키와 값 모두 Object타입으로 데이터를 저장하기 때문에 `HashMap`의 값으로 `HashMap`을 다시 저장할 수 있다.
- 이렇게 함으로써 하나의 키에 다시 복수의 데이터를 저장할 수 있게 된다.
- 전화번호 그룹을 만들고 그 그룹안에 다시 이름과 전화번호를 저장할 수 있게 된다.
- 이름대신 전화번호를 키로 사용했는데 이름은 동명이인이 있을 수 있지만 전화번호는 유일하기 때문이다.

##### `HashMap`예제 4 : `HashMap`을 이용해 그래프 표현해보기
```java
public class HashMapEx4 {
    public static void main(String[] args) {
        String[] data = {"A", "K", "A", "K", "D", "K", "A", "K", "K", "K", "Z", "D"};

        HashMap map = new HashMap();

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
Z : # 1
K : ###### 6
```
- 문자열 배열에 담기 문자열을 하나씩 읽어 `HashMap`에 키로 저장하고 값으로 1을 저장한다.
- `HashMap`에 같은 문자열이 키로 저장되어 있는지 `containsKey()`로 확인하고 저장되어 있으면 문자열의 값에 1을 증가시킨다.
- `HashMap`에 저장된 결과를 `printBar()`를 이용해 그래프로 표현한다.

##### 해싱(Hashing)?
해싱이란 **해시함수(hash function)를 이용해서 데이터를 해시테이블(hash table)에 저장하고 검색하는 기법** 을 말한다. 해시함수는 데이터가 저장되어 있는 곳을 알려주기 때문에 다량의 데이터 중에서도 원하는 데이터를 빠르게 찾을 수 있다. 해싱을 구현한 컬렉션 클래스로는 `HashSet`, `HashMap`, `HashTable` 등이 있다.

![hashing](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%9E%90%EB%B0%94%EC%9D%98_%EC%A0%95%EC%84%9D/11_collection_framework/img/hashing.png?raw=true)

해싱에 사용하는 자료구조는 위의 그림과 같이 배열과 `Linked List`의 조합으로 이루어져 있다. 저장할 데이터의 키를 해시함수에 넣으면 배열의 한 요소를 얻게되고, 다시 그 곳에 연결되어 있는 `Linked List`에 저장하게 된다.

그리고 데이터를 찾는 과정은 아래와 같다
- 검색하고자하는 값의 키로 해시함수를 호출한다.
- 해시함수의 계산결과인 해시코드를 이용해 해당 값이 저장되어 있는 `Linked List`를 찾는다.
- `Linked List`에서 검색하 키와 일치하는 데이터를 찾게된다.
