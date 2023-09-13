# item 78. 공유 중인 가변 데이터는 동기화해 사용하라

## 동기화의 기능
#### (1) 배타적 실행
- 한 스레드가 변경하는 중이라면 다른 스레드가 보지 못하게 막는 용도를 말한다.
- 즉, Lock 을 걸어 동시에 여러 스레드가 접근하는 것을 차단한다.
#### (2) 스레드 사이의 안정적 통신
- 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.
- 즉, Lock 의 보호하에 수행된 모든 이전수정의 최종 결과를 보게 해준다.

## 원자적 연산

- Java 에서는 `long` / `double` 외의 변수를 읽고 쓰는 동작은 원자적(atomic)이다.

- 원자적이란 ?
    - 원자적 연산은 중단이 불가능한 연산은 말한다.
    - 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도 항상 어떤 스레드가 정상적으로 저장한 값을 읽을 수 있음을 보장한다.
    - 좀 더 살펴보자면, 여러 자바의 연산은 바이트 코드로 이루어져 있다.
        - 하나의 연산을 수행하기 위해 바이트코드가 수행될 때 중간에 다른 스레드가 끼어들어서 연산의 결과가 옳바르지 않다면 그건 원자적인 연산이 아니다.
- 원자적 데이터를 읽고 쓸 때 동기화가 왜 필요할까?
  - 스레드가 필드르 읽을 때 수정이 완전히 반영된 값을 얻는다고 보장하지만, 한 스레드가 지정한 값이 다른 스레드에게 보이는가는 보장하지 않기 때문
- long과 double은 왜?
    - JVM 비트수와 관련있음
    - 자바 메모리 모델에 의하면 32비트 메모리에 값을 할당하는 연산은 중단이 불가능 (원자적이다)
    - 그렇지만 long과 double의 경우에는 64비트의 메모리 공간을 갖고있기 때문에 원자적이지x

## 다른 스레드를 멈추는 방법
다른 스레드를 멈추는 `Thread.stop()`은 deprecated API로 지정되었으니 사용하지 말자.
### (1) Synchronized
- 자신의 `boolean` 필드를 두고 `false`로 초기화한 후 반복문을 돌게 만든다. 그리고 다른 스레드에서 해당 스레드를 멈추게 하고 싶은 경우에 true로 초기화하게 만들어 멈추게 하자

#### 잘못된 코드
```java
public class StopThread {
  private static boolean stopRequested; //false 초기화

  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopThread) {
        i++;
      }
    });
    backgroundThread.start();
    
    //1초 뒤에 스레드를 멈추게 하자.
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```
위 코드는 무한 루프를 돌게 된다.

#### why?
동기화의 두번째 기능인 `스레드간의 통신`을 생각하지 않았기 때문
- 동기화하지 않으면, 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤 보게 될 지 보장할 수 없다.

#### synchronized를 사용하여 동기화를 시킨 코드
```java
public class StopThread {
  private static boolean stopRequested; 
  
  //쓰기 메서드
  private static synchronized void requestStop(){
      stopRequested = true;
  }
  
  //읽기 메서드
  private static synchronized boolean stopRequested(){
      return stopRequested;
  }
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(() -> {
      int i = 0;
      while (!stopRequested()) {
        i++;
      }
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    requestStop();
  }
}
```

### (2) volatile으로 선언
- 필드를 `volatile`로 선언하면 동기화를 생략해도 된다.
- `volatile` 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 알게 됨을 보장한다.
#### volatile 필드를 사용해 스레드가 정상 종료되는 코드
```java
public class StopThread{
    // 필드를 volatile로 선언
	private static volatile boolean stopRequested;
    
    public static void main(String[] args) throws InterruptedException{
    	Thread backgroundThread = new Thread(() -> {
        	int i=0;
            while(!stopRequested()){ // 메서드 사용
            	i++;
            }
        });
        backgroundThread.start();
        
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
- 멀티 스레드 환경에서는 원래 성능 향상을 위해 변수 데이터들을 메인 메모리가 아닌 CPU cache에 저장하고 활용한다.
- `volatile` 필드는 멀티 스레드 상황에서도 메인 메모리에 데이터를 저장하고 읽기에 같은 값을 가질 수 있다.

#### volatile 주의점
```java
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber(){
    return nextSerialNumber++;
}
```
- 증가 연산ㄴ자는 코드상으로는 하나지만 실제로는 `nextSerialNumber` 필드에 두번 접근
- `안전 실패 오류`의 가능성 : 첫번째 스레드가 저장하기 이전에 두번째 스레드가 읽어버리면 스레드간 `nextSerialNumber`이 중복된 값이 된다.

#### 해결 방안
- 메서드에 `synchronized`를 붙이고 `volatile`을 제거하여 해결하자

## 동기화 문제를 피하는 방법
- 애초에 **가변 데이터를 공유하지 않을 것**
  - 가변 데이터는 단일 스레드에서만 사용하자
- 사용하려는 프레임워크와 라이브러리가 스레드를 수행하는지 잘 인지할 것
- `안전 발행` : 한 스레드가 데이터를 수정한 후 다른 스레드에 공유할 때 해당 객체에서 공유하는 부분만 동기화 해도 된다

## 핵심 정리
- 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야한다.
- 이에 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어진다
- 배타적 실행은 필요없고 스레드끼리 통신만 필요하다면 volatile 한정자만으로 동기화 할 수 있다.
