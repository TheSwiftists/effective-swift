# Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

### Java의 인터페이스 디폴트 메서드(Interface Default Method))

본문에서는 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 인터페이스(Interface)의 디폴트 메서드(Default Method)를 작성하기 어렵고 때문에 디폴트 메서드로 인해 기존 클라이언트를 망가뜨릴 수 있기 때문에 꼭 필요한 경우가 아니면 인터페이스를 구현하는 쪽으로 생각해 설계하라고 권고하고 있습니다. 

이렇듯 책에서는 인터페이스의 디폴트 메서드 구현을 지양하고 인터페이스를 구현하는 쪽으로 생각해 설계하라고 했는데, Swift에서는 어떨까요?



### Swift의 프로토콜 기본구현(Protocol Default Implementations)

Java에서 인터페이스의 디폴트 메서드를 작성할 수 있듯이 Swift에서도 프로토콜(Protocol)의 요구사항을 익스텐션(Extension)을 통해 구현할 수 있는데, 이를 [**프로토콜 기본구현(Protocol Default Implementations)**](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html#ID521) 이라고 합니다. 

제 생각에 Swift에서는 앞서 언급한 책의 내용과는 달리, 프로토콜의 구현을 지양하는 분위기는 아니라고 생각합니다. 

그 이유는 Swift에서(특히, 프로토콜 지향 프로그래밍(Protocol Oriented Programming)에서) 프로토콜을 익스텐션해서 프로토콜의 요구사항을 구현하는 것은 Swift가 가지고있는 강점 중 하나라고 보기 때문입니다. 

- 기존 타입의 소스 코드에 대한 액세스 권한이 없더라도 익스텐션으로 기존 타입을 확장하여 새 프로토콜을 채택하고 준수할 수 있도록 만들어줄 수 있고 익스텐션으로 기존 타입에 새로운 프로퍼티(저장 프로퍼티 제외), 메서드 및 서브스크립트(subscript)를 추가할 수 있어 프로토콜이 요구할 수 있는 요구사항을 추가할 수도 있습니다. (다만 프로토콜에서 저장 프로퍼티(Stored Property)를 구현할 수 없으므로 저장 프로퍼티는 프로토콜을 채택한 타입에서 직접 구현해야합니다.)  이렇게 프로토콜과 익스텐션을 사용하여서 기본 구현을 해두면 중복된 코드를 줄일 수 있는 장점이 있습니다. 
- 프로토콜에서 정의한 요구사항의 경우 익스텐션에서 기본 구현해 둔 기능을 사용하지 않을 때에는 재정의할 수 있습니다. 특정 프로토콜을 준수하는 타입에 프로토콜의 요구사항이 이미 구현되어 있다면 그 기능을, 그렇지 않다면 프로토콜 기본 구현 기능을 호출합니다. 이처럼 프로토콜 기본 구현을 통해 기능을 구현한다면 프로토콜 채택만으로 타입에 기능을 추가해 사용할 수 있습니다. 

<간단한 예시>

```swift
protocol TextRepresentable {
    var describeSelf: String { get }
}

extension TextRepresentable {
    func describeSelf() {
        print(self)
    }
}

extension Int: TextRepresentable { }
extension String: TextRepresentable { }
extension Double: TextRepresentable { }

8291.describeSelf() // 8291 
3.14.describeSelf() // 3.14 
"Effective Swift".describeSelf() // "Effective Swift"
```

또한 기존에 제공되는 프로토콜들의 익스텐션을 통한 기본 구현 코드를 볼 수는 없지만, 공개되어있는 내부 구현을 살펴봤을 때 스위프트의 많은 기능들이 프로토콜, 익스텐션, 제네릭의 조합으로 구현되어 있는 것 같습니다. 그 중에서 한 가지 예를 들어 살펴보자면, `Array`, `Set`, `Dictionary` 에서 공통으로 채택하고 있는 프로토콜들은 `CustomDebugStringConvertible`, `CustomReflectable`, `CustomStringConvertible`, `CustomDebugStringConvertible` 등이 있는데 타입들에 공통으로 들어가는 코드의 중복을 줄이기 위해 타입별로 공유하는 부분들을 각 프로토콜에서 익스텐션을 통해 기본 구현했을 것이라고 예상할 수 있습니다. 

