# item24. 멤버 클래스는 되도록 static 클래스로 만들라.

## 중첩 클래스 (Nested Class) : 자신을 둘러싼 바깥 클래스에서만 쓰이는 클래스
- 정적 멤버 클래스 (static member class)
- 비정적 멤버 클래스
- 익명 클래스
- 지역 클래스

이중 정적 멤버 클래스를 제외한 클래스를 내부 클래스(inner class)라고 부름

## 정적 멤버 클래스
- 클래스 내부의 static 클래스
> 바깥 클래스의 private 멤버에도 바로 접근 가능. 이외에는 일반 클래스와 같음
정적 멤버 클래스는 흔히 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다.

```java
  class A{
    static class B{}
  }
```
```java
  A.B b = new B();
```

## 비정적 멤버 클래스
- 비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결
- 따라서, 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this 를 사용하여 바깥 인스턴스의 참조를 가져올 수 있음.   
`정규화된 this == 클래스명.this`
- 비정적 멤버 클래스는 인스턴스를 감싸서 마치 다른 클래스처럼 보이게 하는 뷰인 어댑터에서 주로 쓰임

```java
public class MySet<E> extends AbstractSet<E> {
    ... // 생략

    @Override public Iterator<E> iterator() {
    return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
    ...
    }
}
```
- 이렇게 사용하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 됨
- 이 참조를 저장하려면 시간과 공간이 소비  
- GC가 바깥 클래스의 인스턴스를 수거하지 못는 메모리 누수가 생길 수 있음

> 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 **static**을 붙여서 정적 멤버 클래스로 만들자.

```java
public class MySet<E> extends AbstractSet<E> {
    ... // 생략
    @Override public Iterator<E> iterator() {
        return new MyIterator();
    }
    private static class MyIterator implements Iterator<E> {
    ...
    }
}
```

## 익명 클래스
- 익명 클래스는 이름이 없음
- 익명 클래스는 바깥 클래스의 멤버도 아님
- 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어짐
- 주로 즉석에서 작은 함수 객체나 처리 객체를 만드는데 사용했지만, 람다로 대체

```java
/* Person 클래스 */
public class Person {
    void getName() {}
}
```
```java
  public static void main(String[] args) {
    Person person = new Person() {
        String name = "김민지";

        void getName() { System.out.println("이름 : " + name); }
    };

    person.getName();
  }
```
상속 받은 객체를 여러 번 사용한다면 상속을 사용하는 것이 맞음. 하지만 한 번만 사용하고 버려지는 경우, 클래스 파일을 만드는 것은 메모리 낭비가 될 수 있음.   
이런 경우 **익명 클래스**를 사용


그러나 단점이 많기 때문에 주의 해서 사용 해야함
- 익명 클래스의 제약 사항
    - 선언한 지점에서만 인스턴스를 만들 수 있고, instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없음
    - 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없음
    - 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없음
    - 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없음
    - 익명 클래스는 표현식이 중간에 등장하므로 짧지 않으면 가독성이 떨어짐

## 지역 클래스
- 지역 클래스는 블록 안에 정의된 클래스로 메서드 안에서만 클래스를 사용하고 싶을 때 사용 
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며, 정적 멤버는 가질 수 없으며 가독성을 위해 짧게 작성해야 함
```java
public void greeting() {
    class KoreaGreeting { ... }
    class JapanGreeting { ... }
    }
```
