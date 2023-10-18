# Item 71

## 필요 없는 검사 예외 사용은 피하라

### 검사 예외란?
> - 어플리케이션 수행 중에 일어날법한 예외를 검사하고 대비하라는 목적으로 사용.<br>
> - 반드시 예외 처리를 해야한다.<br>
> - 컴파일러가 발견해서 컴파일 오류를 발생시킨다.
> - try/catch, throws 방식이 존재
> - 처리를 강요하는 것이기 때문에 과하게 사용한다면 API 사용자에게 부담을 준다.
<br>

### 검사 예외를 피하는 방법
- 빈 Optional을 반환
- 상태 검사 메서드와 Unchecked Exception을 추가
<br>

#### 예시 1, 빈 Optional
```java
public static Optional<Database> create() {

    try {
        database.create(); // create() 메서드 안에선 throws IOException으로 컴파일 에러 발생 가정.
    } catch (IOException e) {
        return Optional.empty(); // 빈 옵셔널 반환
    }
    return Optional.of(Database);
}
```
- 예외가 발생한 부가 정보를 담을 수 없으므로 좋은 방법은 아님
<br>

#### 예시 2, 상태 검사 메서드와 Unchecked Exception
``` java
public static Optional<Database> create() {

    if (database.canCreateFile()) { // 예외가 던져질 지 여부를 boolean값으로 반환
        database.create();
    } else {
        throw new CustomDatabaseIOException();
    }
}
```
- 여러 스레드가 동시에 접근한다면 canCreateFile와 create 호출부 사이에 외부 요인이 접근 가능.
- 그로 인해 객체의 상태가 변할 수 있다.
<br>

### 결론
1. API호출자가 예외 상황에서 복구할 방법이 없다 => 비검사 예외를 던진다.
2. API호출자가 예외 상황에서 복구할 방법이 있으며, 호출자가 처리하기를 원한다. => optional 반환 고려, 예외가 발생할 수 있음을 암시해야함.
3. API호출자가 예외 상황에서 복구할 방법이 있으며, 예외를 api에서 던진다. => 검사 예외를 던진다.
