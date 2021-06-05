# Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라

<br>

### 목차

- 정적 팩터리 메서드의 장점
  - 호출될 때마다 인스턴스를 새로 생성하지 않아도 됩니다.
  - 반환 타입의 하위 타입 객체를 반환할 수 있습니다.
  - 입력 매개변수에 따라 매번 다른 클래스 객체를 반환할 수 있습니다.
  - 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 됩니다.
- 정적 팩터리 메서드의 단점
  - 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없습니다.
- 정적 팩터리 메서드의 사용 예제
- 핵심 정리



-----



클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자입니다. 하지만 모든 프로그래머가 꼭 알아둬야 할 기법이 하나 더 있습니다. 클래스는 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있습니다. 그 클래스의 인스턴스를 반환하는 단순한 정적 메서드입니다. 

```swift
public static func valueOf(b: Bool) -> Bool {
  return b ? true : false
}
```

> 지금 얘기하는 정적 팩터리 메서드는 디자인 패턴의 팩터리 메서드(Factory Method)와 다릅니다. 디자인 패턴 중에는 이와 일치하는 패턴은 없습니다.

클래스는 클라이언트에 public 생성자 대신 (혹은 생성자와 함께) 정적 팩터리 메서드를 제공할 수 있습니다. 이 방식에는 장점과 단점이 모두 존재합니다. 먼저 정적 팩터리 메서드가 생성자보다 좋은 장점을 알아봅시다.



## 장점

### 첫 번째, 호출될 때마다 인스턴스를 새로 생성하지는 않아도 됩니다.

