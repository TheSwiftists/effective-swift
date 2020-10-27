# Item 14. Comparable을 구현할지 고려하라

이번에는 Swift에서 Java의 `Comparable` 인터페이스의 유일무이한 메서드인 `compareTo`의 역할에 대응하는 프로토콜에 대해 알아보자.

`compareTo`는 단순 동치성 비교와 순서 비교가 가능하다. Java에서 `Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서(natural order)가 있음을 뜻한다. 그래서 `Comparable`을 구현한 객체는 `Arrays.sort(a);` 처럼 손쉽게 정렬할 수 있다. 

이에 대응하는 Swift의 프로토콜로는 `Equatable`, `Sequence`, `IteratorProtocol`, `Comparable`, 비교연산자 등이 있다. 각각의 용도와 쓰임새에 대해 알아보자.

### Equatable



### Sequence



### IteratorProtocol



### Comparable



### 비교 연산자



### 결론

어떨 때 어떤 프로토콜 채택할 지

