# Item 49

## 매개변수가 유효한지 검사하라.

### 유효한지 왜 검사해야하나?
> - 메서드 몸체에서 실행되기 전, 즉각적으로 예외를 던지기 위함.<br>
> - 그렇지 않다면 메서드가 실행되는 중간에 예외를 던질 수 있음.<br>
> - 또는 메서드는 실행이 되지만, 의도하지 않은 결과값을 반환할 수 있음.<br>
> - 즉, 실패 원자성을 보장하지 못하며 해당 메서드와 관련 없는 오류를 발생시킬 수 있다.
<br>

#### 예시 1 , 유효 검사하지 않은 메서드
```java
	private List<String> names;

	public String getNameOf(int index) {
		return names.get(index);
}

```
- 매개변수인 정수 index를 받아, names라는 List의 특정 인덱스를 조회하는 메서드.
- 여기서 index가 음수값을 가져오면 names.get(index); 에서 오류가 발생한다.
- 따라서 예외를 던져서 사전에 방지할 수 있다.
<br>

#### 예시 2, 유효 검사한 메서드
``` java
private List<String> names;

	public String getNameOf(int index) {
		if(index < 0 )
			throw new ArithmeticException("음수 값의 매개변수");
		return names.get(index);
}

```
<br>

### public과 protected 메서드는 던지는 예외를 문서화해야한다
> - public과 protected는 외부 접근도가 높은 제어자이므로 이들 메서드에는 
<br>

#### 예시 3, 문서화된 유효 검사.
- @throws Javadoc 태그를 통해 문서화해야한다.
``` java
/**
 * Returns a BigInteger whose value is {@code (this mod m}).  This method
 * differs from {@code remainder} in that it always returns a
 * <i>non-negative</i> BigInteger.
 *
 * @param  m the modulus.
 * @return {@code this mod m}
 * @throws ArithmeticException {@code m} &le; 0
 * @see    #remainder
 */
public BigInteger mod(BigInteger m) {
    if (m.signum <= 0)
        throw new ArithmeticException("BigInteger: modulus not positive");

    BigInteger result = this.remainder(m);
    return (result.signum >= 0 ? result : result.add(m));
}

```
- BigInteger의 mod 메서드, throw를 통해 유효검사를 함.
- 모든 메서드에 적용되는 유효검사를 하고 싶다면, 클래스 수준에서 JavaDoc을 작성하면된다.
<br>

### assert
> - assert 단언문은 자신이 선언한 조건이 무조건 참이어야 다음 로직을 수행시킬 수 있다.<br>
> - Java 실행시 -ea 또는 --enableassertions 옵션을 주지 않으면 런타임에 영향을 미치지 않으며, 성능 저하 또한 없다.<br>
> - 개발 중에 테스팅 하는 목적.<br>
<br>

#### 예시 4, assert 유효 검사
```java
private String get(int index) {
	assert index >= 0;
	return names.get(index);
}
```

<br>
- 유효성 검사 비용이 지나치게 높거나 실용적이지 않다면 고려해볼 필요가 있다.
	- 대신 Exception에 대한 대응 방안을 둬야한다.

