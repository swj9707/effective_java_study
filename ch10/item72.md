# Item 72

## 표준 예외를 사용하라
<br>

### 표준 예외란?
> - 자바 API에서 제공하는 예외. <br>
> - 가독성이 높아지고, 사용하기 쉽다. <br>
> - 메모리 사용량도 줄어들고, 클래스를 메모리에 적재하는 시간도 적게 걸린다. <br>
<br>

### 대표적인 표준 예외
- IllegalArgumentException
  - 허용하지 않은 값이 인수로 전달되었을 때

- IllegalStateException
  - 객체가 메서드를 사용하기에 적절하지 않은 상태일 때

- NullPointerException
  - null을 허용하지 않는 메서드가 null을 인자로 받을 때

- IndexOutOfBoundsException
  - 인덱스의 범위를 넘어설 때

- ConcurrentModificationException
  - 허용되지 않은 동시 수정이 발견되었을 때

- UnsupportedOperationException
  - 호출한 메서드가 지원하지 않을 때

  <br>

### Exception, RuntimeException, Throwable, Error는 직접 재사용 하지 말 것
- 위 클래스들은 예외 클래스들의 상위 클래스이다. 즉, 구체적인 예외 정보를 알 수 없으므로 안정적으로 테스트할 수 없다.

<br>

### 주의사항

- 상황에 부합한다면 항상 표준 예외를 재사용하자. 단, API 문서를 참고하여 사용하고자 하는 Exception이 이름 뿐만아니라 맥락에도 부합하는지 확인해야한다.

- 더 많은 정보를 제공하고자 한다면 표준 예외를 확장해도 된다. 단, 예외는 직렬화할 수 있다는 것을 감안해야한다.

    직렬화에는 많은 부담이 따르므로 이 사실만으로도 커스텀한 예외를 사용해야할 이유가 사라진다.
