# Item 43

## 람다보다는 메서드 참조를 사용하라.

- 람다가 익명클래스보다 나음 점은 간결한 것이다.
- 자바에서는 함수 객체를 메서드 참조를 통해 람다식보다 더 간결하게 만들 수 있다.
<br>

#### 예시 1, map.merge() 
```java
	map.merge(key,1(count, incr) -> count + incr)
	
	map.merge(key,1,Integer::Sum)
```
- 람다의 매개변수가 늘어날수록 메서드 참조로 제거할 수 있는 코드양도 늘어난다.
- 어떤 람다는 매개변수 이름이 프로그래머에게 중요한 가이드가 되기도한다.
- 보편적인 경우, 람다로 할 수 없는 것은 메서드 참조로도 할 수 없다.
<br>

#### 예시2, 람다가 메서드 레퍼런스보다 간결한 경우

```java
class Example {
    public static void main(String[] args) {
        // 메서드 레퍼런스를 사용하는 경우
	    service.execute(GoshThisClassNameIsHumongous::action);
	    
	    // 람다를 사용하는 경우
	    service.execute(() -> action());
    }
}
```
<br>

### 메서드 레퍼런스의 유형 5가지
|No|메서드 참조 유형|예|같은 기능을 하는 람다|
|:---:|:---:|:---:|:---:|
|1|**정적 메서드를 가리키는 메서드 참조**|Integer::parseInt|str -> Integer.parseInt(str)|
|2|**인스턴스 메서드를 참조하는 유형 <br/> 수신 객체를 특정하는 한정적 인스턴스 메서드 참조**|Instant.now()::isAfter|Instant then = Instant.now(); <br/> t -> then.isAfter(t)|
|3|**인스턴스 메서드를 참조하는 유형 <br/> 수신 객체를 특정하지 않는 비한정적 인스턴스 메서드 참조**|String::toLowerCase|str -> str.toLowerCase()|
|4|**클래스 생성자를 가리키는 메서드 레퍼런스**|TreeMap<K, V>::new|() -> new TreeMap<K,V>()|
|5|**배열 생성자를 가리키는 메서드 레퍼런스**|int[]::new|len -> new int[len]|
