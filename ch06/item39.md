# Item 39
## 명명 패턴보다 애너테이션을 사용하라

- 도구나 프레임워크가 특별히 다뤄야하는 프로그램 요소에는 각각 구분되는 명명패턴을 적용한다
  - 예를들면 테스트 프레임워크인 JUnit 에서는 메서드 이름을 test로 시작하게 지어야 한다
 
- 하지만 이러한 명명 패턴에는 몇가지 단점이 존재한다
  - 오타가 나면 안된다
    - 프로그래머의 실수로 tsetSafetyOverride로 지으면 이 메서드를 무시하고 지나치기 때문
  - 올바른 프로그램 요소에서만 사용된다는 보장이 없다
    - 클래스의 이름을 TestSafetyMechanisms으로 지어 JUnit에게 던진다면 개발자가 의도한 테스트는 수행되지 않는다
  - 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다
    - 특정 예외를 던져야 성공하는 테스트의 경우, 예외 타입을 매개변수로 전달해야할 때 마땅한 방법이 없다


### 어노테이션
- 어노테이션은 이 모든 문제를 해결해주는 개념




```java
// 테스트 메서드임을 선언하는 어노테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}

// @Retention과  @Target은 메타 어노테이션
// @Retention(RetentionPolicy.RUNTIME)은 Runtime 에도 어노테이션이 유지되어야 한다는 의미이다.
// @Target(ElementType.METHOD)은 @Test 어노테이션이 메서드에서만 선언되어야 한다는 의미이다.


public class Sample {
    @Test
    public static void m2() {} // 성공해야 함
    @Test
    public static void m3() { 
        throw new RuntimeException("Failure");
    }
    public static void m4() {}
    @Test
    public void m5() {}
    public static void m6() {}
    @Test
    public static void my() {
        throw new RuntimeException("Failure");
    }
    public static void m8() {}
}

```

```java
import java.lang.reflect.*;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n",
                passed, tests - passed);
    }
}

// 이 테스트 코드는 명령줄로부터 완전 정규화된 클래스 이름을 받아
// 클래스에서 @Test 어노테이션이 달린 메서드를 차례대로 호출하고
// 예외가 발생한다면 오류 메세지를 출력함
```

- 매개변수를 받는 어노테이션 타입
  - 특정 예외를 던져야만 성공하는 테스트를 지원하려면 다음과같은 새로운 어노테이션 타입이 필요
 
```java
//명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();  // 매개변수
}

// 이 어노테이션의 매개변수 타입은 Class<? extends Throwable> 이며
// 모든 예외와 오류타입을 수용한다

- 이런식으로 어노테이션에 따라 초기 언급한 문제점들을 해결할 수 있다
  - 책의 후반부에는 매개변숴를 받는 어노테이션, 반복 가능한 어노테이션 등의 예시가 서술되어 있다




```
# 핵심!

 여러분이 다른 프로그래머가 소스코드에 추가 정보를 제공할 수 있는 도구를 만드는 일을 한다면
적당한 어노테이션 타입도 함께 정의해 제공하자. 도구 제작자를 제외하고는, 일반 프로그래머가
어노테이션 타입을 직접 정의할 일은 거의 없다. 하지만 자바 프로그래머라면 예외 없이 자바가
제공하는 어노테이션 타입들은 사용할 수 있어야 한다(아이템 27, 40 ...)
```
