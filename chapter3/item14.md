# Item 14. Comparable을 구현할지 고려하라

이번에는 Swift에서 Java의 `Comparable` 인터페이스의 유일무이한 메서드인 `compareTo`의 역할에 대응하는 프로토콜에 대해 알아봅니다.

`compareTo`는 단순 동치성 비교와 순서 비교가 가능합니다. Java에서 `Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서(natural order)가 있음을 뜻합니다. 그래서 `Comparable`을 구현한 객체는 `Arrays.sort(a);` 처럼 손쉽게 정렬할 수 있습니다. 

이에 대응하는 Swift의 프로토콜로는 `Equatable`, `Comparable`,  `Sequence` , `IteratorProtocol` 등이 있습니다. 각각의 용도와 쓰임새에 대해 알아봅니다.

<br>

### Equatable

값이 동일한 지를 비교할 수 있는 타입으로 Equatable 프로토콜을 준수하는 타입은 등호 연산자(==) 또는 같지 않음 연산자(!=)를 사용해 동등성을 비교할 수 있습니다. Swift 표준 라이브러리 대부분의 기본 데이터타입은 Equatable을 따릅니다.

``` swift
let one = 1
let two = 2

if one == two {
} else { }
```

그래서 위와같이 기본타입인 Int의 경우 등호연산자(==)로 비교가 가능하고 Float, Double, String, Bool과 같은 타입도 Equatable을 채택하고 있기 때문에 비교가 가능합니다.

그럼 Equatable 프로토콜을 언제 사용해야 할까요?

Equatable 프로토콜은 Hashable, Comparable 프로토콜의 기반이 되므로 해당 프로토콜을 구현하기 위해서는 Equatable 프로토콜을 구현해야 합니다. 그리고 커스텀 타입을 만든 경우 비교를 원한다면 Equatable 프로토콜을 채택하고 구현해주어야 합니다.

- 구조체에서 프로퍼티 정렬 시 Equatable 준수
- 열거형에서 연관 값 사용 시 Equatable 준수 

```swift
class StreetAddress {
    let number: String
    let street: String
    let unit: String?

    init(_ number: String, _ street: String, unit: String? = nil) {
        self.number = number
        self.street = street
        self.unit = unit
    }
}

extension StreetAddress: Equatable {
    static func == (lhs: StreetAddress, rhs: StreetAddress) -> Bool {
        return
            lhs.number == rhs.number &&
            lhs.street == rhs.street &&
            lhs.unit == rhs.unit
    }
}
```

 `public static func ==(lhs: Self, rhs: Self) -> Bool` 는 필수로 구현되어야 하는 함수입니다. 위의 예제처럼 해당 메서드 내에서 개별 요소에 관한 항목을 비교하도록 구현해주면 됩니다.

<br>

### Comparable

연산자 <, <=, >=, > 와 연관된 비교를 가능하게 하는 타입으로 String이나 숫자처럼 고유한 순서를 가진 타입에 주로 사용됩니다. 연산자나 표준 라이브러리 메서드를 사용하여 인스턴스를 비교하는 경우 Comparable 메소드를 채택하면 됩니다.

```swift
struct MyData: Comparable {
    var value: Int = 0
    
    static func < (lhs: MyData, rhs: MyData) -> Bool {
        return lhs.value < rhs.value
    }
}

let v1 = MyData(value: 1)
let v2 = MyData(value: 2)

print(v1 > v2 ? "true" : "false")			//false
print(v1 >= v2 ? "true" : "false")		//false
print(v1 <= v2 ? "true" : "false")		//true
print(v1 == v2 ? "true" : "false")		//false
```

그리고 Comparable이 Equatable을 준수하므로 별도로 채택하여 구현 할 필요는 없습니다.

<br>

### Sequence

해당 요소에 순서와 반복적인 접근을 제공하는 타입으로 Sequence는 한 번에 하나씩 단계별로 실행할 수 있는 값의 목록입니다. 시퀀스의 요소들을 반복하는 일반적인 방법은 for-in 루프가 있습니다. 다시 말해, Sequence 프로토콜을 준수하는 타입은 for-in 루프로 순회할 수 있습니다.

Swift의 기본 라이브러리이고, Array, Dictionary, Set과 같은 Collection 타입의 기반이 되는 프로토콜입니다. Sequence 프로토콜을 구현하면 `forEach`, `map`, `filter`, `flatMap`과 같은 다양한 함수를 사용할 수 있습니다.

```swift
struct Countdown: Sequence, IteratorProtocol {
    var count: Int

    mutating func next() -> Int? {
        if count == 0 {
            return nil
        } else {
            defer { count -= 1 }
            return count
        }
    }
}

let threeToGo = Countdown(count: 3)
for i in threeToGo {
    print(i)
}
// Prints "3"
// Prints "2"
// Prints "1"
```

커스텀 타입을 생성하는 경우, Sequence 프로토콜을 채택하면 유용한 오퍼레이션들을 손쉽게 가져다 쓸 수 있습니다. Sequence 프로토콜을 채택하기 위해선 makeIterator() 메서드를 추가해야합니다. Sequence 내부에 associatedtype으로 IteratorProtocol타입이 있어 순회하려는 대상은 IteratorProtocol 타입이어야 합니다.

Sequence 내에 특정 값이 포함되어 있는지 확인할 때와 Sequence의 끝에 도달하거나 특정값을 찾을 때까지 순차적으로 탐색할 수 있습니다. 이렇게 순회가 가능함은 어떤 시퀀스 상에서든 많은 양의 연산을 위해 접근이 가능하다는 것을 의미합니다. 

또한 Sequence 프로토콜은 `contains(_:)` 메서드를 지원하는데, 이 메서드를 사용하면 수동으로 값을 순회 할 필요 없이 값의 포함 유무를 판단할 수 있습니다.

<br>

### IteratorProtocol

시퀀스 값을 한 번에 하나씩 제공하는 타입으로 Sequence 프로토콜과 함께 사용됩니다. 시퀀스는 반복 프로세스를 트래킹하고, 한번에 한 요소를 반환하는 Iterator를 생성해 개별 요소에 접근할 수 있게 합니다. IteratorProtocol의 목적은 컬렉션을 반복 순회하는 `next()` 메서드를 통해 컬렉션의 반복 상태를 캡슐화 하는 것입니다.

```swift
protocol IteratorProtocol {
  associatedtype Element
  mutating func next() -> Element?
}
```

위 코드에서 associtatedtype으로 선언된 Element는 Iterator가 생성하는 값의 유형을 지정합니다. 그리고 `next()`는 해당 시퀀스에서 다음번 요소를 반환하거나, 다음번 요소가 없는 경우 nil을 반환합니다.



<br>

### 결론

값이 동일한지 비교하고싶다 -> Equatable

값의 크고 작음을 비교하고싶다 -> Comparable

순서를 가지게 하고싶다 -> Sequence

순서를 가진 타입을 순회하고싶다 -> IteratorProtocol



#### References

- [Equatable - Apple Developer Document](https://developer.apple.com/documentation/swift/equatable)
- [Sequence - Apple Developer Document](https://developer.apple.com/documentation/swift/sequence)
- [IteratorProtocol - Apple Developer Document](https://developer.apple.com/documentation/swift/iteratorprotocol)
- [Comparable - Apple Developer Document](https://developer.apple.com/documentation/swift/comparable)
- https://kor45cw.tistory.com/260

