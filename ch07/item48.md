# Item48
## 스트림 병렬화는 주의해서 적용하라

### 스트림 병렬화 나쁜 예시

```java
package effectivejava.chapter7.item48;

import java.math.BigInteger;
import java.util.stream.Stream;

import static java.math.BigInteger.*;

// Parallel stream-based program to generate the first 20 Mersenne primes - HANGS!!! (Page 222)
public class ParallelMersennePrimes {
    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .parallel()
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```
- Item 45 에서 다뤘던 메르센 소수를 생성하는 프로그램을 살펴보면 다음과 같습니다. 
- 이 프로그램의 성능을 높이기 위해 parallel 을 호출하면 벌어지는 일?
  - 아무것도 출력하지 못 하고 CPU를 90%를 잡아먹는다. 
- 스트림 라이브러리가 이 파이프라인을 병렬화 하는 방법을 알아내지 못함. 
- 환경이 아무리 좋더라도 데이터 소스가 Stream.iterate 거나 중간 연산으로 limit을 사용한다면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다. 
- 파이프라인 병렬화는 limit을 다룰 때 CPU 코어가 남는다면 원소를 몇개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다. 
  - 해당 코드는 메르센 소수를 새롭게 찾을 때 마다 그 전 소수를 찾을 때 보다 두 배 정도 더 오래 걸린다. 
  - 원소 하나를 계산하는 비용이 그 이전까지의 원소 전부를 계산한 비용을 합친 것 만큼 든다는 의미

### 그럼 언제 사용해야 하는가?
- 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스이거나 배열, int, long 범위일 때 병렬화의 효과가 가장 좋다. 
  - 공통점 1 : 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있다. 
    - 다수의 스레드에 일을 분배하기 좋다.
  - 공통점 2 : 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어나다. 
    - 이웃한 원소의 참조들이 메모리에 연속해서 저장 되어있다. 

- 스트림 파이프라인의 종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하며 순차적인 연산이라면 파이프라인 병렬 수행의 효과는 제한될 수 밖에 없다. 
- 종단 연산 중 병렬화에 적합한 것은 축소. 
  - reduce, min, max, count, sum
  - anyMatch, allMatch, noneMatch 처럼 조건에 맞으면 바로 반환되는 메소드도 적합하다.
 
- 적합하지 않은 예시
  - collect : 가변 축소 -> 합치는 부담이 너무 큼

- 결론
  - 병렬화의 이점을 제대로 누리게 하고 싶다면 spliterator 메소드를 반드시 재저으이 하고 결과 스트림의 병렬화 성능을 강도 높게 테스트 해야 한다. 
  - 안전 실패 : 스트림의 결과가 잘못되거나 오동작 하는 것
  - Stream 의 reduce 연산에 건네지는 ccumulator 와 combiner 함수는 반드시 결합법칙을 만족하고 간섭받지 않고 상태를 갖지 않아야 한다. 
  - 출력 순서를 순차 버전처럼 정렬하고 싶다면 종단 연산 forEach를 forEachOrdered 로 바꿔주면 된다. 
  - 심지어 데이터 소스 스트림이 효율적으로 나눠지고, 병렬화하거나 빨리 끝나는 종단 연산을 사용하고, 함수 객체들도 간섭하지 않더라도 파이프라인이 수행하는 진짜 작업이 병렬화에 드는 추가 비용을 상쇄하지 못한다면 성능 향상은 미미할 수 있다.
  - 스트림 병렬화는 오직 성능 최적화 수단. 반드시 해야 할 필요가 있는 지 확인 할 필요가 있다. 

### 잘 사용 된 스트림 병렬화 코드의 예시

```java
package effectivejava.chapter7.item48;

import java.math.BigInteger;
import java.util.stream.LongStream;

public class ParallelPrimeCounting {
    // Prime-counting stream pipeline - parallel version (Page 225)
    static long pi(long n) {
        return LongStream.rangeClosed(2, n)
                .parallel()
                .mapToObj(BigInteger::valueOf)
                .filter(i -> i.isProbablePrime(50))
                .count();
    }

    public static void main(String[] args) {
        System.out.println(pi(10_000_000));
    }
}
```

### 정리
- 계산의 결과가 명확하고 성능이 빨라질 것이란 확신 없이 스트림 파이프라인 병렬화를 시도 하지 마라. 오히려 문제를 일으킬 수 있다. 