# item29. 이왕이면 제네릭 타입으로 만들라

## 제네릭타입
- 제네릭 타입은 타입을 파라미터로 가지는 클래스와 인터페이스를 말한다.
- 제네릭 타입은 클래스 또는 인터페이스 이름 뒤에 "<>" 부호가 붙고, 사이에 타입 파라미터가 위치한다.
- JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이다.
- 하지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다. 

## 예시 코드
- item 7 에서 다룬 단순한 스택 코드
```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack(){
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e){
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop(){
        if(size == 0){
            throw new EmptyStackException();
        }
        
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
    
    public boolean isEmpty(){
        return size == 0;
    }
    
    private void ensureCapacity(){
        if(elements.length == size){
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```
- 이 클래스를 제네릭으로 바꾼다고 해도 현재 버전을 사용하는 클라이언트는 아무런 해가 없다.
- 오히러 지금 상태에서는 스택에서 꺼낸 객체의 형변환 때문에 런타임 오류가 날 수 있다.

## 제네릭 클래스로 만들기
### 1. 클래스 선언에 타입 매개 변수를 추가하자.
- 스택이 담을 원소의 타입 하나만 추가하면 된다. 타입 이름으로 보통 E를 사용한다. (Item 68)

### 2. 코드에 쓰인 Object 를 적절한 타입 매개변수로 바꾼다.
```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
 
    public Stack(){
        elements = new E[DEFAULT_INITIAL_CAPACITY];
    }
 
    public void push(E e){
        ensureCapacity();
        elements[size++] = e;
    }
 
    public E pop(){
        if(size == 0){
            throw new EmptyStackException();
        }
 
        E result = elements[--size];
        elements[size] = null;
        return result;
    }
    ... // isEmpty와 ensureCapacity 메서드는 그대로다.
}
```
- 이 단계에서 대체로 오류 발생
- Item 28에서 나온 것처럼, E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다.
- 따라서 배열을 사용하는 코드를 제네릭으로 만들려 할때는 항상 이문제가 발생할 것이다.

### 첫 번째 해결 방안
- **제네릭 배열 생성을 금지하는 제약을 대놓고 우회**하는 방법이다.
1. Object 배열을 생성한 다음, 제네릭으로 형변환한다. 
```java
  public Stack(){
    elements = (E[]) new Object [DEFAULT_INITIAL_CAPACITY]}
  }
```
컴파일러는 이제 오류 대신 (일반적으로) 타입이 안전하지 않다며 경고를 보낸다.
컴파일러는 해당 프로그램의 타입이 안전한지 증명할 방법이 없기 때문이다. 

2. 때문에, 이런 비검사 형변환이 프로그램의 타입 안전성을 해치지 않음을 스스로 확인해야 한다. 
위 코드를 생각해 보자.
- 문제의 배열 elements는 private 필드에 저장
- 클라이언트로 반환되거나, 다른 메서드에 전달되는 일이 전혀 없음
- push 메서드를 통해 배열에 저장되는 원소의 타입은 항상 E

결론은, **이 비검사 형변환은 확실히 안전하다**!

3. 비검사 형변환이 안전하므로, 범위를 최소로 좁혀 @SuppressWarnings 애너테이션으로 해당 경고를 숨긴다(Item 27)   

또는, 이 예에서 생성자가 비검사 배열 생성 말고는 하는 일이 없으니, 생성자 전체에서 경고를 숨기는 것도 방법이다. 
```java
// 배열 elements 는 push(E)로 넘어온 E 인스턴스만 담는다.
// 따라서 타입 안전성을 보장하지만,
// 이 배열의 런타임 타입은 E[]가 아닌 Object[]다!
@SuppressWarnings("unchecked")
public Stack(){
            elements = (E[]) new Object [DEFAULT_INITIAL_CAPACITY]}
  }
```
애너테이션 추가 시, Stack은 깔끔히 컴파일 되고,
명시적으로 형변환 하지 않아도 ClassCaseException 걱정 없이 사용할 수 있다.


### 두 번째 해결 방안
- elements 필드의 타입을 **E[]에서 Object[]로 바꾸자**

```java
 public class Stack<E> {
        private Object[] elements;
    ... // 중략
```

첫 번째와는 다른 오류가 뜬다. 
```java
E result = elements[--size];
```
호환성이 없는 형이라는 오류 발생   

이것을 해결하기 위해 코드를 수정한다.
```java
E result = (E) elements[--size];
```
배열이 반환한 원소를 E로 형변환 하면 오류 대신 경고가 뜬다.
- E는 실체화 불가 타입임으로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다.
- 따라서 이번에도, 개발자는 이를 직접 증명하고, 경고를 숨길 수 있다.

```java
 public E pop(){
            // 비검사 경고를 적절히 숨긴다.
      if(size == 0){
           throw new EmptyStackException();
      }
 
      @SuppressWarnings("unchecked")
      E result = (E) elements[--size];
 
      elements[size] = null; // 다 쓴 참조 해제
      return result;
}
```
pop 메서드 전체에서 경고를 숨기지 말고, 비검사 형변환을 수행하는 할당물에서만 숨기도록 한다(Item 27)


|  | 첫 번째 방법                                                                                       | 두 번째 방법                             |
|----|-----------------------------------------------------------------------------------------------|-------------------------------------|
|    | 배열 타입을 E[]으로                                                                                  | 배열 타입을 Object[]로                    |
| 장점 | * 가독성이 좋음<br/> * 오직 E 인스턴스만 배열로 받음을 선언<br/> * 코드가 더 짧음<br/> * 형변환을 배열 생성시 단 한번만 하면 됨<br/>* 현업에서 더 선호 | * 힙 오염이 발생하지 않아 이를 더 선호하는 프로그래머들이 있음 |
| 단점 | * E가 Object 가 아니므로 배열의 런타임 타입이 컴파일타임 타입과 달라 힙오염(Item 32) 발생                                   | * 배열을 원소를 읽을 때마다 형변환 필요 |

> ***힙 오염(Heap pollution)*** 은 단어 그대로 JVM의 힙(Heap) 메모리 영역에 저장되어있는 특정 변수(객체)가 불량 데이터를 참조함으로써, 만일 힙에서 데이터를 가져오려고 할때 얘기치 못한 런타임 에러가 발생할 수 있는 오염 상태를 일컫는다.
> 간단히 말해, Java generic type에서 ***변수화 된 타입이 가리키는 오브젝트의 타입이 해당 변수화 된 타입의 타입과 다를 때***를 의미한다.

## 배열보다는 리스트를 우선하라?
- stack 예시는 앞의 item 28의 내용과는 모순된다
- 사실 제네릭 타입안에서 리스트를 사용하는게 항상 가능하지 않고, 더 좋은 것은 아니다
- 자바가 리스트를 기본 타입으로 제공하지 않으므로, ArrayList같은 제네릭 타입도 결국 기본타입인 배열을 사용해 구현해야 한다.
- 또한 HashMap 같은 제네릭 타입은 성능 향상을 목적으로 배열을 사용하기도 한다.

## 핵심정리
### 클라이언트에서 직접 형변환 해야 하는 타입보다 제네릭 타입이 더 안전하고 더 쓰기 편하다!
