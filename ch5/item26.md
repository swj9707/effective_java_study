# Item 26

## 로 타입은 사용하지 말라

### 로 타입(raw type) 이란?
> - 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 것을 의미한다.
> - List<E>는 raw type의 List이다. 즉 <E>를 매개변수로 받는 것이라고 생각하면됨.
> - Raw 타입은 제네릭이 도래하기 전 코드와 호환되도록 하기 위해 고안됨.
<br>

#### 예시 1, Raw type
```java

// Stamp 인스턴스만 취급한다.
private final Collection stamps = ...; 
stamps.add(new Coin(...)) // Coin 인스턴스를 add() 한다면 "unchecked call" 경고를 내뱉는다.

```

#### 예시 2, Generic
```java
private final Collection<Stamp> stamps = ...;
stamps.add(new Coin(...)) // 컴파일 오류가 발생한다.
```
<br>

- Raw Type을 사용한 경우 Type이 맞지 않는 인스턴스에 대한 에러를 런타임에 확인할 수 있는 치명적인 단점이 존재한다.
- Generic 을 사용한다면 컴파일 타임에 Type을 체크하여 런타임 오류를 예방할 수 있다.
- Raw Type을 사용하는 것은 Generic이 제공하는 안정성과 표현력을 모두 잃게 되는 셈이다.

### 와일드 카드 타입을 사용하라
- Raw Type을 사용하고 싶은 경우 비한정적 와이드카드 타입(unbounded wildcard type)을 대신 사용하는 것이 좋다.
- 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않다면 <?>를 사용해라.
<br>

#### 예시 3, 와일드 카드
```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) {
  int result = 0;
  for (Object o1 : s1) {
    if (s1.contains(s2))
      result++;
  }
  return result;
}
```
<br>

- 로 타입은 결코 사용해선 안된다. 이것을 컴파일러가 허용하도록 둔 이유는 자바 1.5 버전 이전의호환성 때문이다.
- 제너릭은 특정 파라미터 타입만을 제한한다. 따라서, 모든 파라미터 타입을 허용하도록 하고 싶다면 비한정적 와일드카드를 사용하자.
