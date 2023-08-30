# Item 63
## 문자열 연결은 느리니 주의하라

문자열 '+' 연산은 여러 문자열을 하나로 합쳐주는 편리한 수단이다
### 하지만 본격적으로 사용하기 시작하면 성능 저하를 감내하기 어렵다
- 문자열을 '+' 연산하면 문자열 n개를 잇는 시간은 n^2에 비례한다
- 아래 코드처럼 반복하면서 연산수가 많아지게 되면 심각한 성능 저하를 일으킬 수 있다.
  ```java
  public String statement() {
    String result = "";
    for (int i = 0; i < numItems(); i++ {
      result += lineForItem(i);
    }
    return result;
  }
  ```

### 성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자
참고로 품목을 100개로 하고 길이 80인 문자열을 반환하게해서 테스트하니 6.5배나 빨랐다고 한다
```java
public String statement2() {
  StringBuilder b = new StringBuilder(numItem() * LINE_WIDTH);
  for (int i = 0; i < numItems(); i++) {
    b.append(lineForItem(i));
  }
  return b.toString();
}
```


```java
핵심!
그냥 StringBuilder 써라
```
