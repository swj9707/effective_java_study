# Item 21

## 인터페이스는 구현하는 쪽을 생각해 설계하라
> 구현하는 쪽 : 인터페이스를 활용하는 구현체를 만드는 개발자(클라이언트)라고 이해하면 편함.
<br>

### 인터페이스의 default mehtod란?
> - java 8 이전에는 하위 호환성을 지키기 위해 인터페이스에 메서드를 추가하는 것이 불가능 했다.
> - 인터페이스에 메서드를 추가 했을 때 구현체에 존재할 가능성이 낮고, 대부분 컴파일 오류가 발생함.
> - java 8 이후 default method가 추가되면서 새로운 인터페이스를 구현하지 않아 발생하는 컴파일 에러를 잡을 수 있게 됨.
> - 그러나 default method를 추가한다 해도 기존의 구현체들과 매끄럽게 연동된다는 보장이 없다.
<br>

### 예시 1, default method를 가진 간단한 인터페이스
```java
interface Inter1 {

	default void defaultMethod() {
		System.out.println("인터페이스1의 defaultMethod");
	}
}

interface Inter2 {

	default void defaultMethod() {
		System.out.println("인터페이스2의 defaultMethod");
	}
}

interface Inter3 extends Inter1 {

	default void defaultMethod() {
		System.out.println("인터페이스3의 defaultMethod, 인터페이스 3을 확장함.");
	}
}
```
<br>

### 예시 2, 예시 1의 인터페이스를 구현한 클래스
```java
class Ex1 implements Inter1 {

  // 명시 안해도 인터페이스의 default method가 작동하긴 함. 
  void someMethod() {
  // 인터페이스의 디폴트 메서드를 직접 호출할 수 있음.
  defaultMethod();
}
	
}

class Ex2 implements Inter1 {
	
	@Override
	public void defaultMethod() {
		System.out.println("defaultMethod를 오버라이딩 한 경우.");
	}
}

class Ex3 implements Inter1, Inter2 {
    // 다이아몬드 문제가 발생하므로 반드시 해당 메서드를 오버라이딩해야 함
    @Override
    public void defaultMethod() {
   // 자신이 원하는 인터페이스의 디폴트 메서드를 선택하여 호출해야 함
    	Inter1.super.defaultMethod(); // 또는 Inter2.super.defaultMethod();
    }
}

class Ex4 implements Inter3 {
    ~~~
}
```
<br>

### default 인터페이스를 사용할 때 주의 사항
- 명시적인 호출
  - 인터페이스 내에서 기본 구현을 제공하기 위함이다.
- 메서드 오버라이딩
  - 클래스에서 인터페이스의 default method를 재정의 가능하다.
  - 이 때 호출되는 method는 인터페이스의 메서드가 아닌 구현체의 method가 된다.  
- 다이아몬드 문제
  - class가 여러 인터페이스를 구현할 때, 각 인터페이스에서 같은 default method가 있을 수 있다.
  - 클래스에서 해당 default method를 오버라이딩하여 명시적으로 어떤 것을 호출할 지 정해야 한다. 
- 인터페이스의 확장성
  - default method를 남용하면 인터페이스가 너무 복잡해 질 수 있다.
  - 구현체에서 의도하지 않은 동작 또는 충돌이 발생 할 수 있다.
