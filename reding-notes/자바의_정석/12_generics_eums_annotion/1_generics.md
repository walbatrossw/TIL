# 1 `Generics`

## 1.1 `Generics`?
**`Generics`는 다양한 타입의 객체를 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입체크(complie-time type check)를 해주는 기능** 이다. 객체의 타입을 컴파일 시에 체크하기 때문에 객체의 타입 안정성을 높이고 형변환의 번거로움이 줄어든다.

- 타입의 안정성을 높이는 것?
  - 의도하지 않은 타입의 객체가 저장되는 것을 막고, 저장된 객체를 꺼내올 때 원래의 타입과 다른 타입으로 잘못 형변환되어 발생할 수 있는 오류를 줄여준다는 뜻이다.
- 장점
  - 타입 안전성을 제공한다.
  - 타입체크와 형변환을 생략할 수 있으므로 코드가 간결해진다.

## 1.2 `Generics`의 선언
```java
class Box {

  Object item;

  void setItem(Object item) {
    this.item = item;
  }

  Object getItem() {
    return item;
  }

}
```
```java
// 제네릭 타입 T를 선언
class Box<T> {
  T item;

  void setItem(T item) {
    this.item = item;
  }

  T getItem() {
    return item;
  }
}
```
