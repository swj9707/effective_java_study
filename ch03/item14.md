# Item 14
## Comparable을 구현할지 고려하라

## Comparable 이란?
- 객체의 정렬 기준을 정해주기 위한 인터페이스이다.
- compareTo() 메서드가 정의되어 있다.
```java
package java.lang;
import java.util.*;

public interface Comparable<T> {

    public int compareTo(T o);
}
```
<br>

### Comparable의 메서드 compareTo
- Object.equals의 단순 동치성 + 순서 비교 + 제네릭
- Comparable을 구현한다는 것은 인스턴스들에 자연적인 순서가 있다는 것이다.
- 자바 플랫폼 라이브러리의 모든 값 클래스, 열거 타입은 모두 Comparable을 구현하고 있다.
  - 비교를 활용하는 정렬된 컬렉션인 TreeSet과 TreeMap이 있고, 검색과 정렬 알고리즘을 활용하는 Collections와 Arrays가 있다.
<br>

### compareTo 규약
> 객체가 매개변수로 들어온 객체보다 작으면 음의 정수(-1), 같으면(0), 크다면 양의 정수(+1)을 반환한다.<br>
> 객체와 매개변수로 들어온 객체의 타입이 다르다면 ClassCastException 을 반환한다.
- 반사성 : x.compareTo(y) == -y.compareTo(x)
- 대칭성 : x.compareTo(y) == 0 == (x.compareTo(z) == y.compareTo(z))
- 추이성 : (x.compareTo(y) > 0 && y.compareTo(z) > 0) == x.compareTo(z) > 0
- 권고 : (x.compareTo(y) == 0) == x.eqauls(y)
  - 이 권고는 필수가 아니지만 지키는 것이 좋다.
```java
@Test
void test() {
    BigDecimal number1 = new BigDecimal("1.0");
    BigDecimal number2 = new BigDecimal("1.00");

    Set<BigDecimal> set1 = new HashSet<>();
    set1.add(number1);
    set1.add(number2);

    Set<BigDecimal> set2 = new TreeSet<>();
    set2.add(number1);
    set2.add(number2);

    assertThat(set1).hasSize(2);
    assertThat(set2).hasSize(1);
}
```
<br>

### compareTo 작성 요령
- Comparable은 타입을 인수로 받는 제네릭 인터페이스 → compareTo() 인수 타입은 컴파일 타임에 정해진다.
- compareTo()는 각 필드가 동치인지를 비교하는것이 아니라 순서를 비교한다.
- Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야한다면 Comparator(비교자)를 대신 사용 (비교자는 직접 만들거나 자바가 제공하는 것 골라 쓰기)
- 핵심 필드부터 비교해나가서 순서가 먼저 결정되면 결과를 반환하자.
<br>
  
### 기본 타입 필드가 여럿일 때의 비교
```java
class Student implements Comparable<Student> {
    String Name; // 이름
    int studentId;      // 학번
    int score;       // 학점

    public Student(String studentName, int studentId, double score) {
        this.studentName = studentName;
        this.studentId = studentId;
        this.score = score;
    }

    @Override
    public int compareTo(Student o) {
        int result = CharSequence.compare(name, o.Name);

        if (result == 0) {
            result = Integer.compare(studentId, o.studentId);
            if (result == 0) {
                result = Integer.compare(int, o.int);
            }
        }

        return result;
    }
}
```
<br>

### 비교자 생성 메서드를 이용한 비교
```java
import java.util.Comparator;

public class Student implements Comparable<Student> {
    private static final Comparator<Student> COMPARATOR =
            Comparator.comparingInt((Student student) -> student.grade)
                    .thenComparing((Student student) -> student.name)
                    .thenComparingInt((Student student) -> student.age);

    private int grade;
    private String name;
    private int age;

    @Override
    public int compareTo(Student o) {
        return COMPARATOR.compare(this, o);
    }
}
```
<br>

### 두 값의 차이를 가지고 비교
```java
private static final Comparator<Student> HASHCODE_COMPARATOR = new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```
- 이 방법은 오버플로를 일르키거나 부동소수점 계산 방식에 따른 오류가 발생할 수도 있다.
  
#### 정적 compare 메서드 활용
```java
static Comparator<Student> HASHCODE_COMPARATOR = new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
```

#### 비교자 생성 메서드 활용
```java
static  Comparator<Student> HASHCODE_COMPARATOR = Comparator.comparingInt(
            s -> s.hashCode());
```
