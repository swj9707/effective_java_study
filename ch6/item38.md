# Item 38
## 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

- 타입 안전 열거 패턴?
  - JDK 1.5 이전에서 공식적으로 enum을 지원하지 않았기 때문에 생겨난 열거 패턴
  - 거의 모든상황에서 enum 타입이 타입 안전 열거 패턴보다 우수하다
  - 단, 하나의 예외상황은 열거타입은 확장이 불가능하다는 것


```java
// 열거한 값들을 가져온 다음 값을 더 추가하여 다른 목적으로 사용 가능a
public class Country {
  private String name;

  private Shape(String name) {
    this.name = name
  }

  public static final Country ROK = new Shape("republic of korea");
  public static final Shape USA = new Shape("united states");
  public static final Shape JPA = new Shape("japan");
}
```

- enum 타입은 왜 확장이 불가능하게 설계 되었을까?
  - 사실 대부분의 상황에서 열거타입을 확장하는 건 좋지 않은 생각이다
  - 확장할 수 있는 enum 타입이 어울리는 쓰임이 존재한다
  - 예를들어 연산코드, 연산코드의 각 원소는 특정 기계가 수행하는 연산을 뜻한다. 연산코드는 이따금 API가 제공하는 기본 연산 이외에도 사용자 확장 연산을 추가할 수 있도록 열어줘야할 때가 있음
  - enum에도 interface를 이용하여 확장성을 갖게 할 수 있다!

```java
// 기본 아이디어는 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하는 것
// 연산 코드용 인터페이스를 정의하고 열거 타입이 인터페시르르 구현하게 하면 된다
public interface Operation {
  double apply(double x, double y);
}



// 열거 타입인 BasicOperation은 확장할 수 없지만
// 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산 타입으로 사용하면 됨
public enum BasicOperation implements Operation {
  PLUS("+") {
    @Override
    public double apply(double x, double y) { return x + y }
  },
  MINUS("-") { ... }
  ...
}

// 위의 코드에서 *를 추가하고 싶다면?
  TIMES("*) {
    @Override
    public double apply(double x, double y) { return x * y }
  }
// 우리가 할 일은 Operation 인터페이스를 구현한 열거 타입을 작성하는 것 뿐




// interface를 구현하는 여러 enum에 접근하여 사용 가능!
public static void main(String[] args) {
    double x = 3d;
    double y = 2d;

    test(ExtendedOperation.class, x, y);
}

private static <T extends Enum<T> & Operation> void test(Class<T> opEnumType, double x, double y) {
    for(Operation op : opEnumType.getEnumConstants()) {
        System.out.printf("%f %s %f = %f%n",
                            x, op, y, op.apply(x, y));
}
```
  


```
# 핵심!

 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을
함께 사용해 같은 효과를 낼 수 있다. 이렇게하면 클라이언트는 이 인터페이스를 구현해 자신만의
열거 타입을 만들 수 있다.
```
