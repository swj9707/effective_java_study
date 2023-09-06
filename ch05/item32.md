# Item32

## 제네릭과 가변인수를 함께 쓸 때는 신중하라

```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // Heap pollution
    String s = stringLists[0].get(0); // ClassCastException
}
```
- 가변 인수는 메소드에 넘기는 인수의 갯수를 클라이언트가 조절할 수 있게 해준다. 
- 가변 인수 메소드를 호출하면 가변인수를 담기 위한 배열이 자동으로 만들어진다
- 내부로 감춰야 했을 이 배열이 클라이언트에게 노출되는 문제가 생긴다. 
- 그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다. 

- 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다. 
- 이렇게 다른 타입 객체를 참조하는 상황에서는 컴파일러가 자동 생성한 형변환이 실패할 수 있다. 
- 즉 제네릭 타입 시스템이 약속한 타입 안전성을 보장할 수 없게 된다. 

## 제네릭 varargs 매개변수를 허용하는 이유?
- 제네릭 배열을 프로그래머가 직접 생성하는 것은 허용하지 않는다. 
- 하지만 제네릭 varargs 매개변수를 받는 메서드를 선언할 수 있는 이유는 실무에서 유용하기 때문이다. 

```java
Arrays.asList(T... a);
Collections.addAll(Collection<? super T> c, T... elements);
EnumSet.of(E first, E... rest);
```

## @SafeVarargs
- @SuppressWarnings("unchecked") 어노테이션을 통해 제네릭 가변인수 메소드의 경고를 숨겨줄 수 있지만 반복 작업이고 정작 진짜 중요한 경고를 숨기는 결과로 돌아올 수 있다. 
- @SafeVarargs 어노테이션을 통해 해당 메소드의 타입 세이프를 보장시켜줄 수 있지만 무조건 사용해서는 안된다. 

- 가변 인수 메소드가 안전한 지 확신하는 방법
  - 가변인수 메소드를 호출할 때 varargs 매개변수를 담는 제네릭 배열이 생성된다. 
  - 이 배열에 메소드가 아무런 값도 저장하지 않아야 한다. (매개변수들을 덮어쓰지 않아야 한다.)
  - 그 배열의 참조가 밖으로 노출되지 않아야 한다. (신뢰할 수 없는 코드가 배열에 접근할 수 없어야 한다. )
  - 즉 순수하게 인수들을 메소드에 전달해주는 역할만 해야 한다. 
  ```java
   static <T> T[] toArray(T... args) {
        return args;
    }
  ```
  - 메소드의 인수를 그대로 리턴하는 케이스
  - 메소드가 반환하는 배열의 타입은 메소드가 인수를 넘기는 컴파일 타임에 결정된다. 
  - 그 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다. 
  - 메소드를 호출한 콜 스택까지 힙 오염을 전이시킬 수 있다. 
  
  ```java
    static <T> T[] pickTwo(T a, T b, T c) {
        switch(ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // Can't get here
    }

  ```
  - toArray는 가변인수를 받지만, pickTwo 에 어떤 타입을 넘겨도 담길 수 있는 구체적인 타입인 Object[] 를 반환한다. 
  
  ```java
   public static void main(String[] args) {
        String[] attributes = pickTwo("Good", "Fast", "Cheap");
        System.out.println(Arrays.toString(attributes));
    }
  ```
  - String[] 에 아무 문제없이 담기지만, 실제로 코드를 실행해보면 컴파일 결과와는 다르게 ClassCaseException 이 발생한다. 
  - String[] attributes 에 결과를 담기 위해 컴파일러가 이 코드를 읽고 String[] 로 반환하는 코드를 생성한다. 
  - 결론
    - 아무리 몇단계 떨어져서 작동하는 varargs 매개변수 메소드라고 해도 컴파일 타임에서 코드를 읽어들이는 상황에 따라서 이런 형태의 메소드가 에러를 발생시킬 수 있다. 
    - 제네릭 varargs 매개변수 배열에 다른 메소드가 접근하도록 허용하면 안전하지 않다.
 
  ```java
  @SafeVarargs
  static <T> List<T> flatten(List<? extends T>... lists) {
     List<T> result = new ArrayList<>();
     for (List<? extends T> list : lists)
         result.addAll(list);
     return result;
  }
  ```
  - 안전하게 제네릭 varargs 메소드를 사용하는 예시. 
  - 매개변수는 매개변수의 역할대로 메소드에 값을 전달하는 역할. 
  - 새로운 List에 해당 값들을 담아서 List로 반환
  - @SafeVarargs 어노테이션이 달려있으므로 어느 쪽에서도 경고를 울리진 않는다. 
  
  ```java
  static <T> List<T> flatten(List<List<? extends T>> lists) {
     List<T> result = new ArrayList<>();
     for (List<? extends T> list : lists)
         result.addAll(list);
     return result;
  }
  ```
  - @SafeVarargs 를 쓰지않고 리스트를 연속으로 사용해서 가변 인수를 구현 한 예시
  ```java
  List<Integer> flatList = flatten(List.of(List.of(1,2), List.of(3,4,5), List.of(6,7)));
  ```
  - List.of 자체에 @SafeVarargs 어노테이션이 붙어있다. 
  - 단점 : 코드가 지저분해지고 속도가 느려질 수 있다.


  ```java
  static <T> List<T> pickTwo(T a, T b, T c) {
     switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0: return List.of(a, b);
        case 1: return List.of(a, c);
        case 2: return List.of(b, c);
    }
    throw new AssertionError();
  }
  ```
  - varargs 메소드를 안전하게 작성하는 게 불가능한 상황
  - 