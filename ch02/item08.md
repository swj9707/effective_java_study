item 08. finalizer와 cleaner 사용을 피하라
=============

자바의 두가지 객체 소멸자
------------
### finalizer
- 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요
- 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 함
- Java 9부터 deprecated API로 지정되어 cleaner를 대안으로 소개

### cleaner
- finalizer 보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요함

지양하는 이유
----------
- 즉시 수행이 보장되지 않는다
- 수행 여부 또한 보장되지 않는다
- 심각한 성능 문제도 동반한다
- 심각한 보안 문제를 일으킬 수 있다

어디에 쓰는 물건일까
---------
자원 반납의 안전망 역할이나 네이티브 자원 정리 외에는 사용하지 않는다.


### 정리
cleaner(Java 8 까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자.
