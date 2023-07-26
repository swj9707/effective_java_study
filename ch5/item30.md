# Item 31
## 이왕이면 제네릭 메서드로 만들어라

- 클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다!
  - 매개변수와 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다
  - 메서드에 제네릭을 사용할 때도 앞선 아이템 29에서 클래스에 제네릭을 사용했던 것 처럼 타입이 오는 자리에 제네릭을 넣어주면 된다
  - 다만, 메서드에 제네릭을 사용할 때는 메서드의 제한자와 반환 타입 사이에 타입 매개변수 목록을 넣어주어야 한다는 것을 잊지 말아야 한다.

```java
// 제네릭을 사용하지 않고 Raw 타입을 사용한 정적 메서드의 예시
public static Set union(Set s1, Set s2) {
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```

컴파일은 가능하지만 new HashSet(s1), result.addAll(s2) 두 곳에서 Raw 타입에 대한 경고가 발생한다!!

```java
// 안전하게 수정한 코드, 이 때의 매개변수 목록은 <E>이고 반환 타입은 Set<E> 이다
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet(s1);
  result.addAll(s2);
  return result;
}


// 이전의 메서드보다 더 안전하며 사용하기도 쉽다!
public static void main(String[] args) {
  Set<String> guys = Set.of("톰", "딕", "해리");
  Set<String> stooges = Set.of("래리", "모에", "컬리");

  Set<String> aflCio = Union(guys, stooges);
}
```

- 제네릭을 사용해서 싱글턴 메서드를 활용하라
  - 제네릭은 런타임 시점에 Object 타입으로 타입이 소거된다. 이로 인해 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다.
  - 하지만 이렇게 하려면 요청한 타입에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다.

```java
// 어떠한 타입으로 요청을 해도 불변의 빈 Set 객체를 반환해주는 팩터리 메서드
// 이 때, T에 어떤 타입 요청이 오더라도 타입 안전을 보장할 수 있다.
public class GenericFactoryMethod {
    private static final Set IMMUTABLE_EMPTY_SET = Set.copyOf(new HashSet());

    @SuppressWarnings("unchecked")
    public static <T> Set<T> immutableEmptySet() {
        return (Set<T>) IMMUTABLE_EMPTY_SET;
    }
}
// @SuppressWarnings("unchecked") 를 사용하면 오류나 경고를 숨길 수 있다


public class GenericFactoryMethod {
    private static final Set IMMUTABLE_EMPTY_SET = Set.copyOf(new HashSet());

    public static void main(String[] args) {
        Set<String> immutableEmptyStringSet = immutableEmptySet();
        // String을 element 타입으로 가지는 불변의 빈 Set 타입 객체가 반환됨
        Set<Integer> immutableEmptyIntSet = immutableEmptySet();
        // int를 element 타입으로 가지는 불변의 빈 Set 타입 객체가 반환됨
    }
}
```




```
# 핵심!

 메서드에서 제네릭을 사용하지 않고 로 타입이나 Object 타입을 사용한다면 입력 매개변수와 반환값을 명시적 형변환해야 하며,
비검사 형변환 과정에서 예외 상황이 발생할 수 있어 안전하지 않다. 따라서 불필요한 형변환을 없애기 위해 메서드의 매개변수와
반환값에 적절히 제네릭을 사용하여 제네릭 메서드로 만들면 편의성과 안정성을 모두 잡는 좋은 방법이 될 수 있다.
```
