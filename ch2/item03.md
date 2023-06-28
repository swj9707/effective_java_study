# Item 03
## private 생성자나 열거타입으로 싱글턴임을 보장하라

## 싱글턴이란?
>인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

- 함수와 같은 무상태 객체
- 설계상 유일해야하는 시스템 컴포넌트
- DBCP(DataVase Connection Pool)
- 로그 기록 객체

# public static final 필드 방식의 싱글턴
```java
public class Honggi {
  public static final Honggi INSTANCE = new Honggi();
  private Honggi() {...}

  public void run() {...}
}
```
> public, protected 생성자가 없으므로 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나 뿐임이 보장된다.




