# Item 27

## 비검사 경고를 제거하자.

### 비검사 경고란?
- unchecked warning, 컴파일러가 발생시키는 경고이다.

<br>

#### 예시 1, 비검사 경고문
```java

public class Item27 {

    public static void main(String[] args) {
        Set<String> mySet = new HashSet(); // 비검사 경고 발생
    }
}

```
- 위의 명령어를 실행 시키면 아래와 같은 경고 문구가 발생한다.
```java
javac -Xlint:unchecked Item27.java

Item27.java:10: warning: [unchecked] unchecked conversion
        Set<String> mySet = new HashSet();
                            ^
  required: Set<String>
  found:    HashSet
1 warning
```
- 컴파일러가 알려준 대로 수정하면 경고가 사라진다.

### 비검사 경고 제거 @SuppressWarnings("unchecked")
>- 가능한 모든 비검사 경고를 제거시키면 좋다.
>- 모두 제거한다면 코드 타입 안정성이 보장된다.

- 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있으면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기자
 - 이 경우 타입 안전함을 검증해야한다.
 - 반대로 안전하다고 검증된 비검사 경고를 그대로 두면, 진짜 문제를 알리는 경고를 묵시하게 될 수 있다,
 - 즉, @SuppressWarnings("unchecked") 애너테이션을 사용하여 불필요한 비검사 경고를 제거하되, 반드시 타입 안전함을 검증해야 한다
- @SuppressWarnings 애너테이션은 항상 가능한 한 좁은 범위에 적용하자
 - 보통은 변수 선언, 아주 짧은 메서드, 혹은 생성자가 될 수 있다.
 - 해당 애너테이션은 개별 지역 변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다.
<br>

#### 예시 2, 비검사 경고 제거
```java
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            return (T[]) Arrays.copyOf(elementData, size, a.getClass()); // 비검사 경고 발생
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```

```java
    public <T> T[] toArray(T[] a) {
        if (a.length < size) {
            // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로
            // 올바른 형변환이다.
            @SuppressWarnings("unchecked") T[] result =
                (T[]) Arrays.copyOf(elementData, size, a.getClass());
            return result;
        }
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
```
<br>
- 변수를 선언하고, 변수에 적용함으로써 에너테이션이 적용되는 범위를 최소화 시켰다.
- @SuppressWarnings("unchecked") 애너테이션을 사용할 땐 반드시 비검사 경고를 무시해도 안전한 이유를 주석으로 남긴다.
