# Item 16

## public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

### 인스턴스 필드란?
> - 객체지향 프로그래밍에서 클래스에 속한 변수를 말함. <br>
> - 클래스는 객체의 특성을 정의하는 템플릿.<br>
> - 클래스의 인스턴스는 이러한 특성을 가진 실제 객체. <br>
> - 인스턴스 필드는 클래스 내부에 선언되는 변수로, 해당 클래스의 인스턴스 마다 독립적인 값을 가질 수 있다.<br>
> - 즉, 객체마다 다른 상태를 유지할 수 있다.