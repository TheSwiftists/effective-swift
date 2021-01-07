# item 29.  이왕이면 제네릭 타입으로 만들라

클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편리합니다. 따라서 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하는 것이 좋고 그러기 위해 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하는 것을 권장하고 있습니다.

### 제네릭의 작동 방식

```swift
func min<T: Comparable>(_ x: T, _ y: T) -> T {
       return y < x ? y : x
}
```

컴파일러에는 이 함수를 위해 코드를 보내는데 필요한 두 가지 필수 정보인 1. _T타입 변수의 사이즈_와 2. _런타임에 호출해야하는_ `<` 메서드의 특정 오버로드 주소가 없습니다.

컴파일러가 제네릭 타입을 가진 값(value)을 발견할 때마다 해당 값을 컨테이너에 보관합니다. 이 컨테이너는 값을 저장할 수 있는 고정 크기가 있습니다. 값이 너무 크면 Swift는 힙(heap)에 할당하고 해당 컨테이너에 대한 참조(주소값)를 저장합니다.

컴파일러는 또한 제네릭 타입 매개 변수당 하나 이상의 감시 테이블(witness table)의 리스트를 유지 관리합니다. 하나는 값 감시 테이블(value witness table)이고, 또 하나는 해당 타입의 각 프로토콜 제약 조건에 대한 프로토콜 감시 테이블(protocol witness table)입니다. 감시 테이블은 런타임에 올바른 구현에 대한 함수 호출을 동적으로 디스패치하는 데 사용됩니다.(The witness tables are used to dynamically dispatch function calls to the correct implementations at runtime.)

이 내용에 대해 더 자세한 내용은 '참고 - Witness Table에 대해 참고한 자료'에 있는 자료들을 참고해주세요.

### 제네릭 타입으로 해결 가능한 문제

제네릭 타입을 사용하는 대표적인 예는 Stack(스택)입니다. 일반적인 스택의 특징은 아래와 같습니다. 

* 스택의 특징
  1. 스택의 요소로 한 타입을 지정해주면 그 타입으로 계속 스택이 동작 
  2. 처음 지정해주는 타입은 스택을 사용하고자 프로그래머가 지정할 수 있음.
     스택의 인스턴스를 생성할 때 실제로 어떤 타입을 사용할지 명시.

제네릭 타입을 사용했을 때와 사용하지 않았을 때를 비교하여 살펴보겠습니다.

< 제네릭 타입을 사용하지 않았을 경우 >

타입에 따른 Stack을 일일이 구현해줘야 합니다. 

```swift
struct IntStack {
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
}

// 사용 예
var integerStack: IntStack = IntStack()
integerStack.push(3)
print(integerStack.items) // [3]
integerStack.pop()
print(integerStack.items) // []
```

<. 제네릭 타입을 사용했을 경우 >

하지만 제네릭 타입을 사용하여 Stack을 구현했을 경우 모든 타입을 대상으로 동작할 수 있기 때문에 타입별 Stack을 일일이 구현할 필요가 없습니다.

```swift
struct Stack<Element> {
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}

// 사용 예
var doubleStack: Stack<Double> = Stack<Double>()
doubleStack.push(1.0) 
print(doubleStack.items) // [1.0]
doubleStack.pop()
print(doubleStack.items) // []

var stringStack: Stack<String> = Stack<String>()
stringStack.push("Effective Swift") 
print(stringStack.items) // ["Effective Swift"]
stringStack.pop()
print(stringStack.items) // []

// Any 타입을 사용하면 요소로 모든 타입을 수용할 수 있습니다.
var anyStack: Stack<Any> = Stack<Any>()
anyStack.push("Effective Swift") 
print(anyStack.items) // ["Effective Swift"]
anyStack.push(3.0) 
print(anyStack.items) // ["Effective Swift", 3.0]
anyStack.pop()
print(anyStack.items) // ["Effective Swift"]
```

이렇게 제네릭 타입을 사용하면 **훨씬 유연하고 광범위**하게 사용할 수 있습니다. 또한 Element의 타입을 정해주면 그 타입에만 동작하도록 제한할 수도 있기 때문에 프로그래머가 의도한 대로 기능을 사용하도록 유도할 수 있습니다.

### 타입 제약 (Type Constraints)

제네릭 타입은 타입의 제약 없이 사용할 수 있지만, 때로는 아래와 같이 타입 제약이 필요한 상황이 있을 수 있습니다. 

- 제네릭 함수가 처리해야 할 기능이 특정 타입에 한정되어야만 처리할 수 있는 경우
  Ex) 아래 '타입 제약 예시'의 `substractTwoValue(_: _:)` 뺄셈 함수의 경우,
  _뺄셈 연산자를 사용할 수 있는 타입_이어야 하기 때문에 매개변수를 BinaryInteger 프로토콜을 준수하는 타입으로 한정.

