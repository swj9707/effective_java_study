# Item35

## ordinal 매소드 대신 인스턴스 필드를 사용하라.

Enum 타입에서는 몇 번째 상수인지 리턴해주는 ordinal 메소드를 제공한다. 

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, DOUBLE_QUARTET,
    NONET, DECTET, TRIPLE_QUARTET;

    public int numberOfMusicians() {return ordinal() + 1}
}
```

- 상수들의 순서를 바꾸는 순간 문제가 생긴다. 
- 값을 중간에 비울 수 없다. 


```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```
- 간단하게 해결 가능
- 그럼 ordinal 의 목적은?
  - 일반적인 프로그래머가 사용하는 목적은 X
  - EnumSet, EnumMap 과 같은 열거 타입 기반의 범용 자료구조에서 사용 됌