# item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

> Swift Protocol의 네가지 용법에 대해 알아보자



<br>

## Java의 Interface

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 합니다. 달리 말해, **클래스가 어떤 인터페이스를 구현한다는 것은 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것입니다.**

이 지침에 맞지 않는 예로 소위 상수 인터페이스라는 것이 있습니다. 상수 인터페이스란 메서드 없이, 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스를 말합니다. 그리고 이 상수들을 사용하려는 클래스에서는 정규화된 이름(qualified name)을 쓰는 걸 피하고자 그 인터페이스를 구현하곤 합니다.  다음의 예를 봅시다.

```java
public interface PhysicalConstants {
  // 아보가드로 수 (1/몰)
  static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  // 볼츠만 상수 (J/K)
  static final double BOLTZMAN_NUMBER = 1.380_648_52e-23;
}
```

상수 인터페이스 안티패턴은 인터페이스를 잘못 사용한 예시입니다. 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당합니다. 따라서 상수 인터페이스를 구현하는 것은 이 내부 구현을 클래스의 API로 노출하는 행위입니다. 클래스가 어떤 상수 인터페이스를 사용하든 사용자에게는 아무런 의미가 없습니다. 오히려 사용자에게 혼란을 주기도 하며, 더 심하게는 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속되게 합니다. 

상수를 공개할 목적이라면 더 합당한 선택지가 몇 가지 있습니다. 특정 클래스나 인터페이스와 강하게 연관된 상수라면 그 클래스나 인터페이스 자체에 추가해야 합니다. 열거타입으로 나타내기 적합한 상수라면 열거 타입으로 만들어 공개하면 됩니다. 그것도 아니라면, 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개합니다.

다음 코드는 앞서 보여준 `PhysicalConstants`의 유틸리티 클래스 버전입니다.

```java
public class PhysicalConstants {
  private PhysicalConstants() {}		// 인스턴스화 방지
  
  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  // 볼츠만 상수 (J/K)
  public static final double BOLTZMAN_NUMBER = 1.380_648_52e-23;
}
```



Java의 Interface를 이용한 안티패턴과 그것을 피할 수 있는 방법에 대해 알아보았으니 이번에는 Swift의 Protocol을 적절하게 사용하는 방법에 대해 알아보겠습니다.

<br>



## Swift의 Protocol

Swift의 프로토콜 구현은 언어적인 측면에서 가장 흥미로운 부분 중 하나입니다. 앞서 말했듯이 프로토콜의 주된 역할은 구체적인 구현에 앞서 추상화를 정의할 수 있도록 하는 것입니다. 프로토콜을 사용하는 것은 공용 API에 영향을 주지 않고 구현부를 swap하거나 변형하도록 할 수 있습니다. 프로토콜 지향 프로그래밍 방식을 통해 다양한 방식으로 프로토콜을 사용할 수 있습니다. 크게 네가지로 나눌 수 있는 프로토콜의 용법에 대해 알아봅시다.

 

- **Action enablers** : 각각의 주어진 타입에 정의된 동작을 하도록 함. 일반적으로 `Equatable`처럼 "able"로 끝나는 이름을 가짐

- **Requirement definitions** : `Numeric`, `Sequence` 처럼 특정한 종류의 객체가 되기 위한 요구사항을 정함

- **Type conversion** : `CustomStringConvertible`, `ExpressibleByStringLiteral` 처럼다양한 유형이 다른 유형으로 변환 가능하거나 원시 값 또는 리터럴을 통해 표현할 수 있음을 선언하는데 사용

- **Abstract interfaces** : 여러 타입이 구현할 수 있는 통합된 API의 역할을 함. 이를 통해 원하는 대로 구현을 교체하거나, 코드를 캡슐화 하거나, mock된 객체를 테스트에서 사용할 수 있음

특히 associated type 및 protocol extension을 활용해서 정의하고 사용할 수 있는 방법의 수는 아주 다양해서 프로토콜이 얼마나 강력하게 쓰일 수 있는지를 보여줍니다. 

