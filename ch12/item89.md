# Item 89

## 인스턴스 수를 통제해야한다면 readResolve보다는 열거 타입을 사용하라

### readResolve 란?
> - 인스턴스 통제 목적으로 사용함 .<br>
> - 단 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야한다.<br>
> - 그렇지 않으면 readResolve 메서드가 수행되기 전에 역직렬화된 객체의 참조를 공격할 여지가 남는다.
<br>

### transient 란?
- 
- 
<br>

### 싱글톤을 직렬화하는 경우 문제점
- 클래스에 implements Serializable 을 추가하는 순간 더 이상 싱글턴이 아니게 된다.
- 기본 직렬화를 쓰지 않고, 명시적인 readObject를 제공하더라도 소용없다.
- 어떤 readObject를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다.

###

#### 예시 1, 싱글톤 객체
```java
public final class Elvis implements Serializable {
  private static final Elvis INSTANCE = new Elvis();

  private static Elvis() {
  }

  public static Elvis getINSTANCE() {
    return INSTANCE;
  }

  private String[] favoriteSongs = {"Sandman", "aquaman"}

  public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs);
  }

  private Object readResolve() {
    return INSTANCE;
  }
}
```
- 위 예시에서는 favoriteSongs를 transient로 선언할 것이 아니라면 enum타입으로 만드는 것이 좋다.
- 싱글턴이 non-transient 참조필드를 가지고 있으면 이 내용은 readResolve가 실행되기 전에 역직렬화 된다.
- 그렇게 되면 조작된 스트림을 사용하여 해당 참조 필드의 내용이 역직렬화되는 시점에 참조를 훔쳐올 수 있다.
<br>

``` java
public enum Elvis {
    INSTANCE;
    private String[] favoriteSongs = {"sandman", "aquaman"};

    public void printFavorites() {
    System.out.println(Arrays.toString(favoriteSongs);
  }
}
```
- 이렇게 enum 타입으로 하면 된다.
<br>

##결론
- 직렬화가 필요하며 인스턴스 수가 하나임을 보장하고 싶을 때, enum 타입을 가장 먼저 고려해라.
- 만약 enum 타입을 사용하는 것이 불가능하며, 인스턴스 수를 제한해야 한다면, readResolve를 구현해라.
- 또한, 해당 클래스의 모든 필드를 원시타입 또는 임시타입으로 정의되어야 한다.


