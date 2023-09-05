# Item 50

## 적시에 방어적 복사본을 만들어라.
- 어떠한 객체든 그 객체의 허락 없이는 외부에서 내부를 수정할 수 있게 하면 안된다.
- 하지만 주의깊게 작성하지 않으면 내부를 수정하도록 하는 경우가 생긴다.
<br>

### 예시 1, 불변식 처럼 보이는 코드
```java
public final class Period { 
    private final Date start;
    private final Date end;
	
	public Period(Date strart, Date end) {
		if (start.compareTo(end) > 0) 
			throw new IllegalArgumentException( 
				start + "가 " + end + "보다 늦다.");
		this.start = start;
		this.end = end;		 
	}
	
	public Date start() { 
		return start;
	}
	
	public Date end() {
		return end;
	}
}
```
- 이 예시는 불변처럼 보인다. 하지만 불변을 깨뜨릴 수 있다.
- 이유는 Date가 가변이기 때문에 Date를 통해서 기존의 불변식을 깨뜨릴 수 있다.
<br>

### 예시 2, 불변식 내부의 가변식..
```java
	Date start = new Date();
	Date end = new Date();
	Period p = new Period(start, end);
	end.setYear(78); 
```
- 외부에서 이 코드를 작성하면 예시 1의 불변식의 정보는 변하게 된다.
<br>

### 예시 3, 방어적 복사를 통한 불변식 유지.
```java

/..
// 생성자
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());
	
	if (this.start.compareTo(this.end) > 0 ) {
		throw new IllegalArgumentException (
			this.start + "가 " + this.end + "보다 늦다.");
	}
}
```
<br>

### 예시 4, 가변식 사용시 단순한 방어적 복사에 대한 공격

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78)

```
- 이러한 방법으로 불변식을 유지할 수 없다.
<br>

### 예시 5, 불변식을 유지하는 방법
``` java
public Date start() {
    return new Date(start.getTime());
}
public Date end() {
    return new Date(end.getTime());
}
```
- 접근자가 가변 필드의 방어적 복사본을 반환하면 간단히 막아낼 수 있다.
- 이 식을 통하여 불변식을 위배할 방법이 없어진다.
- 모든 필드가 객체 안에서 완벽히 캡슐화 된다.

