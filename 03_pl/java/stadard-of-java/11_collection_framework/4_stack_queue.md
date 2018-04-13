본 내용은 자바의 정석 3rd Edition을 참고하여 작성되었습니다. 개인적으로 학습한 내용을 복습하기 목적이기 때문에 내용상 오류가 있을 수 있습니다.

## 1. `Stack`, `Queue`

**`Stack`**
```
push -----┐    ┌-----▶ pop
          ▼    |
        |         |
        |---------|
        |    2    |
        |---------|
        |    1    |
        |---------|
        |    0    |
        |---------|
        |---------|
```
**마지막에 저장한 데이터를 가장 먼저 꺼내게 되는 LIFO(Last In First Out)의 구조** 로, 상자에 책을 쌓아둔 것을 위에서 부터 차례로 다시 꺼내는 것과 동일하다. 예를 들면 `Stack`에 0, 1, 2, 3을 차례로 데이터를 넣었다면, 꺼낼 때는 넣은 순서와 반대로 3, 2, 1, 0의 순서로 꺼내게 된다. `Stack`을 구현하기 위해서는 순차적으로 데이터를 추가하고 삭제하는 `Stack`에는 **`ArrayList`와 같은 배열기반의 컬렉션클래스가 적합** 하다.

**`Queue`**
```
offer -------┐    
             ▼    
        |         |
        |---------|
        |    2    |
        |---------|
        |    1    |
        |---------|
        |    0    |
        |---------|
        |         |
              |
              └-------▶ poll
```
**처음 저장한 데이터를 가장 먼저 꺼내게 되는 FIFO(First In First Out)의 구조** 로, 양옆에 뚫린 파이프에서 한쪽은 넣고, 다른 한쪽은 빼는 것과 동일하다. 예를 들면 `Queue`에 0, 1, 2, 3, 4를 차례로 데이터를 넣었다면, 꺼낼 때는 넣은 순서와 마찬가지로 0, 1, 2, 3, 4의 순서로 꺼내게 된다. `Queue`를 구현하기 위해서는 **`LinkedList`로 구현** 하는 것이 적합하다. 그 이유는 만약 `ArrayList`와 같이 배열기반의 컬렉션클래스를 사용한다면 데이터를 꺼낼 때 항상 첫번째 저장한 데이터를 삭제하므로 빈 공간을 채우기 위해 데이터의 복사가 발생해 비효율적이기 때문이다.

##### `Stack`, `Queue` 예제 1 : 비교
```java
public class StackQueueEx {
    public static void main(String[] args) {
        Stack stack = new Stack();
        Queue queue = new LinkedList();

        stack.push("0");
        stack.push("1");
        stack.push("2");
        stack.push("3");

        queue.add("0");
        queue.add("1");
        queue.add("2");
        queue.add("3");

        System.out.println("stack");
        while (!stack.empty())
            System.out.println(stack.pop());

        System.out.println();

        System.out.println("Queue");
        while (!queue.isEmpty())
            System.out.println(queue.poll());
    }
}
```
```
Stack
3
2
1
0

Queue
0
1
2
3
```
- 스택과 큐는 각각 0, 1, 2, 3을 같은 순서로 넣고 꺼냈을 때 결과가 다르게 나온다는 것을 알 수 있다.
- java에서는 스택을 `Stack`클래스로 구현하여 제공하지만, 큐는 `Queue`인터페이스만 정의해놓았을 뿐 별도의 클래스를 제공하지 않기 때문에 `Queue`인터페이스를 구현한 클래스들 중에 하나를 사용해야한다.

##### `Stack` 구현 예제
```java
public class MyStack extends Vector {

    public Object push(Object item) {
        addElement(item);
        return item;
    }

    public Object pop() {
        Object obj = peek(); // stack에 저장된 마지막 요소를 읽어옴, stack이 비어있으면 EmptyStackException 발생
        removeElementAt(size() - 1);    // 마지막 요소삭제, 0부터시작하기 때문에 1을 빼줌
        return obj;
    }

    public Object peek() {
        int length = size();
        if (length == 0) {
            throw new EmptyStackException();
        }
        // 마지막 요소를 반환, 배열의 인덱스가 0부터 시작하기 때문에 1을 빼줌
        return elementAt(length - 1);
    }

    public boolean empty() {
        return size() == 0;
    }

    public int search(Object o) {
        int i = lastIndexOf(o); // 끝에서부터 객체를 찾는다.
        // 반환값은 저장된 위치(배열의 인덱스)
        if (i >= 0) {
            return size() - i;
        }
        // 객체를 찾지 못했을 경우 -1 반환
        return - 1;
    }
}
```

**`Stack`, `Queue`활용**
스택과 큐에 대해 알아보았는데 그렇다면 스택과 큐는 어떻게 활용되는지에 대해 알아보자.
- `Stack` : 수식계산, 수식괄호검사, 웹브라우저의 뒤로/앞으로 등
- `Queue` : 최근사용문서, 인쇄작업 대기목록, 버퍼(Buffer)

