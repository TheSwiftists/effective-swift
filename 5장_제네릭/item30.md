# item30. 이왕이면 제네릭 메서드로 만들라

### 개요 
제네릭 메서드에 대해서 알아봅니다.

<br>

### Swift에서 Generic 메서드 만들어보기
Java와 선언하는 방식이나 개념이 크게 다르지 않습니다.
Swift에서는 이렇게 정의하고 있습니다.
> 어떠한 타입과도 같이 사용할 수 있는 유연하고, 재사용 가능한 함수와 타입을 작성할 수 있습니다.

Java에서는 `Collections` 내부의 일부 메서드들(`sort`, `binarySearch`)등은 제네릭 메서드로 되어있고, Swift에서는 기본 제공 클래스 중`Array`, `Dictionary`가 제네릭 콜렉션입니다. 

그 중 `Array`를 살펴보면 타입이 `Array<Element>`이고, `append`, `shuffle` 등과 같은 메서드가 제네릭 메소드로 선언되어 있습니다. 

<br>

#### 제네릭 메서드 예제
제네릭 메서드는 아래 예제 코드들을 통해 설명드리겠습니다.

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
```
이렇게 `Int`타입인 두 값(someInt, anotherInt)을 바꿔주는 메서드가 있습니다.

유용한 함수긴 하지만 매개변수로 `Int`타입인 값밖에 사용할 수가 없습니다. 혹여 다른 타입을 사용하려고 하면 똑같은 함수를 타입만 바꿔서 만들어야 할것입니다.

```swift
func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someDouble = 6.0
var anotherDouble = 99.0
swapTwoDoubles(&someDouble, &anotherDouble)
```

이렇게 Double타입인 두 값을 바꿔주는 메서드를 만들어 보았습니다.
그럼 String이나 다른 타입을 바꿔주고 싶으면 메서드를 또 만들어야 할까요?🧐

위와 아래의 메서드를 보시면 아시겠지만 똑같은 로직을 가지고 있고 다른점은 오직 받아들이는 값의 타입입니다.

그럼 이제는 제네릭 메서드를 만들어 어떤 타입의 두 값이라도 바꿔줄 수 있는 메서드를 작성해보겠습니다.

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

이렇게 메서드명 옆에 자리표시용 타입인 `<T>`를 붙여 Swift에게 제네릭 메서드임을 알려주게 됩니다.
`T`의 자리에 사용될 실제 타입은` swapTwoValues(_:_:)` 함수를 호출하는 매 순간 결정됩니다.

<br>

#### 타입 매개 변수
위에서 보았던 `T`는 타입 매개 변수의 한 가지 예입니다.
타입 매개 변수는 자리 표시용 타입을 지정하고(함수이름 바로 뒤에`<T>` 붙임) 이름을 정하는 것입니다.

타입 매개 변수를 지정하게 되면, 이를 사용해서 매개 변수의 타입을 정의하거나, 반환 타입을 지정할 수 있습니다.

하나 이상의 타입 매개 변수를 제공하려면 `<>`안에 쉼표로 구분하여 작성할 수 도 있습니다.
예를 들면 `Dictionary<Key, Value>`가 되겠네요.

<br>

#### 타입 매개 변수 이름짓기
대부분의 경우 `descriptive name`을 가지게 되는데 위에 언급되었던 `Dictionary`의 `Key`, `Value` 및 `Array`의 `Element`를 예를 들 수 있겠습니다.

이처럼 타입 매개 변수와 이것을 사용하는 제네릭 타입/메서드 사이의 관계에 대해서 말해주지만, `swapTwoValues(_:_:)`처럼 의미있는 관계가 아닐때는 `T`, `U`와 같은 단일 문자 + Upper camel case를 사용하여 이름을 짓는 것이 전통적이라고 설명하고 있습니다.

<br>

### 참고한 곳
- https://docs.swift.org/swift-book/LanguageGuide/Generics.html
- http://xho95.github.io/swift/language/grammar/generic/2020/02/29/Generics.html
- https://zeddios.tistory.com/226
