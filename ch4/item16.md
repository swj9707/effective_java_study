# Item 16

## public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

### 인스턴스 필드란?
> - 객체지향 프로그래밍에서 클래스에 속한 변수를 말함. <br>
> - 클래스는 객체의 특성을 정의하는 템플릿.<br>
> - 클래스의 인스턴스는 이러한 특성을 가진 실제 객체. <br>
> - 인스턴스 필드는 클래스 내부에 선언되는 변수로, 해당 클래스의 인스턴스 마다 독립적인 값을 가질 수 있다.<br>
> - 즉, 객체마다 다른 상태를 유지할 수 있다.

### 예시1
```java
class Animal {
  public String tiger;
  public String lion;
}
```
- animal 클래스는 인스턴스 필드를 모아둔 것 이외에는 목적이 없다.
- 이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
- API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
- 불변식을 보장할 수 없다.
- 외부에서 필드에 접근할 때 부수 작업을 수행할 수 없다.
- 객체 지향적이지 않다.

### 예시2
```java
class Animal {
  private String tiger;
  private String lion;

  public Animal(String tiger, String Lion) {
    this.tiger = tiger;
    this.lion = lion;
}

public String getTiger() { return tiger;}
public String getLion() { return lion;}

public void setTiger(String tiger) {this.tiger = tiger;}
public void setTiger(String lion) {this.lion = lion;}
} 
```

  
