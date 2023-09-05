# Item 53
## 가변인수는 신중히 사용하라

### 가변인수
- 가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다
  - 가변인수를 메서드를 호출하면 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 전달

```java
// 가변인수 등장 전
static int sum(int arg1) {
  ...
}
static int sum(int arg1, int arg2) {
  ...
}
static int sum(int arg1, int arg2, int arg3) {
  ...
}


// 가변인수 등장 후
static int sum (int... args) {
  int sum = 0;
  for (int arg : args) {
    sum += arg;
  }
  return sum;
}
```

- 위와 같이 가변인수를 사용함으로써 코드의 중복도 줄일 수 있다.

### 왜 신중하게 사용해야 할까?
- 가변인수를 사용할 때는 몇가지 주의사항이 잇다.

#### 인수가 반드시 1개 이상이어야 할 때
잘못 구현한 예
```java
// 최소값을 구하는 로직인데 인수의 예외처리를 해주었음에도
// 인수를 0개 넣어 호출하면 런타임에러가 발생한다
// (코드도 지저분함)
static int min(int... arg) {
  if (args.length == 0) {
    throw new IllegalArgumentException("인수가 1개 이상 필요!");
  }

  int min = args[0];
  for (int i = 1; i < args.length; i++) {
    if (args[i] < min) {
      min = args[i];
    }
  }
  return min;
}
```

제대로 사용한 방법
```java
// 매개변수를 2개 받도록 하여 해결
// 첫 번째로는 평범한 매개변수를 받고, 두 번째로 가변인수를 받으면 앞의 문제가 사라진다
static int min(int firstArg,int... remainingArgs) {

  int min = firstArg;
  for (int arg : remainingArgs) {
    if (arg < min) {
      min = arg;
    }
  }
  return min;
}
```

### 성능은 어떨까?
이처럼 가변인수는 인수의 개수가 정해지지 않았을 때 아주 유용하다. 그러나, 성능이 중요한 상황이라면?
- 성능이 중요한 상황이라면 가변인수는 걸림돌이 될 수 있다.
  - 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문
  - 다행히 이런 비용을 감당하지않고 가변인수의 유연성을 사용할 수있는 패턴이 있음
    - 예를들어 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 하면
    - 인수가 0개부터 4개인 것까지, 총 5개를 다중정의 해보자
    - 마지막 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당하는 것
```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
// 메서드 호출중 단 5%의 경우만이 배열을 생성하기 때문에 성능 최적화가 가능하다
// 대다수의 성능 최적화와 마찬가지로 보통때는 별 이득이 없지만 꼭 필요한 특수 상황에서는
// 오아시스가 되어줄 것이다
public void foo(int a1, int a2, int a3, int... rest) {}

```




# 핵심!

 인수 개수가 정해지지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다. 메서드를
정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자!
```
