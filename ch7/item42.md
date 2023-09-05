# Item 42

## 익명클래스보다는 람다를 사용해라.

### 익명클래스란?
> - 클래스를 생성하면서 동시에 인스턴스를 생성할 수 있는 방법.
> - 이름이 없다. 주로 인터페이스나 추상 클래스로 구현하거나, 상속받는 형태로 사용한다.

### 람다란?
> - 익명클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았음.
> - 람다는 익명클래스보다 코드가 훨씬 간결하다.
> - 매개 변수와 반환값의 타입은 컴파일러가 문맥을 살펴 추론해준다.

#### 예시 1, 익명클래스
``` java
Button myButton = new Button();
myButton.setOnClickListener(new onClickListener()) {
	@Override
	public void onClick(View v) {
	...
	}
}

```

#### 예시 2, 람다식
``` java
Button myButton = new Button();
myButton.setOnClickListener(v -> { ... })

```

#### 예시 3-1), 람다 활용
```java
public enum Operation {
    PLUS("+") {
	    public douvle apply(double x, double y) { return x + y };
	},
	MINUS("-") {
		public douvle apply(double x, double y) { return x - y };
	},
	TIMES("*") {
		public douvle apply(double x, double y) { return x * y };
	},
	DIVIDE("/") {
		public douvle apply(double x, double y) { return x / y };
	};
	
	private final String symbol;
	Operation(String symbol) {this.symbol = symbol;}
	@Override public String toString() {return symbol;}
	public abstract double apply(double x, double y);
}
```

#### 예시 3-2), 람다 활용
```java
public enum Operation {
	PLUS("+", (x, y) -> x + y),
	MINUS("-", (x, y) -> x - y),
	TIMES("*", (x, y) -> x * y),
	DIVIDE("/", (x, y) -> x / y);

	private final String symbol;
	private final DoubleBinaryOperator op;

	Operation(String symbol, DoubleBinaryOperator op) {
		this.symbol = symbol;
		this.op = op;
	}

	@Override public String toString() {
		return symbol;
	}

	public double apply(double x, double y) {
		return op.applyAsDouble(x, y);
	}
}

```


- 이러한 람다식도 비교자 생성  메서드나, 참조 메서드를 사용하여 더 짧게 만들 수 있다. <br>
  but,
- 메서드나 클래스와 달리, 람다는 이름이 없고 문서화를 하지 못한다.
  - 코드 자체로 동작이 명확히 설명되지 않거나 코드의 줄 수가 많아지면 람다를 사용하는 것이 역효과가 날 수 있다.
- 람다식은 자기 자신은 참조할 수 없다.
- 람다로 대체하지 못하는 곳들도 있다.
  - 람다는 함수형 인터페이스에서만 사용가능하다.
  - 추상클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니. 이 때는 익명클래스를 사용한다.
  

