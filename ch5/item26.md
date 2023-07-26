# Item 26

## 로 타입은 사용하지 말라

### 로 타입(raw type) 이란?
> - 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않는 것을 의미한다.
> - List<T>는 raw type의 List이다. 즉 <T>를 매개변수로 받는 것이라고 생각하면됨.
> - Raw 타입은 제네릭이 도래하기 전(java 9) 코드와 호환되도록 하기 위해 고안됨.
> - 컴파일 시점에서 타입을 체크하지 않고 런타임 시점에 타입을 체크함.
<br>

#### 예시 1, 제네릭 개념
```java
class List<T> {  }
```
>- List<T> : 제네릭 클래스 혹은 제네릭 인터페이스. 제네릭 클래스와 제네릭 인터페이스를 통틀어 제네릭 타입이라 한다.
>- T : 타입 변수 또는 타입 매개변수 (T는 타입 문자)
>- List : 원시 타입(raw type) 
<br>

- Raw Type을 사용하면 런타임에 예외가 일어날 수 있으므로 사용하면 안된다.
- 제네릭을 사용하면 컴파일 시점에서 타입을 체크함으로써 타입 안정성을 제공해준다.
- 타입체크와 형변환을 생략함으로써 코드가 간결해진다.


#### 예시 2, Raw type 사용
```java
// Stamp 인스턴스만 취급한다.
private final Collection stamps = ...; 
stamps.add(new Coin(...)) // Coin 인스턴스를 add() 한다면 "unchecked call" 경고를 내뱉는다.
```

#### 예시 3, Generic 사용
```java
private final Collection<Stamp> stamps = ...;
stamps.add(new Coin(...)) // 컴파일 오류가 발생한다.
```
<br>

- Raw Type을 사용한 경우 Type이 맞지 않는 인스턴스에 대한 에러를 런타임에 확인할 수 있는 치명적인 단점이 존재한다.
- Generic 을 사용한다면 컴파일 타임에 Type을 체크하여 런타임 오류를 예방할 수 있다.
- Raw Type을 사용하는 것은 Generic이 제공하는 안정성과 표현력을 모두 잃게 되는 셈이다.
 - 안정성 : 컴파일 시점에 타입을 체크함.
 - 표현력 : 특정 타입의 인스턴스를 사용한다는 정보가 주석이 아닌 타입 선언 자체에 명시함.
- 추가하자면, 에러는 가능한 발생 즉시 또는 컴파일 할 때 발견하는 것이 좋다.

### 로 타입의 대안
- 임의의 객체를 허용하는 매개변수화 타입을 사용하라.
- 제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경쓰고 싶지 않다면 <?>를 사용해라.
<br>

#### 예시 4, 임의 객체를 허용하는 매개변수화 타입 

```java
public class Item26 {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다, 런타임 에러 발생
    }

    private static void unsafeAdd(List list, Object o) {
        list.add(o); // 경고 발생
    }
}
```
- 위 예시는 컴파일은 되지만 경고가 발생한다.

```java
public class Item26 {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();
        unsafeAdd(strings, Integer.valueOf(42)); // 컴파일 에러 발생
        String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다
    }

    private static void unsafeAdd(List<Object> list, Object o) {
        list.add(o);
    }
}
```
- 위 예시는 컴파일 에러가 발생하여, 컴파일이 되지 않는다.
<br>

#### 예시 5, 비한정적 와일드 카드

```java
public int numElementsInCommon(Set s1, Set s2) {
    int result = 0;
    for (Object o1 : s1)
        if (s2.contains(o1))
            result++;
    return result;
}
```
- 위 예시는 동작은 하나, 로 타입 사용으로 안전하지 않다.
- 아무 원소나 넣을 수 있으므로 타입 불변식을 훼손하기 쉽다.

```java
public int numElementsInCommon(Set<?> s1, Set<?> s2) {
    int result = 0;
    for (Object o1 : s1)
        if (s2.contains(o1))
            result++;
    return result;
}
```
- 비한정적 와일드카드 타입을 사용해 안전하게 코드를 작성할 수 있다.
- Collection<?>에는 null 외엔 어떤 원소도 넣을 수 없으므로 컬렉션의 타입 불변식을 훼손하지 못한다
<br>

- 로 타입은 결코 사용해선 안된다.
- 제너릭은 특정 파라미터 타입만을 제한한다. 따라서, 모든 파라미터 타입을 허용하도록 하고 싶다면 비한정적 와일드카드를 사용하자.
