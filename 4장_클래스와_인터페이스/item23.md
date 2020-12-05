# Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

두 가지 이상의 의미를 표현할 수 있으며, 그 중 현재 표현하는 의미를 태그 값으로 표시하는 클래스는 불필요한 코드가 많으며 가독성 면에서도 좋지 않습니다.

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

우선, 해당 모양에 쓰이지 않는 프로퍼티들까지 초기화해야 합니다.

스위프트는 init 시점에 모든 프로퍼티를 초기화시켜야 하므로 옵셔널로 선언

예를 들어 삼각형 기능을 추가하려 한다면, 열거형에 case를 하나 더 추가 후 이 열거형이 사용되는 모든 switch문을 찾아 수정해야 합니다.

스위프트에서는 switch 문이 모든 열거형의 case를 처리하지 않을 경우 컴파일 에러를 발생시켜 도와주지만, 모든 switch 문들을 찾아 수정해야 한다는 사실에는 변함이 없습니다.

마지막으로, 인스턴스의 타입 만으로는 해당 인스턴스가 어떤 모양을 나타내고 있는지 알기 어렵습니다.

위와 같은 클래스는 계층 구조로 만들어 개선하는 것이 좋습니다.

```swift
protocol Figure {
    var area: Double { get }
}

class Circle: Figure {
    private let radius: Double
    
    var area: Double { return Double.pi * radius * radius }
    
    init(radius: Double) {
        self.radius = radius
    }
}

class Rectangle: Figure {
    private let length: Double
    private let width: Double
    
    var area: Double { return length * width }
    
    init(length: Double, width: Double) {
        self.length = length
        self.width = width
    }
}
```

태그 달린 클래스를 계층 구조로 바꾸려면, 계층 구조의 root가..

### 계층 구조 활용법

<center><img src="resources/buttons.png" width="200"></center>

이렇게 비슷하지만 약간씩 다른 버튼을 만들어야 한다면, enum으로 case를 나누지 말고, Rounded Button을 상속한 세 가지 버튼으로 만드는 것이 좋습니다.

새로운 버튼을 추가할 때도 변경에 유연
