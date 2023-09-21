# Item81

## wait와 notify 보다는 동시성 유틸리티를 애용하라

## concurrent collection

- List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 고려하여 구현 한 고성능 컬렉션
- 각자 내부에서 동기화를 수행.
- 동시성 컬렉션에서 동시성을 무력화하는 것은 불가능.

  - 외부에서 락을 추가로 사용하면 더 느려짐.

- 상태 의존적 수정 메소드
  - 여러 기본 동작을 하나의 원자적 동작으로 묶음

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

public class ConcurrentMapExam {
    private static final ConcurrentMap<String, String> map =
            new ConcurrentHashMap<>();

    public static String intern(String s) {
        String result = map.get(s);
        if (result == null) {
            result = map.putIfAbsent(s, s);
            if (result != null)
                result = s;
        }
        return result;
    }
}
```

- putIfAbsent(key, value)
  - 주어진 키에 매핑 된 갑싱 아직 없을 때만 새 값을 넣고 null을 리턴
  - 기존 값이 있다면 그 값을 리턴

## synchronizer

- 쓰레드가 다른 쓰레드를 기다릴 수 있도록 해서 서로 작업을 조율할 수 있게 도와줌
  - CountDownLatch, Semaphore 등등...

```java
public class CountDownLatchExam {
    public static long time(Executor executor, int concurrency,
                            Runnable action) throws InterruptedException {
        CountDownLatch ready = new CountDownLatch(concurrency);
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch done = new CountDownLatch(concurrency);

        for (int i = 0; i < concurrency; i++) {
            executor.execute(() -> {
                // 타이머에게 준비를 마쳤음을 알린다.
                ready.countDown();
                try {
                    // 모든 작업자 스레드가 준비될 때까지 기다린다.
                    start.await();
                    System.out.println("Start at " + Thread.currentThread().getName());
                    action.run();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown();
                }
            });
        }

        ready.await();  // 모든 작업자가 준비될 때까지 기다린다.
        System.out.println("Ready!");
        long startNanos = System.nanoTime();
        start.countDown();  // 작업자들을 깨운다.
        done.await();   // 모든 작업자가 일을 끝마치기를 기다린다.
        System.out.println("Done!");
        return System.nanoTime() - startNanos;
    }

    public static void main(String[] args) throws Exception {
        try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in))) {
            ExecutorService executorService = Executors.newFixedThreadPool(10);

            while (true) {
                String str = br.readLine();
                if (str.equals("exit")) break;

                Runnable action = () -> System.out.println(Thread.currentThread().getName() + " : " + str);
                System.out.println(time(executorService, 10, action));
            }
        }
    }
}
```

## wait, notify

- Java 5 이후로는 사실 쓸 일이 없음
- wait() 메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용
- 락 객체의 wait() 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출 해야 함

```java
synchronized (obj) {
    while (<조건이 충족되지 않았다>)
        obj.wait();	// 락을 놓고 대기 상태로 들어간다.

    // 조건이 충족됐을 때의 동작을 수행한다.
}
```

- wait() 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용해야 함
- 반복문 밖에서는 절대로 호출하지 말아야 합니다.
- while 반복문은 wait() 호출 전후로 주건이 만족하는지를 검사하는 역할

- notify와 notifyAll 중에서 선택해야 하는 경우에는 일반적으로 notifyAll을 사용하는게 안전하다.

  - 깨어나야 하는 모든 스레드가 깨어남을 보장할 수 있으니 항상 정확한 결과를 얻을 수 있기 때문.

- 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 notifyAll 대신 notify를 사용해 최적화할 수도 있다.
