# Item74

## 메소드가 던지는 모든 예외를 문서화하라

- 예외를 문서로 남기지 않는다면 해당 클래스와 인터페이스를 사용하는 데 애로사항이 생긴다.
  - 어떤 부분을 조심하며 써야할 지에 대해 전혀 알 수 없다.

## 문서화 하는 법

```java
/ **
 * 주석의 설명문
 *
 * @throws java.io.FileNotFoundException 지정된 파일을 찾을 수 없습니다
 * /
```

- JavaDoc의 `@throws` 태그를 사용한다.
- 단, 한 클래스에 정의 된 많은 메소드가 같은 이유로 같은 예외를 던진다면 각각의 메소드가 아닌 클래스 설명에 추가하는 방향도 고려해볼 수 있다.

- 주의사항

  - 메소드가 Exception이나 Throwable 같은 공통 상위클래스를 던진다고 선언해서는 안된다.
  - 단 main은 오직 JVM 만이 호출하므로 Exception 을 던지도록 선언해도 괜찮다.

- 비검사 예외도 문서화 하면 좋지만 throws 목록에 작성하지 않는다. (구별 목적)
  - 인터페이스 메소드에서 해당 오류에 대한 명세는 구현체들의 명세로 전파될 수 있다.
