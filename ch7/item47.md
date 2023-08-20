# item47. 반환 타입으로는 스트림보다 컬렉션이 낫다.

## 자바 8 이전

목록을 표현하는 자료구조 반환 타입

- `Collection` 인터페이스 (ex. `Collection`, `List`, `Set`)
    - 기본적으로 사용한다.
- 배열
    - 반환 원소가 기본 타입이거나, 성능이 중요시될 때 사용한다.
- `Iterable` 인터페이스
    - `for-each`문에서만 사용되거나, 일부 `Collection`을 구현할 수 없을 때 사용한다.

## 자바 8 이후, Stream의 등장

Java 8 이후 **Stream**이 생겨났다


## 반복(loop)을 지원하지 않는 Stream

`Stream`은 반복을 지원하지 않는다.

(`Iterable`의 추상 메소드를 모두 구현하고 있지만 정작 `extend`는 하지 않고 있기 때문이다.)

## 스트림을 반복하기 위한 우회
```java
// 컴파일 오류
for (ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {	
}
```
- 이 오류를 잡기 위해 메서드 참조를 메개변수화된 Iterable로 적절히 형변환
```java
// 스트림을 반복하기 위한 '끔직한 우회 방법'
for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
}
```
- 작동은 하나 직관성이 떨어짐

```java
// Stream<E>를 Iterable<E>로 중개해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
	return stream::iterator;
}
```
- 어댑터 메서드 제공을 통해 보안
- 이 경우, 자바의 타입 추론이 문맥을 잘 파악하여 어댑터 메서드 안에서 따로 형변환 하지 않아도 된다.


```java
// Iterable<E>를 Stream<E>로 중개해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
	return StreamSupport.stream(iterable.spliterator(), false);
}
```

## 핵심 정리
- 원소 시퀀스를 반환하는 메서드를 작성할 때 Iterable 과 Stream 두 방식 모두 작성하자
- 하지만 가능하면 Iterable 하위타입이면서 Stream을 지원하는 Collection 의 하위 타입을 반환하도록 하자.
- 반환할 컬렉션의 원소 개수가 적다면 ArrayList와 같은 표준 컬렉션 구현체를 반환하라
- 시퀀스 크기가 크다면 전용 컬렉션을 구현하는 것을 고민하라
