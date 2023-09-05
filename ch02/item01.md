# Item 01
## 생성자 대신 정적 팩터리 메서드를 고러하라

- 단순 생성자를 사용했을 때 
  - 생성자 그 자체는 이름을 가질 수는 없다. 
    ```java
    Instance instance = new Instance(a, b, c, d);
    ```
- 정적 팩터리 메소드의 장점
1. 이름을 가질 수 있다. 
   1. 날 생성자는 전달되는 파라미터만 보고 의미를 유추해야 한다. 
   2. 협업을 하는 상황이라면?
2. 호출 될 때마다 인스턴스를 새로 생성할 필요 없다.
   1. 불변 클래스의 경우 어차피 값이 변화하지 않기 때문에 굳이 새로운 인스턴스를 생성할 이유가 없다. 
   2. `Boolean.valueOf(boolean)` 과 같은 경우는 굳이 새로운 인스턴스를 생성하지 않는다. 
      1. 객체의 생성 비용을 줄이므로 자주 이러한 객체가 요청되는 경우엔 성능을 끌어올려 줄 수 있다.  
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다. 
   ```java
    public class Level {

        public static Level of(int score) {
            if (score < 50) {
                return new Basic();  //상위 클래스 
            } else if (score < 80) {
                return new Intermediate();   // 하위 클래스
            } else {
                return new Advanced();    // 하위 클래스
            }
        }
    }

    private class Intermediate extends Level {

    }
   ```
   - 생성자와 다르게 반드시 종속되어 있는 클래스에 대한 객체를 리턴할 필요가 없다. 
   - public 으로 선언되지 않은 클래스의 객체를 반환할 수 있다. 
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
   1. 반환 타입의 하위 타입이기만 하다면 어떤 클래스의 객체를 반환하든 상관없다 
   2. 위의 예시 참조
5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다. 
   ```java
    String url = "jdbc:mysql://localhost:3306/XX";
    String user = "root";
    String password = "1234@";
 
    try {
        Class.forName("com.mysql.jdbc.Driver");  // 리플렉션  
        Connection conn = DriverManager.getConnection(url, user, password);  // 정적 팩터리 메서드
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    }
   ```
   - mysql, oracle 등 어떤 DB를 사용할 지 모르는 상황에서도 개발이 가능함. 

- 단점
1. 상속을 하려면 public 이나 protect 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다. 
2. 정적 팩토리 메소드는 프로그래머가 찾기 힘들다. 
   - 정적 팩토리 메소드에 주로 쓰는 이름
  
    1. from : 매개변수 하나 받아 해당 타입의 인스턴스 반환하는 형변환 메서드
    ```java
    Date d = Date.from(instant);
    ```
     2. of : 여러 매개변수 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
    ```java
    Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
    ```
     3. valueOf : from 과 of 의 더 자세한 버전
    ```java
    BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
    ```
     4. instance / getInstance : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스 보장하지 X
    ```java
    StackWalker luke = StackWalker.getInstance(options);
    ```

     5. create / newInstance : 매번 새로운 인스턴스를 생성해 반환함을 보장
    ```java
    Object newArray = Array.newInstance(classObject, arrayLen);
    ```

     6. getType : getInstance 와 같으나, 생성 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할때 사용
    ```java
    FileStore fs = Files.getFileStore(path)
    ```

     7. newType : newInstance 와 같으나, 생성 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할때 사용
    ```java
    BufferedReader br = Files.newBufferedReader(path);
    ```
     8. type : getType / newType 간결 버전
    ```java
    List<Complaint> litany = Collections.list(legacyLitany);
    ```

