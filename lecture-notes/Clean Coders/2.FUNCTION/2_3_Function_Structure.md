# Clean Coders : Function Structure Part1

## 1. Agrumenet
- 인자가 많이지면 복잡도가 증가한다
- 인자의 최대 갯수는 3개까지만
  - 왜냐면 인자가 많이지면 외우기가 어렵다
  - Introducde Parameter Object 기법을 사용하자(https://www.refactoring.com/catalog/introduceParameterObject.html)
  - 많은 인자들을 추상화하는 클래스를 작성하고 그것을 인자로 쓰자.
- 아래 코드와 같이 생성자에 여러개의 인자를 넘겨야한다면?
  ```java
  pubilc class NutritionFacts {
    private final int servingSize; // reqired
    private final int servings; // reqired
    private final int calories; // optional
    private final int fat; // optional
    private final int sodium ;// optional
    private final int carbohydrate; // optional

    public NutritionFacts(int servingSize, int servings) {};
    public NutritionFacts(int servingSize, int servings, int calories) {};
    public NutritionFacts(int servingSize, int servings, int calories) {};
    public NutritionFacts(int servingSize, int servings, int calories
                          int fat, int sodium) {};
    public NutritionFacts(int servingSize, int servings, int calories
                          int fat, int sodium, int carbohydrate) {};
  }
  ```
  ```java
  public class NutritionFactsTest {
    @Test
    public void canCreate() {
      NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
  }
  ```
  - 만약 생성자의 하나의 타입으로 많은 인자가 넘어가게 되면 호출하는 쪽은 무엇을 인자로 넣어야하는지 모르게 된다.
    - 첫번째 해결책 : Java Bean Pattern
      - 좋은 이름을 갖는 setter를 사용하면된다. 하지만 그렇게 set이 완료되기 전까지는 객체는 불안정해진다.
    - 두번째 해결책 : Builder Pattern
      ```java
      pubilc class NutritionFacts {
        private final int servingSize; // reqired
        private final int servings; // reqired
        private final int calories; // optional
        private final int fat; // optional
        private final int sodium ;// optional
        private final int carbohydrate; // optional

        // Builder class
        public static class Builder {
          // reqired parameters
          private final int servingSize;
          private final int servings;
          // Optional parameters = initialized to default values
          private int calories = 0;
          private int fat = 0;
          private int sodium = 0;
          private int carbohydrate = 0;

          // Builder Contstructor
          public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
          }

          public Builder calories(int val) {
            calories = val;
            return this;
          };

          public Builder fat(int val) {
            fat = val;
            return this;
          };

          public Builder sodium(int val) {
            sodium = val;
            return this;
          };

          public NutritionFacts build() {
            return new NutritionFacts(this);
          };
        }

        private NutritionFacts(Builder builder) {
            servingSize = builder.servingSize;
            servings = builder.servings;
            calories = builder.calories;
            fat = builder.fat;
            sodium = builder.sodium;
            carbohydrate = builder.carbohydrate;
        }
      }
      ```
      ```java
      public class NutritionFactsTest {
        @Test
        public void canCreate() {
          NutritionFacts cocaCola =
                new NutritionFacts.Builder(240, 8)
                                  .calories(100)
                                  .sodium(35)
                                  .carbohydrate(27)
                                  .build();
        }
      }
      ```
- Boolean 인자 사용금지
  - 2가지 이상의 일을 하는 것이다.
    - true 경우를 위한 일
    - false 경우를 위한 일
  - 2개의 함수로 분리하는 것이 좋다.
- Innies not Outies
  - 인자를 output으로 사용하지 않아야한다.
  - 인자는 함수로 전달되는 것일뿐 함수로부터 변경되어 나오는 것이 아니다.
  - 인자로 받은 값을 로컬 변수에 저장, 변경하고 retuen value로 사용해야한다.

- the null of defense
  - null을 전달/기대하는 함수는 boolean을 전달하는 것 만큼 잘못된 것이다. null인 경우, 아닌 경우의 행위 2개의 함수를 만드는 것이 맞다.

## 2. The Stepdown Rule
- 모든 public 메서드, 함수는 위쪽으로
- 모든 private 메서드, 함수는 아래쪽으로
- 중요한 부분은 위로, 상세한 부분은 밑으로
- top에서 bottom으로 읽을 수 있도록 작성해야한다.

## 3. switches and cases
- Switch문을 쓰는 것을 왜 꺼리는가?
  - 멋지지 않다? 만족스러운 대답은 아니다...
- 객체지향의 가장 큰 이점은 의존성 관리 능력이다.
  1. 모듈 A가 모듈 B의 함수를 사용하는 경우
      ```
      Souce Code Dependency
      --------------------->
      Module A -> Module B
      --------------------->
      runtime Dependency ->
      ```
      - 의존성이 높아져 모듈 B의 코드가 변경되면 모듈 A에 영향을 미치게 된다.
      - 독립적인 배포/컴파일/개발이 불가능하다.
  2. 객체지향이 가능하게 하는 대단한 것은?
      ```
            Souce Code Dependency
            <---------------------
      Module A -> Interface <- Module B
            --------------------->
            runtime Dependency
      ```
      - runtime 의존성을 그대로 둔채로 Source Code의 의존성을 역전시킨다.
      - 절차
        1. 본래의 의존성을 제거
        2. polymophic interface를 삽입
        3. 모듈 A는 인터페이스에 의존하고, 모듈 B는 인터페이스로부터 파생된다.
- Switch 문장은 위의 1번처럼 독립적인 배포에 방해
  - 각 case문장은 외부 모듈에 의존성을 갖는다.
  - 다수의 다른 모듈에 의존성을 갖을 수 있다.
  - Switch문에서 souce code의 의존성은 flow of control과 방향이 같으므로 모든 외부 모듈에 영향을 받게된다.
  - 외부 모듈 중 하나라도 변경이 일어나게 된다면 Switch문장이 영향을 받게된다.
  - 독립적 배포가 불가능하게 만드는 많은 의존성을 만들게 된다.
- Switch 문장 제거 절차
  1. Switch문장을 polymophic interface 호출로 변환
  2. case에 있는 문장들을 별도의 클래스로 추출하여 변경 영향이 발생하지 않도록 한다.
- Switch 문장 제거 실습 (https://github.com/walbatrossw/videostore) : remove-switch-statement 브랜치
  1. Test부터 polymophic하게 수정
  2. make it pass
  3. 중복제거
  4. Push member down
  5. run with coverage
  6. remove unused code
  7. remove type code

## 4. Temporal Coupling

## 5. CQS

## 6. Tell Don't Ask

## 7. Law of Demeter

## 8. early returns

## 9. error handling

## 10. Special Cases

## 11. Null is not a error

## 12. Null is a value

## 13. try도 하나의 역할/기능이다.
