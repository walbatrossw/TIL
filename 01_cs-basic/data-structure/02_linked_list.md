# 연결리스트(Linked List)

## 1. 연결리스트란?

![](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/linked_list/LinkedList.png?raw=true)

단순 연결리스트(Singly Linked List)는 동적 메모리 할당을 이용해 리스트를 구현하는 가장 간단한 형태의 자료구조이다. 동적 메모리를 할당 받아 노드(Node)를 저장하고, 노드는 참조를 이용하여 다음 노드를 가리키도록 만들어 노드들을 한 줄로 연결한다.

#### # 단순 연결리스트의 구성
단순 연결리스트의 노드는 데이터와 참조값으로 구성되는데 참조 값은 한 노드에서 다른 노드를 가리키는 포인터 역할을 한다. 그리고 마지막 참조값은 `null`을 가리키게 된다.
```java
class Node<T> {
  T data;         // 데이터
  Node nextNode;  // 다음 노드를 가리키는 참조변수
}
```
하나의 노드는 위와 같이 데이터와 다음노드를 가리키는 참조값으로 구성된다.


#### # 연결리스트 특성
- 연결 리스트는 단순하고 매우 일반적인 데이터 구조로 `Stack`, `Queue` 등의 다른 일반적인 데이터 유형을 구현하는 데 사용할 수 있다.
- 연결리스트 자체로는 데이터에 대한 랜덤 액세스를 허용하지 않으므로 인덱스를 사용할 수 없다.
- 리스트의 마지막 노드를 가져오거나, 지정된 데이터가 들어 있는 노드를 찾거나, 새 노드를 삽입할 위치를 찾는 등의 많은 기본 작업을 수행하려면 대부분 또는 전체 리스트의 요소를 순차적으로 검사해야만 한다.

#### # 연결리스트의 장점
- 배열과 다르게 연결리스트는 동적 자료구조이기 때문에 런타임 시에 필요한 메모리를 할당 할 수 있다.
- 첫번째 요소 조작에 매우 효율적이다.
- 구현이 간편하다.
- 크기가 서로 다른 요소를 저장할 수 있다.
- 연결리스트의 크기는 유기적으로 확장할 수 있지만, 배열의 경우 최초에 배열의 크기를 미리 예측하여 결정해야하고, 확장해야할 경우에는 다시 생성해야만한다.

#### # 연결리스트의 단점
- 참조로 인해 메모리 낭비가 된다.
- 연결리스트는 항목을 탐색하려면 항상 첫 노드부터 원하는 노드를 찾을 때까지 차례로 방문하는 순차 탐색을 해야만한다.
- 역방향 이동에는 제약이 발생하기 때문에 이중연결리스트를 구현해야하고, 이렇게 되면 이전 노드의 참조가 필요하게 됨으로 메모리가 추가적으로 필요하게 된다.

## 2. 연결리스트의 삽입, 삭제 작업

#### # 맨 앞에 노드를 삽입할 경우
![](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/linked_list/LinkedList_insert_front.gif?raw=true)
```java
linkedList.insertAtStart(10); // 10 첫노드에 삽입
linkedList.insertAtStart(4);  // 4 첫노드에 삽입
linkedList.insertAtStart(-5); // -5 첫노드에 삽입
```
연결리스트에 새로운 노드를 맨 앞에 삽입하는 방법은 새로운 노드의 참조값을 첫 노드로 변경하기만 하면 된다.

#### # 삽입 : 맨 뒤에
![](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/linked_list/LinkedList_insert_last.gif?raw=true)
```java
linkedList.insertAtEnd(25); // 25 마지막 노드에 삽입
```
연결리스트에 새로운 노드를 맨 마지막에 삽입하는 것은 비교적 간단하지는 않다. 연결리스트의 첫번째 노드부터 마지막 노드(참조값이 `null`인 노드)까지 탐색을 한 뒤 마지막 노드의 참조값을 새로운 노드로 변경하고, 새로운 노드의 참조값을 `null`을 가리키게 하면된다.

#### # 삭제 : 맨 앞의 노드
![](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/linked_list/LinkedList_remove_first.gif?raw=true)
```java
linkedList.removeStart()
```
연결 리스트의 첫 노드는 매우 빠르게 제거할 수 있다. 노드를 추가적으로 탐색할 필요가 없으므로 참조값만 변경하면 된다.

#### # 삭제 : 특정 데이터를 가진 노드
![](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/linked_list/LinkedList_remove.gif?raw=true)
```java
linkedList.remove(10)
```
연결 리스트에서 특정 데이터를 가진 노드를 제거하는 것은 빠르지 않을수도 있다. 만약 특정 데이터를 가진 노드가 마지막에 존재한다면 첫번째 노드부터 마지막 노드까지 탐색을 해야되기 때문이다.
특정 데이터를 가진 노드의 탐색을 완료한 뒤 특정 데이터를 가진 노드의 이전 노드가 다음 노드를 가리키도록 참조값을 변경해주면된다.


## 3. 단순 연결리스트의 문제점
![](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/linked_list/LinkedList_problem.png?raw=true)
위의 그림에서 보면 4에서 25까지 탐색을 수행할 수 있지만 역으로 25에서 4로 탐색을 수행할 수 없다. 그 이유는 참조 값의 반대방향이기 때문이다.

![](https://github.com/walbatrossw/TIL/blob/master/01_cs-basic/data-structure/img/linked_list/DoublyLinkedList_problem.png?raw=true)
이중 연결리스트 구현을 통해 역방향으로도 탐색이 가능하게 할 수 있다. 이중 연결리스트는 노드에 두개의 참조값을 가지는데 하나는 다음노드, 또다른 하나는 이전 노드를 가진다. 하지만 이중 연결리스트는 참조값이 2개이기 때문에 단순 연결리스트에 비해 메모리의 공간을 더 많이 차지하게 되는 단점이 존재한다.

## 4. 연결리스트와 배열 차이비교

#### # 검색
- 배열은 랜덤 엑세스가 가능하다.
- 배열이 연결리스트보다 검색이 빠르다.
- 배열은 인덱스를 통해 검색하는 속도가 빠르지만 연결리스트는 특정 요소를 검색하기 위해서는 모든 항목을 탐색해야한다.

#### # 삭제
- 연결리스트의 삭제 작업은 참조값을 변경하면 되기 때문에 빠르게 수행할 수 있다.
- 배열의 경우 삭제 작업을 할 때 배열을 재구성해야되기 때문에 연결리스트에 비해 상대적으로 느리다.

#### # 메모리
- 배열은 추가 메모리가 필요하지 않다.
- 연결리스트는 참조값 때문에 추가적으로 메모리가 필요하다.

## 5. 연결리스트의 구현
