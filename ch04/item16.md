# Item 16

## public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

### 인스턴스 필드란?
> - 객체지향 프로그래밍에서 클래스에 속한 변수를 말함. <br>
> - 클래스는 객체의 특성을 정의하는 템플릿.<br>
> - 클래스의 인스턴스는 이러한 특성을 가진 실제 객체. <br>
> - 인스턴스 필드는 클래스 내부에 선언되는 변수로, 해당 클래스의 인스턴스 마다 독립적인 값을 가질 수 있다.<br>
> - 즉, 객체마다 다른 상태를 유지할 수 있다.
<br>

### 예시 1
```java
class Zoo {
  public String tiger;
  public String lion;
}
```
- animal 클래스는 인스턴스 필드를 모아둔 것 이외에는 목적이 없다.
- 이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식을 보장할 수 없다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다.
- 객체 지향적이지 않다.
<br>

### 예시 2, 접근자와 변경자 메서드를 활용한 데이터 캡슐화
- 예시 1의 클래스를 객체지향적으로 사용하기 위해 필드의 접근제어자를 private으로 선언한다.<br>
- 또한, 외부에서 이 인스턴스 변수에 접근할 수 있도록 public 접근자(getter)를 추가한다.<br>
```java
class Zoo {
  private String tiger;
  private String lion;

  public Zoo(String tiger, String Lion) {
    this.tiger = tiger;
    this.lion = lion;
}

public String getTiger() { return tiger;}
public String getLion() { return lion;}

public void setTiger(String tiger) {this.tiger = tiger;}
public void setTiger(String lion) {this.lion = lion;}
} 
```
- 이 과정을 통해 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.
  - 그러나 public 클래스가 필드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없다.
<br>

### 예시 3, 데이터 필드를 노출해도 되는 package-private 클래스 혹은 private 중첩 클래스.
- 클래스가 표현하려는 추상 개념만 제대로 표현해주면 된다.
  - 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 깔끔하다.
  - 클라이언트 코드가 해당 클래스 내부 표현에 종속되나, 클라이언트도 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다.
  - 따라서 패키지 바깥 코드는 전혀 손대지 않고 데이터 표현 방식을 바꿀 수 있다.
  - private 중첩 클래스의 경우 수정 범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한된다.

```java
public class OuterClass {

    private InnerClass innerClass;

    public OuterClass() {
        this.innerClass = new InnerClass();
    }

    // 내부 클래스의 수정 기능을 제공하기 위한 퍼블릭 인터페이스
    public void changeValue(int value) {
        innerClass.value = value;
    }

    // 접근자(getter)
    public int value() {
        return innerClass.value;
    }

    // 내부 클래스, 필드 값이 public
    class InnerClass {
        public int value;
    }
}
```

```java
class OuterClassTest {

    void testCase1() {
        // given
        OuterClass outerClass = new OuterClass();

        // when
        outerClass.changeValue(1);

        // then
        assertThat(outerClass.value()).isEqualTo(1);
    }
}
```
<br>

### 예시 4, 불변 필드를 노출한 public 클래스
- 다시 돌아와서, 예제 1번의 문제점에서 조금 보완할 수 있는 방법은 불변 필드를 선언해주는 것이다.
- 하지만 여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없다.
- 필드를 읽을 때 부수 작업을 수행할 수 없다.
- 그러나 데이터의 불변은 보장 가능하다.
```java
//각 인스턴스가 유효한 시간을 표현함을 보장 (final)
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOURS = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOURS)
            throw new IllegalArgumentException(": " + minute);
        this.hour = hour;
        this.minute = minute;
    }
    //..생략
```

#### 요약
- public 클래스는 가변 필드를 노출하면 안된다.
- 불편 필드라면 노출해도 되지만 여전히 불완전하다.
- package-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을 때도 있다.
