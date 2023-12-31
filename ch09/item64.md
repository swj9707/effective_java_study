# Item 64
## 객체는 인터페이스를 사용해 참조하라

Item51에서 매개변수 타입으로 클래스가 아닌 인터페이스를 사용하라고 했었다
#### 이는 "객체는 클래스가 아닌 인터페이스로 참조하라" 까지 확장 가능하다
- 적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스로 선언하라

```java
// 좋은 예!
// 인터페이스를 타입으로 사용했다
Set<Son> sonSet = new LinkedHashSet<>();
```

```java
// 나쁜 예!
// 클래스를 타입으로 사용했다
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

위의 예시처럼 인터페이스를 타입으로 사용하는 습관을 길러두면 프로그램이 훨씬 유연해진다
- 나중에 클래스를 교체하고자 한다면 그저 새 클래스의 생성자를 호출해주기만 하면 된다
```java
// 다른 코드의 변경 없이 LinkedHashSet -> HashSet 으로 변경한 것 만으로 끝!
Set<Son> sonSet = new HashSet<>();
```

#### 하지만 적합한 인터페이스가 없다면?
아쉽지만 당연히 클래스를 참조해야 한다
- String, BigInteger 등의 값 클래스
  - 값 클래스는 여러가지로 구현될 수 있다고 생각하고 설계되지 않았으므로 상응하는 인터페이스가 없는경우가 대부분
  - 이런경우는 클래스를 매개변수, 변수, 필드, 반환 타입으로 사용해도 무방하다
- OutputStream 과 같은 클래스 기반으로 작성된 프레임워크가 제공하는 객체들
  - 이런 경우라도 특정 구현 클래스보다는 기반 클래스를 참조하는게 좋다 (추상 클래스)

```java
핵심!
적합한 인터페이스가 있으면 전부 인터페이스 타입으로 선언하라
(적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 클래스를 사용하자)
```
