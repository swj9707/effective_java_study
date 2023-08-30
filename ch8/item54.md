# Item 54
## null 이 아닌 빈 컬렉션이나 배열을 반환하라

- null을 반환하는 경우 에러 처리를 해야 할 거리가 늘어나고 성능도 좋지 못하다.
- 예시

```java
private final List<Cheese> cheesesInStock = ... ;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 * 단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null : new ArrayList<>(cheesesInStock); 
}
```
- null을 굳이 처리해야하는 수고를 들여야 한다. 

```java
List<Cheese> cheeses = shop.getCheeses();
if (cheeses != null && cheeses.contains(Cheese.STILTON))
    System.out.println("좋았어, 바로 그거야.");
```
- null을 반환하는 메소드는 반드시 방어코드를 작성해야한다.
  - 예를 들면 JPA에서 findById 등
  - 방어코드를 누락하면 오류가 생길 수 있다.

### 빈 컬렉션 반환하기
```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```
- 빈 컬렉션을 매번 할당시키는 게 아닌 빈 불변 컬렉션을 반환하는 방법
```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList() : new ArrayList<>(cheesesInStock);
}
```
- 배열을 반환하는 예시
```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```
- 권장하지 않음 : 배열을 미리 선언하고 매번 그 배열을 반환하는 방법
  - 오히려 성능을 하락시킴
```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```