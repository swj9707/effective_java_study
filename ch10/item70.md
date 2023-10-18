# item 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라

## throwable
자바는 문제 상황을 알리는 타입(throwable)으로 `검사 예외`, `런타임 예외`, `에러`를 제공함. 


## 검사 예외
- 반드시 예외처리를 해야한다.
    - `throw`, `try-catch`를 이용하여 예외처리를 강제화한다.
    - 호출하는 쪽에서 복구하리라 여겨지는 상황이라면 검사 예외를 사용해야한다.
- 복구에 필요한 정보를 알려주는 메소드를 정의하는게 좋다.
    - 예를 들어 물건을 구입하는데 잔고가 부족하다면, 잔고가 얼마나 부족한지 알려줘야한다. (아이템 75)

## 비검사 예외
- 종류
    - 런타임 예외
    - 에러
- 프로그램에서 잡을 필요가 없다.
    - 복구가 불가능하거나 더 실행해봤자 의미가 없기 때문 → 예외처리를 강제화하지 않는다.
    - 프로그래밍 오류를 나타낼 때에는 비검사 예외를 사용해야한다.
### (1) 런타임 예외
- 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용
- 대부분 전제 조건을 만족하지 못했을 때 발생한다.
  - 배열의 인덱스는 0에서 `배열 크기` -1 사이여야 하는데 `ArrayIndexOutOfNoundsException`이 발생했다는 건 이 전제조건이 지켜지지 않았음을 명시
- 구현하는 비검사 `throwable` 은 모두 `RuntimeException`의 하위 클래스여야 함.

### (2) 에러
- 에러는 자바 언어 명세가 요구하는 것은 아니지만 업계에 널리 퍼진 규약
  - JVM이 자원 부족, 불변식 깨짐
- 시스템 레벨에서 심각하여 예측하거나 처리할 수 있는 방법이 없음
- Error 클래스를 상속해 하위 클래스를 만들지 말아야 하고 throw문으로 던지지도 말아라.

## 핵심 정리
- 복구해야하는 상황이면→ **검사 예외**
- 프로그래밍 시 나타나는 오류라면 -> **런타임 예외**
- 확실하지 않은 예외라면 → **비검사 예외**
- `Error` 클래스를 상속해 하위 클래스를 만드는 일 → **하지말자**
- `Exception`, `RuntimeException`, `Error`를 상속하지 않은 `throwble` → **만들지 말자**