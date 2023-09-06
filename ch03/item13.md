# Item 13
## Clone 재정의는 주의해서 진행해라.
-----
결론만 말하자면, 새로운 인터페이스를 만들 땐 Clonable 을 Extend 해서는 안되고, 새로운 클래스도 이를 구현해서는 안됀다. 
가능하면 생성자나 팩토리를 활용해서 복제 기능을 구현해야 한다. 
단, 배열은 예외.
## Clonable Interface
```java
public interface Cloneable {
}
```

위의 인터페이스는 Object의 protected 메소드 `clone` 의 동작 방식을 결정함.  
`Clonable` 을 구현한 클래스의 인스턴스에서 clone을 호출하면, 그 객체들의 필드를 하나하나 복사한 객체를 반환.   
구현하지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportException`을 던진다. 

인터페이스를 구현한다 -> 해당 클래스가 그 인터페이스에서 정의한 기능을 제공하고 선언한다는 의미.  
`Clonable`의 경우는 상위 클래스에서 정의 된 동작 방식을 변경하는 것이므로 따라하지 않는 것이 좋다. 
> 인터페이스를 구현한다는 의미는, 일반적으로 해당 클래스가 그 인터페이스에서 정의한 기능을 제공한다고 선언하는 행위인데, 인터페이스인 Cloneable는 상위 클래스에 정의된 protected 메서드의 동작 방식을 변경한 것이기 때문


실제로 구현 된 public clone 메소드를 통해 생성되는 객체는 복제가 제대로 이뤄질 것이라는 기대를 하지 않는 것이 좋다. -> 생성자를 쓰지 않고도 객체를 생성할 수 있게 된다. 

```java
public PhoneNumber clone() {
    try{
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

객체를 복사하거나 CloneNotSupportedException 을 던지거나 둘 중 하나를 선택하는 경우인데, 애초에 구현하지 않은 클래스에서 `clone` 메소드를 호출했을때 `CloneNotSupportedException` 이 호출된다고 언급했다시피, 위의 catch 코드는 일어날 수 없는 코드다. 

Clonable 인터페이스를 구현한 클래스가 불변 객체만 참조하는 경우는 상관없지만, 가변 객체를 참조하게 된다면 분명 복제 된 객체지만, 가변 객체이기 떄문에 생기는 문제가 발생할 수 있다. 
## Example
Stack을 Clonable 인터페이스를 가져와서 구현한 예시
```java
public class Stack implements Cloneable {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    public boolean isEmpty() {
        return size ==0;
    }

    // Clone method for class with references to mutable state
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }

    // Ensure space for at least one more element.
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
- 생성자를 통해 개겣를 생성한게 아니므로 같은 element 배열을 공유한다.
  - 여기서 추가적인 상식. 객체가 생성되면 힙메모리에 할당된다. 
  - 복제가 된다면, 새로운 메모리를 할당시킬까? 아니면 해당 메모리를 가리키는 데이터를 리턴할까?
    - Pointer 를 이해한다면, 이게 왜 문제가 되는 지 알 수 있음.  
- 한쪽에서 element 의 변경이 발생한다면 다른 한쪽에 영향이 가게 된다. 
  - 그래서 아래와 같은 방법을 쓰면 되지만, elements 필드가 final인 경우는 통하지 않는다.
    ```java
    @Override public Stack clone() {
        try {
            Stack result = (Stack) super.clone();
            result.elements = elements.clone();
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    ```
  - 왜? final 필드에는 새로운 값을 할당할 수 없기 때문이다. 
## Replace
- 이미 Clonable을 구현한 클래스를 확장한다면 어쩔 수 없지만, 그렇지 않다면 복사 생성자와 복사 팩토리를 사용한다. 
- 복사 생성자
  ```java
  public Element(Element element) {
    this.a = element.a;
    ...
  }
  ```
- 복사 팩토리
  ```java
  public static Element newInstance(Element element) {
    return new Element(element);
  }
  ```
- 배열은 왜 clone 메소드 방식이 가장 깔끔한가?
  - 배열 그자체는 class로 정의 되어있지 않기 때문