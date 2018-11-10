> 본 글은 [이펙티브 자바 2nd](https://book.naver.com/bookdb/book_detail.nhn?bid=8064518)를
읽고 개인적으로 학습한 내용을 복습하기 위해 작성된 글로 내용상 오류가 있을 수 있습니다.
오류가 있다면 지적 부탁드리겠습니다.

# Rule 2. Builder 패턴의 사용 : 생성자의 인자가 많을 때

**객체를 생성하기 위해 사용되는 정적 팩터리 메서드, 생성자 둘다 선택적 인자가 많은 상황에
적응하지 못하는 문제점이 있다.**

포장 판매되는 음식의 영양 성분표를 나타내는 클래스를 예를 들어보자. 반드시 포함되어야
하는 필수 항목과 포함되지 않아도 되는 선택적인 항목은 아래와 같다.

- 필수 항목
  - 총 제공량(serving size)
  - 1회 제공량(servings per cotainer)
  - 1회 제공량 당 칼로리(calories per servings)
- 선택 항목
  - 지방
  - 나트륨
  - 탄수화물
  - 칼로리
  - 콜레스테롤

이러한 클래스에는 어떤 생성자나 정적 팩터리 메서드가 적합한지 3가지 방법을 통해 알아보자.

## 2.1 점층적 생성자 패턴(Telescoping Constructor Pattern)

### 2.1.1 점층적 생성자 패턴이란?

**점층적 생성자 패턴은 필수 인자만 받는 생성자를 정의하고, 선택적 인자를 하나, 둘, 셋 하나씩
추가 시킨 생성자를 쌓아 올리듯 정의한다.**

```java
public class NutritionFact {

    // 필수 인자
    private final int servingSize;
    private final int servings;

    // 선택 인자
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;


    // 필수 인자 생성자
    public NutritionFact(int servingSize, int servings) {
        this(servingSize, servings, 0); // 필수 + 선택 인자1 생성자 호출
    }

    // 필수 + 선택 인자1 생성자
    public NutritionFact(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0); // 필수 + 선택 인자2 생성자 호출
    }

    // 필수 + 선택 인자2 생성자
    public NutritionFact(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0); // 필수 + 선택 인자3 생성자 호출
    }

    // 필수 + 선택 인자3 생성자
    public NutritionFact(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0); // 전체 인자 생성자 호출
    }

    // 전체 인자 생성자
    public NutritionFact(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }

}
```

```java
NutritionFact cocaCola = new NutritionFact(240, 8, 3, 35, 27);
```

### 2.1.2 점층적 생성자 패턴의 단점

- 인자 수가 늘어나면 클라이언트에서 코드를 작성이 어렵고, 혼동하기 쉽다.
- 가독성이 떨어지기 때문에 어떤 인자인지 알 수가 없다.

## 2.2 자바빈 패턴(Java Bean Pattern)

### 2.2.1 자바빈 패턴이란?

**자바빈 패턴은 인자없는 생성자를 호출하여 객체부터 만든 다음, 설정 메서드(setter)들을
호출하여 필수, 선택 필드의 값을 채우는 방식이다.**

```java
public class NutritionFact {

    // 필수 인자
    private int servingSize = -1;       // 필수
    private int servings = -1;          // 필수

    // 선택 인자
    private int calories = 0;           // 선택
    private int fat = 0;                // 선택
    private int sodium = 0;             // 선택
    private int carbohydrate = 0;       // 선택

    // Setter 메서드
    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }

}
```

```java
NutritionFact cocaCola = new NutritionFact();   // 객체 생성

// 필수, 선택 필드 값 채우기
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setFat(35);
cocaCola.setCarbohydrate(27);
```

### 2.2.2 단점

- 점층적 생성자 패턴의 단점인 가독성 면에서는 좋아졌지만 코드의 양이 증가한다.
- 1회 함수 호출로 객체 생성을 끝낼 수 없으므로, 객체 일관성이 일시적으로 깨질 수 있다.
- 변경불가능 클래스를 만들 수 없다.

## 2.3 빌더 패턴(Builder Pattern)

### 2.3.1 빌더 패턴이란?

**빌더 패턴은 점정측 생성자 패턴의 안정성에 자바빈 패턴의 가독성을 결합한 형태이다.**

```java
public class NutritionFact {

    // 필수 인자
    private final int servingSize;
    private final int servings;

    // 선택적 인자
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    // private 생성자 : 변경 불가능 객체 생성
    private NutritionFact(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
        this.sodium = builder.sodium;
        this.carbohydrate = builder.carbohydrate;
    }

    // 정적 내부 클래스
    public static class Builder {

        // 필수 인자
        private int servingSize;   // 필수
        private int servings;      // 필수

        // 선택적 인자 : 기본값 초기화
        private int calories = 0;     // 선택
        private int fat = 0;          // 선택
        private int sodium = 0;       // 선택
        private int carbohydrate = 0; // 선택

        // 내부 클래스 생성자 : 필수 인자만
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        // 선택적 인자 설정 메서드
        public Builder calories(int value) {
            this.calories = value;
            return this;
        }

        // 선택적 인자 설정 메서드
        public Builder fat(int value) {
            this.fat = value;
            return this;
        }

        // 선택적 인자 설정 메서드
        public Builder sodium(int value) {
            this.sodium = value;
            return this;
        }

        // 선택적 인자 설정 메서드
        public Builder carbohydrate(int value) {
            this.carbohydrate = value;
            return this;
        }

        // 변경 불가능 객체 생성 메서드
        public NutritionFact build() {
            return new NutritionFact(this);
        }

    }

}
```

```java
NutritionFact cocaCola = new NutritionFact.Builder(240, 8)
        .calories(180).sodium(35).carbohydrate(27).build();
```

빌더 패턴을 통해 객체를 생성하는 과정은 아래와 같다.

1. 필요한 객체를 직접 생성하는 대신 먼저 필수 인자들을 생성자(정적 팩터리 메서드)에 전달하여
빌더 객체를 만든다.  
2. 빌더 객체에 정의된 설정 메서드들을 호출하여 선택적 인자를 추가한다.
3. `bulid`메서드를 호출하여 변경 불가능한 객체를 만든다.

### 2.3.2 빌더 패턴의 장단점

#### 장점
- 빌더 패턴은 작성하기도 쉽고, 가독성도 높아진다.
- 인자에 불변식을 적용할 수 있다.
  - `build`메서드에서 불변식을 위반한 경우, `IllegalStatementException`을 던져야 한다.
  - 설정자 메서드에서 불변식을 위한한 경우, `IllegalArgumentException`을 던져야 한다.
- 각 설정 메서드마다 가변인자(varargs) 사용이 가능하다.
- 하나의 빌더 객체로 여러 객체를 만들 수 있기 때문에 유연하다.

#### 단점
- 객체를 생성하려면 빌더 객체부터 생성해야 하기 때문에 성능이 중요한 상황에서는 문제가
발생할 수 있다.
- 빌더 패턴은 인자가 충분히 많은 상황에서만 사용하는 것이 바람직하다.
- 앞으로 인자가 늘어날 경우를 대비해 사용하는 것이 나을 수 있다.

## 2.4 요약

빌더 패턴은 인자가 많은 생성자 또는 정적 팩터리가 필요한 클래스를 설계할 때, 대부분의
인자가 선택적 인자인 상황에 유용하다. 클라이언트 코드의 가독성은 점층적 생성자 패턴보다
좋고, 결과물은 자바빈보다 훨씬 안전하다.
