# Item 18
## 상속보다는 컴포지션을 사용하라

[참고자료](https://tecoble.techcourse.co.kr/post/2020-05-18-inheritance-vs-composition/)

- 상속은 상위 클래스의 구현에 따라 하위 클래스의 동작에 이상이 생길 수 있다. 
- 상위 클래스에 코드가 변경되어 릴리즈 된다면, 하위 클래스는 코드 한줄 수정하지 않았음에도 오동작 할 수 있다. 
- 상위 클래스에 대한 설계나 문서가 제대로 되어있지 않다면, 하위 클래스는 상위 클래스의 변화에 발맞춰 수정 되어야 한다. 
  - 즉...결합도가 높다는 이야기입니다. 

### 잘못 된 상속 예시
```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```
- 상위의 addAll을 사용했지만, addCount에 값을 한번 더하고 addAll을 호출한다.
- addAll은 실제로 add 메소드를 활용해서 개발 되어있는데, 이미 재정의 한 코드에서 addCount++ 코드를 실행시키고 있다. 
    ```java
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e)) // 여기서 호출되는 add는 우리가 재정의한 메서드이다.
                modified = true;
        return modified;
    }
    ```
- 즉 코드가 중복으로 실행되어서 아래와 같은 코드를 실행시키면 addCount에는 6이 저장된다. 

```java
InstrumentedHashSet<String> s = new InstrumentHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));
```

- addAll 메소드를 구현하지 않는 것도 방법이지만, HashSet이라는 클래스에서나 통하는 방법이지 어떤 클래스를 상속했을 지는 아무도 모른다. 
- 책에 여러 설명들이 나오지만 결론만 말하자면, 결국 상위 클래스에 의존하게 되는 구조라는 한계는 **_극복 할 수 없다._** 


### 래퍼 클래스 - 상속 대신 컴포지션 (데코레이터 패턴)
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
}
```
```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```
- 두 예시의 공통점은 굳이 상속을 통해 새로운 객체를 만드는게 아니라 상위 클래스에서 생성 된 인스턴스를 감싸고 있는 구조. 그래서 Wrapper Class 라고도 불린다. 
- 뭔 차이인가 싶겠지만 분리 된 별개의 객체가 아니라 교집합으로써 상위 클래스의 인스턴스를 보유하고 있다는게 다르다. 
- 해당 클래스로 생성 된 인스턴스 내에 있는 교집합 역할을 하는 객체가 가진 기능들을 통해 새로운 기능들을 덧씌우듯 구현하는 방법. 
- 단점의 거의 없으나, 콜백 프레임워크와는 어울리지 않는다. 

### 정리
- 상속은 하위 클래스가 상위 클래스의 진정한 하위 타입일 때만 사용해야 한다. 기능을 확장시켜서 또 다른 클래스를 생성하려는 목적이라면 데코레이터 패턴을 고려하라. 
  - 예를들면 Java API 내에 Stack은 vector가 아니지만 vector를 확장하여 구현되어있다. 그래서 공식적으로 대체제로 Deque를 사용하는 것을 권장하고 있음.
- 컴포지션 대신 상속을 사용할 때의 체크리스트
   1. 확장하려는 클래스의 API 에 아무런 결함이 없는가?
   2. 결함이 있다면 이 결함이 다른 클래스의 API 에 전파되어도 괜찮은가?