# Item 09
## try-finally 보다는 try-with-resources를 사용하라

```java
import java.io.*;

public class Copy {
    private static final int BUFFER_SIZE = 8 * 1024;

    // try-finally is ugly when used with more than one resource! (Page 34)
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }

    public static void main(String[] args) throws IOException {
        String src = args[0];
        String dst = args[1];
        copy(src, dst);
    }
}
```
- 자원이 여러 개 생기는 상황에서 각 자원들을 회수하는 코드가 전달된다. 
  - 그럼 그냥 finally 블록 하나에서 다 해결하면 안됩니까?
    - No! 예외는 try와 finally 모두에서 발생할 수 있다. 
    - 소프트웨어에는 문제가 없을 수 있다. 하지만 장치에서 물리적인 문제가 발생할 수 있다.
    - 코딩할 때 항상 예상치도 못한 예외에 대비해서 코딩하는 것 잊지 마세요!

```java
public class TopLine {
    // try-with-resources - the the best way to close resources!  (Page 35)
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        }
    }

    public static void main(String[] args) throws IOException {
        String path = args[0];
        System.out.println(firstLineOfFile(path));
    }
}
```

- try-with-resources 를 사용했을 때 장점
  - try catch 가 종료될 때 반드시 해제 해 줘야하는 자원들을 자동으로 관리 해 준다. 
    - readLine, close 양쪽에서 예외가 발생한다면 closeㅔ서 발생한 예외는 숨겨지고 readLine 에서 발생환 예외가 기록된다. 
    - 보여줄 예외를 제외하고 여러개의 다른 예외는 숨겨질 수 있다. 
  - 마찬가지로 catch절을 쓸 수 있다.
    - try catch 절을 중첩하지 않고도 다수의 예외를 처리할 수 있다. 
 