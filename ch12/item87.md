# item 87. 커스텀 직렬화 형태를 고려해보라

- 클래스가 `Serializable`을 구현하고 기본 직렬화 형태를 사용한다면 현재의 구현에 종속적이게 된다. 즉, 기본 직렬화 형태를 버릴 수 없게 된다.
- 따라서 유연성, 성능, 정확성과 같은 측면을 고민한 후에 합당하다고 생각되면 기본 직렬화 형태를 사용해야 한다.
- 일반적으로 직접 설계하더라도 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 형태를 사용해야 한다.

## 기본 직렬화가 적합한 경우
- 기본 직렬화 형태는 객체가 포함한 데이터뿐만 아니라 그 객체를 시작으로 접근할 수 있는 모든 객체와 객체들의 연결된 정보까지 나타낸다.
- 그러나, 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만을 표현해야 한다.
- **객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태를 선택해도 무방하다**.

```java
public class Name implements Serializable {
    /**
     * 성. null이 아니어야 함.
     * @serial
     */
    private final Stirng lastName;

    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;

    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;

    ... // 나머지 코드는 생략
}
```
- 성명은 논리적으로 성, 이름, 중간 이름이라는 3개의 문자열로 구성하는데, 위 클래스의 인스턴스 필드들은 이 논리적인 구성 요소를 정확하게 반영했다.
- 기본 직렬화 형태가 적합해도 불변식 보장과 보안을 위해 `readObject` 메서드를 제공해야 하는 경우가 많다.
- 앞에서 살펴본 `Name` 클래스를 예로 들면, `lastName`과 `firstName` 필드가 null이 아님을 `readObject` 메서드가 보장해야 한다.

## 기본 직렬화가 적합하지 않은 경우
```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    
    ... // 나머지 코드는 생략
}
```
- 논리적으로 문자열을 표현했고 물리적으로는 문자열들을 이중 연결 리스트로 표현했다.
- 이 클래스에 기본 직렬화 형태를 사용하면 양방향 연결 정보를 포함해 각 노드에 연결된 노드들까지 모두 표현할 것이다.

## 객체의 물리적 표현과 논리적 표현의 차이가 클 때는 생기는 문제
#### 1. 공개 API가 현재의 내부 표현 방식에 종속적이게 된다.
- 예를 들어, 향후 버전에서는 연결 리스트를 사용하지 않게 바꾸더라도 관련 처리는 필요해진다. 따라서 코드를 절대 제거할 수 없다.
#### 2. 너무 많은 공간을 차지할 수 있다.
- 위의 StringList 클래스를 예로 들면, 기본 직렬화를 사용할 때 각 노드의 연결 정보까지 모두 포함될 것이다.
- 하지만 이런 정보는 내부 구현에 해당하니 직렬화 형태에 가치가 없다. 오히려 네트워크로 전송하는 속도를 느리게 한다.
#### 3. 시간이 많이 걸린다.
- 직렬화 로직은 객체 그래프의 위상에 관한 정보를 알 수 없으니, 직접 순회할 수밖에 없다.
#### 4. 스택 오버플로를 발생시킨다.
- 기본 직렬화 형태는 객체 그래프를 재귀 순회한다. 호출 정도가 많아지면 이를 위한 스택이 감당하지 못할 것이다.

## 합리적인 직렬화 형태
#### StringList를 위한 합리적인 직렬화 형태는 무엇일까?
- 단순히 리스트가 포함한 문자열의 개수와 문자열들만 있으면 될 것이다.
- 물리적인 상세 표현은 배제하고 논리적인 구성만을 담으면 된다.
```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;

    // 이번에는 직렬화 하지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // 지정한 문자열을 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * StringList 인스턴스를 직렬화한다.
     */
    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeInt(size);

        // 모든 원소를 순서대로 기록한다.
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStream stream)
            throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        int numElements = stream.readInt();

        for (int i = 0; i < numElements; i++) {
            add((String) stream.readObject());
        }
    }
    ... // 나머지 코드는 생략
}
```
- `transient` 키워드가 붙은 필드는 기본 직렬화 형태에 포함되지 않는다.
- StringList.Entry 클래스 (물리적 데이터)를 직렬화 하지 않는다.
- 클래스의 모든 필드가 transient로 선언되었더라도 `writeObjec`와 `readObject` 메서드는 각각 `defaultWriteObject`와 `defaultReadObject` 메서드를 호출한다.
  - `wirteObject`와 `readObject` 메서드를 통해 논리적인 직렬화 형태를 구현한다
  - int size와 Entry head를 `transient` 변경자를 통해 기본 직렬화 형태에 포함시키지 않았기 때문에, 역직렬화하게 되면 기본 값을 얻게 된다.
- **이러한 독자적인 직렬화 형태를 통해, 불필요한 물리적 데이터 없이 논리적인 데이터만으로 직렬화를 구현할 수 있다.**

## SerialVersionUID
- 어떤 직렬화 형태를 선택하더라도 직렬화가 가능한 클래스에는 `SerialVersionUID`(이하 SUID)를 명시적으로 선언해야 한다.
- 이렇게 하면 직렬 버전 UID가 일으키는 잠재적인 호환성 문제가 사라진다.(아이템 86)
- 물론 선언하지 않으면 자동 생성되지만 런타임에 이 값을 생성하느라 복잡한 연산을 수행해야 한다.
```java
private static final long serialVersionUID = <무작위로 고른 long 값>;
```
- SUID가 꼭 유니크할 필요는 없다. 다만 이 값이 변경되면 기존 버전 클래스와의 호환을 끊게 되는 것이다.
- 구버전으로 직렬화된 인스턴스들과의 호환성을 끊는 경우가 아니라면 SUID 값을 절대 수정해서는 안 된다.

## 핵심 정리
- 직렬화 하기로 결정했다면 어떤 직렬화(기본, custom)를 선택할지 심사숙고 해야한다
- 기본 직렬화는 물리적, 논리적 표현이 일치할때만 사용하라
- 일치하지 않으면 커스텀 직렬화를 고안해야 한다.
- 일단 직렬화를 하기로 했다면 설계가 탄탄해야 하고 모든것이 API로 제공되는 것이므로 이 호환성을 영원히 지원해야 한다.
