# Item 57

## 지역변수의 범위를 최소화하라

### 왜 최소화를 해야하나?
> - 코드가독성이 높아진다다.<br>
> - 유지보수성이 높아진다.<br>
> - 오류가능성이 낮아진다.
<br>

#### 예시 1 , 가장 처음 쓰일 때 선언하라
```java
public class Account {
	public static void main(String[] args) {
	  int myBalance = 51500;
		//
    //
    //
	}
}
```
- 미리 변수를 선언하면 코드가 길어질 때 가독성이 나빠진다.
- 막상 쓰일 시점에 어떤 타입인지, 무슨 값으로 초기화 되었는지 파악하기 힘들다.
<br>

#### 예시 2, 모든 지역변수는 선언과 동시에 초기화 하라.
``` java
public class Account {
	public static void main(String[] args) {
	  int myBalance;
  //
  //
  //
		myBalance = 51500;
	}
}

public class Animal {
	public static void main(String[] args) {
	  int myBalance;

		try {
			myBalance = Account.getBalance();
		} catch (IOException e) {
			// 예외처리
		}
	}
}

```
- 초기화에 필요한 정보가 충분하지 않으면, 선언을 미룬다.
- 그러나 try-catch-finally 구문에서는 예외이다. 초기화 과정에서 확인된 예외 발생시, try 블록 안에서 초기화를 해야한다.   
<br>

#### 예시 3, for문을 고려하라.
``` java
for (Iterator<Integer> i = c.iterator(); i.hasNext();) {
	Integer number = i.next();
}

i.hasNext(); // compile error!
    
Iterator<Integer> i = c.iterator();
while (i.hasNext()) {
	doSomething(i.next());
}

// ...
Iterator<Integer> i2 = c2.iterator();
while (i.hasNext()) { // 버그 발생
	doSomething(i2.next());
}
```
- 반복문 마다 접근할 수 있는 변수의 위치가 다르다.
<br>

#### 예시 4, 메서드를 작게 유지하고 한 기능에 집중하라.
``` java
public class MyData {
    public void process() {
		// 데이터를 읽어오는 로직
        String myData = DataSource.readData();
				
		// 데이터를 파싱하는 로직
        RenderedData renderedData = parseData(myData);
        renderedData.render();
				
		// 데이터를 정리하는 로직
		if (renderedData.getRendingComplete) {
			renderedData.cleanup();
		}
    }
}
```
- 한 메서드에서 여러 가지 기능을 처리한다면 그 중 한 기능만 관련된 지역 변수라도 다른 기능에 접근할 수 있다.
<br>

###
