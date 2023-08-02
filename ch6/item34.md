# Item34

## int 상수 대신 열거형을 사용하라

```java
public static final int APPLE_FUJI          = 0;
public static final int APPLE_PIPPIN        = 1;
public static final int APPLE_GRANNY_SMITH  = 2;

public static final int ORANGE_NAVEL        = 0;
public static final int ORANGE_TEMPLE       = 1;
public static final int ORANGE_BLOOD        = 2;
```

enum 이 없을 경우에는 이런 식으로 정수 상수를 묶어서 선언하여 사용. (int enum pattern, 정수 열거 패턴)  
정수 열거 패턴의 단점
- 타입 안전을 보장할 수 없다.
- 표현력이 좋지 않다. 
- 오렌지를 보내야 할 메소드에 사과를 보내고 == 연산자로 비교해도 아무런 경고 메시지가 출력되지 않는다. 

```java
int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
```

## 정수, 문자열 상수 열거패턴 -> 대안은 열거 타입 (Enum Type)
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH };
public enum Orange { NAVEL, TEMPLE, BLOOD };
```
- 상수 변수가 아니라 하나의 클래스로서 역할 (타입 안전을 보장할 수 있다.)
- 값 그자체가 아니라 인스턴스로서 생성된다. 
- 컴파일 타임 타입 안전성을 제공해줄 수 있다. 
- 각자의 네임스페이스가 있기 때문에 같은 이름의 상수가 존재할 수 있다. 
    ```java
    public enum Apple { GREAT };
    public enum Orange { GREAT } 
    ```
- final 키워드를 사용하지 않기 때문에 컴파일 시 상수로 취급받지 않는다. 
- Enum 타입 자체가 toString 메소드를 지원한다. 
- 클래스이므로 메소드나 필드를 추가할 수 있다. 
```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // In kilograms
    private final double radius;         // In meters
    private final double surfaceGravity; // In m / s^2

    // Universal gravitational constant in m^3 / kg s^2
    private static final double G = 6.67300E-11;

    // Constructor
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
```
### Planet 열거 타입을 활용 한 예시
  - `values` 메소드 : 선언 된 순서대로 상수들 묶음을 리턴하는 메소드
```java
public class WeightTable {
   public static void main(String[] args) {
      double earthWeight = Double.parseDouble(args[0]);
      double mass = earthWeight / Planet.EARTH.surfaceGravity();
      for (Planet p : Planet.values())
         System.out.printf("Weight on %s is %f%n",
                 p, p.surfaceWeight(mass));
   }
}
```
- 만약 상수 하나를 제거한다 하더라도 제거한 상수를 참조하지 않는 클라이언트에는 아무런 영향이 없음. 
- 참조한다 하더라도 컴파일 에러가 발생하되, 해당 위치에 대해 에러가 발생하므로 금방 트러블 슈팅 할 수 있다. 

## 상수마다 동작이 달라져야 하는 상황

```java
public enum Operation {
    PLUS, MINUS, TIMES, DIVIDE;

    public double apply(doublex, double y) {
        switch(this) {
            case PLUS : return x + y;
            case MINUS : return x - y;
            case TIMES : return x * y;
            case DIVIDE : return / y;
        }
        throw new AssertionError("알 수 없는 연산 : " + this);
    }
}
```
- 크게 문제는 없지만, throw 코드가 실행 될 일이 없지만 없으면 오류가 나기 때문에 작성 되어있음 -> 아무런 의미 없음
- 새로운 상수를 추가했다면 새로운 case 코드를 추가해야 함. 
- 개선하려면 아래처럼 해결 할 수 있음. 

```java
public enum Operation {
    PLUS {public double apply(double x, double y) {return x + y}},
    MINUS {public double apply(double x, double y) {return x - y}},
    TIMES {public double apply(double x, double y) {return x * y}},
    DIVIDE {public double apply(double x, double y) {return x / y}};

    public abstract double apply(double x, double y)
}
```

- toString 을 재정의 해 연산을 의미하는 기호를 리턴하게 하는 방법
- fromString : toString 이 리턴하는 문자열을 해당 열거타입 상수로 변환
  
```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);

    // Implementing a fromString method on an enum type (Page 164)
    private static final Map<String, Operation> stringToEnum =
            Stream.of(values()).collect(
                    toMap(Object::toString, e -> e));

    // Returns Operation for string, if any
    public static Optional<Operation> fromString(String symbol) {
        return Optional.ofNullable(stringToEnum.get(symbol));
    }

    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}
```

## 값에 따라 분기하여 코드를 공유하는 열거타입 - 좋은 방법일까?
상수 별 메소드 구현에서는 열거타입 상수 끼리 코드를 공유하기 어렵다는 단점이 있다. 

```java
enum PayrolLDay {
    MONDAY, TUESDAY, WEDSENDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY:

    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minuitesWorked * payRate;
        
        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY:
                overtimePay = basePay /2;
                break;
            default:
                overtimePay = minuutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate /  2;
        }
        return basePay + overtimePay;
    }
}
```

- 간결하지만, 새로운 값이 열거 타입에 추가된다면 case 문을 반드시 넣어 주어야 한다. 
- 상수 별 메소드 구현을 통해 급여가 계산되는 방법
  - 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣는다
  - 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메소드로 작성한 다음 상수가 자신에게 필요한 메소드를 적절히 호출한다. 
  - 둘 다 코드가 장황해지고 오류 발생 가능성이 커진다. 
  - PayrollDay 에 평일 잔업수단 계산용 메소드인 `overtimePay`를 구현 해 놓고 주말 상수에만 재 정의 해서 쓰면 장황함을 줄일 수는 있지만, switch 를 썼을때와 똑같은 단점이 나타난다. 
- 가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수단 전략을 선택하도록 하는 것
```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // The strategy enum type
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }
}
```

- switch 문을 넣는 기준 => 기존 열거 타입에 상수별 동작을 혼합 해 놓을 때
  - 서드파티에서 가져오는 경우
  ```java
  public static Operation inverse(Operation op) {
    switch(op) {
        case PLUS:   return Operation.PLUS;
        case MINUS:  return Operation.MINUS;
        case TIMES:  return Operation.DIVIDE;
        case DIVIDE: return Operation.TIMES;

        default : throw new AssertionError("unknown operation : " + op);
    }
  }
  ```
  - 추가하려는 메소드가 의미상 열거타입에 속하지 않는다면

## 열거 타입을 쓰는 타이밍
- 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 사용한다. 