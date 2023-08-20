# item46. 스트림에서는 부작용 없는 함수를 사용하라

## 스트림
- 단순 API가 아닌, 함수형 프로그래밍에 기초한 패러다임
- 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야함
  - 순수 함수 : 오직 입력만이 결과에 영향을 주는 함수
  - 함수 객체는 모두 부작용(side effect)이 없어야 한다.
  
## 안좋은 예시
```java
Map<String, Long> freq = new HashMap<>();
try(Stream<String> words = new Scanner(file).tokens())
	words.forEach(word -> {
	freq.merge(word.toLowerCase(), 1L, Long::sum);
           });
   }
```
- 스트림 코드를 가장한 반복적 코드
- 이 코드의 모든 작업이 종단 연산인 forEach에서 일어나는데, 이 때 외부 상태(빈도표)를 수정하는 람다를 실행하면서 문제가 생김
- forEach는 그저 스트림이 수행한 연산 결과를 보여주는 일만 해야하는 데, 그 이상을 함

## 올바른 예시
```java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
freq = words.collect(groupingBy(String::toLowerCase, counting()));
    }
```
- 수집기(collector)를 사용
- for-each 반복문은 forEach 종단 연산과 비슷하게 생겼다.
  - 'for-each'는 간단한 반복을 위해 사용되고, 'forEach'는 스트림을 사용하여 더 복잡한 데이터 처리 작업을 수행할 때 유용하다.
  - 하지만 forEach 연산은 종단 연산 중 기능이 가장 적고 가장 '덜' 스트림답다. 대놓고 반복적이라서 병렬화할 수도 없다.
- **forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자**
  - 물론 가끔은 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 용도로도 쓸 수 있다.
  
## 수집기(collector)
- 축소(reduction) 전략을 캡슐화한 블랙박스 객체
  - 축소 : 스트림의 원소들을 객체 하나에 취합
- 수집기가 생성하는 객체는 일반적으로 컬렉션이다
- toList(), toSet(), toCollection(colelctionFactort)
```java
// 빈도표에서 가장 흔한 단어 10개를 뽑아내는 스트림 파이프라인
List<String> topTen = freq.keySet().stream()
    .sorted(comparing(freq::get).reversed())
    .limit(10) 
    .collect(toList());  
}
```
- comparing : 키 추출 함수를 받은 비교 생성 메서드
- freq::get 입력받은 단어(키)를 빈도표에서 찾아(추출) 빈도 반환
- reversed : 가장 흔한 단어가 위에 오도록 비교자(comparing)을 역순으로 정렬

### toMap
**toMap(keyMapper, valueMapper)**
- 가장 간단한 맵 수집기
- 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다
```java
// toMap 수집기를 사용하여 문자열을 열거 타입 상수에 매핑한다.
private static final Map<String, Operation> stringToEnum = 
    Stream.of(values()).collect(
        toMap(Obejct::toString, e->e));
```
- 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다
- 스트림 요소들이 key를 중복해서 사용하면 IllegalStateException 을 던지며 종료된다

```java
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
```
- 비교자 : BinaryOperator에서 정적 임포트한 maxBy라는 정적 패터리 메서드 사용

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```
- 네 번째 인수 : 맵 팩터리
- 이 인수로는 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직정 지정 가능

### groupingBy
- 입력으로 분류 함수를 받고, 출력으로 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기 반환
- 분류 함수는 입력받은 원소가 속하는 카테고리를 반환
```java
// 알파벳화한 단어를 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 맵 생성
words.collect(groupingBy(word -> alphabetsize(word)))
```

### joining
- 이 메서드는 (문자열 등의) CharSequence 인스턴스의 스트림에만 적용할 수 있다.
- 이 중 매개변수가 없는 joining은 단순히 원소들을 연결(concatenate)하는 수집기를 반환한다.
- 한편 인수 하나짜리 joining은 CharSequence 타입의 구분문자(delimiter)를 매개변수로 받는다.


## 핵심 정리
- 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다.
  - 스트림뿐 아니라 스트림 관력 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다.
- 종단 연상 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다.
  - 계산 자체에는 이용하지 말자.
- 스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다.
  - 가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다.
