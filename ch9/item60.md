# item 60. 정확한 답이 필요하다면 float와 double은 피하라

## float과 double의 문제점
`float` 과 `double` 타입은, 이진 부동소수점 연산에 쓰이며 넓은 범위의 수를 빠르게 정밀한 **근사치**로 계산하도록 설계되었다.
따라서, 정확한 결과가 필요할때는 부적합하다. 특히나, 금융 관련 계산에는 사용하면 안된다. 0.1 혹은 10의 음의 거듭제곱 수를 표현할 수 없기 때문이다.

```java
System.out.println(1.03-0.42); // 0.6100000000000001
```

### 금융 계산에 부동소수 타입 사용
```java
  public static void main(String[] args) {
        double funds = 1.00;
        int itemsBought = 0;
        for (double price = 0.10; funds >= price; price += 0.10) {
            funds -= price;
            itemsBought++;
        }
        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds); // 0.3999999999999
    }
```

## 해결 방법
### (1) BigDecimal
```java
  public static void main(String[] args) {
      final BigDecimal TEN_CENTS = new BigDecimal(".10");

      int itemsBought = 0;
      BigDecimal funds = new BigDecimal("1.00");
      for (BigDecimal price = TEN_CENTS;
           funds.compareTo(price) >= 0;
           price = price.add(TEN_CENTS)) {
           funds = funds.subtract(price);
           itemsBought++;
      }
      System.out.println(itemsBought + "개 구입");
      System.out.println("잔돈(달러): " + funds); // 4개 구입시 잔돈 0
  }
```
- BigDecimal 은 여덟 가지 반올림 모드를 제공하기 때문에 **반올림을 완벽히 제어할 수 있다**는 장점이 존재한다.   
- 하지만, 기본 타입보다 **속도가 느리고 쓰기 불편**하다.

### (2) 정수 타입 사용
```java
 public static void main(String[] args) {
      int itemsBought = 0;
      int funds = 100;
      for (int price = 10; funds >= price; price += 10) {
           funds -= price;
           itemsBought++;
      }
      System.out.println(itemsBought + "개 구입");
      System.out.println("잔돈(센트): " + funds);
 }
```
- 기본 타입이므로 성능은 좋지만, **다를 수 있는 값의 크기가 제한**되고 **소수점을 직접 관리**해야 한다.

참고로 int 는 숫자를 아홉 자리의 십진수로 표현 가능하고, long 은 숫자를 열여덟 자리의 십진수까지 표현할 수 있으므로 그 이상의 자리를 표현하고자 한다면 BigDecimal 을 사용하는 것을 추천한다.

## 핵심 정리
- 정확한 답이 필요한 계산에는 float나 double을 피하자.   
- 법으로 정해진 반올림을 수행해야 하는 비즈니스 계산에서는 BigDecimal을 사용하자.  
- 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 너무 크지 않다면 int나 long을 사용하자.  
