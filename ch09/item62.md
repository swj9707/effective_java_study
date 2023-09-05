# item 62. 다른 타입이 적절하다면 문자열 사용은 피하라

## 문자열은 다른 타입을 대신하기에 적절하지 않다.
- 데이터를 입력받을 때 무작정 문자열로 입력받지 말자.
    - 숫자면 숫자, 예/아니오라면 boolean 과 같은 명확한 타입으로 받는게 더 좋다.
- 열거 타입을 문자열로 대신하는 경우도 있다. (아이템 34)
- 혼합 타입을 문자열로 대신하는 경우도 있다.
- 권한을 문자열로 표기하는 경우도 있다.

## 혼합 타입을 남용하면 안된다
```java
String compoundKey = className + "#" + i.next();
```
- 가운데 `#` 을 기준으로 파싱하려고 하는 의도가 살짝 보인다.
- `#` 이 만약 `className` 에서 이용되거나 `i.next()` 에서 이용되면 문제가 일어난다.
- 문자열 파싱이라는 추가적인 성능 악영향까지 있다.
- `equals()`, `toString()`, `compareTo()` 등의 메서드도 제공 불가능하다.
- 3가지 필드를 보관할 수 있는 클래스를 만들지 않을 이유가 없다.
  - 보통 private 정적 멤버 클래스로 선언한다 (아이템 24)

## 문자열은 권한을 표현하기에 적합하지 않다.
### 권한을 문자열로 관리하는 ThreadLocal 예시
#### ThreadLocal 이란?
-  간단하게 말하면, 해당 쓰레드만 접근할 수 있는 특별한 저장소
- 여러 쓰레드가 같은 인스턴스의 필드에 접근하면 처음 쓰레드가 보관한 데이터가 사라질 수 있다.
- 반면, ThreadLocal을 사용하면 각 쓰레드마다 별도의 내부 저장소가 제공되어 같은 인스턴스의 쓰레드 로컬 필드에 접근해도 문제가 발생하지 않는다.

```java
public class ThreadLocal {
  private ThreadLocal() {} // 객체 생성 불가

  // 현 스레드의 값을 키로 구분해 저장한다.
  public static void set(String key, Object value);

  // 현 스레드의 값을 반환한다.
  public static Object get(String key);
}
```
- 코드 내에서 구체적인 스레드 로컬 스토리지 매커니즘을 구현하고 있지 않음.
- 즉, 스레드 구분용 키가 전역 이름 공간 (global namespace) 에서 공유된다는 뜻이다.
  - 보안에 취약할 수 있다.
- 서로 다른 문자열 키를 사용함을 보장해야 하고 같은 키를 이용할 경우 충돌이 발생한다.
- 다른 클라이언트가 문자열만 알면 다른 스레드가 사용하는 ThreadLocal 변수를 가져올 수 있다.
- 스레드 로컬 스토리지를 올바르게 사용하려면 스레드 간에 공유되지 않는 유일한 키를 사용해야 한다.

> 이를 해결하기 위해서 API는 문자열 대신 위조할 수 없는 키를 사용하면 된다.

### Key 클래스로 권한 구분하기
```java
public class ThreadLocal {
    private ThreadLocal() { } // 객체 생성 불가
    
    public static class Key { // (권한)
        Key() { }
    }
    
    // 위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
        return new Key();
    }
    
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```
위 방법은 문자열 기반 API의 문제 두 가지를 모두 해결해주지만, 개선의 여지가 존재한다.
- set, get메서드는 정적 메서드일 이유가 없으니 Key 클래스의 인스턴스 메서드로 옮긴다. 
  - Key는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다.
- 결과적으로 지금 톱레벨 클래스인 ThreadLocal은 별달리 하는 일이 없어지므로 치워버리고, 중첩 클래스 Key의 이름을 ThreadLocal로 바꾼다.

```java
// Key -> ThreadLocal
public final class ThreadLocal {
    public ThreadLocal() { }
	public void set(Object value);
    public Object get();
}
```
- 이 API에서는 get으로 얻은 Object를 실제 타입으로 형변환해 써야 해서 안전하지 않다
- ThreadLocal을 매개변수화 타입으로 선언하여 문제를 해결한다.
  - java.lang.ThreadLocal처럼 구성하여 문자열 기반 API의 문제를 해결

```java
public final class ThreadLocal<T> {
    public ThreadLocal();
    public void set(T value);
    public T get();
}
```


## 핵심 정리
- 데이터의 내용이 문자열인 경우에만 String으로 관리한다.
- 그 외에 이미 라이브러리내에 존재하는 타입인 경우, 해당 타입에 맞춰서 관리한다.
- 특정 값에 대한 정의가 가능한 경우, 사용자 정의에 따른 클래스를 정의하여 관리한다.
- 문자열을 잘못 관리하는 흔한 예로 기본 타입, 열거 타입, 혼합타입 등이 있다.
