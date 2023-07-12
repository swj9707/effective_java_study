# Item17

## 변경 가능성을 최소화 하라.

### 불변 클래스란?
> - 클래스의 인스턴스 값을 수정할 수 없는 클래스이다.
> - 가변 클래스보다 설계, 구현하기가 쉽다.
> - 오류가 생길 여지도 적고 훨씬 안전하다.
> - String, Wrapper Class 등이 있다.
<br>

### 불변 클래스를 만드는 다섯 가지 규칙
- 객체의 상태를 변경하는 메서드를 제공하지 않는다.
  - setter 메서드 등
- 클래스를 확장할 수 없도록 한다.
  - 하위 클래스에서 객체의 상태를 변하게 만들 수 있는 것을 방지해야함.
- 모든 필드를 final로 선언한다.
  - 설계자의 의도를 명확히 드러내는 방법.
- 모든 필드를 private으로 선언한다.
  - 클라이언트에서 직접 접근해 수정하는 일을 막을 수 있다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.
  - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야한다.
<br>

### 예제 1, 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 하라.
```java
public class person {
    private final String name;
    private final Address address;

    public Member(String name, Address address) {
        this.name = name;
        this.address = address;
    }
    //...getter
}
```
```java
public class Address {
    private String post;

    //... getter & setter
}
```
Person 클래스에서는 가변 객체인 Address의 참조를 적용하고 있다.<br>
Address의 객체가 변경되면 Member 객체도 변경된다.

```java
    void mutableReference(){
        Address address = new Address();
        address.setPost("post1");

        //Person에서 address를 직접 참조
        Person person = new Person("name", address);
        String before = person.getAddress().getPost();
        
        //address를 수정
        address.setPost("post2");
        //변경 후의 person의 address
        String after = person.getAddress().getPost();

        assertThat(before).isNotEqualTo(after);     //pass
    }
```
<br>

이를 불변으로 만들기 위해서는 아래와 같이 Address를 새롭게 정의해준다.
```java
public class Person {
    private final String name;
    private final Address address;

    public Person(String name, Address address) {
        this.name = name;
        this.address = new Address(address.getPost());
        // 매번 새로운 객체를 할당 받기 때문에 
    }
    //...getter
}
```

### 불변 객체의 장점
- 불변 객체는 단순하다.
  - 생성된 시점의 상태를 파괴될 때까지 그대로 가져간다.
- 불변 객체는 근본적으로 스레드가 안전하여 따로 동기화할 필요가 없다.
   - 객체의 변동이 없으니 여러 쓰레드에서 접근해도 훼손되지 않는다.
- 한 번 만든 인스턴스는 재활용 할 수 있다.
   - 정적 팩터리를 제공할 수 있다.
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
  - 값이 바뀌지 않는 요소들만 있다면, 구조가 아무리 복잡해도 불변식을 유지하기 쉽다.
- 실패 원자성을 제공한다.
  - 상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에서 빠질 가능성이 없다.
<br>

### 불변 객체의 단점
- 값이 다르면 반드시 독린된 객체로 만들어야한다.
- 즉, 새로운 인스턴스의 생성해야한다.

#### 캐싱, 가변 동반클래스를 활용하여 성능 개선이 가능하다.
<br>

### 불변 클래스를 만드는 다른 설계 방법
1. 클래스를 final로 선언한다.
2. 모든 생성자를 private or package-private으로 만들고 정적팩터리를 제공한다.
<br>

### 주의사항
1. getter 메서드가 있다고 해서 무조건 setter를 만들지는 말자.
    - 클래스는 꼭필요한 경우가 아니라면 불변이어야 한다.

2. 불변으로 만들 수없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
    - 꼭 변경해야 할 필드를 뺀 나머지는 final로 선언하자.
    - 다른 합당한 이유가 없다면 모든 필드는 private final 이어야한다.

3. 생성자는 불변식 설정이 모두 완료된 ,초기화가 완벽히 끝난 상태의 객체를 생성해야한다.
    - 확실한 이유가 없다면 생성자와 정적 팩터리 외에는 그 어떤 초기화 메서드도 public 으로 제공 하면 안된다.
