# Item 20
## 추상 클래스보다는 인터페이스를 우선하라

- 자바가 제공하는 다중 구현 메커니즘은 인터페이스와 추상클래스 이렇게 두가지
  - 추상클래스가 정의한 타입을 구현하는 클래스는 반드시 추상 클래스의 하위 클래스가 되어야 함
  - 인터페이스가 정의한 타입을 구현하는 클래스는 어떤 다른 클래스를 상속했든 같은 타입으로 취급된다
 
- 인터페이스는 기존 클래스에도 손쉽게 구현해넣을 수 있다
  - implement 구문을 추가하고 인터페이스가 요구하는 메서드만 추가하면 됨
- 추상클래스는 기존 클래스에 끼워넣기는 일반적으로 어렵다
  - 두 클래스가 같은 추상 클래스를 확장하길 원한다면, 그 추상클래스는 계층구조상 두 클래스의 공통 조상이어야한다
  - 이 방식은 클래스 계층구조에 커다란 혼란을 일으킨다

- 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다
  - 믹스인이란 클래스에 원래의 주된타입 외에도 특정 선택정 행위를 제공한다고 선언하는 효과를 준다
- 추상클래스는 믹스인을 정의할 수 없다
  - 앞서 설명과 같이, 기존 클래스에 덧씌울 수 없기 때문
 
- 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.
  - 타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있다, 하지만 현실에서는 계층 구분이 모호한 경우가 많음

```java
// 예를들어 가수와 작곡가 인터페이스가 있다
public interface Singer {
  AudioClip Sing(Song s);
}

public interface SongWriter {
  Song compose(int chartPosition);
}


// 현실에서는 작곡가 이면서 가수인 사람이 있다
public interface SingerSongWriter extends Singer, SongWriter {
  AudioClip Sing(Song s);
  Song compose(int chartPosition);
}
```

- 인터페이스의 메서드 중 구현 방법이 명백한 것이있다면, 그 구현을 디폴트 메서드로 제공해라
  - Java 8부터 제공하는 디폴트 메서드를 사용하면 인터페이스에도 메서드를 구현할 수 있다
  - equals나 hashcode 같은 Object의 메서드는 디폴트 메서드로 제공해서는 안된다
  - 인터페이스는 인스턴스 필드를 가질 수 없다
  - public이 아닌 정적 멤버도 가질 수 없다
  - 여러분이 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다

- 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공할 수 있다 (템플릿 메서드 패턴)
  - 이 방법을 사용하면 인터페이스와 추상클래스의 장점을 모두 취할 수 있다
  - 인터페이스로 타입을 정의, 필요하면 디폴트 메서드 몇개도 함께 제공
  - 골격 구현 클래스는 나머지 메서드들까지 구현한다
  - 이렇게 하면 단순히 골격 구현을 확장하는 것 만으로 이 인터페이스를 구현하는데 필요한 일이 대부분 완료된다
  - 관례상 인터페이스명 앞에 Abstract를 붙여 골격 구현 클래스의 이름을 짓는다
  - ex) Collection 프레임워크의 골격 구현 클래스의 이름은 각각 AbstractCollection, AbstractSet, AbstractList ...

아래의 코드는 완벽히 동작하는 List 구현체를 반환하는 정적 팩터리 메서드로, AbstractList 골격 구현으로 활용했다
```java
static List<Integer> intArrayAsList(int[] a) {
  Object.requireNonNull(a);

  return new AbstractList<>() {
    @Override
    public Integer get(int i) {
      return a[i];
    }

    @Override
    public Integer set(int i, Integer val) {
      int oldVal = a[i];
      a[i] = val;
      return oldVal;
    }

    @Override
    public int size() {
      return a.length;
    }
  }
}
```
List 구현체가 제공하는 기능들을 생각하면 이 코드는 골격 구현의 힘을 잘 보여주는 예이다.


```
# 핵심!

 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 복잡한 인터페이스라면 구현하는
수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자. 골격 구현은 '가능한 한'
인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하는 것이 좋다
```
