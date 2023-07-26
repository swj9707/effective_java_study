# item28. 배열보다는 리스트를 사용하라

## 배열의 제네릭 타입의 차이
### 1. 공변과 불공변

- 공변이란?
  - 경험적 관계에서 첫 번째 변수의 크기가 두 번째 변수의 크기에 따라 변하는 것을 말한다.   


- 배열
    - 공변(covariant)
    - Sub가 Super의 하위 타입이라면 배열 Sub[]은 배열 Super[]의 하위 타입이 된다.(상속 관계)
    - 함께 변한다.

```java
Object[] objectArray = new Long[1];
objectArray[0] = "어떤 타입이든 저장할 수 있는데.........."; // ArrayStoreException 던짐
```
배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.   
그래서 컴파일에는 아무런 영향을 끼치지 않지만 런타임 시 ArrayStoreException을 던진다.


- 제네릭
    - 불공변(invariant)
    - 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고, 상위 타입도 아니다.

```java
// 컴파일 시 예외
List<Object> list = new ArrayList<Long>();
list.add("컴파일에서 체크!"); // Compile Error!
```
컴파일러에게 타입을 명시해주어 컴파일 시 에러가 발생한다.


### 2. 실체화(reify)
- 다시 말하면 배열은 구체적이다.
- 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.
- Long 배열에 String을 넣으려 하면 ArrayStoreException이 발생한다.
- 반면, 제네릭은 타입 정보가 런타임에는 소거(erasure)된다.


제네릭 배열을 만들지 못하게 막은 이유?
- 타입의 안정성을 보장하기 어려움 : *체크하는 시점이 다르기 때문에...*
- 만약 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException을 던짐
- 타입의 안정성을 보장하는 제네릭의 목표에 맞지 않는다.


#### 제네릭 배열 생성을 허용하지 않는 이유 - 컴파일되지 않는다.
```java
        List<String>[] stringLists = new List<String>[1]; // (1)
        List<Integer> intList = List.of(42); // (2)
        Object[] objects = stringLists; // (3)
        objects[0] = intList; // (4)
        String s = stringLists.get(0); // (5) 컴파일 에러가 난다!
```
- 제네릭 배열을 생성하는 (1)이 허용된다고 가정함.
- (2)는 원소가 하나인 List<Integer>을 생성함.
- (3)은 (1)에서 생성한 List<String>의 배열을 Object 배열에 할당함.
- (4)는 (2)에서 생성한 List<Integer의 인스턴스를 Object배열의 첫 원소로 저장함. 제네릭은 소거 방식으로 구현되어서 이 역시 성공.
- 즉, 런타임에는  List<Integer>인스턴스의 타입은 단순히 List가 되고, List<Integer>[] 인스턴스 타입은 List[]가 됨.
- (5)번에서 문제가 일어나는데, List<String> 인스턴스만 담겠다고 선언한 stringLists 배열에는 지금 List<Integer>인스턴스가 저장되어 있음
- 컴파일러는 꺼낸 원소를 자동으로 String으로 형변환하는데 이 원소는 Integer이므로 런타임에 ClassCastException이 발생하게됨.

> 예외를 방지하기 위해서는 제네릭 배열이 생성되지 않도록 컴파일 오류를 내야 한다.


## 배열 대신 리스트를 사용하자
- 배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 배열타입 대신 제네릭 컬렉션을 사용하면 해결된다.
- 조금 복잡해지고 성능이 나빠질 수 있지만 타입 안정성과 상호운용성은 좋아진다.


[예제코드]
```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        this.choiceArray = choices.toArray();
    }

    public Object choose() {
        Random random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)]
    }
}
```
- 이 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야한다.
- 혹시나 다른 원소가 들어 있었다면 런타임에 형변환 오류가 날 것이다.

[제네릭 수정]
```java
public class Chooser<T> {
  private final T[] choiceArray;

  public Chooser(Collection<T> choices) {
    this.choiceArray = choices.toArray(); // 컴파일 에러
  }

/// choose 메서드는 그대로
}
```
제네릭을 도입하여 수정하였다. 하지만 컴파일에서 에러를 내주었다.   
choies.toArray() 의 반환타입이 Object[]이므로 T[] 타입으로 캐스팅하면 된다.

```java
public Chooser(Collection<T> choices) {
        this.choiceArray = (T[]) choices.toArray(); // 컴파일 경고 warinng!!
}
```
- 컴파일러가 T에 대한 타입을 알고 있지 않아 경고가 발생한 것이다.  
- 이 형변환은 런타임에도 안전하지 못하다는 경고 메시지다.
- 단순히 경고 메시지다. 만약 개발자가 타입이 안전하다고 확신한다면 주석을 남기고 애너테이션을 달아 경고를 숨겨도 된다.
- 하지만 경고를 제거하는 것이 더 낫기에 배열보다 리스트를 사용하여 경고를 없애자.(item27)

```java
public class Chooser<T> {
    private final List<T> choiceArray;

    public Chooser(Collection<T> choices) {
        this.choiceArray = new ArrayList<>(choices);
    }

    public Object choose() {
        Random random = ThreadLocalRandom.current();
        return choiceArray.get(random.nextInt(choiceArray.size()));
    }
}
```


## 핵심 정리

- 배열은 공변이고 구체적이다.
- 제네릭은 불공변이고 비 구체적이다.
- 결과적으로
  - 배열은 런타임에는 타입 안전하지만 컴파일 타임에는 그렇지 않다.
  - 제네릭은 런타임에는 타입 안전하지 않고 컴파일 타임에는 안전하다.
- 둘을 섞어서 쓰기란 쉽지 않기 때문에 컴파일 오류나 경고를 만나면 가장 먼저 **배열을 리스트로 대체**하는 방법을 적용하도록 한다.
