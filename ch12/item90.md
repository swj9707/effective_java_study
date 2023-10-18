# Item 90

## 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라
- Serializable을 구현하기로 결정한 순간, 언어의 생성자 이외의 방법으로도 인스턴스를 생성할 수 있게 된다.
- 버그 및 보안 문제가 일어날 가능성이 커진다.
- 직렬화 프록시 패턴 (Serialization Proxy Pattern)을 이용하면 이러한 위험을 크게 줄여줄 수 있다.
- <br>

### 직렬화 프록시 패턴이란?
> - 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해준다.<br>
> - 필드들을 final로 선언해도 되므로 Period 클래스를 진정한 불변으로 만들 수 있다.<br>
> - 역직렬화때 유효성 검사를 하지 않아도 된다.
> - 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 작동한다.
<br>

#### 예시 1, 직렬화 프록시 패턴
```java
  public final class Period implements Serializable {
  
    private final Date start;
    private final Date end;
  
    public Period(Date start, Date end) {
      if (this.start.compareTo(this.end) > 0) {
        throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
      }
      this.start = new Date(start.getTime());
      this.end = new Date(end.getTime());
    }
  
    //자바의 직렬화 시스템이 바깥 클래스의 인스턴스 말고 SerializationProxy의 인스턴스를 반환하게 하는 역할
    //이 메서드 덕분에 직렬화 시스템은 바깥 클래스의 직렬화된 인스턴스를 생성해낼 수 없다
    //"프록시야 너가 대신 직렬화되라"
    private Object writeReplace() {
      return new SerializationProxy(this);
    }
  
    //불변식을 훼손하고자 하는 시도를 막을 수 있는 메서드
    //"Period 인스턴스로 역직렬화를 하려고해? 안돼"
    private void readObject(ObjectInputStream stream) throws InvalidObjectException {
      throw new InvalidObjectException("프록시가 필요합니다");
    }
  
    //바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스(Period의 직렬화 프록시)
    private static class SerializationProxy implements Serializable {
  
      private static final long serialVersionUID = 234098243823485285L;
  
      private final Date start;
      private final Date end;
  
      //생성자는 단 하나여야 하고, 바깥 클래스의 인스턴스를 매개변수로 받고 데이터를 복사해야 함
      SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
      }
  
      //역직렬화시 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해줌.
      //역직렬화는 불변식을 깨뜨릴 수 있다는 불안함이 있는데, 
      //이 메서드가 불변식을 깨뜨릴 위험이 적은 정상적인 방법(생성자, 정적 팩터리, 다른 메서드를 사용)으로 역직렬화된 인스턴스를 얻게 한다.
      //"역직렬화 하려면 이거로 대신해"
      private Object readResolve() {
        return new Period(start, end);
      }
    }
  }
  ```

### 직렬화 프록시 패턴의 한계?
- 클라이언트가 조건없이 확장할 수 있는 클래스에는 적용할 수 없다.<br>
- 객체그래프 순환이 있는 클래스에는 적용할 수 없다.<br>
- 방어적 복사보다 느리다.<br>


## 결론
- 확장할 수 없는 클래스라면 가능한 직렬화 프록시 패턴을 사용하자.


