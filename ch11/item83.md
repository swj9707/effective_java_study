# Item 83

## 지연 초기화는 신중히 사용하라

### 지연초기화란?
> - 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법.<br>
> - 주로 최적화 용도로 사용.<br>
> - 클래스와 인스턴스 초기화 때 발생 하는 위험한 순환문제를 해결하는 효과도 있음.
<br>

### 지연초기화는 양날의 검이다.
- 지연초기화를 사용한다면 인스턴스 생성 시의 초기화 비용은 줄지만 그 대신 필드에 접근하는 비용은 커진다.
- 지연 초기화가 실제로는 성능을 느려지게 할 수 있다.
> - 초기화가 이뤄지는 비율은 어떠한가.<br>
> - 실제 초기화에 비용이 얼마나 드는가.<br>
> - 초기화된 각 필드를 얼마나 빈번히 호출되는가.
<br>

따라서, 성능을 생각한다면 지연초기화는.
- 해당 클래스의 인스터스 중 그 필드를 사용하는 인스턴스의 비율이 낮고
- 그 필드를 초기화하는 비용이 크다면 사용하자.
- 즉. 초기화 시점 해당 필드 or 클래스가 필요할 때 시키는 것이다!
- 그러나, 성능을 측정해보기 전까지는 어떤 것이 더 나은지 확신하지 못함.

또한, 멀티스레드 환경에서는 지연초기화 하는 필드를 둘 이상의 스레드가 공유한다면
어떠한 형태로든 반드시 동기화를 해야함!

#### 예시 1, 일반적인 초기화 방법
```java
public class TestClass {
    private final Member member = createMember();

    private Member createMember() {
        return new Member("KJJ");
    }
}
```
- 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.
<br>

#### 예시 2, 인스턴스 필드의 지연초기화
``` java
public class TestClass {
    private Member member;

    public synchronized  Member getMember(){
        if(member==null){
            member = createMember();
        }
        return member;
    }
    private Member createMember() {
        return new Member("KJJ");
    }
}
```
- 지연 초기화가 초기화 순환성(initialization circularity) 을 깨뜨릴 것 같으면 synchronized 를 단 접근자를 사용하자.
<br>

#### 예시 3, 정적 필드용 지연 초기화 홀더 클래스 관용구
``` java
public class TestClass {

    static final private Member member = createMember();

    private static Member createMember() {
        System.out.println("init");
        return new Member("KJJ");
    }

    public static Member getMember(){
        return TestClass.member;
    }
}
```
- 해당 방식은 클래스는 클래스가 처음 쓰일 때 비로소 초기화 된다는 특성을 이용한 관용구다.
- 이러한 방식은 getMember 메서드가 필드에 접근하면서 동기화를 전혀 하지않으니 성능이 느려질 거리가 전혀 없다.
<br>

#### 예시 4, 인스턴스 필드 지연 초기화용 이중검사 관용구
``` java
public class TestClass {

    private volatile Member member;

    private Member createMember() {
        return new Member("KJJ");
    }

    public Member getMember() {
        Member result = member;
        if(result!=null){
            return result;
        }
        synchronized (this){
            if(member == null){
                member = createMember();
            }
            return member;
        }
    }
}
```
- 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라.
- 해당 관용구는 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.
- 필드가 초기화 된 후로는 동기화 하지 않으므로 해당 필드는 반드시 volatile로 선언한다.
- result 지역변수는 필드가 이미 초기화된 상황에서는 이 필드를 딱 한번만 읽도록 보장하는 역할을 한다
<br>
