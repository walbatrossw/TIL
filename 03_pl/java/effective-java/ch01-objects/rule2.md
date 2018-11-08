# 2. 생성자의 인자가 많을 때는 Builder 패턴 적용을 고려하라.

정적 팩터리 메서드를 사용하는 방법이나 생성자를 사용하는 방법 모두 객체를 생성할 때
선택적 인자가 많은 상황에서 적응하지 못한다는 문제를 가지고 있다.

포장 판매되는 음식에 붙어있는 영양 성분표를 나타내는 클래스를 예를 들어보자. 이 클래스에는
아래와 같이 반드시 포함되어야하는 항목이 있다.

-   총 제공량(serving size)
-   1회 제공량(servig per container)
-   1회 제공량당 칼로리(calories per serving)

그리고 선택적인 항목들은 지방, 나트륨, 탄수화물, 칼로리, 등 20가지 넘는다. 이러한 경우
어떻게 객체를 생성해야 할까? 그 방법에 대해서 알아보자.

## 2.1 점층적 생성자 패턴(telescoping constructor pattern)

첫번째 방법으로 점층적 생성자 패턴을 사용할 수 있다. 필수 인자만 받는 생성자를 생성하고,
선택적 인자를 하나씩 늘려가며 생성자를 각각 정의 하는 방식으로 생성자드을 쌓아 올리듯
추가한다.

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
        this(servingSize, servings, 0);
    }

    // 선택 인자 생성자
    public NutritionFact(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    // 선택 인자 생성자
    public NutritionFact(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    // 선택 인자 생성자
    public NutritionFact(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
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
NutritionFact cocaCola = new NutritionFact(240, 8, 100, 3, 35, 27);
```

하지만 문제점은 설정할 필요가 없는 필드에도 인자를 전달해야한다. 인자 수가 늘어나게 되면
클라이언트 코드를 작성하기 어렵고, 가독성이 낮아진다.

## 2.2 자바 빈 패턴(Java Bean Pattern)

두번째 방법은 자바 빈 패턴으로 인자가 없는 생성자를 호출하여 객체를 만들고, `set` 메서드를
호출하여 필수 필드, 선택적 필드의 값을 채우는 방식이다.

```java
public class NutritionFact {
    private int servingSize = -1;        // 필수
    private int servings = -1;           // 필수
    private int calories = 0;           // 선택
    private int fat = 0;                // 선택
    private int sodium = 0;             // 선택
    private int carbohydrate = 0;       // 선택

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
NutritionFact cocaCola = new NutritionFact();
cocaCola.setServigSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setFat(35);
cocaCola.setCarbohydrate(27);
```

자바 빈 패턴의 문제점은 1회 함수 호출로 객체 생성을 끝낼 수 없고, 객체 일관성이 일시적으로 깨질 수 있다. 그리고 변경불가능(immutable) 클래스를 만들 수 없고, 스레드 안전성을 제공하기 위해 해야할
일도 많아진다.

## 2.3 빌더 패턴(builder pattern)

세번째 빌더 패턴은 점층적 생성자 패턴의 안전성에 자바빈 패턴의 가독성을 결합한 형태이다.
빌더 패턴은 아래와 같이 코드를 작성한다.

1. 필요한 객체를 직접 생성하지 않고, 클라이언트에서 먼저 필수 인자들을 빌더 생성자에
전달하여 빌더 객체(builer object)를 만든다.
2. 빌더 객체에 정의된 설정 메서드를 호출하여 선택적 인자들을 추가해 나간다.
3. 인자가 없는 `build` 메서드를 호출하여 변경 불가능 객체를 만든다.


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
  .calories(100).sodium(35).carbohydrate(27).build();
```

- 빌더 클래스는 빌더가 만드는 객체 클래스의 정적 멤버 클래스 즉, 내부 클래스로 정의한다.
- 빌더에 정의된 설정 메서드는 빌더 객체 자신을 반환하기 때문에 설정 메서드를 호출하는
코드는 위와 같이 쭉 이어서 쓸 수 있다.

생성자와 마찬가지로 빌더 패턴을 사용하면 인자에 불변식을 적용할 수 있다.
```java

```
- `build`메서드 안에서 해당 불변식이 위반되었는지 검사할 수 있다.
- 불변식이 적용될 값 전부를 인자로 받는 설정자 메서드를 정의한다.
