# Item 03
## private 생성자나 열거타입으로 싱글턴임을 보장하라

## 싱글턴이란?
>인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

- 함수와 같은 무상태 객체
- 설계상 유일해야하는 시스템 컴포넌트
- DBCP(DataBase Connection Pool)

#### 1. public static final 필드 방식의 싱글턴
```java
public class Honggi {
  public static final Honggi INSTANCE = new Honggi();
  private Honggi() {...} //외부에서 인스턴스를 생성할 수 없도록 방지.

  public void run() {...}
}
```
> public, protected 생성자가 없으므로 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나 뿐임이 보장된다.

#### 장점
- public 필드 방식의 큰 장점은 해당 클래스가 싱클턴임이 명백히 드러난다는 것이다.
  - public static 필드가 final이니 절대로 다른 객체를 참조할 수 없다.
- 간결하다.


