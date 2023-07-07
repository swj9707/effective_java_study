# Item 14
## Comparable을 구현할지 고려하라

## Comparable 이란?
> 인터페이스 관련 내용

### Comparable의 메서드 compareTo
>Object.equals
>> 단순 동치성

### Comparable을 구현한 배열들.

## 정적 메서드와 정적 필드만 담은 클래스
- 객체지향적이지 않다.
- 전역으로 사용 가능하다.
- 유틸리티 클래스라고 불린다.
<br>

### compareTo 규약
반사성, 대칭성, 추이성
```java
public class HonggiInfo {

  public static final int height = 185;
  public static final int weight = 82;
  public static final int foot = 270;

}
```
- 이 클래스는 인스턴스 생성이 가능하다.
  - 인스턴스를 생성하기 위해서는 생성자 호출이 필요함.
  - 생성자가 명시되어 있지 않으면 컴파일러가 자동으로 기본 생성자를 만들어줌.
<br>

### 정적 메서드와 정적 필드만 담은 클래스 예시 2
```java
public class HonggiInfo {
  private HonggiInfo() {} // 외부에서 인스턴스가 생성되는 것을 방지.
  public static final int height = 185;
  public static final int weight = 82;
  public static final int foot = 270;

}
```
- 이 클래스는 인스턴스 생성이 불가능하다.
<br>

### 왜 차단할까?
- 유틸리티 클래스는 인스턴스를 사용하려고 설계한 것이 아니다.
  - 즉 "인스턴스 생성" 이라는 여지를 주지 않기 위함.
  - 협업시 코드 작성의 의도를 파악하지 못하여 혼란을 야기할 수 있음.
