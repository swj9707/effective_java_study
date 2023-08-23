# Item 53
## 가변인수는 신중히 사용하라

### 가변인수란?
```java
// Simple use of varargs (Page 245)
static int sum(int... args) {
    int sum = 0;
    for (int arg : args)
        sum += arg;
    return sum;
}

```
- 연속 된 파라미터 값들을 컬렉션으로 변활 할 필요가 없는 예시
- 하지만 특정 경우에는 문제가 생길 수 있다. 
  

```java
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("Too few arguments");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```

- 이 경우 인수가 만약 0개라면 무조건 런타임 에러를 발생 시킨다. 
  - 차라리 else 로 빼던가...
- 코드도 지저분하다.

개선 된 코드

```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```
- 반드시 첫 번째 인수는 입력하도록 강제하는 방법

### 성능이 민감한 상황이라면 가변 인수의 사용은 고려해야한다. 
- 가변인수 메소드는 호출 될 때마다 배열을 새로 하나 할당하고 초기화 한다. 
- 이런 경우 무작정 가변 인수를 사용했다간 메모리 할당 및 해제 과정에서 비용이 많이 발생한다.

- 이런 경우 인수의 갯수에 따라 다중정의 후 가장 마지막 다중정의 메소드에 가변 인수를 쓰는 방법이 있다. 

```java
public void foo() { }
public void foo(int a1) { }
public void foo(int a1, int a2) { }
public void foo(int a1, int a2, int a3) { }
public void foo(int a1, int a2, int a3, int... rest) { }
```
