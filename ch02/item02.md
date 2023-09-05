# Item 02
## 생성자에 매개변수가 많다면 빌더를 고려해라

# 점층적 생성자 패턴
```java
public class NutritionFacts {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
                new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
    
}
```
- 단점
  - 매개변수 갯수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다. 
  - (추가) 매개변수의 입력 순서가 헷깔리고 순서가 변경되는 경우 오작동 할 가능성이 있다. 

# 자바빈즈 패턴
```java
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize  = -1; // Required; no default value
    private int servings     = -1; // Required; no default value
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)     { servings = val; }
    public void setCalories(int val)     { calories = val; }
    public void setFat(int val)          { fat = val; }
    public void setSodium(int val)       { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```
- 단점 
  - 객체 하나를 생성하기 위해 여러개의 메소드를 호출해야 한다. 
  - 객체가 완전히 생성되기 전 까지 일관성이 무너진 상태가 된다. 
    - 객체를 구성하는 데이터가 _*언제든*_ 바뀔 수 있는 상태로 유지된다. 

# 빌더 패턴
```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
        { calories = val;      return this; }
        public Builder fat(int val)
        { fat = val;           return this; }
        public Builder sodium(int val)
        { sodium = val;        return this; }
        public Builder carbohydrate(int val)
        { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
```
- 장점
  - 빌더 내에 setter 메소드들은 빌더 자신을 리턴하므로 연쇄저깅ㄴ 호출이 가능하다. 
  - 읽기 쉽다. 

# 계층 빌더

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // Subclasses must override this method to return "this"
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}
```
- 여기서 잠시 정리
  - 추상 클래스 vs 인터페이스
    - 추상 클래스는 다중 상속이 가능하지 않다.
      - 추상 클래스를 상속해서 새로운 클래스를 생성하는 목적이고 그 자체로는 인스턴스 생성 불가. 
    - 인터페이스는 다중 상속이 가능하다. 
      - 추상 클래스와 비교했을 때 특정 기능에 대한 구현을 강제하되, 하나의 큰 클래스를 생성하는 목적은 아니다. 
      - 예를 들면 스마트폰이란 클래스에서 통신, 배터리 충전 등의 기능들을 인터페이스로 다중 상속할 수 있다. 
    - 추상클래스 사용 시기 : 상속 관계를 쭉 타고 올라갔을때 같은 조상클래스를 상속하는데 기능까지 완변히 똑같은 기능이 필요한 경우
    - 인터페이스 사용 시기 : 상속 관계를 쭉 타고 올라갔을때 다른 조상클래스를 상속하는데 같은 기능이 필요할 경우 인터페이스 사용 

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }

    @Override public String toString() {
        return "New York Pizza with " + toppings;
    }
}
```

```java
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }

    @Override public String toString() {
        return String.format("Calzone with %s and sauce on the %s",
                toppings, sauceInside ? "inside" : "outside");
    }
}
```

- 두 코드의 차이점
  - 뉴욕 피자는 size 매개변수를 필수로 받는다. 
  - 칼초네 피자는 소스를 안에 넣을지를 선택하는 매개변수를 필수로 받는다. 
  - 각 클래스별 빌더에 각자 필요한 매개변수 데이터를 추가할 수 있다. 
  - 하나의 클래스에서 파생 된 다른 클래스라고 해도 복잡하게 생성자를 다시 만들 필요 없다. 

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
    .addTopping(SAUSAGE).addTopping(ONION).build();

Calzone calzone = new Calzone.Builder()
    .addTopping(HAM).sauceInside().build();
```

# 빌더의 단점
- 객체를 만들기 전에 빌더부터 생성해야 한다. 
  - 크게 문제될 건 아니지만...성능에 민감한 상황이라면 빌더를 생성하는 비용 마저도 아껴야 할 수 있다. 