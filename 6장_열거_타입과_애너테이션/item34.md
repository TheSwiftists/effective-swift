# Item 34. int 상수 대신 열거 타입을 사용하라



### 열거 타입(Enumerations)이 있기 이전

자바에 열거 타입을 지원하기 전에 사용했던 패턴으로는 정수 열거 패턴(int enum pattern)과 문자열 열거 패턴(string enum pattern) 등이 있습니다. 저자는 이 두 가지 패턴의 문제점에 대해서 지적하고 있습니다. 자세한 설명은 책에서 상세히 설명해주고 있으므로 간략하게만 짚고 넘어가겠습니다.

* **정수 열거 패턴(int enum pattern)**: 
  1. 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않습니다. 
  2. 자바는 정수 열거 패턴을 위한 별도 이름공간(namespace)를 지원하지 않기 때문에 접두어를 써서 이름 충돌을 방지해야 합니다. 
  3. 정수 열거 패턴을 사용한 프로그램은 깨지기 쉽습니다. 평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨지기 때문에 상수 값이 바뀌면 클라이언트도 반드시 다시 컴파일해야합니다.
  4. 정수 상수는 문자열로 출력하기에 까다롭습니다. 그 값을 출력하거나 디버거로 살펴보면 의미를 담은 문자열이 아니라 그냥 숫자로 보여서 직관적으로 파악하기 어렵습니다. 
  5. 같은 정수 열거 그룹에 속한 모든 상수를 한 바퀴 순회하는 방법도 마땅치 않습니다. 심지어 그 안에 상수가 몇 개인지 알 수도 없습니다.
* **문자열 열거 패턴(string enum pattern)**: 
  1. 상수의 의미를 출력할 수 있다는 점은 좋지만 경험이 부족한 프로그래머가 문자열 상수의 이름대신 문자열 값을 그대로 하드코딩하게 만들기 때문에 좋지 않습니다. 
  2. 오타가 있어도 컴파일러는 확인할 길이 없어 자연스럽게 런타임 버그가 생길 가능성이 있습니다. 
  3. 문자열 비교에 따른 성능 저하가 야기될 수 있습니다.

### Java에서의 열거타입(Enumerations)

* 자바의 열거 타입은 완전한 형태의 클래스로 주로 단순히 정수 타입 값의 별칭 형태로 사용되는 다른 언어의 열거 타입보다 훨씬 강력합니다. 
* 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개합니다.
* 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final입니다. 
  따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재한다는게 보장됩니다.(인스턴스 통제됨)
* 열거타입은 컴파일 타입 안전성을 제공합니다.

**열거 타입을 사용하는 경우**

저자는 필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하라고 권고하고 있습니다. 또한 열거 타입은 나중에 상수가 추가 되어도 바이너리 수준에서 호환되도록 설계되었기 때문에 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없습니다.

**핵심 정리**

Java에서 열거 타입은 확실히 정수 상수보다 뛰어납니다. 더 읽기 쉽고 안전하고 강력합니다. 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요합니다. 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있습니다. 이런 열거 타입에서는 switch 문 대신 상수별 메서드 구현을 사용하라고 권장합니다. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하는 것이 좋습니다.

<br><br>

**그렇다면 Swift 에서의 열거 타입(Enumerations)에 대해서 알아보겠습니다.**

### Swift에서의 열거 타입(Enumerations)

 **C 또는 Objective-C vs Swift**

열거 타입은 연관된 항목들을 묶어서 표현할 수 있는 타입입니다. 

앞서 살펴본 봐와 같이 자바나 스위프트의 열거 타입과 C언어 등에서의 열거 타입은 사용의 방향이 다릅니다. 주로 기존의 언어와 비교해봤을 때 Swift에서는 좀 더 진보된 enum을 제공하는 것 같습니다. 

