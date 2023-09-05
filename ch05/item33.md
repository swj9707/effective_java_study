# Item33

## 타입 안전 이종 컨테이너를 고려하라

- 타입 안전 이종 컨테이너 패턴
  - 컨테이너 대신 키를 매개변수화 한 다음 컨테이너에 값을 넣거나 뺄 때 매개변수화 한 키를 함께 제공하면, 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해주는 패턴

```java
// Typesafe heterogeneous container pattern (Pages 151-4)
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }

//    // Achieving runtime type safety with a dynamic cast
//    public <T> void putFavorite(Class<T> type, T instance) {
//        favorites.put(Objects.requireNonNull(type), type.cast(instance));
//    }

    public static void main(String[] args) {
        Favorites f = new Favorites();
        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);
        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite(Class.class);
        System.out.printf("%s %x %s%n", favoriteString,
                favoriteInteger, favoriteClass.getName());
    }
}
```
- 객체가 가질 수 있는 값을 타입과 함께 저장하는 구조
- String 타입을 요청했는 데 Integer를 리턴하는 일은 없다. 

```java
public <T> T getFavorite(Class<T> type) {
    return type.cast(favorites.get(type));
}
```
- cast 메소드의 반환 타입은 Class 객체의 타입 매개변수와 같다. 
- cast 메소드는 Class 클래스가 제네릭이라는 이점을 활용한다. 
- getFavorite 메소드에서 T 로 비검사 형변환 하는 과정 없이도 Favorites 를 타입 세이프하게 만들어준다. 

- 만약 클라이언트가 Class 객체를 제네릭이 아닌 Raw 타입으로 넘기면 타입 안정성이 깨지게 된다. 
- 컴파일 시 비검사 경고가 뜬다. 
- 타입 안정성을 확보하고 싶다면 값 인수로 주어진 타입이 키로 명시한 타입과 같은 지 확인하면 된다. 

```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), instance);
}
```

위와 같은 형태로 구현 된 자바 컬렉션 예시들
- checkedSet, checkedList, checkedMap

## 실체화 불가 타입에는 사용할 수 없다. 
- String, String[] 은 사용할 수 있지만, List<String> 은 사용할 수 없다. 
- List<String> 용 Class 객체를 얻을 수 없기 때문이다. 
  - List<String>, List<Integer> 는 List.class 라는 Class 객체를 공유한다. 
  - 같은 타입의 객체 참조를 반환한다면 객체 내부에서 이를 구별 할 방법이 없어진다. 

## 한정적 타입 토큰
- 한정적 타입 매개변수나 한정적 와일드카드를 사용해서 표현 가능한 타입을 제한하는 타입 토큰
- 토큰으로 명시한 타입의 어노테이션이 대상 요소에 달려있다면 그 어노테이션을 반환하고 그렇지 않다면 null을 반환

```java
public class PrintAnnotation {
    static Annotation getAnnotation(AnnotatedElement element,
                                    String annotationTypeName) {
        Class<?> annotationType = null; // Unbounded type token
        try {
            annotationType = Class.forName(annotationTypeName);
        } catch (Exception ex) {
            throw new IllegalArgumentException(ex);
        }
        return element.getAnnotation(
                annotationType.asSubclass(Annotation.class));
    }

    // Test program to print named annotation of named class
    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.out.println(
                "Usage: java PrintAnnotation <class> <annotation>");
            System.exit(1);
        }
        String className = args[0];
        String annotationTypeName = args[1]; 
        Class<?> klass = Class.forName(className);
        System.out.println(getAnnotation(klass, annotationTypeName));
    }
}
```
- asSubclass : 형 변환을 안전하게, 동적으로 수행 해주는 메소드. 
- 호출 된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환
- 실패 시 ClassCaseException 을 던진다. 

## 요약
- 컬렉션 API로 대표되는 일반적인 제네릭형태는 한 컨테이너가 다룰수 있는 매개변수의 수가 고정적이다.
- 하지만 컨테이너자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.
- 타입 이종 컨테이너는 Class를 키로 사용하며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다.
- 또한 직접 구현한 키 타입도 쓸 수 있다. 예컨데 DB의 행(컨테이너)을 표현한 DatabaseRow 타입에는 제네릭 타입인 Column<T>를 사용할 수 있다.
- 하지만 타입이종 컨테이너를 사용하는데 제약이 있으니 이런 제약들을 주의해서 사용하자.