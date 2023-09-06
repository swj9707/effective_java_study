# Item 11
## equals를 재정의 하려거든 hashCode도 재정의하라

- equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다

- Object 명세 내용 중..
  - equals 비교에 사용되는 정보가 변경되지 않았다면, 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
  - equals가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다
  - equals가 두 객체를 다르게 판단하더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 값을 반환해야 해시테이블의 성능이 좋아진다

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
```

```java
// 최악의 (하지만 틀리지 않은) hashCode 구현 (사용금지!)
@Override
public int hashCode() {
  return 42;
}
```

```java
// 전형적인 hashCode 메서드
@Override
public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCode(lineNum);
  return result;
}
```

### 성능을 높인답시고 해시코드를 계산할 떄 핵심 필드를 생략해서는 안됨!