기존의 C나 Objective-C 언어 등에서 열거 타입은 주로 Int 타입의 값에 의미를 명확히 하기 위한 이름을 부여하는 목적으로 사용되었습니다. 하지만 스위프트에서 enum은 [1급 객체](https://ko.wikipedia.org/wiki/일급_객체) 로 하나의 타입으로 사용할 수 있습니다. 

> ```swift
> enum CompassPoint {
>     case north
>     case south
>     case east
>     case west
> }
> ```
>
> C나 Objective-C 와는 다르게 Swift에서 열거형은 생성될 때 각 case 별로 기본 integer값을 할당하지 않습니다. 위 `CompassPoint`를 예로 들면, north, south, east, west는 각각 암시적으로 0, 1, 2, 3값을 갖지 않습니다. 대신 Swift에서 열거형의 각 case는 CompassPoint으로 선언된 온전한 값입니다.
> -출처: Enumerations - the swift programming languageswift 5.3

> ... 그렇기 때문에 모든 열거형의 데이터 타입은 같은 타입(주로 정수 타입)으로 취급합니다. 
> 이는 열거형 각각이 고유의 타입으로 인식될 수 없다는 문제 때문에 여러 열거형을 사용할 때 프로그래밍의 실수로 인한 버그가 생길 수도 있습니다. 
> 그러나 스위프트의 열거형은 각 열거형이 **고유의 타입**으로 인정되기 때문에 실수로 버그가 일어날 가능성을 원천 봉쇄할 수 있습니다.
> -출처: 스위프트 프로그래밍 Swift5 3판(지은이: 야곰) 4.5 열거형

<br>**Swift의 Enum의 특징**

1. 배열이나 딕셔너리 같은 타입과 다르게 프로그래머가 정의해준 항목 값 외에는 추가/수정이 불가합니다. 
2. 항목별로 원시값(rawValue)을 가질 수도, 가지지 않을 수도 있습니다. 
   예를 들어 C언어는 열거타입의 각 항목 값이 정수 타입으로 기본 지정되지만, 스위프트의 열거타입은 각 항목이 그 자체로 고유의 값이 될 수 있습니다. 
3. Swift에서는 열거 타입은 case 값으로 `string`, `character`, `integer`, `floting` 타입을 사용할 수 있습니다. 
4. 계산 프로퍼티(computed property)를 가질 수 있습니다. enum의 현재 값에 대한 추가적인 정보를 제공하기 위한 수단으로 사용됩니다.
5. 인스턴스 메소드(instance method)를 가질 수 있습니다. enum이 나타내는 값에 관련된 추가 기능을 제공합니다.
6. 생성자(initializer)를 가질 수 있습니다. enum의 기본 값을 제공하기 위해 사용합니다.
7. 익스텐션(extension)을 적용할 수 있습니다.
8. 프로토콜(protocol)을 채택할 수 있습니다.
9. 연관값(Associated Value)을 사용하여 다른 언어에서 공용체라고 불리는 값의 묶음도 구현할 수 있습니다.
10. 저장 프로퍼티를 가질 수 없고 enum은 값 타입(value type)이라는 점에서 클래스와 차이가 있습니다.

<br>

**enum의 raw value 지정 규칙**

Swift의 enum은 기본 타입으로 Int를 사용하지 않고 아래와 같은 규칙을 갖습니다.

1. enum의 타입 명시를 하지 않은 경우 
   : enum의 모든 케이스들은 값을 가지지 않습니다. 따라서 값을 기반으로 하는 생성자도 사용할 수 없습니다.
2. 정수 / 실수 타입으로 지정한 경우
   : 첫 case의 기본 값은 0으로 설정 되고, 이후에는 이전 케이스의 값에서 1을 더한 값을 사용합니다. 값을 제공해 줄 경우 제공된 값을 우선적으로 사용합니다.
3. String 타입으로 지정한 경우: case명을 값으로 사용합니다. 값을 제공해 줄 경우 제공된 값을 우선적으로 사용합니다.
4. Character타입으로 지정한 경우: 컴파일러가 값을 줄 수 없습니다. 모든 case에 대해 명시적으로 값을 제공해야만 합니다.

<br>

**enum의 모든 경우를 순회하고 싶을 때**

enum에 대한 모든 경우를 순회하고 싶을 때 **CaseIterable** 프로토콜을 적용하면, 컴파일러가 `allCases`라는 프로퍼티를 추가해주며  `allCases` 프로퍼티를 통해 case의 개수, 전체 case 순회 등이 가능해집니다.

```swift
enum Beverage: CaseIterable {
    case coffee, wine, tea
}

let numberOfCafeMenu = Beverage.allCases.count
print("\(numberOfChoices) beverages available") //3 beverages available

for beverage in Beverage.allCases {
    print(beverage)
}
// coffee
// wine
// tea
```

<br>

**Associated Value(연관 값)**

swift의 enum에는 Associated Value(연관 값) 기능이 있습니다. 연관 값은 특정 case에 대한 추가 정보를 저장하기 위한 것으로 enum을 사용하는 문맥에 따라 다른 값을 가질 수 있습니다. 연관 값으로 기본 타입 뿐만 아니라 어떠한 타입도 가능하며 수의 제한도 없습니다.또한 case별로 다른 타입의 연관 값을 가질 수 있습니다. 

```swift
enum School {
	case student(name:String, birthday:String, age:int)
  case professor(name:String, age:int)
}

let student = School.student(name:"Tom", birthday:"2012345", age:20)
let professor = School.professor(name:"Jack", age:56)
```

<br>

**열거 타입을 사용하는 경우**

1. 제한된 선택지를 주고 싶을 때
2. 정해진 값 외에는 입력받고 싶지 않을 때
3. 예상된 입력 값이 한정되어 있을 때
4. 코드의 가독성을 높이고 싶을 때
5. 가능한 최대한의 캡슐화를 통해서 코드를 간결하게 만들고 쉽게 수정할 수 있도록 하고 싶을 때

**열거 타입을 사용을 권장하지 않는 경우**

> That’s a key part though, because enums are only really useful when the number of states can be specified up-front — so for more free-form values that can only be determined at runtime, other constructs (like structs, protocols, or classes) are most likely going to be more appropriate.
> -출처: Enums - Swift by Sundell

하지만 중요한 부분은 열거형이 상태 수를 미리 지정할 수 있을 때만 유용하기 때문에 런타임에 결정될 수 있는 더 많은 자유 형식 값(free-form values)의 경우 구조체(structure), 프로토콜 또는 클래스와 같은 다른 구성 요소가 더 적합할 가능성이 높습니다.

### 전략 열거 패턴 (The strategy enum pattern)

전략 열거 패턴에서 중요한 것은 각각의 케이스에 맞는 특정한 행동이 정의하는 것 (for each constant, implement constant-specific method) 이라고 생각합니다.
> To perform the pay calculation safely with **constant-specific method** implementations, you would have to duplicate the overtime pay computation **for each constant**, or move the computation into two helper methods, one for weekdays and one for weekend days, and invoke the appropriate helper method from each con- stant. 
- Effective Java 3rd Edition

```swift
// 코드 34-8를 Swift로 변환
enum PayrollDay {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
    static private var MINS_PER_SHIFT: Int = 8 * 60 // enums must not contatin stored properties
    func pay(minutesWorked: Int, payRate: Int)-> Int {
        let basePay: Int = minutesWorked * payRate
        var overtimePay: Int
        switch(self) {
        case .SATURDAY, .SUNDAY: // Weekend
            overtimePay = basePay / 2;
            break;
        default: // Weekday
            overtimePay =
                minutesWorked <= PayrollDay.MINS_PER_SHIFT ?
                    0 : (minutesWorked - PayrollDay.MINS_PER_SHIFT) * payRate / 2
        }
        return basePay + overtimePay
    }
}
```

코드 34-8를 제가 이해한 바대로 전략 열거 타입 패턴을 적용하여 바꿔봤습니다.

```swift
// 코드34-9를 Swift에 맞게 변환
struct PayrollDay {
    private let payType: PayType
    private static var MINS_PER_SHIFT = 8 * 60 
    
    init(_ payType: PayType = .WEEKDAY) {
        self.payType = payType
    }
    
    func pay(minutesWorked: Int, payRate: Int) -> Int {
        return payType.pay(minsWorked: minutesWorked, payRate: payRate)
    }
    
    enum PayType {
        
        case WEEKDAY, WEEKEND
        
        func pay(minsWorked: Int, payRate: Int) -> Int {
            switch self {
            case .WEEKDAY:
                return minsWorked <= PayrollDay.MINS_PER_SHIFT ? 0 : (minsWorked - PayrollDay.MINS_PER_SHIFT) * payRate / 2
            case .WEEKEND:
                return minsWorked * payRate / 2
            }
        }
    }
}

// 사용
let weekendPay = PayrollDay(.WEEKEND)
weekendPay.pay(minutesWorked: 1500, payRate: 2) // 1500
let weekdayPay = PayrollDay()
weekdayPay.pay(minutesWorked: 1500, payRate: 2) // 1020
```

* `PayrollDay`의 `payType`에 따라 pay가 다르게 계산(`PayType.pay(minsWorked:,payRate:)`)됩니다. 
* `PayrollDay`의 init에서 `WEEKDAY`로 기본값을 주었고, `WEEKDAY`가 아니라 `WEEKEND`일 경우  init을 통해 초기화할 `PayrollDay`의 `payType`을 설정할 수 있습니다.
* `PayrollDay`를 enum이 아니라 **Structure**로 한 이유: 책에 나와있는 Java 예제 코드에서는 `PayType` 타입의 `payType`을 enum 안에 저장하고 있습니다(Java에서 enum은 class입니다). 하지만 Swift의 enum에는 프로퍼티를 저장할 수 없습니다. `PayrollDay`의 각 case(MONDAY - SUNDAY)가 사용되는 곳이 없고, `WEEKDAY`와 `WEEKEND` 를 구분하여 pay를 계산하는게 핵심이라고 생각해 `PayrollDay`를 enum으로 구현하지 않고 Structure로 구현했습니다. 

### 추가적으로 위 코드에서 더 개선된 코드를 제안해주셔서 문서에 기록합니다.

* 첫번째

> private init 과 static let을 사용해 자바 Enum 과 유사하게 사용되게끔 하였습니다.
PayrollDay의 MONDAY 부터 SUNDAY 까지 case 구문이 계속 있는 형태입니다!
덧붙여, 위의 코드는 basePay와 overtimePay가 생략되어 있는 것 같아 추가하였습니다.

```swift
struct PayrollDay {
    static let monday = PayrollDay(.WEEKDAY)
    static let tuesday = PayrollDay(.WEEKDAY)
    static let wednesday = PayrollDay(.WEEKDAY)
    static let thursday = PayrollDay(.WEEKDAY)
    static let friday = PayrollDay(.WEEKDAY)
    static let saturday = PayrollDay(.WEEKEND)
    static let sunday = PayrollDay(.WEEKEND)

    private let payType: PayType
    private static var MINS_PER_SHIFT = 8 * 60

    private init(_ payType: PayType) {
        self.payType = payType
    }

    func pay(minutesWorked: Int, payRate: Int) -> Int {
        return payType.pay(minsWorked: minutesWorked, payRate: payRate)
    }

    enum PayType {
        case WEEKDAY, WEEKEND

        func pay(minsWorked: Int, payRate: Int) -> Int {
            let basePay = minsWorked * payRate
            return basePay + overtimePay(minsWorked: minsWorked, payRate: payRate)
        }
        
        private func overtimePay(minsWorked: Int, payRate: Int) -> Int {
            switch self {
            case .WEEKDAY:
                return minsWorked <= PayrollDay.MINS_PER_SHIFT ?
                    0 : (minsWorked - PayrollDay.MINS_PER_SHIFT) * payRate / 2
            case .WEEKEND:
                return minsWorked * payRate / 2
            }
        }
    }
}

let curPayrollDay: PayrollDay = .monday
```
> 책에서 예시로 든 API가 클라이언트 측에서 일당을 요청했을 때 요일에 따라 이를 계산해주는 기능이기 때문에 요일을 고르는 절차가 필요한것 같습니다! 
이 방법대로라면 클라이언트 측에서 요일을 고를 수도 있고, 생성자를 private으로 선언해서 기존 enum과 같이 case를 제한할 수도 있어서 좋은 것 같습니다.

다만 모든 case를 순회하거나 모든 case의 개수를 구할 수는 없는데, 이에 대비해 CaseIterable 프로토콜을 구현해 놓으면 모든 case를 순회할 수도 있고, case의 개수도 구할 수 있을 것 같아요.

```swift
struct PayrollDay: CaseIterable {
    typealias AllCases = [PayrollDay]
    
    static var allCases: [PayrollDay] {
        return [monday, tuesday, wednesday, thursday, friday, saturday, sunday]
    }
}
```

* 두번째

> 책에서 switch문을 지양하고 있어서 switch를 빼기 위해 프로토콜을 사용해서 추상화 해봤습니다.
> 하지만 스위프트에서는 switch문에서 모든 case를 처리하지 않을 때 컴파일 에러로 알려주기 때문에 PayType이 enum이어도 저는 충분하다고 생각하긴 합니다. 새 payType이 추가됐을 때 case를 수정하지 않을 방법이 필요하다면 이 방법을 쓰면 좋을 것 같습니다.
> 그리고 만약 책에서 나온 상수별 메서드 구현과 같은 기능이 필요하다면 이런 식으로 구현할 수도 있을 것 같습니다!

```swift

private protocol PayType {
    func pay(minutesWorked: Int, payRate: Int) -> Int
    func overtimePay(minutesWorked: Int, payRate: Int) -> Int
}

private extension PayType {
    func pay(minutesWorked: Int, payRate: Int) -> Int {
        let basePay = minutesWorked * payRate
        return basePay + overtimePay(minutesWorked: minutesWorked, payRate: payRate)
    }
}

struct PayrollDay {
    private struct Weekday: PayType {
        func overtimePay(minutesWorked: Int, payRate: Int) -> Int {
            let minutesPerShift = 480
            return minutesWorked <= minutesPerShift ? 0 : (minutesWorked - minutesPerShift) * payRate / 2
        }
    }
    
    private struct Weekend: PayType {
        func overtimePay(minutesWorked: Int, payRate: Int) -> Int {
            return minutesWorked * payRate / 2
        }
    }
    
    static let monday = PayrollDay(payType: Weekday())
    static let tuesday = PayrollDay(payType: Weekday())
    static let wednesday = PayrollDay(payType: Weekday())
    static let thursday = PayrollDay(payType: Weekday())
    static let friday = PayrollDay(payType: Weekday())
    static let saturday = PayrollDay(payType: Weekend())
    static let sunday = PayrollDay(payType: Weekend())
    
    private let payType: PayType
    
    private init(payType: PayType) {
        self.payType = payType
    }
    
    func pay(minutesWorked: Int, payRate: Int) -> Int {
        return payType.pay(minutesWorked: minutesWorked, payRate: payRate)
    }
}

// 시급 9000원, 일요일 4시간 근무 -> 54000원
PayrollDay.sunday.pay(minutesWorked: 4 * 60, payRate: 9000 / 60)
```

### 참고

1. [Enumerations - the swift programming languageswift 5.3](https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html)
2. [enum 살펴보기 - 사용법](https://jcsoohwancho.github.io/2019-08-18-enum-%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0-%EC%82%AC%EC%9A%A9%EB%B2%95/)
3. [Enums - Swift by Sundell](https://www.swiftbysundell.com/basics/enums/)
