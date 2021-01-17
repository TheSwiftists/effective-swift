# Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

두 가지 이상의 의미를 표현할 수 있으며, 그 중 현재 표현하는 의미를 태그 값으로 표시하는 클래스는 불필요한 코드가 많으며 가독성 면에서도 좋지 않습니다.

### 태그 달린 클래스 예시

이번 아이템에서 말하는 "태그 달린 클래스"의 문제점은 자바와 스위프트 두 언어 모두에서 거의 비슷하기 때문에 바로 스위프트 코드로 설명하겠습니다.

```swift
class Figure {
    enum Shape {
        case rectangle, circle
    }
    
    private let shape: Shape
    
    // 사각형일 때만 쓰이는 프로퍼티
    private let length: Double?
    private let width: Double?
    
    // 원일 때만 쓰이는 프로퍼티
    private let radius: Double?
    
    var area: Double? {
        switch shape {
        case .rectangle:
            guard let length = length, let width = width else { return nil }
            return length * width
        case .circle:
            guard let radius = radius else { return nil }
            return Double.pi * radius * radius
        }
    }
    
    init(length: Double, width: Double) {
        self.shape = .rectangle
        self.length = length
        self.width = width
        self.radius = nil
    }
    
    init(with radius: Double) {
        self.shape = .circle
        self.length = nil
        self.width = nil
        self.radius = radius
    }
}
```

위 태그 달린 클래스에는 문제점이 많습니다. 우선 여러 구현이 한 클래스에 혼합되어 있어서 가독성이 좋지 않습니다. 코드를 읽는 사람 입장에서, 모양이 어떤 경우인지 먼저 생각하고 읽어야 합니다.

또 다른 문제점은, 객체를 생성할 때 `shape`을 제외하고는 해당 모양에 쓰이지 않는 프로퍼티들에 0과 같은 의미 없는 값을 넣거나, `shape`을 제외한 모든 프로퍼티들을 옵셔널로 선언하여 `nil`을 입력해 두거나 해야 합니다. 의미 없는 값을 넣는 방식의 경우 프로그램이 런타임에 오동작할 위험이 높아지며, 옵셔널로 선언할 경우 옵셔널 언래핑 등의 추가 코드를 작성해야 합니다.

삼각형 등의 새 도형을 추가하려 한다면, `Figure` 클래스의 전반적인 수정이 필요합니다. 새 열거형 case와 저장 프로퍼티를 추가하고, 모든 switch 문을 찾아 새 case를 처리하는 코드를 추가해야 합니다. 스위프트에서는 자바와 달리 switch 문에서 모든 열거형 case가 처리되지 않으면 컴파일 에러를 발생시켜서 도와주지만, 그 부분들을 모두 수정해야 한다는 사실에는 변함이 없습니다.

마지막으로, 인스턴스의 타입만으로는 해당 인스턴스가 어떤 모양을 나타내고 있는지 알기 어렵습니다. 해당 인스턴스의 `shape` 프로퍼티를 보고 판단해야 하는데, 이는 런타임에 결정되므로 코드의 의미를 파악하기가 더 어려워집니다.

이렇게 태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적입니다.

### 계층 구조로 개선하기

태그 달린 클래스는 계층 구조로 만들어 개선하는 것이 좋습니다. 태그 달린 클래스를 계층 구조로 바꾸려면, 계층 구조의 root가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언합니다. 태그 값에 상관없는 공통적인 메서드나 프로퍼티들은 루트 클래스에 추가합니다.

그 다음, 루트 클래스를 확장한 구체 클래스를 각각 정의합니다. 여기서는 `Circle`과 `Rectangle`입니다. 각 구체 타입들에 각자의 의미에 해당하는 프로퍼티들을 추가하고, 추상 클래스에 정의된 메서드를 각 의미에 맞게 구현합니다.

저는 스위프트의 프로토콜이 인스턴스를 생성할 수 없으며 필요에 따라 선택적으로 extension을 통해 기본 구현을 제공할 수 있다는 점에서 추상 클래스와 유사하면서, 이번 예제에도 적합하다는 생각이 들어 프로토콜로 추상화하는 방법을 선택했습니다.

```swift
protocol Figure {
    var area: Double { get }
}

struct Circle: Figure {
    private let radius: Double
    
    var area: Double { return Double.pi * radius * radius }
    
    init(radius: Double) {
        self.radius = radius
    }
}

struct Rectangle: Figure {
    private let length: Double
    private let width: Double
    
    var area: Double { return length * width }
    
    init(length: Double, width: Double) {
        self.length = length
        self.width = width
    }
}
```

계층 구조로 구성된 클래스들은 태그 달린 클래스의 단점을 모두 해결해 줍니다. 각 타입이 간결하고 명확하며, 옵셔널 언래핑 코드, switch 문 등 불필요한 코드들도 모두 사라졌습니다. 클래스가 표현하지 않는 모양에 관련된 프로퍼티들도 없어졌기 때문에, 의미 없는 값을 넣거나 옵셔널로 선언하지 않아도 됩니다.

또 다른 장점은, 기존 코드의 변경 없이 계층구조를 확장하거나 새 도형을 추가할 수 있다는 점입니다. 예를 들어 삼각형을 추가하려 한다면, `Figure`의 변경 없이 `Triangle` 타입을 하나 더 구현하면 됩니다.

### 계층 구조 활용법

<center><img src="resources/buttons.png" width="200"></center>

위 이미지처럼, 비슷하지만 약간씩 다른 여러 종류의 버튼을 만들어야 한다면, 열거형으로 date, people, price 등의 case로 나누어 태그 달린 클래스로 구현하는 것 보다, Rounded Button을 상속한 세 가지 버튼으로 만드는 편이 더 좋습니다.

만약 새로운 필터 버튼을 추가하려 할 때도, 계층 구조로 설계했다면 변경에 유연하여 기존 코드의 변경 없이 새 필터 버튼을 추가할 수 있습니다.
