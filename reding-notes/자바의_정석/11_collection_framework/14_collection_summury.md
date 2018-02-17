본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. 컬렉션 클래스간의 관계도
![summary](https://github.com/walbatrossw/develop-notes/blob/master/reding-notes/%EC%9E%90%EB%B0%94%EC%9D%98_%EC%A0%95%EC%84%9D/11_collection_framework/img/collection_class_relation.png?raw=true)

## 2. 컬렉션 클래스의 특징 요약
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
