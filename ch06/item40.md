# item40. @Override 애너테이션을 일관되게 사용하라

## @Override

- 자바에서 기본적으로 제공하는 애너테이션
- 상위 타입의 메서드를 재정의했음을 의미

#### @Override 애너테이션을 일관되게 사용하면 악명 높은 버그를 예방할 수 있음


```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram bigram) {
        return bigram.first == this.first &&
                bigram.second == this.second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }

        System.out.println(s.size()); //260
    }
}
```
- set은 중복을 허용하지 않으니 **26**이 출력 될 것으로 예상되지만, 실제로는 **260**이 출력됨.

## @Override를 일관되게 사용해야 하는 이유
- `equals` 는 `Object` 를 파라미터로 갖는데, `Bigram` 으로 받은 문제
  - equals 메소드를 **재정의(overriding)** 한 게 아니라 **다중정의(overloading)** 함
  - 즉 Object의 equals() 를 재정의하려면 매개변수 타입을 Object로 해야 하는데 그렇지 않아서 **상속이 아닌 별개의 equals 메서드를 정의**한 꼴이 되었다.
- 이 문제를 통해 `Set` 에서 `==` 연산 시 `equals(Object)` 를 호출
    - `Bigram` 에서 같은 값을 가지는 값이더라도 다른 객체(참조)이므로 다른 값으로 인식해 의도한 목적대로 `Set` 이 수행되지 않는 문제가 발생

### 문제를 해결하기 위해선 `@Override` 를 사용하자


```java
@Override
public boolean equals(Bigram bigram) {
    return bigram.first == this.first &&
        bigram.second == this.second;
}
```
- `@Override` 을 사용하고 컴파일하면 컴파일 오류가 뜸

#### equals 메서드 수정
```java
@Override public boolean equals(Object o) {
        if (!(o instanceof Bigram2))
            return false;
        Bigram2 b = (Bigram2) o;
        return b.first == first && b.second == second;
}
```

## 예외 사항
- **구체 클래스에서 상위 클래스의 메서드를 재정의**할 때는, 굳이 @Override를 달지 않아도 된다.
- 구체 클래스인데 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 자동으로 사실을 알려주기 때문이다.
- 물론 @Override를 붙여도 상관없다.

### 구체클래스?
- new 연산자를 사용하여 인스턴스를 생성할 수 있는 클래스를 Concrete(구상) 클래스라고 한다.
- 미완성클래스인 추상클래스에 정의된 기능을 구현하는 클래스이다.



## 핵심 정리

- 앞선 문제를 방지하기 위해, **상위 클래스의 메서드를 재정의하는 모든 메서드에 `@Overriding` 애너테이션을 달자 !**
  - 오타, 실수 등을 IDE 및 컴파일 단에서 잡아내어 문제를 찾기 위한 시간을 단축시킬 수 있음
- 구체 클래스에서 상위 추상 메서드를 재정의한 경우엔 **달지 않아도 되나, 단다고 해서 해로울 것도 없다.**
