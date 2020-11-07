# Item 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자입니다. 하지만 모든 프로그래머가 꼭 알아둬야 할 기법이 하나 더 있습니다. 클래스는 생성자와 별도로 정적 팩터리 메서드(static factory method)를 제공할 수 있다. 그 클래스의 인스턴스를 반환하는 단순한 정적 메서드입니다. 

```swift
public static func valueOf(b: Bool) -> Bool {
  return b ? true : false
}
```

> 지금 얘기하는 정적 팩터리 메서드는 디자인 패턴의 팩터리 메서드(Factory Method)와 다릅니다. 디자인 패턴 중에는 이와 일치하는 패턴은 없습니다.

클래스는 클라이언트에 public 생성자 대신 (혹은 생성자와 함께) 정적 팩터리 메서드를 제공할 수 있습니다. 이 방식에는 장점과 단점이 모두 존재합니다. 먼저 정적 팩터리 메서드가 생성자보다 좋은 장점을 알아봅시다.



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

- (내가 생각하기에) 이 방법의 장점은 DB 드라이버의 구현체를 아무데서도 명시하지 않아서, 모든 드라이버들이 컴파일은 되지만, Class.forName으로 명시한 클래스만 메모리에 로드되는 것인듯하다.
- 또한, 여기서 정적 팩터리 메서드를 이용하면 반환할 객체의 클래스를 명시하지 않고 인터페이스만으로 다룰 수 있어서 해당 클래스 정보가 메모리에 로드되지 않습니다.

<br>

이제 단점을 알아볼 차례입니다.

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



### 핵심 정리

정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋습니다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고쳐야 합니다.