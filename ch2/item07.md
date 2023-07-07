item 07. 다 쓴 객체 참조를 해제하라
=============

### 메모리 관리하기
객체들의 다 쓴 참조를 주의해야한다.   
프로그램을 오래 실행하다 보면 점차 가비지 컬렌션 활동과 메모리 사용량이 늘어나 성능 저하를 일으킬 수 있다.
> null 처리를 통해 참조 해제를 하자.
> - null처리한 참조를 실수로 사용하려 하면 프로그램은 NullPointerException을 던지며 종료한다.

#### 하지만, 객체 참조를 null 처리 하는 일은 예외적인 경우여야 한다.
다 쓴 참조를 해제하는 가장 좋은 방법은 유효 범위 밖으로 변수를 밀어내는 것이다.

### null 처리는 언제?
책의 예시인 Stack 클래스는 자기 메모리를 직접 관리하기 때문에 메모리 누수에 취약하다.   
일반적으로 자기 메모리를 직접 관리하는 클래스라면 메모리 누수에 주의해야한다.

### 캐시
캐시 또한 메모리 누수를 일으킬 수 있다.   

#### Java에서는 세 가지 주요 유형의 참조(Reference) 방식

1. 강한 참조(Strong Reference)
- Integer prime = 1;   
와 같은 가장 일반적인 참조 유형이다.    
prime 변수 는 값이 1 인 Integer 객체에 대한 강한 참조를가진다.  
이 객체를 가리키는 강한 참조가 있는 객체는 GC대상이 되지않는다.

2. 부드러운 참조 (Soft Reference)
- SoftReference<Integer> soft = new SoftReference<Integer>(prime);    
와 같이 SoftReference Class를 이용하여 생성이 가능하다.   
만약 prime == null 상태가 되어 더이상 원본(최초 생성 시점에 이용 대상이 되었던 Strong Reference) 은 없고 대상을 참조하는 객체가 SoftReference만 존재할 경우 GC대상으로 들어가도록 JVM은 동작한다.   
다만 WeakReference 와의 차이점은 메모리가 부족하지 않으면 굳이 GC하지 않는 점이다.  
때문에 조금은 엄격하지 않은 Cache Library들에서 널리 사용되는 것으로 알려져있다.

3. 약한 참조(Weak Reference)
- WeakReference<Integer> soft = new WeakReference<Integer>(prime);   
와 같이 WeakReference Class를 이용하여 생성이 가능하다.  
prime == null 되면 (해당 객체를 가리키는 참조가 WeakReference 뿐일 경우) GC 대상이 된다.  
앞서 이야기 한 내용과 같이 SoftReference와 차이점은 메모리가 부족하지 않더라도 GC 대상이 된다는 것이다.    
다음 GC가 발생하는 시점에 무조건 없어진다.

#### WeakHashMap
WeakHashMap은 WeakReference의 특성을 이용하여 HashMap의 Element를 자동으로 제거, GC 해버린다.   
Key에 해당하는 객체가 더이상 사용되지 않는다고 판단되면 제거한다는 의미이다.


### 리스너 혹은 콜백
클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 콜백은 계속 쌓일 것이다.
이럴 때 콜백을 약한 참조로 저장하면 GC가 수거한다.
