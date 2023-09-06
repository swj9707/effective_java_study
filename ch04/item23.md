# Item23

## 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### 태그 달린 클래스

```java
// Tagged class - vastly inferior to a class hierarchy! (Page 109)
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // Tag field - the shape of this figure
    final Shape shape;

    // These fields are used only if shape is RECTANGLE
    double length;
    double width;

    // This field is used only if shape is CIRCLE
    double radius;

    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

- 태그가 달린 클래스는 두 가지 이상의 의미를 표현할 수 있고 그 중 현재 표현하는 의미를 태그 값으로 알려준다.
- 단점
  1. 쓸데없는 코드가 너무 많다.
  2. 새로운 의미 (enum) 을 추가할 때 마다 switch 코드를 수정해야한다.
  3. 인스턴스의 타입 만으로는 현재 나타내는 의미를 알 길이 없다.

### 서브 타이핑

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
```

- 타입이 의미 별로 따로 존재하기 때문에 변수의 의미를 명시하거나 제한할 수 있다.
- 특정 의미만 매개변수로 받을 수도 있다.
- 타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성 뿐만 아니라 컴파일 타임 타입 검사 능력을 높힐 수 있다.

### 결론

- 태그 달린 클래스를 걍 쓰지 마세요
