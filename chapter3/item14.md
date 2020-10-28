# Item 14. Comparable을 구현할지 고려하라

이번에는 Swift에서 Java의 `Comparable` 인터페이스의 유일무이한 메서드인 `compareTo`의 역할에 대응하는 프로토콜에 대해 알아보자.

`compareTo`는 단순 동치성 비교와 순서 비교가 가능하다. Java에서 `Comparable`을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서(natural order)가 있음을 뜻한다. 그래서 `Comparable`을 구현한 객체는 `Arrays.sort(a);` 처럼 손쉽게 정렬할 수 있다. 

이에 대응하는 Swift의 프로토콜로는 `Equatable`, `Sequence`, `IteratorProtocol`, `Comparable`, 비교연산자 등이 있다. 각각의 용도와 쓰임새에 대해 알아보자.

### Equatable

값이 동일한 지를 비교할 수 있는 타입으로 Equatable 프로토콜을 준수하는 타입은 등호 연산자(==) 또는 같지 않음 연산자(!=)를 사용해 동등성을 비교할 수 있다. Swift 표준 라이브러리 대부분의 기본 데이터타입은 Equatable을 따른다.

``` swift
let one = 1
let two = 2

if one == two {
} else { }
```

그래서 위와같이 기본타입인 Int의 경우 등호연산자(==)로 비교가 가능하고 Float, Double, String, Bool과 같은 타입도 Equatable을 채택하고 있기 때문에 비교가 가능하다.

그럼 Equatable 프로토콜을 언제 사용해야 할까?

Equatable 프로토콜은 Hashable, Comparable 프로토콜의 기반이 되므로 해당 프로토콜을 구현하기 위해서는 Equatable 프로토콜을 구현해야 한다. 그리고 커스텀 타입을 만든 경우 비교를 원한다면 Equatable 프로토콜을 채택하고 구현해주어야 한다.

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

 `public static func ==(lhs: Self, rhs: Self) -> Bool` 는 필수로 구현되어야 하는 함수이다. 위의 예제처럼 해당 메서드 내에서 개별 요소에 관한 항목을 비교하도록 구현해주면 된다.

### Sequence



### IteratorProtocol



### Comparable



### 비교 연산자



### 결론

어떨 때 어떤 프로토콜 채택할 지



#### References

- [Equatable - Apple Developer Document](https://developer.apple.com/documentation/swift/equatable)

  