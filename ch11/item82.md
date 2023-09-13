# Item82

## 스레드 안전성 수준을 문서화 하라

## API 문서의 synchronized 한정자

- API 문서에 synchronized 한정자가 보이는 메소드는 스레드 안전하다고 100% 확신할 수 없다.
- 자바독이 기본 옵션에서 생성한 API 문서에는 synchronized 한정자가 포함되지 않는다.
- 메소드 선언에 synchronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않기 떄문이다.
- synchronized 유무로 스레드 안전성을 알수있다는 것은 모 아니면 도라는 위험한 발상이다.
- 따라서 멀티스레드 환경에서도 API를 안전하게 사용하려면 클래스가 지원하는 스레드 안정성 수준을 정확히 명시해야 한다.

## 스레드의 안전성 수준

- 불변(immutable)
  - 해당 클래스의 인스턴스는 마치 상수와도 같아서 외부 동기화도 필요 X
  - 예를 들면 String, Long, BigInteger
- 무조건적인 스레드 안전(unconditionally thread-safe)
  - 해당 클래스의 인스턴스는 수정될 수 있지만 내부에서도 충실히 동기화하여 별도의 외부 동기화없이 동시에 사용해도 안전
  - 예를 들면 AtomicLong, ConcurrentHashMap
- 조건부 스레드 안전(conditionally thread-safe)
  - 무조건적인 스레드 안전성과 같지만 일부 메서드는 동시에 사용하려면 외부 동기화가 필요.
  - Collections.synchronized 래퍼 메서드가 반환한 컬렉션
- 스레드 안전하지 않음(not thread-safe)
  - 해당 클래스의 인스턴스는 수정될 수 있으며 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 로직으로 감싸야 한다.
  - 예를 들면 ArrayList, HashMap
- 스레드 적대적(thread-hostile)
  - 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 X
  - 동시성을 고려하지 않고 만들다보면 우연히 만들어짐

## 동기화에 대한 문서화

- 조건부 스레드 안전한 클래스는 주의하여 문서화해야 한다.
- 어떠한 순서로 호출할 때 외부 동기화 로직이 필요한지 그리고 그 순서대로 호출하려면 어떤 락 혹은 락을 얻어야만 하는지 알려주어야 한다.
- ex) Collections.synchronizedMap의 API의 문서

```java
/**
 * It is imperative that the user manually synchronize on the returned
 * map when iterating over any of its collection views
 * 반환된 맵의 콜렉션 뷰를 순회할 때 반드시 그 맵으로 수동 동기화하라
 *
 *  Map m = Collections.synchronizedMap(new HashMap());
 *      ...
 *  Set s = m.keySet();  // Needn't be in synchronized block
 *      ...
 *  synchronized (m) {  // Synchronizing on m, not s!
 *      Iterator i = s.iterator(); // Must be in synchronized block
 *      while (i.hasNext())
 *          foo(i.next());
 *  }
 */
```

- 반환 타입만으로 명확히 알 수 없는 정적 팩토리 메서드라면 위의 예시 코멘트처럼 자신이 반환하는 객체에 대한 스레드 안전성을 문서화해야 한다.

## 외부에 공개된 Lock

- 외부에 공개된 락(Lock)을 사용하면 유연한 코드를 만들 수 있지만, 라이언트가 공개된 락을 가지고 놓지 않는 서비스 거부 공격(denial-of-service attack)을 수행할 수 있습니다. - 그렇기 때문에 아래와 같은 비공개 락 객체를 사용해야 합니다.

```java
// 비공개 락 객체, final 선언!
private final Object lock = new Object();

public void someMethod() {
    synchronized(lock) {
        // do something
    }
}
```

- 여기서 lock 멤버를 final로 선언한 이유는 우연히라도 락 객체가 교체되는 상황을 방지하기 위함.
  - 일반적인 락이든 java.util.concurrent.locks 패키지에서 가져온 락이든 동일.
  - 이러한 방법은 클라이언트 또는 이를 상속하는 하위 클래스에서 동기화 로직을 깨뜨리는 것을 예방할 수 있다.
