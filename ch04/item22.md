# Item22

## 인터페이스는 타입을 정의하는 용도로만 사용하라

**여담이지만 타입스크립트에서 의미하는 인터페이스도 비슷한 역할을 한다.**

### 인터페이스의 역할 => 어떤 기능을 구현해야 하는가, 등등

- 클래스의 타입을 정의시키는 데 사용된다.
- Q : 클래스 그 자체가 사실 타입 아닌가요?
- A : 맞는 말이지만, 인터페이스는 클래스보다는 좀 더 작은 단위라고 볼 수 있다.
- 인터페이스는 클래스와 다르게 다중 상속이 가능하다. A라는 타입을 위해 B, C 가 필요하다면?

- 클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지 클라이언트에게 알려주는 것.

### 잘못된 인터페이스 사용 예시 : 상수 인터페이스

```java
// Constant interface antipattern - do not use!
public interface PhysicalConstants {
    // Avogadro's number (1/mol)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // Boltzmann constant (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // Mass of the electron (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

- 클래스 내부에서 사용하는 상수는 내부 구현에 해당하므로 클래스의 API를 노출하는 해위이다.
- final이 아닌 클래스가 상수 인터페이스를 구현한다면 모든 하위클래스의 이름공간이 그 인터페이스가 정의한 상수로 덮어진다.

### 상수를 공개 할 목적이라면, 클래스나 인터페이스 자체에 추가해야한다.

- 예를 들면 Integer, Double에 선언 된 MIN_VALUE, MAX_VALUE

```java
// Constant utility class (Page 108)
public class PhysicalConstants {
    private PhysicalConstants() { }  // Prevents instantiation

    // Avogadro's number (1/mol)
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    // Boltzmann constant (J/K)
    public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

    // Mass of the electron (kg)
    public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```

- 인스턴스화 할 수 없는 클래스에 이런 데이터들을 담아서 생성해야 한다.

### 요약

- 인터페이스는 타입을 정의하는 용도로만 사용해야 하지, 상수 공개용 수단으로 사용하지 말자.