그렇기 때문에 모든 프로토콜을 동일한 방식으로 취급하는 것이 아니라 어떤 카테고리에 속하는지에 따라 디자인 하는 것이 중요합니다. 위에서 말한 프로토콜의 용법에 대해 Apple의 프레임워크는 어떻게 사용하고 있는지 알아봅시다.



<br>

### Enabling unified actions

특정한 작업을 수행하기 위해 이를 준수하는 타입이 필요한 프로토콜부터 살펴보겠습니다. 예를들어 표준 라이브러리인 `Equatable` 프로토콜은 두 인스턴스간에 동등성 검사를 수행할 수 있음을 표시하는 데 사용되는 반면 `Hashable` 프로토콜은 해시될 수 있는 유형에 의해 채택됩니다.

```swift
protocol Equatable {
    static func ==(lhs: Self, rhs: Self) -> Bool
}

protocol Hashable: Equatable {
    func hash(into hasher: inout Hasher)
}
```

예를 들어 다음은 배열의`Element` 유형이`Equatable`을 준수하는 경우, 값의 모든 발생을 계산할 수있는 메서드로 `Array`를 확장하는 방법입니다.

```swift
extension Array where Element: Equatable {
    func numberOfOccurences(of value: Element) -> Int {
        reduce(into: 0) { count, element in
            // 두 값이 Equatable을 따른다고 보장이 되어 있기 때문에
            // 값이 같은지 비교할 수 있습니다
            if element == value {
                count += 1
            }
        }
    }
}
```

**특정 영역에 너무 묶여있지 않고 행동 자체에 집중할 수 있기 때문에, (Equatable과 Hashable처럼) 행동 기반의 프로토콜을 정의할 때는 가능한 한 일반적으로 만드는 것이 좋습니다.**



예를 들어, 다양한 객체나 값을 로드하는 여러 유형을 통합하는 `Loadable`의 경우, `associated type`으로 프로토콜을 정의할 수 있습니다. `associated type`을 이용하면 프로토콜을 채택하는 객체들이 로드되는 결과의 타입을 선언하도록 합니다.

```swift
protocol Loadable {
    associatedtype Result
    func load() throws -> Result
}
```

그러나 모든 프로토콜이 동작을 정의하는 건 아닙니다. 예를들어, 아래의 `Cacheable` 프로토콜의 이름은 캐싱 작업이 포함되어있음을 암시하지만 실제로는 다양한 유형이 자체 캐싱 키를 정의할 수 있도록 하는데 사용됩니다.

```swift
protocol Cacheable: Codable {
    var cacheKey: String { get }
}
```

위의 코드를 인코딩과 디코딩 모두에 대한 작업을 정의하는 기본 내장 프로토콜인 `Codable`과 비교해보면 이름이 적절하지 않다는 것을 알 수 있습니다.

모든 프로토콜이 "-able" 접미사를 사용할 필요는 없습니다. 프로토콜을 정의하기 위해 주어진 명사에 "-able" 접미사를 사용하는 경우 다음과 같이 많은 혼란을 초래할 수 있습니다.

```swift
protocol Titleable {
    var title: String { get }
}

protocol Colorable {
    var foregroundColor: UIColor { get }
    var backgroundColor: UIColor { get }
}
```

그렇다면 어떻게 이러한 프로토콜을 이름과 구조적인 측면에서 개선할 수 있을까요? 동작을 통합하는 역할로서의 프로토콜 외에 프로토콜의 다른 사용 방법을 살펴봅시다.



<br>

### Defining requirements

프로토콜이 사용될 수 있는 또 다른 방법으로는 API에 대한 요구사항이나 객체를 정의하는데 사용할 수 있습니다. 표준 라이브러리에서 `Collection`, `Numeric`, `Sequence`와 같은 프로토콜은 **해당 프로토콜의 의미를 정의하는 데 사용**됩니다.

```swift
protocol Sequence {
    associatedtype Iterator: IteratorProtocol
    func makeIterator() -> Iterator
}
```

> 위의 프로토콜은 `Sequencable` 로 불리지 않습니다. 왜냐하면 객체가 Sequencable 해지기 위한 요구사항을 정의하는 것이 아니라 객체를 Sequence로 바꾸는 것이기 때문입니다.

