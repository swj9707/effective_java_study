# Item73

## 추상화 수준에 맞는 예외를 던져라.

- 무슨 의미냐면, 무작정 던지지 말라는 의미.
- 저수준, 즉 하위 메소드에서 해결해야 할 예외를 무작정 던졌다간 다른 윗 레벨의 API를 오염시키는 결과를 초례한다.

## 예외 번역

- 저수준에서 생긴 예외를 만약 던져야 하는 케이스라면, 적당한 예외로 바꿔서 던져야 한다.

```java
try {
    //예외가 일어날 수도 있는 코드
} catch (LowerLevelException e) {
    throw new HigherLevelException("예외 발생!");
}
```

- 또 다른 예시 (AbstractSequentialList)
  ```java
  public E get(int idex) {
      ListIterator<E> i = listIterator(index);
      try {
          return i.next();
      } catch (NoSuchElementException e) {
  	        throw new IndexOutOfBoundsException("인덱스: " + index);
      }
  }
  ```

## 예외 연쇄

- 문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방법
  ```java
  try {
      //예외가 일어날 수도 있는 코드
  } catch (LowerLevelException e) {
      throw new HigherLevelException(e);
  }
  ```
- 이 경우 저수준 예외의 정보까지 전달 되므로 상위 레이어에서 해당 예외에 대한 정보 유실 없이 디버깅을 할 수 있다.

## 결론

- 예외를 전파하는 것 보다 예외를 번역하는 게 더 좋다. 단, 무작정 사용해서는 안되고 우선순위가 있다.

  1. 가능하다면 저수준 메서드가 반드시 성공해 예외를 발생시키지 않도록 한다.
  2. 상위 계층 메서드의 매개변수 값을 아래로 넘기기 전에 미리 검사한다.
  3. 예외를 예방 및 처리 불가능할때 예외 번역을 사용한다.

- 또 다른 방법
  - 굳이 예외를 API 호출자에게 까지 전달하지 말고 로거로 기록만 하는 방법.
