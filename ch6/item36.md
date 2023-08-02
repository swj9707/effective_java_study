# Item 36

## 비트 필드 대신 EnumSet을 사용하라

### 비트 필드란?
> - Enum과 EnumSet이 나오기 전에 사용하던 방법.
> - 참/거짓 값을 여럿 결합하여 하나의 바이트로 만들어 메모리를 절약할 수 있음.
> - 바이트 내 비트가 2의 제곱수로 커지기 때문에, 항상 2의 배수를 값으로 가짐.
> - 그래서 다른 상수와 구분하기 쉽다?
<br>

#### 예시 1, 비트 필드 열거 상수
```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0; // 1
    public static final int STYLE_ITALIC = 1 << 1; // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
    public void applyStyles(int styles) {
        // ...
    }
}
```
- 아래와 같은 식으로 비트별 OR을 사용해 여러 상수를 하나의 집합으로 모은다.
- 이렇게 만들어진 집합을 비트 필드라 한다.
```java
public class Item36 {
    public static void main(String[] args) {
        text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
    }
}
```
<br>

- 비트 필드를 사용하면 비트별 연산을 사용해 집합 연산을 효율적으로 수행 가능하다.
- 그러나 로직을 실행해야하는데 숫자는 BOLD, ITALTC 이라는 정보를 가지고 있지 않아, 해석하기 어렵다.
- 또한, 열거 값들을 순회하고 싶다면 다른 로직을 만들어야한다. 
<br>

#### 예시 2, 비트 필드를 대체할 수 있는 Enum.

```java
public enum Text {
    BOLD, ITALIC, UNDERLINE, STRIKETHROUGH ...
}
```
- 비트 필드는 Enum의 등장으로 완전 대체할 수 있다.
<br>

#### 예시 3, 비트 필드를 대체할 수 있는 EnumSet.
```java
public class Text {

    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    public static void applyStyles(Set<Style> styles) {
        //...
    }

    public static void main(String[] args) {
        // ...
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC, Style.UNDERLINE));
    }
}
```
- Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 Set 구현체와도 함께 사용할 수 있다.

#### 정리
- 열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.
- EnumSet 클래스가 비트 필드 수준의 명료함과 성능, 그리고 열거 타입의 장점까지 제공해준다.