이 덕분에 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있습니다. 따라서 (특히 생성비용이 큰) 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 줍니다. 플라이웨이트 패턴([Flyweight pattern](https://ko.wikipedia.org/wiki/플라이웨이트_패턴))도 이와 비슷한 기법이라 할 수 있습니다.

반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있습니다. 이런 클래스를 인스턴스 통제(instance-controlled) 클래스라 합니다. 그렇다면 인스턴스를 통제하는 이유는 무엇일까요? 


인스턴스를 통제하면 클래스를 싱글턴(singleton)으로 만들 수도, 인스턴스화 불가(noninstantiable)로 만들 수도 있습니다. 

```swift
// 인스턴스화 불가
class NonInstanceClass {
    private init() { }
    
    public static func initMethod() -> NonInstanceClass {
        return NonInstanceClass()
    }
}
let n = NonInstanceClass.initMethod()
```



<br>

### 두 번째, 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있습니다.

 이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 '엄청난 유연성'을 선물합니다. 이러한 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있습니다. 


```swift
class Animal {
    private var hasWings: Bool?
    private var hasLegs: Bool?
    private var legCount: Int?
    
    init(hasWings: Bool? = false, hasLegs: Bool? = false, legCount: Int? = nil) {
        self.hasWings = hasWings
        self.hasLegs = hasLegs
        self.legCount = legCount
    }
    
    static func owl() -> Owl {
        return Owl(hasWings: true, hasLegs: true, legCount: 2)
    }
    
    static func snake() -> Snake {
        return Snake()
    }
}

final class Owl: Animal {
    func fly() {
        print("fly")
    }
}

final class Snake: Animal {
    func crawl() {
        print("crawl")
    }
}

let owl = Animal.owl()
let snake = Animal.snake()
```

위와 같이 작성하게 되면 Owl이나 Snake의 실제 구현 클래스를 알지 않아도 Animal을 통해 생성할 수 있게 됩니다. 

<br>

### 세 번째, 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있습니다. 

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관 없습니다. 


```swift
class Animal {
    private var hasWings: Bool?
    private var hasLegs: Bool?
    private var legCount: Int?
    
    init(hasWings: Bool? = false, hasLegs: Bool? = false, legCount: Int? = nil) {
        self.hasWings = hasWings
        self.hasLegs = hasLegs
        self.legCount = legCount
    }
    
    static func canFly(hasWings: Bool) -> Animal {
        return hasWings ? Owl(hasWings: true, hasLegs: true, legCount: 2) : Snake()
    }
}
```

<br>

### 네 번째, 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 됩니다.

서비스 제공자 프레임워크에 대해 설명해 놓은 글 - [이펙티브 자바 01. 정적 팩토리 메소드와 서비스 제공자 인터페이스 (JDBC 예제)](https://plposer.tistory.com/61)

- 자바 클래스를 컴파일하면 .class 파일로 변환됩니다. 여기에 해당 클래스의 정보가 들어있습니다.
- 실제 해당 라인이 (처음?)실행될 때 해당 클래스 정보가 메모리의 클래스 영역(메서드 영역)에 로드됩니다. (마치 인터프리터 언어처럼)
- 또한 Class.forName("com.mysql.jdbc.Driver");를 호출하면 런타임에 클래스 로더에 의해 해당 클래스 정보가 메모리에 로드됩니다.
- 어떤 클래스 정보가 메모리에 로드될 때는 그 클래스 안의 static 멤버들(static 필드, static block 등..)이 실행(분석?)되며, 그로 인해 예를 들어 static 메서드 안에 명시된 클래스의 정보 또한 클래스 영역에 로드됩니다. 이 때문에 구현체 클래스를 코드에 명시할 경우 이 클래스의 정보가 클래스 영역에 로드됩니다.

```java
class A {
  static AInterface of() {
    // 명시했으므로 클래스 A의 정보가 메모리에 로드될 때
    // Aimpl의 클래스 정보 또한 클래스 영역에 로드된다.
    // 그래야 of 메서드가 실행될 때 Aimpl을 인스턴스화해서 돌려줄 수 있음
    return Aimpl()
  }
}
```


(내가 생각하기에) 이 방법의 장점은 DB 드라이버의 구현체를 아무데서도 명시하지 않아서, 모든 드라이버들이 컴파일은 되지만, Class.forName으로 명시한 클래스만 메모리에 로드되는 듯합니다.
- 또한, 여기서 정적 팩터리 메서드를 이용하면 반환할 객체의 클래스를 명시하지 않고 인터페이스만으로 다룰 수 있어서 해당 클래스 정보가 메모리에 로드되지 않습니다.


<br>

## 단점

### 상속을 하려면 public이나 internal 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없습니다.


```swift
// 인스턴스화 불가
class NonInstanceClass {
  private init() { }

  public static func initMethod() -> NonInstanceClass {
    return NonInstanceClass()
  }
}

class InheritedClass: NonInstanceClass {
  
}

let inheritedClass = InheritedClass()	// 에러 발생 'InheritedClass' cannot be constructed because it has no accessible initializers
```

이런식으로 인스턴스화가 불가하도록 만들어진 클래스는 상속이 불가합니다. 불변타입으로 만들기 위해서는 이러한 제약이 장점으로 받아들여질 수도 있습니다. 

<br>

## 사용 예제

정적 팩터리 메서드가 유용하게 사용될 수 있는 두가지 방법에 대해 소개하고자 합니다.

- UI 요소 생성시
- 테스트 stub 객체 생성시

UIViewController의 하위 요소들을 configure할 때 보통은 아래와 같은 방법을 사용합니다.

```swift
class TitleLabel: UILabel {
    override init(frame: CGRect) {
        super.init(frame: frame)

        font = .boldSystemFont(ofSize: 24)
        textColor = .darkGray
        adjustsFontSizeToFitWidth = true
        minimumScaleFactor = 0.75
    }
}
```

위와 같은 접근방식에 문제가 있는 것은 아니지만, 우리는 종종 같은 종류의 UI에 대해 세부 요소만 다른 여러 개의 하위 클래스를 갖게 됩니다.(`TitleLabel`, `SubtitleLabel`, `FeaturedTitleLabel` 등) 

서브클래싱은 중요한 작업이지만, 현재 작업은 새로운 동작을 추가하는 것이 아닌 인스턴스화 하는 과정이어서 서브클래싱 하는 것이 이 목적에 부합한가 하는 의문이 듭니다. 그래서 어떤 동작을 가지고 있는 것이 아니라면, 정적 팩터리 메서드를 통해 새로운 인스턴스를 만들 수 있습니다.

```swift
private extension UILabel {
    static func makeForTitle() -> UILabel {
        let label = UILabel()
        label.font = .boldSystemFont(ofSize: 24)
        label.textColor = .darkGray
        label.adjustsFontSizeToFitWidth = true
        label.minimumScaleFactor = 0.75
        return label
    }
}
```

이런 접근의 장점은, 설정 부분을 실질적인 동작 부분과 분리할 수 있다는 것입니다. 또한 private 접근 제한자를 추가해 단일 파일로 범위를 지정할 수 있어서 앱의 일부에만 단일 기능을 가지도록 확장할 수 있습니다.

```swift
class ProductViewController {
    private lazy var titleLabel = UILabel.makeForTitle()
}
```

그럼 위와 같이 간단하게 UI 요소를 만들 수 있습니다. 



그리고 테스트 코드를 작성하는 경우가 있습니다. 특히 특정 모델에 의존하여 테스트 코드를 작성할 때, 보일러플레이트가 많이 발생하는 코드를 작성하게 되는 경우가 많아 읽기 어렵고 디버깅이 어려워질 수 있습니다. 

이럴 때 정적 팩터리 메서드로 stub 데이터를 가진 모델객체를 생성하게끔 만들어 놓으면, 테스트 시 해당 메소드만을 호출하여 간단하게 stub을 가져다 쓸 수 있습니다.

```swift
extension User {
    static func makeStub(permissions: Set<User.Permission>) -> User {
        return User(
            name: "TestUser",
            age: 30,
            signUpDate: Date(),
            permissions: permissions
        )
    }
}
```

정적 팩토리 메서드의 이름을 지정하여 메인 앱에 추가하지 않고 테스트용으로만 사용할 수 있습니다. 이렇게 하게되면 코드를 실제 로직과 명확하게 구분할 수 있고, 깨끗한 테스트 코드를 쉽게 작성하는데에 도움이 됩니다.

<br>

## 핵심 정리

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋습니다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고쳐야 합니다.



<br>

### 참고
https://www.swiftbysundell.com/articles/static-factory-methods-in-swift/