##### `Stack` 활용 예제 1 : 브라우저의 뒤로가기, 앞으로 가기 버튼 구현
```java
public class StackEx1 {

    public static Stack back = new Stack();
    public static Stack forward = new Stack();

    public static void main(String[] args) {
        goURL("1.naver");
        goURL("2.google");
        goURL("3.daum");
        goURL("4.youtube");

        printStatus();

        goBack();
        System.out.println("뒤로가기 버튼 클릭");
        printStatus();

        goBack();
        System.out.println("뒤로 버튼 클릭");
        printStatus();

        goForward();
        System.out.println("앞으로 버튼 클릭");
        printStatus();

        goURL("github.com");
        System.out.println("새로운 주소 이동");
        printStatus();
    }

    public static void printStatus() {
        System.out.println("back : " + back);
        System.out.println("forward : " + forward);
        System.out.println("현재화면은 " + back.peek() + "입니다.");
        System.out.println();
    }

    public static void goURL(String url) {
        back.push(url);
        if (!forward.empty())
            forward.clear();
    }

    public static void goForward() {
        if (!forward.empty())
            back.push(forward.pop());
    }

    public static void goBack() {
        if (!back.empty())
            forward.push(back.pop());
    }
}
```
```
back : [1.naver, 2.google, 3.daum, 4.youtube]
forward : []
현재화면은 4.youtube입니다.

뒤로가기 버튼 클릭
back : [1.naver, 2.google, 3.daum]
forward : [4.youtube]
현재화면은 3.daum입니다.

뒤로 버튼 클릭
back : [1.naver, 2.google]
forward : [4.youtube, 3.daum]
현재화면은 2.google입니다.

앞으로 버튼 클릭
back : [1.naver, 2.google, 3.daum]
forward : [4.youtube]
현재화면은 3.daum입니다.

새로운 주소 이동
back : [1.naver, 2.google, 3.daum, github.com]
forward : []
현재화면은 github.com입니다.
```
- 웹브라우저의 뒤로, 앞으로 가기 버튼의 기능을 구현한 것인데 이 기능을 구현하기 위해서는 2개의 스택을 사용해야한다.

##### `Queue`활용 예제 1 : 유닉스의 History명령어 구현
```java
public class QueueEx1 {

    static Queue queue = new LinkedList();
    static final int MAX_SIZE = 5;

    public static void main(String[] args) {
        System.out.println("help를 입력하면 도움말을 볼 수 있습니다.");

        while (true) {
            System.out.print(">>");
            try {

                Scanner scanner = new Scanner(System.in);
                String input = scanner.nextLine().trim();

                if ("".equals(input))
                    continue;

                if (input.equalsIgnoreCase("q")) {
                    System.exit(0);
                } else if (input.equalsIgnoreCase("help")) {
                    System.out.println("help - 도움말을 보여줍니다.");
                    System.out.println("q 또는 Q - 프로그램을 종료합니다.");
                    System.out.println("history - 최근에 입력한 명령어를 " + MAX_SIZE +" 개 보여줍니다.");
                } else if (input.equalsIgnoreCase("history")) {
                    int i = 0;
                    save(input);

                    LinkedList temp = (LinkedList) queue;
                    ListIterator listIterator = temp.listIterator();

                    while (listIterator.hasNext()) {
                        System.out.println(++i+"."+listIterator.next());
                    }
                } else {
                    save(input);
                    System.out.println(input);
                }

            } catch (Exception e) {
                System.out.println("입력오류입니다.");
            }
        }
    }

    public static void save(String input) {
        if (!"".equals(input))
            queue.offer(input);

        if (queue.size() > MAX_SIZE)
            queue.remove();
    }
}
```
```
help를 입력하면 도움말을 볼 수 있습니다.
>>help
help - 도움말을 보여줍니다.
q 또는 Q - 프로그램을 종료합니다.
history - 최근에 입력한 명령어를 5 개 보여줍니다.
>>ls -al
ls -al
>>cd ~
cd ~
>>mkdir
mkdir
>>history
1.ls -al
2.cd ~
3.mkdir
4.history
>>q
```

##### `PriorityQueue`(우선순위 큐)와 예제
`Queue`인터페이스의 구현체 중의 하나로 저장한 순서와 관계없이 우선순위가 높은 것부터 꺼내게 된다는 특징이 있다. 그리고 `null`은 저장할 수 없으며, 만약 `null`을 저장하면 `NullPointerException`이 발생한다.
```java
public class PriorityQueueEx {
    public static void main(String[] args) {
        Queue priorityQueue = new PriorityQueue();
        priorityQueue.offer(3);
        priorityQueue.offer(1);
        priorityQueue.offer(5);
        priorityQueue.offer(2);
        priorityQueue.offer(4);
        System.out.println(priorityQueue);

        Object obj = null;
        while ((obj = priorityQueue.poll()) != null)
            System.out.println(obj);
    }
}
```
```
[1, 2, 5, 3, 4]
1
2
3
4
5
```
- 저장순서가 3, 2, 0, 1, 4 인데도 출력결과는 1, 2, 3, 4, 5 이다. 우선순위는 숫자가 작을수록 높은 것이므로 1이 가장 먼저 출력되었다.
- 숫자뿐만아니라 객체를 저장할 수도 있는데 그럴 경우 각 객체의 크기를 비교할 수 있는 방법을 제공해야한다.
- 참보변수 `priorityQueue`를 출력하면 내부적으로 가지고 있는 배열의 내용이 출력되는데 저장한 순서와 다르게 나온다. 힙이라는 자료구조의 형태로 저장되었기 때문이다.

## 2. `Deque`(Double-Ended Queue)
```
offerLast -------┐   ┌-----▶ pollLast
                 ▼   |
              |         |
              |---------|
              |    2    |
              |---------|
              |    1    |
              |---------|
              |    0    |
              |---------|
              |         |
                 ▲   |
offerFirst ------┘   └-------▶ pollFirst
```
`Queue`의 변형으로 한쪽 끝으로만 추가/삭제할 수 있는 `Queue`와 달리 `Deque`는 양쪽 끝에 추가/삭제가 가능하다. `Deque`의 조상은 `Queue`이며 구현체로는 `ArrayDeque`와 `LinkedList`등이 있다.
