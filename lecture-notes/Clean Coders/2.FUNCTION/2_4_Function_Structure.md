# Clean Coders : Function Structure Part2

## 4. Temporal Coupling
- 참수는 순서를 지키며 호출되어야한다.
  - open, execute, close

## 5. CQS(Command Query Separation)
- Side Effect를 관리하는 좋은 방법
- Command? : 리턴타입은 항상 `void`
  - 시스템의 상태를 변경 가능
  - side effect를 갖는다.
  - 아무것도 반환하지 않는다.
- Query?
  - 계산값이나 시스템 상태를 반환
  - side effect가 없다
  - 언제 어디서든 호출하면 같은 값을 반환해야한다.
- CQS???
  - 상태를 변경하는 함수는 값을 반환해서는 안된다.
  - 값을 반환하는 함수는 상태를 변경해서는 안된다.
- CQS의 예
  - CQS가 잘지켜지지 않은 경우
    ```java
    User user = authorizer.login(userName, password);
    ```
    - user를 사용하기 위해서는 항상 로그인을 해야만하는 상황
    - login을 하면 원치 않아도 user정보를 읽어야만하는 상황
  - CQS가 지켜진 경우
    ```java
    authorizer.login(userName, password);
    User user = authorizer.getUser(userName);
    ```
    - user를 사용하기 위해서 항상 로그인을 하지 않아도 된다.
    - login을 하면서 user정보도 함께 읽을 필요가 없어졌다.
- 약속을 지켜서 작성해야한다.
  - 값을 반환하는 함수는 상태를 변경해서는 안된다.
  - 상태를 변경하는 함수는 exception을 발생시킬 수는 있지만, 값을 반환해서는 안된다.

## 6. Tell Don't Ask(TDA법칙)
- Extreme한 CQS는 C와 Q를 사용하지 않도록 해야한다.
  - 3가지의 동일한 로직 중에 어느 것이 보기에 좋은가?
    - `if else`문 사용
      ```java
      // 로그인 되어있으면(Q)
      if(user.isLoggedIn())
        // 명령을 실행(C)
        user.execute(command);
      else
        // 그렇지 않으면 로그인을 하도록 유도(C)
        authenticator.promptLogin();
      ```
    - `Exception`를 사용
      ```java
      try {
        // 명령을 실행(C)
        user.execute(command);
      } catch(User.NotLoggedIn e) {
        // 예외가 발생되면 로그인 유도(C)
        annuciator.promptLogin();
      }
      ```
    - user객체가 모든 것을 처리
      ```java
      // 명령을 실행하면서 로그인되어 있지않다면 로그인을 유도
      user.execute(command, annuciator);
      ```
  - 로그인 여부는 user객체 스스로가 알고 있어야만한다.
  - user의 상태를 가져와 user를 대신해서 결정해서는 안된다.
  - user가 해당 행위를 수행하는 것이 바람직하다.
- Tell Don't Ask는 CQS를 강화하는 역량이다.
  - 객체가 무엇을 할지 지시해야하는 것이지 객체의 상태를 묻고 직접 변경하려해서는 안된다.
  - 이 규칙을 준수하다보면 Q의 필요가 많이 없어지게 된다.
  - Q는 곧 out of control이 되는 경향이 많기 때문이다.
  - 객체의 상태를 많이 드러내게 되면 객체 스스로가 제어할 수가 없어지게 된다.
- Long Chain of Function(Train wrecks) : Tell, Don't ask의 위반, Tell을 하기전에 계속 Ask를 해야한다.
  ```java
  o.getX()
      .getY()
        .getZ()
          .doSomeThing();
  ```
  - `o`는 `X`를 `X`는 `Y`를 `Y`는 `Z`를 그리고 `Z`는 `doSomeThing()`을 할 수 있다.
  - 한라인의 코드가 알아야하는 것으로는 방대한 내용이고, 함수가 시스템에 너무 많은 의존성을 지니게 된다.

  ```java
  o.DoSomeThing();
  ```
  - `o`에게 무엇을 하라고 지시를 했지만 어떻게 해야할지를 모른다.
  - 하지만 `o`는 어떤 다른 객체에게 tell을 하면되는지 알기 때문에 최종적으로 처리되기까지 요청이 전파된다.
  - 위의 코드는 `doSomeThing()`을 실행하기까지 여러 함수를 알아야하지만, 아래의 코드는 `DoSomeThing()`만 알면된다.
  - 아래의 코드는 `DoSomething()`이 호출되었는지만 알면 테스트를 진행할 수 있지만 위의 코드는 테스트를 하기 위해 `getX()`부터 `getZ()`까지 모든 `mock`을 설정해줘야하는 번거로움이 발생한다.

## 7. Law of Demeter(드미터의 법칙)
- 함수가 시스템의 전체를 알게 하면 안된다.
- 개별 함수는 아주 제한된 정보만을 가져야 한다.
- 객체는 요청된 기능을 수행하기 위해 이웃 객체에게 요청해야한다.
- 요청을 수신하면 적절한 객체가 수신할 때까지 전파되어야만 한다.
- 아래와 같은 객체들의 메서드만 호출이 가능하다
  - 인자로 전달된 객체의 메서드
  - 로컬에서 생선한 객체의 메서드
  - 필드로 선언된 객체의 메서드
  - 전역 객체
- Ask대신 Tell을 하면 주변 객체와의 의존성이 낮아지게 된다.

## 8. early returns
```java
private boolean nameIsValxxxx {
  if (name.equals(""))
    return true;
  if (WikiWordWidget.xxx)
    return true;
  return false;
}
```
- early return, guarded return은 허용된다.
- `if else`문에서 `if`에서 모든 조건을 전부 확인하고 `else`에서 최종적으로 리턴을 하는 것은 바람직하지 않다. 오히려 `if`에서 부정조건을 넣어 확인한 다음바로 리턴할 수 있도록 하는 것이 더 좋다.
- 하지만 roof 중간에서 리턴하는 것은 문제가 발생한다.
  - break, roof 중간에서 return은 roof를 복잡하게 만들어버린다.
  - 그렇기 때문에 좀더 깊이 있게 생각을 해봐야할 문제이다.
- 코드가 동작하도록하는 것보다 이해할 수 있도록하는 것이 매우 중요하다.

## 9. error handling

## 10. Special Cases

## 11. Null is not a error

## 12. Null is a value

## 13. try도 하나의 역할/기능이다.
