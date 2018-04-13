# 2018 02 26 검색 노트2

#### 검색 키워드 : `field injection is not recommended intellij`

#### 검색 이유
게시판 예제를 intellij를 통해 작업을 하면서 매번 `@Inject`나 `@Autowired`애너테이션에 위와 같은 경고문구(?)같은게 나왔었다. 그래서 이 문제는 무엇일까 궁금해졌고 이에 대해 검색을 하게 되었고, 외국의 개발자 블로그에서 잘 정리된 내용을 번역하게 되었다.

#### 전체 내용 출처
http://vojtechruzicka.com/field-dependency-injection-considered-harmful/

# Field Dependency Injection Considered Harmful(번역)

스프링과 같은 프레임워크에서 Field Injection은 매우 널리 사용되는 방법이지만, 몇가지 심각한 트레이드오프가 있기 때문에 일반적으로 피해야 한다.

**트레이드오프(tradeoff)란?**
여기서 말하는 트레이드오프(tradeoff)란 객체의 어느 한부분의 품질을 높이거나, 낮추는게 다른 부분의 품질을 높이거나 낮추는데 영향을 끼치는 상황을 말한다. 일반적으로는 한 쪽의 품질을 높이면, 다른 쪽의 품질은 떨어지는 방향으로 흐르게 된다. 즉 간단하게 말해 한 쪽을 추구하면 다른 한 쪽을 포기 해야하는 상황을 말한다.
출처(https://www.joinc.co.kr/w/man/12/%ED%8A%B8%EB%A0%88%EC%9D%B4%EB%93%9C%EC%98%A4%ED%94%84)

## Injection Types
클래스에 의존성을 주입하는 방법에는 `Contstructor`, `Setter method`, `Field`가 있다. 동일한 의존성 주입을 하는 코드를 통해 비교해보자

#### `Contstructor`
```java
private DependencyA dependencyA;
private DependencyB dependencyB;
private DependencyC dependencyC;

@Autowired
public DI(DependencyA dependencyA, DependencyB dependencyB, DependencyC dependencyC) {
    this.dependencyA = dependencyA;
    this.dependencyB = dependencyB;
    this.dependencyC = dependencyC;
}
```
#### `Setter`
```java
private DependencyA dependencyA;
private DependencyB dependencyB;
private DependencyC dependencyC;

@Autowired
public void setDependencyA(DependencyA dependencyA) {
    this.dependencyA = dependencyA;
}

@Autowired
public void setDependencyB(DependencyB dependencyB) {
    this.dependencyB = dependencyB;
}

@Autowired
public void setDependencyC(DependencyC dependencyC) {
    this.dependencyC = dependencyC;
}
```

#### `Field`
```java
@Autowired
private DependencyA dependencyA;

@Autowired
private DependencyB dependencyB;

@Autowired
private DependencyC dependencyC;
```

#### 무엇이 문제일까?(What is wrong?)
위에서 보다시피 Field는 매우 짧고 간결

#### 단일 책임 원칙 위배(Single Responsibilty Principal Violation)

#### 의존성 은닉 (DI Hiding)

#### 의존성 컨테이너의 결합도(DI Container Coupling)

#### 불변성(Immutability)

## `Contstructor`와 `Setter`의 차이
