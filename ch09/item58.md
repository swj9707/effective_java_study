# Item 58

## 전통적인 for 문보다는 for-each문을 사용하라

### 전통적인 for문과 차이점
> - for-each 문은 명료, 유연하고 버그를 예방해준다.<br>
> - 성능 저하가 없다.<br>
<br>

#### 예시 1, 기존의 for문
```java
for (Iterator<String> iter = names.iterator(); iter.hasNext()) {
    String name = iter.next();
}
    
for (int i = 0; i < names.length; i++) {
    // ...
}
```
- 전통적인 for 문은 Iterator와 인덱스 변수를 사용해야한다.
- 코드의 가독성이 떨어진다.
<br>

#### 예시 2, for-each 문
``` java
for (String name : names) {
    // ...
}

```
- 정식 이름은 enhanced for statement.
- 반복 대상이 컬렉션이든 배열이든, for-each 문을 사용해도 속도는 일정함.
<br>

### for-each 를 사용하지 못하는 상황
- 컬렉션을 순회하면서 선택된 원소를 제거하는 상황
- 리스트나 배열을 순회하면서 원소의 값 일부 또는 전체를 교체 하는 상황
- 병렬 반복을 해야하는 상황
<br>
> Iterator 또는 Index 변수가 없어 foreach 자체만으로는 할 수 없다.
> java 8에 추가된 default 메서드의 remove를 통해 원소 제거 가능.
