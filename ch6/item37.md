# Item 37

## ordinal 인덱싱 대신 EnumMap을 사용하라

### ordinal()란?
> - 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 메소드

<br>

### 예시1, ordinal()을 배열 인덱스로 사용 - 따라 하지 말 것!
```java
public class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    public Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }

    public static void main(String[] args) {
        Set<Plant>[] plantsByLifeCycle = (Set<Plant>[]) new Set[LifeCycle.values().length];
        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            plantsByLifeCycle[i] = new HashSet<>();
        }
        List<Plant> garden = new ArrayList<>(); // 편의상 빈 리스트로 초기화 했다.
        for (Plant plant : garden) {
            plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
        }

        // 결과 출력
        for (int i = 0; i < plantsByLifeCycle.length; i++) {
            System.out.printf("%s : %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
        }
    }
}
```
- 배열은 제네릭과 호환되지 않으니 비검사 형변환을 진행해야한다.
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
<br>

### EnumMap을 사용해 데이터와 열거 타입을 매핑한다.
- 위 예제에서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 역할을 한다. 따라서 Map을 사용할 수도 있을 것이다.
- EnumMap은 열거 타입을 키로 사용하도록 설계한 아주 빠른 Map 구현체이다.

#### 에시 2, EnumMap을 활용한 열거 타입과 데이터의 매핑
```java
public class Plant {

    // ...

    public static void main(String[] args) {
        Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
        for (Plant.LifeCycle lifeCycle : Plant.LifeCycle.values()) {
            plantsByLifeCycle.put(lifeCycle, new HashSet<>());
        }
        List<Plant> garden = new ArrayList<>(); // 편의상 빈 리스트로 초기화 했다.
        for (Plant plant : garden) {
            plantsByLifeCycle.get(plant.lifeCycle).add(plant);
        }
        System.out.println(plantsByLifeCycle);
    }
}
```
- 더 짧고 명료하고 안전하다.
  - 안전하지 않은 형변환은 쓰지 않는다.
  -  맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공한다.
  - 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄된다.
- 내부 구현 방식을 안으로 숨겨서 Map의 타입 안전성과 배열의 성능을 모두 얻었다.
<br>

### Stream 사용
- 스트림을 사용하여 코드를 더 단순하게 바꿀 수 있다.

#### 예시 3, EnumMap을 사용하지 않고 Stream을 사용한 경우
```java
public class Plant {

    // ...

    public static void main(String[] args) {
        List<Plant> garden = new ArrayList<>(); // 편의상 빈 리스트로 초기화 했다.
        System.out.println(garden.stream().collect(groupingBy(plant -> plant.lifeCycle)));
    }
}

```
- 위 코드는 EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다.

- #### 예시 4, EnumMap을 이용해 데이터와 열거 타입을 매핑한다.
 ```java
public class Plant {

    // ...

    public static void main(String[] args) {
        List<Plant> garden = new ArrayList<>();
        System.out.println(garden.stream().collect(
                groupingBy(plant -> plant.lifeCycle,
                        () -> new EnumMap<>(LifeCycle.class), toSet())));
    }
}

```
- EnumMap 버전은 언제나 열거 타입당 하나씩 중첩 맵을 만들지만, Stream에서 사용하면 해당 열거 타입에 속하는 객체가 있을 때만 만든다.
<br>
### 정리
- 배열의 인덱스를 얻기 위해서 Ordinal()을 사용하는 것 보다 EnumMap을 사용하는 것이 좋다.
