# item28. 배열보다는 리스트를 사용하라

## Array와 List

### Array

- 크기가 정해져 있다 -> 컴파일 이전에 크기를 정해주어야 하고, 컴파일 이후에 크기를 변동할 수 없다
- 연속된 메모리 공간으로 이루어져 있어 지역성을 띈다(정적 표현) -> 관리가 편하지만 한 데이터를 삭제하더라도 배열은 연속해야 하므로 공간이 남는다(메모리 낭비)
- 인덱스를 이용해 표현된다



### List

- 불연속적으로 메모리 공간을 차지한다(동적 표현) -> 크기가 정해져 있지 않아 메모리 낭비를 줄일 수 있다
- 인덱스가 없고 포인터를 통해 접근할 수 있다 -> 삽입, 삭제가 용이하다
- 다 흩어져 있어서 검색 성능이 좋지 않고, 포인터를 통해 가리켜야하므로 추가적인 메모리 공간이 필요하다



배열과 제네릭 타입에는 중요한 차이가 두 가지 있습니다. 첫 번째, 배열은 공변(covariant)입니다. 어려워 보이는 단어지만 뜻은 간단하다. Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 됩니다.(공변, 즉 함께 변한다는 뜻) 반면, 제네릭은 불공변(invariant)입니다. 즉, 서로 다른 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아닙니다. 이것만 보면 제네릭에 문제가 있다고 생각할 수도 있지만, 사실 문제가 있는 건 배열 쪽입니다. 다음은 문법상 허용되는 코드입니다.

```java
//런타임에 실패하는 코드
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다";	//ArrayStoreExtension을 던짐
```



하지만 다음 코드는 문법에 맞지 않습니다

```java
//컴파일이 되지 않는 코드
List<Object> ol = new ArrayList<Long>();	//호환되지 않는 타입
ol.add("타입이 달라 넣을 수 없다");
```

어느 쪽이든 Long용 저장소에 String을 넣을 수 없습니다. 다만 배열에서는 그 실수를 런타임에야 알게 되지만, 리스트를 사용하면 컴파일할 때 바로 알 수 있습니다.

두 번째 주요 차이로는 배열은 실체화(reify)됩니다. 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인합니다. 그래서 위 코드에서 보듯 Long 배열에 String을 넣으려 하면 ArrayStoreException이 발생합니다. 반면, 앞서 이야기했듯 제네릭은 타입 정보가 런타임에는 소거(erasure)됩니다. 원소 타입을 컴파일타임에만 검사하며 런타임에는 알 수조차 없다는 뜻입니다. 소거는 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 해주는 메커니즘

위의 차이들로 배열과 제네릭은 잘 어우러지지 못합니다. 예컨대 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없습니다. 즉 코드를 new List<E>[], new List<string>[], new E[] 식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킵니다. 

제네릭 배열을 만들지 못하게 막은 이유는 무엇일까요? 타입 안전하지 않기 때문입니다. 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 ClassCastException이 발생할 수 있습니다. 런타임에 ClassCastException이 발생하는 일을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋나는 것입니다.



 

## Swift의 Array

- [`init()`](https://developer.apple.com/documentation/swift/array/1539784-init) - 빈 배열 생성

- [`init(S)`](https://developer.apple.com/documentation/swift/array/2853698-init) - 시퀀스 타입을 받아 같은 순서로 나열된 배열 생성

- [`init(S)`](https://developer.apple.com/documentation/swift/array/2907560-init)

시퀀스의 요소를 포함하는 배열을 만듭니다.

- [`init(repeating: Element, count: Int)`](https://developer.apple.com/documentation/swift/array/1641692-init)

반복되는 단일 값의 지정된 수를 포함하는 새 배열을 만듭니다.

- [`init(unsafeUninitializedCapacity: Int, initializingWith: (inout UnsafeMutableBufferPointer, inout Int) -> Void)`](https://developer.apple.com/documentation/swift/array/3200717-init)

지정된 용량의 배열을 만든 다음 배열의 초기화되지 않은 메모리를 덮는 버퍼로 지정된 클로저를 호출합니다.

### Array의 크기 늘리기

모든 어레이는 내용을 보관하기 위해 특정 양의 메모리를 예약합니다. 배열에 요소를 추가하고 해당 배열이 예약 된 용량을 초과하기 시작하면 배열은 더 큰 메모리 영역을 할당하고 해당 요소를 새 스토리지에 복사합니다. 새 저장소는 이전 저장소 크기의 배수입니다. 이 기하 급수적 성장 전략은 요소 추가가 일정한 시간에 발생하여 많은 추가 작업의 성능을 평균화한다는 것을 의미합니다. 재 할당을 트리거하는 추가 작업에는 성능 비용이 발생하지만 어레이가 커질수록 발생 빈도가 점점 줄어 듭니다.

저장해야하는 요소 수를 대략적으로 알고있는 경우  `reserveCapacity(_:)` 메서드를 사용하면 중간 재 할당을 방지할 수 있습니다. capacity 및 count 속성을 사용하여 대용량 스토리지를 할당하지 않고 배열에 저장할 수있는 요소 수를 결정합니다.

대부분의 `Element` 타입 배열의 경우, 이 저장소는 연속적인 메모리 블록입니다. 클래스거나 @objc 프로토콜 타입의 Element요소인 배열의 경우, 이 저장소는 연속적인 메모리 블록이나 NSArray의 인스턴스가 될 수 있습니다. 왜냐하면 NSArray의 임의의 어떤 서브클래스도 Array가 될 수 있기 때문에 이 경우 표현이나 효율성에 대한 보장이 없습니다.

### 참고

- [Array | Apple Developer Documentation](https://developer.apple.com/documentation/swift/array)

- https://opentutorials.org/module/1335/8677
- https://zeddios.tistory.com/117