- 제네릭 타입을 특정 프로토콜을 따르는 타입만 사용할 수 있도록 제약을 두어야 하는 경우

  등

이런 경우, 타입 제약을 사용하여 타입 매개변수가 가져야 할 제약사항을 지정할 수 있습니다. 이때, 타입 제약은 클래스 타입 또는 프로토콜로만 줄 수 있습니다.

<타입 제약 예시> 

```swift
// Dictionary의 키는 Hashable 프로토콜을 준수하는 타입으로만 사용 가능
public struct Dictionary<Key: Hashable, Value: Collection> : Collection, ExpressibleByDictionaryLiteral { /* 상세 구현부 생략 */ }>

// T를 뺼셈 연산자를 사용할 수 있는 BinaryInteger 타입으로 제한
func substractTwoValue<T: BinaryInteger>(_ a: T, _ b: T) -> T {
  return a - b
}

// 여러 제약을 추가하고 싶을 때 - where 절 사용
// T를 BinaryInteger 프로토콜을 준수하고 FloatingPoint 프로토콜도 준수하는 타입으로 제약
func swapTwoValues<T: BinaryInteger>(_ a: inout T, _ b: inout T) where T: FloatingPoint { /* 상세 구현부 생략 */ }

```



### Any vs Generic

Swift에서는 불특정 타입(nonspecific types) 작업을 위해 제공하는 두 가지 특수 타입(Any, AnyObject) 중 하나로 `Any`는 함수 타입을 포함해 모든 타입의 인스턴스를 나타낼 수 있습니다. 위의 '제네릭 타입으로 해결 가능한 문제 - <제네릭 타입으로 해결 가능한 문제>의 `anyStack`'에서 알 수 있듯이 **Any**는 타입을 고정하지 않고 계속해서 변경할 수 있습니다. **Any**로 선언된 변수의 값을 가져다 쓰려면 매번 타입 확인 및 변환을 해줘야 하기 때문에 불편할 뿐더러 예기치 못한 오류의 위험을 증가시키기 때문에 사용을 지양해야 합니다. 때문에 타입 안전성을 중요시하는 프로그래밍 언어인 Swift에서는 **Any**는 될 수 있으면 사용하지 않는 것을 권장하고 있습니다.

반면 **Generic**은 제네릭을 사용하여 구현한 타입(struct, class 등)의 인스턴스를 생성할 때 실제로 어떤 타입을 사용할지 지정해준 이후로 Any와 같이 변경할 수 없습니다. 

<br>

번외로 연관 타입에 대해서도 조사해봤습니다.

### 연관 타입 (Accociated Type)

위에서 알아본 제네릭을 정리하자면, 어떤 타입이 들어올지 모를때 타입 매개변수를 통해 종류는 알 수 없지만 어떤 타입이 여기에 쓰일 것이라는 걸 표시해주는 것을 의미합니다. 타입 매개변수의 이러한 역할을 프로토콜에서 수행할 수 있도록 만들어진 기능이 바로 연관타입입니다.

<연관 타입 예시>

``` swift
protocol Container {
    associatedtype Item // 연관 타입
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

struct IntStack: Container {
    // original IntStack implementation
    var items = [Int]() 
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias Item = Int // 🙌🏻
    mutating func append(_ item: Int) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
}

```

`IntStack`에서 `typealias Item = Int` (🙌🏻 표시) 는 Container 프로토콜의 구현을 위해 추상 타입(abstract type) 이었던 items를 구체 타입(concrete type) Int로 바꿉니다.

### 참고

* [Generic - The Swift Programming Language](https://docs.swift.org/swift-book/LanguageGuide/Generics.html#ID184) 
* [Power of Swift Generics — Part 1](https://medium.com/swift-india/power-of-swift-generics-part-1-ab722a030dc2)
* [The Swift Programming Language Swift 5.3 - Type Casting for Any and AnyObject](https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html#ID342)
* Witness Table에 대해 참고한 자료
  1. [WWDC2015 Understanding Swift Performance - Protocol Witness Table](https://developer.apple.com/videos/play/wwdc2016-416/?time=1570)
  2. [WWDC2015 Understanding Swift Performance - Value Witness Table](https://developer.apple.com/videos/play/wwdc2016/416/?time=1681)
  3. [Understanding method dispatch in Swift](https://heartbeat.fritz.ai/understanding-method-dispatch-in-swift-684801e718bc)
  4. [Apple Github - Type Layout](https://github.com/apple/swift/blob/main/docs/ABI/TypeLayout.rst)
  5. [Why does Swift need witness tables?](https://softwareengineering.stackexchange.com/a/331993)

