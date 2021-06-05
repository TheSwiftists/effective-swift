# item 24. 멤버 클래스는 되도록 static으로 만들라

### 개요
이번 아이템에서는 Java에서 사용하는 4가지 중첩 클래스와 언제 그리고 왜 사용해야하는지 살펴봅니다.

<br>

### 중첩 클래스의 종류
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스

<br>

#### 정적 멤버 클래스
흔히 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰입니다.

예제에서는 `enum`인 `Operation`(item 34)을 예로 들어주는데 Java의 경우 `enum`의 타입은 `class`이고 `enum` 상수 하나당 인스턴스가 만들어지며, 각각의 타입은 `public static final` 입니다.
그렇기 때문에 Java에서는 아래 예제 코드와 같이 상수 선언시 몸체를 만들 수 있습니다.

```java
public class Calculator {
    public enum Operation {
        PLUS {public double apply(double x, double y){return x + y}};
    }
}

// 호출시
Calculator.Operation.PLUS
```
이렇게 계산기(Calculator)가 지원하는 연산 종류를 정의해서 사용할 수 있겠네요.
<br>

하지만 Swift에서는 `static class`를 지원하지 않으며(정확히 말하자면 이미 static하게 되어있습니다.) enum 또한 저러한 형태로 사용되지 못합니다. 
최대한 호출하는 구조를 비슷하게 만들어보면 이러한 형식이 될것 같습니다.

```swift
class Calculator {
    enum Operation {
        case minus
        case plus
        case time
        case divide
        
        func calculate(x: Double, y: Double) -> Double {
            switch self {
            case .minus: return x - y
            case .plus:  return x + y
            case .time: return x * y
            case .divide: return x / y
            }
        }
    }
}

Calculator.Operation.minus.calculate(x: 3, y: 4)
```

<br>

#### (비정적) 멤버 클래스
어떤 클래스의 인스턴스를 감싸 마치 다른 클래스의 인스턴스처럼 보이게 하는 `어댑터`를 정의할때 자주 쓰입니다.

책에서는 Java코드로 자신의 반복자를 구현한다는 예제 코드가 올라와있는데, 해당 코드를 이해하는데 어려움이 있어서 중첩 클래스로 어댑터를 구현하는 대신에 `어댑터 패턴`을 예시로 들겠습니다.

추후 보충하도록 하겠습니다.
```swift
// 기본적으로 사용되고 있는 Target Class
class Target {
    func request() -> String {
        return "Target: The default target's behavior."
    }
}

// Target과 동작은 비슷하지만 같은 타입은 아님.
class Adaptee {
    public func specificRequest() -> String {
        return ".eetpadA eht fo roivaheb laicepS"
    }
}

// 어댑터를 통해 타입을 맞춰준다.(클래스 래핑) Adaptee -> Traget
class Adapter: Target {
    private var adaptee: Adaptee

    init(_ adaptee: Adaptee) {
        self.adaptee = adaptee
    }

    override func request() -> String {
        return "Adapter: (TRANSLATED) " + adaptee.specificRequest().reversed()
    }
}

// 사용자는 target만을 주입받음.
class Client {
    // ...
    static func someClientCode(target: Target) {
        print(target.request())
    }
    // ...
}


Client.someClientCode(target: Target())
let adaptee = Adaptee()
Client.someClientCode(target: Adapter(adaptee))
```

#### 익명 클래스
Swift에서는 지원하지 않아 다루지 않겠습니다.

#### 지역 클래스

지역클래스는 메소드 내부의 지역변수처럼 선언해서 쓸수 있으나 Swift와의 용법에서는 맞지 않는 것 같아 역시 익명 클래스와 마찬가지로 다루지 않겠습니다.

<br>

### Swift는 중첩클래스를 언제 흔하게 사용할까?
Swift는 Java처럼 중첩클래스를 자세하게 나누어서 쓰는 용법은 흔하게 볼수 있지 않았습니다.
보통 namespace를 정의할때 많이 쓰이는것 같았습니다.

Java의 경우 정적 멤버 클래스를 제외하고 자동으로 외부 클래스의 인스턴스에 대한 참조를 가지기 때문에 외부 클래스의 인스턴스가 있는 경우에만 내부 클래스의 인스턴스를 만들 수 있습니다.

Swift에서는 내부 클래스의 인스턴스는 외부 클래스의 인스턴스와 독립적입니다.
따라서 인스턴스화 할때 차이가 있는 것을 보실 수가 있습니다.

#### Swift 예시 코드

```Swift
class Master {
    var testProperty = 2;

    class Nested{
        init() {
            // ...
        }
    }

    func foo() {
        // ...
    }
}

var master = Master()
// Java랑은 다르게 Master의 인스턴스가 필요없다.
var nested = Master.Nested()
nested.foo()

// error : Static member 'Nested' cannot be used on instance of type 'Master'
var nested2 = master.Nested()
```

#### Java 예시 코드
```java
public class Master {
    private static int testProperty = 100;
    
    class Nested {
        private int testProperty = 200;
        
        public void display() {
            // ...
        }
    }
}

Master out = new Master();
// Master의 인스턴스가 존재해야지만 인스턴스화 가능
Master.Nested in = out.new Nested();
        
in.display();
```

<br>

### 참고한 곳
- https://johngrib.github.io/wiki/java-enum/
- https://refactoring.guru/design-patterns/adapter/swift/example
- https://stackoverflow.com/questions/26806932/swift-nested-class-properties
- https://academy.realm.io/kr/posts/swift-namespace-typealias/
- https://onelife2live.tistory.com/15
- https://gyrfalcon.tistory.com/entry/JAVAJ-Nested-Class [Minsub's Blog]
