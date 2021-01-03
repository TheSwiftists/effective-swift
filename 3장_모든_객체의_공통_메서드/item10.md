# Item 10. equals는 일반 규약을 지켜 재정의하라

Item 10에서는 equals를 재정의하기에 적합한 상황을 설명하고, 재정의할 때 지켜야 할 규약들을 설명합니다.

### equals를 재정의하기 적합한 경우

자바의 Object는 equals의 기본 구현을 제공하는데, 이 메서드에서는 두 레퍼런스 변수가 같은 인스턴스를 가리키고 있는지를 비교합니다.

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

만약 두 객체가 물리적으로 같은지가 아니라 논리적으로 같은지를 비교하려 한다면, 위 메서드를 재정의하여 사용할 수 있습니다.

### 스위프트에서는…

스위프트에서는 커스텀 타입에 Equatable 프로토콜을 채택하는 방식으로 객체 간의 논리적 동치 확인을 구현할 수 있습니다. 또한 Equatable을 채택하여 구현한 객체들은 `==` 연산자를 이용해 같은지를 비교할 수 있으며, Equatable을 채택한 객체들로 이루어진 컬렉션에서는 `firstIndex(of:)`, `contains()` 등의 메서드들을 사용할 수 있습니다.

### equals를 재정의할 때 지켜야 할 규약들

컬렉션 클래스들을 포함한 수많은 클래스들에서는 equals 메서드를 사용하고 있으며, equals가 특정 규약들을 지킨다고 가정하고 구현되어 있으므로 equals를 재정의할 때는 이 규약들을 반드시 따라야 합니다.

- Reflexivity(반사성)
- Symmetry(대칭성)
- Transitivity(추이성)
- Consistency(일관성)
- Non-nullity

### 스위프트에서는…

스위프트 또한 Equatable에 의존하는 여러 타입들과 메서드들이 있어서, Equatable을 따르는 커스텀 타입들 또한 몇몇 규약을 만족해야 합니다. — 참조: [Equatable](https://developer.apple.com/documentation/swift/equatable/1539854)

다만, 자바에서는 Object의 equals 메서드를 재정의하는 방식으로 논리적 동치 확인을 구현하지만, 스위프트에서는 프로토콜을 채택하는 방식으로 구현하도록 만들어 위에서 언급한 규약들을 지키는 데에 도움을 주는 것 같습니다.

예를 들어 책에 쓰여 있는 올바른 equals 메서드 구현 단계 중, `instanceof` 연산자를 이용해 입력되는 인스턴스의 타입이 올바른지 확인하고 해당 타입으로 형변환하는 절차가 있습니다. 자바에서는 equals 메서드의 파라미터 타입이 항상 Object이기 때문에 거쳐야 하는 절차들입니다.

```swift
public protocol Equatable {
    static func == (lhs: Self, rhs: Self) -> Bool
}
```

반면 Equatable 프로토콜에서는 left hand side와 right hand side 파라미터들의 타입이 둘 다 Self 키워드로 명시되어 있어서, 구현체에서 `==` 연산자를 구현할 때 타입이 다른 경우를 고려하지 않아도 되며, 번거로운 형변환 작업도 필요없습니다.

또한, (의도된 것인지는 모르겠지만) class가 아닌 static으로 선언되어 있어 하위 클래스에서 오버라이드할 수 없도록 강제해 놓았습니다. 책에서 “구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다”라고 쓰여 있는데, 어차피 규약을 만족시킬 방법이 없으므로 오버라이딩을 막아둔 것이 아닐까 생각됩니다.

자바보다 자율성은 낮지만, 프로그램이 의도치 않게 런타임에 이상 동작할 가능성을 줄이기 위해 최대한 컴파일 시점에 문제를 발견하도록 설계한 스위프트의 언어적 특성이 보입니다.

### 상속 관계에서의 Equivalence 확인 방법 제안

[코드스쿼드 자판기 앱 프로젝트](https://github.com/seizze/swift-vendingmachineapp)에서 음료수 객체들 간의 상속 관계를 구현하였는데, 이때 인스턴스들 간의 동치를 확인할 때 hashValue를 이용하였습니다.

먼저, 가장 상위 클래스에서는 Hashable을 채택하여 `==` 연산자와 `hash(into:)` 메서드를 구현합니다.

```swift
class Beverage: Hashable {
    
    let price: Int
    let name: String
    
    init(price: Int, name: String) {
        self.price = price
        self.name = name
    }
    
    func hash(into hasher: inout Hasher) {
        hasher.combine(price)
        hasher.combine(name)
    }
    
    static func == (lhs: Beverage, rhs: Beverage) -> Bool {
        return lhs.hashValue == rhs.hashValue
    }
}
```

이 때 `==` 연산자에서는 hashValue가 같은지를 비교합니다. `hash(into:)` 메서드에서는 동치를 확인하기 위한 클래스 안의 핵심 프로퍼티들을 사용합니다.

```swift
class Coffee: Beverage {
    
    private let caffeineContent: Int
    private let temperature: Int
    
    init(
        price: Int,
        name: String,
        caffeineContent: Int,
        temperature: Int
    ) {
        self.caffeineContent = caffeineContent
        self.temperature = temperature
        super.init(price: price, name: name)
    }
    
    override func hash(into hasher: inout Hasher) {
        super.hash(into: &hasher)
        hasher.combine(caffeineContent)
        hasher.combine(temperature)
    }
}
```

하위 클래스에서는 `==` 연산자가 아닌 `hash(into:)` 메서드를 오버라이드하며, 상위 `hash(into:)` 메서드를 호출 후 하위 클래스의 핵심 필드를 hash 계산에 포함시킵니다.

Beverage와 Coffee의 인스턴스들을 비교한 결과는 다음과 같습니다.

```swift
let beverage = Beverage(price: 1000, name: "A")
let coffee = Coffee(price: 1000, name: "A", caffeineContent: 32, temperature: 99)

print(beverage == coffee) // false
```

Coffee 클래스에서 `hash(into:)`를 오버라이드하지 않았을 경우, `print(beverage == coffee)`는 true를 출력합니다.

### 주의할 점

위 방법은 인스턴스의 개수가 아주 많아졌을 경우, 해시 충돌 가능성이 있으므로 주의해야 합니다.

### References

- [Equatable](https://developer.apple.com/documentation/swift/equatable/1539854)
- [코드스쿼드 자판기 앱 프로젝트](https://github.com/seizze/swift-vendingmachineapp)
- [자판기 앱 PR #204](https://github.com/code-squad/swift-vendingmachineapp/pull/204)
