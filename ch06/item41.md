# item41. 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

## 마커 인터페이스 
- 자신을 구현하는 클래스가 특정 속성을 가짐을 나타내는 인터페이스
- 아무 메서드도 담지 있지 않음
- 예를 들어, Serializable, Cloneable 인터페이스 등이 있다

#### Serializable 인터페이스
```java
public interface Serializable {
}
```
-  Serializable은 자신을 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 쓸수 있다고(직렬화할 수 있다고) 알려준다.

## 마커 애너테이션 
- 멤버변수 없이 단순히 대상에 마킹하는 어노테이션
- 예를 들어, @Override

## 마커 인터페이스 vs 마커 애너테이션
### 마커 인터페이스가 더 나은 점
1. 마커 인터페이스는 이를 구현한 클래스의 인스턴스들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 않다.
   - 마커 인터페이스는 어엿한 타입이기 때문에, 마커 애너테이션을 사용했다면 런타임에야 발견될 **오류를 컴파일타임에 잡을 수 있다.**
2. 적용 대상을 더 정밀하게 지정할 수 있다.
    - 마크 애너테이션은 적용 대상( @Target )을 Element.Type으로 선언한 애너테이션은 모든 타입(클래스, 인터페이스, 열거 타입, 애너테이션)에 달 수 있지만, 부탁할 수 있는 타입을 더 세밀하게 제한할 수는 없다.
    - 반면, 마크 인터페이스는 그냥 **마킹하고 싶은 클래스에서만 그 인터페이스를 구현하면 마킹된 타입은 자동으로 그 인터페이스의 하위 타입임이 보장**된다.

### 마커 애너테이션이 더 나은 점
1. 거대한 애너테이션 시스템의 지원을 받는다.
    - 애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 쓰는 쪽이 일관성을 지키는 데 유리하다


## 마커 애너테이션 vs 마커 인터페이스, 언제 사용해야 할까?
- 클래스와 인터페이스 외의 프로그램 요소(모듈/패키지/필드/지역변수 등)에 마킹해야 할때, 애너테이션을 활발히 활용하는 프레임워크에서 사용할 때는 **애너테이션**을 사용한다.
-  클래스와 인터페이스에 마커를 적용해야 할때, "마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있다면" 마커 **인터페이스**를 사용해야 한다. 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일 타임에 오류를 잡아낼 수 있기 때문이다.


## 핵심 정리

- 마커 인터페이스와 마커 애너테이션은 각자의 쓰임이 있다.
- 새로 추가하는 메서드 없이 단지 타입 정의가 목적이라면? → 마커 인터페이스
- 클래스나 인터페이스 외의 프로그램 요소에 마킹해야 하거나, 애너테이션을 적극 활용하는 프레임워크의 일부로 마커를 편입시키고자 한다면? → 마커 애너테이션
- 적용 대상이 ElementType.TYPE 인 마커 애너테이션을 작성하고 있다면, 잠시 여유를 갖고 정말 애너테이션으로 구현하는게 좋은지, 혹은 마커 인터페이스가 낫지는 않을지 곰곰히 생각해보자.
