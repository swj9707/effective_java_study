# Item 65
## 리플렉션보다는 인터페이스를 사용하라

### 리플렉션?
리플렉션 기능을 이용하면 임의의 클래스에 접근 가능 (생성자, 메서드, 필드 등을 가져오고 조작 가능)
- 단점
  1. 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다
  2. 리플렉션을 이용하면 코드가 지저분하고 장황해진다
  3. 성능이 떨어진다

 리플렉션을 써야하는 복잡한 애플리케이션에서도 (코드 분석 도구, 의존관계 주입 프레임워크 등)
 리플렉션 사용을 점차 줄이고 있다.
```java
class Cat {
    public void introduceMyself() {
        System.out.println("고양이");
    }
}

public class Reflection {
    public static void main(String[] args) {
        Object 고양이 = new Cat();
        // 컴파일 에러
        // 고양이.introduceMyself();
    }
}
```
위 처럼 Cat이라는 구체적인 타입을 알지 못할 때 메서드를 실행시킬 수 있는 것이 리플렉션이다
```java
public class Reflection {
    public static void main(String[] args)
            throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Object 고양이 = new Cat();
        Class 고양이_클래스 = 고양이.getClass();
        Method[] 고양이_클래스_Methods = 고양이_클래스.getMethods();
        System.out.println(Arrays.toString(고양이_클래스_Methods));

        Method 고양이_함수1 = 고양이_클래스.getMethod("introduceMyself"); // NoSuchMethodException 에러 
        고양이_함수1.invoke(고양이, null); // IllegalAccessException
    }
}
```

### 하지만
동적으로 클래스를 생성하고 접근해야 한다면 사용할 이유는 분명하다 (특수한 목적으로 만든 기능이기 때문)
- 어쩔수 없이 리플렉션을 써야한다면
  1. 리플렉션은 인터페이스 생성시에만 사용
  2. 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하기
 
```java
public interface Crew {
    void setName(String name);
    void sayMyCourseWithName();
}
```

```java
public class Backend implements Crew {
    private String name;

    @Override
    public void sayMyCourseWithName() {
        System.out.println(name + "은 Backend 입니다.");
    }

    @Override
    public void setName(String name) {
        this.name = name;
    }
}
```

```java
public class Frontend implements Crew {
    private String name;

    @Override
    public void sayMyCourseWithName() {
        System.out.println(name + "은 Frontend 입니다.");
    }

    @Override
    public void setName(String name) {
        this.name = name;
    }
}
```

```java
public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        Class crew = null;
        crew = (Class<? extends Crew>) Class.forName(args[0]);
        

        Constructor crewConstructor = null;
        crewConstructor = crew.getDeclaredConstructor();


        Crew eden = null;
        eden = crewConstructor.newInstance(); // new Backend() or new Frontend()

        eden.setName(args[1]);
        eden.sayMyCourseWithName();
    }
}
```


```java
핵심!
리플렉션은 특수 시스템을 개발할 때 필요한 강력한 기능이지만, 단점도 많다. 컴파일 타임에서는
알 수 없는 클래스를 사용하는 프로그램을 작성한다면 리플렉션을 사용해야 할 것이다.
단, 되도록이면 객체 생성에만 사용하고, 만든 객체를 이용할 때는 인터페이스나 상위 클래스로
형변환해서 사용해야 한다.
```
