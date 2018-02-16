본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## `Properties`란?
`Properties`는 `HashMap`의 구버전인 `Hashtable`을 상속받아 구현한 것으로, `Hashtable`은 키와 값을 `(Object, Object)`의 형태로 저장하는데 비해 **`Properties`는 `(String, String)`형태로 저장하는 보다 단순화된 컬렉션 클래스** 이다.
**주로 애플리케이션의 환경설정과 관련된 속성을 저장하는데 사용되며 데이터를 파일로부터 읽고 쓰는 편리한 기능을 제공한다.** 그래서 간단한 입출력은 `Properties`를 활용하면 몇 줄의 코드로 쉽게 해결할 수 있다.

##### `Properties`예제 1 : 저장, 읽기, 출력
```java
public class PropertiesEx1 {
    public static void main(String[] args) {
        Properties properties = new Properties();

        properties.setProperty("timeout", "30");
        properties.setProperty("language", "kr");
        properties.setProperty("size", "10");
        properties.setProperty("capacity", "10");

        // properties에 저장된 요소들을 Enumeration을 이용해 출력
        Enumeration enumeration = properties.propertyNames();

        while (enumeration.hasMoreElements()) {
            String element = (String) enumeration.nextElement();
            System.out.println(element + "=" + properties.getProperty(element));
        }

        System.out.println();

        properties.setProperty("size", "20");
        System.out.println("size=" + properties.getProperty("size"));
        System.out.println("capacity=" + properties.getProperty("capacity", "20"));
        System.out.println("loadfactor=" + properties.getProperty("loadfactor", "0.75"));

        System.out.println(properties);

        System.out.println();

        properties.list(System.out);
    }
}
```
```
capacity=10
size=10
timeout=30
language=kr

size=20
capacity=10
loadfactor=0.75
{capacity=10, size=20, timeout=30, language=kr}

-- listing properties --
capacity=10
size=20
timeout=30
language=kr
```
- `properties`의 기본적인 메서드를 이용해서 저장하고, 읽어오고, 출력하는 방법을 보여주는 예제이다.
- 데이터를 저장하는데 사용되는 `setProperty()`는 단순히 `Hashtable`의 `put`메서드를 호출한다.
- `setProperty()`는 기존에 같은 키로 저장된 값이 있는 경우 그 값을 `Object`타입으로 반환하며, 그렇지 않을 경우 `null`을 반환한다.
- `getProperty()`는 `properties`에 저장된 값을 읽어오는 일을 하는데 만일 읽어오려는 키가 존재하지 않으면 지정된 기본값을 반환한다.
- `Properties`는 컬렉션프레임워크 이전의 구버전이기 때문에 `Iterator`가 아닌 `Enumeration`을 사용한다.
- `list()`를 이용하면 `Properties`에 저장된 모든 데이터를 화면 또는 파일에 편리하게 출력할 수 있다.
