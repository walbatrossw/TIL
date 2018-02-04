## 1.14 `Collection Framework` 간단 정리

### 1) 컬렉션 클래스간의 관계도
![]()

### 2) 컬렉션 클래스의 특징 요약
|컬렉션|특징|
|---|---|
|`ArrayList`|배열기반, 데이터의 추가/삭제 불리, 순차적인 추가/삭제 제일 빠름, 임의의 요소에 대한 접근성이 뛰어남|
|`LinkedList`|연결기반, 데이터의 추가/삭제 유리, 임의의 요소에 대한 접근성이 좋지 않음|
|`HashMap`|배열과 연결의 결합형태, 추가/삭제/검색/접근성 모두 뛰어남, 검색에 최고성능|
|`TreeMap`|연결기반, 정렬과 검색(특히 범위검색)에 적합, 검생성능은 `HashMap`보다 떨어짐|
|`Stack`|`Vector`를 상속받아 구현|
|`Queue`|`LinkedList`가 `Queue`인터페이스를 구현|
|`Properties`|`Hashtable`을 상속받아 구현|
|`HashSet`|`HashMap`을 이용해서 구현|
|`TreeSet`|`TreeMap`을 이용해서 구현|
|`LinkedHashMap`/`LinkedHashSet`|`HashMap`과 `HashSet`에 저장순서유지기능을 추가|
