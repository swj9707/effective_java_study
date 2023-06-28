# Item 03
## private 생성자나 열거타입으로 싱글턴임을 보장하라

## 싱글턴이란?
>인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

- 함수와 같은 무상태 객체
- 설계상 유일해야하는 시스템 컴포넌트
- DBCP(DataBase Connection Pool)
<br>

### 1. public static final 필드 방식의 싱글턴
```java
public class Honggi {
  public static final Honggi INSTANCE = new Honggi();
  private Honggi() {...} //외부에서 인스턴스를 생성할 수 없도록 방지.

  public void run() {...}
}
```
> public, protected 생성자가 없으므로 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나 뿐임이 보장된다.
>- 클래스가 로드될 때 초기화되고 이후에는 변경되지 않는 상수 형태의 필드를 사용한다.

#### 장점
- public static 이므로 다른 클래스에서 쉽게 접근할 수 있다.
- public static final이니 절대로 다른 객체를 참조할 수 없다.
- 스레드의 안전성을 보장한다.
- 간결하다.
  
#### 단점
- 클래스가 로딩되는 시점에 인스턴스가 생성되므로, 항상 인스턴스를 생성하는 것이 아니라면 메모리를 낭비할 수 있다.
<br>

### 2. 정적 팩토리 방식의 싱글턴
```java
public class Honggi {
  private static final Honggi INSTANCE = new Honggi();
  private Honggi() {...} //외부에서 인스턴스를 생성할 수 없도록 방지.
  public static Honggi getInstance() { return INSTANCE; }

  public void run() {...}
}
```
> `Honggi.getInstance`는 항상 같은 객체의 참조를 반환하므로 제 2의 인스턴스를 만들지 않는다. 

#### 장점
- 인스턴스를 처음 사용(getInstance)하는 시점에 생성 가능하다.
 - 초기화 작업이 필요한 경우에 유연하게 처리 할 수 있다.
 - 필요한 시점에 인스턴스를 생성하므로 메모리를 더욱 효율적으로 관리할 수 있다.

#### 단점
- 멀티스레드 환경에서 동기화 처리를 해주지 않으면 여러 스레드가 동시에 인스턴스를 생성할 수 있는 위험이 있다.
