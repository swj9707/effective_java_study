# Item 10
## equals는 일반 규약을 지켜 재정의하라. 

- equals를 따로 재정의 하지 않아도 되는 경우
  - 각 인스턴스가 본질적으로 고유하다
  - 인스턴스의 논리적 동치성을 검사할 일이 없다.
  - 상위 클래스에서 재정의한 equals가 하위 클래스에서도 들어맞는다.
  - 클래스가 private이거나 package-private이고 equal 메소드를 호출할 일이 없다. 

- equals를 재정의 해야 하는 경우
  - 객채 식별성이 아니라 논리적 동치성을 확인해야 하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않았을 때

<br>

### equals 메서드를 재정의 할 때는 반드시 일반 규약을 따라야 한다!
- Object 명세에 적힌 규칙
  - 반사성 : null이 아닌 모든 참조 x에 대해, x.equals(x)는 true다.
  - 대칭성 : null이 아닌 모든 참조 값 x,y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
  - 추이성 : null이 아닌모든 참조 값 x,y,z에 대해 x.equals(y)가 true이고 y.equals(z)도 true이면 x.equals(z)도 true다.
  - 일관성 : null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다
  - null-아님 : null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.

- 양질의 equals 구현 방법
  - == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다
  - instanceof 연산자로 입력이 올바른 타입인지 확인한다
  - 입력을 올바른 타입으로 형변환한다.
  - 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다

```java
public final class PhoneNumber {
  private final short areaCode, prefix, lineNum;

  public PhoneNumber(int areaCode, int prefix, int lineNum) {
    this.areaCode = areaCode;
    this.prefix = prefix;
    this.lineNum = lineNum;
  }

  @Override
  public boolean equals(Object o) {
    if (o == this)
      return true;
    if (!(o instanceof PhoneNumber))
      return false;
    PhoneNumber pn = (PhoneNumber)o;
    return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
  } 
}
```

(equals를 재정의할 땐 hashCode도 반드시 재정의 하자!)

---

### 핵심정리 : 꼭 필요한 경우가 아니라면 equals를 재저의하지 말자.
### 대부분의 경우에는 Object의 equals 메서드가 원하는 비교를 정확히 수행해준다. 
### 단, 재정의 해야할 때는 클래스의 핵심 필드 모두를 빠짐없이, 규약을 확실히 지켜가며 비교해야 한다.
