# Item 31
## 한정적 와일드카드를 사용해 API 유연성을 높여라

- 매개변수화 타입은 불공변 이다 (서로 다른 타입 Type1, Type2가 있을 때 List<Type1>은 List<Type2>의 하위 타입도 상위 타입도 아니다)
  - 예를들어 List<String>은 List<Object>의 하위타입이 아니라는 뜻
  - 하지만 때론 불공변 방식보다 유연한 무언가가 필요! (특히나 API를 설계할 때는 유연한 API가 필요할 때가 있다)
 

```java
// 다음은 Stack 클래스에 pushAll() 이라는 일련의 원소를 스택에 넣는 메서드를 추가한 상태
public class Stack<E> {
    private List<E> list = new ArrayList<>();
    
    public void pushAll(Iterable<E> src) {
        for(E e : src) {
            push(e);
        }
    }
}

// Iterable의 원소 타입이 스택의 원소 타입과 일치하면 잘 작동하지만
// 타입이 다를 경우 매개변수화 타입이 불공변이기 때문에 컴파일 에러가 발생한다!
// 예를들면 논리적으로는 Number을 담을 수 있는 스택이 Integer도 담을 수 있어야 할 것 같지만
// 제네릭은 불공변이기 때문에 Integer가 Number의 하위타입인 것은 전혀 상관이 없다
@Test
void pushAll() {
    Stack<Number> stack = new Stack<>();
    List<Integer> intList = Arrays.asList(1, 2, 3, 4);
    
    // Compile Error
    stack.pushAll(intList);
}

// 한정적 와일드카드를 통해 해결 가능!
// pushAll() 입력 매개변수 타입은 E의 Iterable이 아니라
// E의 하위타입의 Iterable 이라고 변경해주어 타입을 안전하게 사용 가능
public class Stack<E> {
    private List<E> list = new ArrayList<>();

    public void pushAll(Iterable<? extends E> src) {
        for(E e : src) {
            push(e);
        }
    }
}
```
- 위의 코드에서 Iterable<? extends E> 의 의미는 pushAll()의 입력 매개변수 타입은 E의 Iterable이 아니라 E의 하위 타입의 Iterable 이라는 뜻!

이번에는 정반대의 상황을 살펴보자!

```java
// pushAll과 대비되는 popAll 메서드 (스택 안의 원소를 주어진 컬렉션으로 옮겨닮는 역할)
// 해당 코드 역시 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 말끔히 컴파일되고 문제없이 동작한다
public void popAll(Collection<E> dst) {
    while (!isEmpty()) {
        dst.add(pop());
    }    
}


// 하지만 역시 Stack<Number>의 원소를 Object용 컬렉션으로 옮기려 한다고 하면
// 결과는 앞에서 보았던것 처럼 Compile Error가 발생한다
@Test
void popAll() {
    Stack<Number> stack = new Stack();
    List<Integer> intList = Arrays.asList(1, 2, 3, 4);
    stack.pushAll(intList);
    
    List<Object> objList = new ArrayList<>();
    List<Number> numberList = new ArrayList<>();
    
    // Compile Error
    stack.popAll(objList);
    stack.popAll(numberList);
    
    assertThat(numberList).contains(1, 2, 3, 4);
}
// 이것 역시 원인은 제네릭이 불공변이기 때문


// E의 Collection이 아니라 E의 상위 타입의 Collection 이라고 변경해주면 해결!
public void popAll(Collection<? super E> dst) {
    dst.addAll(list);
    list.clear();
}
```
- 위의 코드에서 Iterable<? super E> 의 의미는 popAll()의 입력 매개변수 타입은 E의 Collection이 아니라 E의 상위 타입의 Collection 이라는 뜻!

### PECS 공식!

- PECS를 풀어서 설명하면 매개변수화 타입 T가 생산자라면 <? extends T>를 사용하고, 소비자라면 <? super T>를 사용한다는 의미
- Producer Extends Consumer Super



```
# 핵심!

 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리를 작성한다면 와일드카드 타입을
적절히 사용해줘야 한다. PECS 공식을 기억하자!
```