`Sequence` 의 정의에서 알 수 있는 것은 Array, Dictionary와 같은 Swift의 시퀀스의 기본 역할은 iterator를 생성하는 [팩토리](https://www.swiftbysundell.com/articles/using-the-factory-pattern-to-avoid-shared-state-in-swift/) 역할을 하는 것입니다. 이는 다음 프로토콜을 통해 공식화됩니다.

```swift
protocol IteratorProtocol {
    associatedtype Element
    mutating func next() -> Element?
}
```

> iterator가 실제로 각 반복 작업을 자체적으로 수행하기 때문에 위의 프로토콜은 대신 `Iterable`이라고 불릴 수 있습니다.그러나 `IterableProtocol`이라는 이름은 `Sequence`와 더 일관된 느낌을 주기 위해 선택되었을 것입니다. 그리고 관련된 타입과 동일한 이름의 충돌을 방지하기 위해 `Iterator`라는 이름을 하지 않은 것입니다.

위의 `Sequence`와 `IteratorProtocol`을 염두에 두고 `Cacheable`과 `Colorable` 프로토콜로 돌아가 요구사항을 담고 있는 프로토콜로서 개선할 수 있는지 살펴보겠습니다.

일단 `Colorable`을 `ColorProvider`로 이름을 바꾸는 것부터 시작하겠습니다. 이 프로토콜은 요구사항이 정확히 동일하더라도 완전히 새로운 의미를 부여합니다. 더 이상 색상을 지정할 수 있는 객체를 정의하는 데 사용되는 것처럼 들리지 않지만, 시스템의 다른 부분에 색상 정보를 제공하는 타입으로서 느껴집니다. 

```swift
protocol ColorProvider {
    var foregroundColor: UIColor { get }
    var backgroundColor: UIColor { get }
}
```

마찬가지로 `IteratorProtocol`에서 영감을 얻어 `Cacheable`을 아래와 같이 바꿀 수 있습니다.

```swift
protocol CachingProtocol: Codable {
    var cacheKey: String { get }
}
```



<br>

### Type conversions

다음으로, 다른 값으로부터 convertible이 가능한 타입을 정의하는 프로토콜에 대해 알아보겠습니다.

`CustomStringConvertible` 이라는 표준 라이브러리가 있습니다. 모든 타입을 사용자가 정의한 문자열로 반환하는 데 사용합니다.

```swift
protocol CustomStringConvertible {
    var description: String { get }
}
```

여러 타입에서 단일 데이터 조각을 추출할 때 주로 유용합니다. 아까 앞서 살펴봤던 `Titleable` 프로토콜의 목적과 완벽하게 일치합니다.

`Titleable`프로토콜의 이름을 `TitleConvertible`로 변경하면 해당 프로토콜의 용도를 더 쉽게 이해할 수 있을 뿐만 아니라, 코드를 표준 라이브러리와 일관성있게 만들 수 있습니다.

```swift
protocol TitleConvertible {
    var title: String { get }
}
```

이러한 타입 변환 프로토콜은 특정 구현에 많은 양의 계산이 필요할 때 프로퍼티보다 메소드에 유용하게 사용할 수 있습니다.  

```swift
protocol ImageConvertible {
    //렌더링 되는 유형에 따라 다르지만, 이미지 렌더링은 다소 비쌀 수 있기 때문에
	  //프로퍼티보다는 메소드로 정의합니다.
    func makeImage() -> UIImage
}
```

특정 타입을 반환하는 프로토콜을 앞서 본 방식과 또 다른 방법으로도 표현할 수 있습니다. 특히 문자열 및 배열 리터럴과 같은 Swift의 모든 내장 리터럴 지원을 구현하는 데 사용되는 방법입니다. `nil` 할당조차도 프로토콜을 통해 구현됩니다.

```swift
protocol ExpressibleByArrayLiteral {
    associatedtype ArrayLiteralElement
    init(arrayLiteral elements: ArrayLiteralElement...)
}

protocol ExpressibleByNilLiteral {
    init(nilLiteral: ())
}
```

> 자체 코드 내에서 대부분의 기본 제공 리터럴 프로토콜을 자유롭게 준수 할 수 있지만, `ExpressibleByNilLiteral` 준수는 권장되지 않습니다. - 'Optional'은 해당 프로토콜을 채택하는 유일한 유형이 될 것으로 예상됩니다. ( `@frozen public enum Optional<Wrapped> : ExpressibleByNilLiteral` )



<br>

### Abstract interfaces

마지막으로, 여러 타입을 추상화하는 인터페이스로서 프로토콜을 사용하는 방법에 대해 알아보겠습니다.

흥미로운 예시로는 저수준 그래픽 프로그래밍 API인 Apple의 Metal 프레임워크에서 찾을 수 있습니다. GPU는 장치마다 많이 달라지는 경향이 있고 Metal은 지원하는 모든 유형의 하드웨어에 대해 프로그래밍을 위한 통합 API를 제공하는 것을 목표로 하기 때문에 프로토콜을 사용해 API를 추상 인터페이스로 정의합니다.

```swift
protocol MTLDevice: NSObjectProtocol {
  var name: String { get }
  var registryID: UInt64 { get }
  ...
}
```

Metal을 사용할 때 `MTLCreateSystemDefaultDevice` 함수를 호출하면, 시스템은 현재 프로그램이 실행되고있는 장치에 적합한 위 프로토콜의 구현을 반환합니다.

```swift
func MTLCreateSystemDefaultDevice() -> MTLDevice?
```



자체 코드 내에서 동일한 인터페이스의 여러 구현을 지원할 때마다 정확히 동일한 패턴을 사용할 수도 있습니다. 예를 들어, 특정 한 네트워킹 수단에서 네트워크를 호출하는 방식을 분리하기 위해 `NetworkEngine` 프로토콜을 정의할 수 있습니다.

```swift
protocol NetworkEngine {
    func perform(
        _ request: NetworkRequest,
        then handler: @escaping (Result<Data, Error>) -> Void
    )
}
```

네트워크 작업을 수행할 객체가 해당 `NetworkEngine`이라는 프로토콜을 채택함으로써 네트워킹 실행부 구현을 필요한만큼 자유롭게 정의할 수 있습니다. 아래 코드를 보면 `NetworkEngine` 프로토콜을 실제 프로덕트에서 사용될 `URLSession`이  채택할 수도 있고, 테스트를 위한 mock 버전의 `MockNetworkEngine`이 채택 할 수도 있습니다.

```swift
extension URLSession: NetworkEngine {
    func perform(
        _ request: NetworkRequest,
        then handler: @escaping (Result<Data, Error>) -> Void
    ) {
        ...
    }
}

struct MockNetworkEngine: NetworkEngine {
    var result: Result<Data, Error>

    func perform(
        _ request: NetworkRequest,
        then handler: @escaping (Result<Data, Error>) -> Void
    ) {
        handler(result)
    }
}
```

이렇게 프로토콜을 통해 캡슐화하는 것은 비슷한 코드가 이곳저곳 흩어져 있는 것을 방지할 수 있습니다. 따라서 추후에도 의존성을 쉽게 제거하거나 교체할 수 있는 이점이 있습니다.

<br>

### 결론

Java의 Interface의 사용 목적과 그에 반하는 안티패턴, 그리고 적절하게 사용할 수 있는 방법에 대해 알아보았습니다. 또한 Swift의 Protocol의 사용 목적과 Apple이 제공하는 프레임워크를 통해 네가지 용법에 대해 알아보았습니다. 

또 한가지 기억해야 할 것은, 프로토콜이라고 무조건 -able 접미사만 붙이는 것이 아니라 용법에 따라 적절한 이름을 붙여야 어떤 기능을 하는지 정확하게 표현할 수 있고, 개발자간 소통에 드는 비용을 줄일 수 있을 것입니다.



<br>

### 참고

[The different categories of Swift protocols](https://www.swiftbysundell.com/articles/different-categories-of-swift-protocols/)
