# Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

자세한 설명은 책에서 상세히 설명하고 있으므로 생략하겠습니다. 아래는 요약입니다.

* **하지말 것**: ordinal 메서드로 인덱스를 얻는 코드 작성
  **이유**: 
  < 배열이나 리스트에서 원소를 꺼내는 경우 >

  1. ordinal을 사용하면 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일 되지 않습니다.
  2. 배열은 각 인덱스의 의미를 모르니 출력 결과가 직접 레이블을 달아야 합니다. 
  3. 특히나 정확한 정숫값을 사용한다는 것을 프로그래머가 직접 보증해야한다는 점이 문제입니다.

   < enum 안에 enum이 있는 경우 >

  1. 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없습니다. 
     즉, 열거타입을 수정하면서 이와 관련된 다른  값들을 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 날 수 있습니다. 

* **권장 방법**: 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않습니다. 여기서 배열은 실질적으로 열거 타입 상수를 값으로 매핑하는 일을 하기 때문에 Map을 사용할 수도 있으니 열거 타입을 키(key)로 사용하는 EnumMap 사용합니다.  

참고로, 

item37 본문 마지막에 

> 실제 내부에서는 맵들의 맵이 배열들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 좋다. 

라는 말이 나오는데 이 부분에 대해서는 이 [링크](https://www.tutorialspoint.com/difference-between-enummap-and-hashmap-in-java)를 참고하면 좋을 것 같습니다. (이번 주제와 연관성이 적어 직접 설명하지는 않겠습니다.) 

### Plant-LifeCycle 예시를 Swift 버전으로 바꾸기 
해당 아이템의 코드 37-1 부터 코드 37-2 까지의 예시인 Plant-LifeCycle을 Swift 버전으로 바꾸어 보았습니다.

* 자바 enum의 ordinal()에 대응되는 것을 구현해보자면 CaseIterable을 채택하고, Int를 채택한 enum의 `rawValue` 를 사용하는 것일 겁니다. 이 방식은 자바의 `ordianl()` 예시처럼 `Index out of Range` 에러가 발생할 수 있는 방식입니다. 

```swift
struct Plant: Hashable, Equatable {
    enum LifeCycle: Int, CaseIterable {
        case annual
        case perennial
        case biennial
    }
    
    let name: String
    let lifeCycle: LifeCycle
    
    init(name: String, lifeCycle: LifeCycle) {
        self.name = name
        self.lifeCycle = lifeCycle
    }
}

let garden: [Plant] = [Plant(name: "foo", lifeCycle: .annual),
                       Plant(name: "goo", lifeCycle: .perennial),
                       Plant(name: "koo", lifeCycle: .biennial)]
var plantsByLifeCycle: Array<Set<Plant>> = Array.init(repeating: Set<Plant>(), count: Plant.LifeCycle.allCases.count)

for plant in garden {
    plantsByLifeCycle[plant.lifeCycle.rawValue].insert(plant)
}

for i in 0 ..< plantsByLifeCycle.count {
    print("\(Plant.LifeCycle.allCases[i]): \(plantsByLifeCycle[i])")
}

// annual: [Plant(name: "foo", lifeCycle: Plant.LifeCycle.annual)]
// perennial: [Plant(name: "goo", lifeCycle: Plant.LifeCycle.perennial)]
// biennial: [Plant(name: "koo", lifeCycle: Plant.LifeCycle.biennial)]
```

* 아래처럼 `Plant.LifeCycle.allCases`의 `reduce` 메서드를 사용하면 자바의 `EnumMap<Plant.LifeCycle, Set<Plant>` 에 대응하는 자료구조를 만들 수 있습니다. 그리고 `var` 가 아닌 상수 `let` 으로 선언할 수 있습니다. 

```swift
let plantsByLifeCycle: [Plant.LifeCycle: Set<Plant>] = Plant.LifeCycle.allCases.reduce([Plant.LifeCycle: Set<Plant>]()) { (result, lifeCycle) -> [Plant.LifeCycle: Set<Plant>] in
    var result = result
    result[lifeCycle] = Set<Plant>(garden.filter{ $0.lifeCycle == lifeCycle })
    return result
}

print(plantsByLifeCycle)

// [Plant.LifeCycle.biennial: Set([Plant(name: "koo", lifeCycle: Plant.LifeCycle.biennial)]), Plant.LifeCycle.annual: Set([Plant(name: "foo", lifeCycle: Plant.LifeCycle.annual)]), Plant.LifeCycle.perennial: Set([Plant(name: "goo", lifeCycle: Plant.LifeCycle.perennial)])]
```


### Phase-Transition 예시를 Swift 버전으로 바꾸기 

해당 아이템의 코드 37-5 부터 코드 37-7 까지의 예시인 Phase-Transition을 Swift 버전으로 바꾸어 보았습니다. Transition을 Enum 타입이 아닌 Struct 타입으로 두고 `CaseIterable` 과 `Equatable` 을 채택하게 해서 Swift 버전으로 구현할 수 있었습니다. 개인적으로 새로 알게 된 것은 Enum 타입뿐만 아니라 Struct, Class 타입도 CaseIterable 을 채택할 수 있다는 점입니다.

```swift
enum Phase: CaseIterable {
    case solid
    case liquid
    case gas
    
    struct Transition: CaseIterable, Equatable {
        public static var allCases: [Transition] {
            return [melt, freeze, boil, condense, sublime, deposit]
        }
        
        let from: Phase
        let to: Phase
        
        static let melt = Transition(from: .solid, to: .liquid)
        static let freeze = Transition(from: .liquid, to: .solid)
        static let boil = Transition(from: .liquid, to: .gas)
        static let condense = Transition(from: .gas, to: .liquid)
        static let sublime = Transition(from: .solid, to: .gas)
        static let deposit = Transition(from: .gas, to: .solid)
        
        private init(from: Phase, to: Phase) {
            self.from = from
            self.to = to
        }
        
        static let m: [Phase: [Phase: Transition]] = Transition.allCases.reduce([Phase: [Phase: Transition]]()) { (result, transition) -> [Phase: [Phase: Transition]] in
            var result = result
            
            if result[transition.from] == nil {
                result[transition.from] = [transition.to: transition]
                return result
            }
            
            result[transition.from]![transition.to] = transition
            return result
        }
        
        static func from(from: Phase, to: Phase) -> Transition? {
            guard m[from] != nil else { return nil }
            guard m[from]![to] != nil else { return nil }
            
            return m[from]![to]!
        }
    }
}
```
### Swift에서 EnumMap 사용 예시

**< Fonts Dictionary >**

```swift
enum TextType {
    case title
    case subtitle
    case sectionTitle
    case body
    case comment
}

// fonts
let fonts: [TextType : UIFont] = [
    .title : .preferredFont(forTextStyle: .headline),
    .subtitle : .preferredFont(forTextStyle: .subheadline),
    .sectionTitle : .preferredFont(forTextStyle: .title2),
    .comment : .preferredFont(forTextStyle: .footnote)
]
```

위 Fonts Dictionary 예시에서 보면 .body에 대한 내용이 fonts에서 누락되었음을 컴파일러가 알려주지 않기 때문에 알아차리기 어려울 수 있습니다. 이렇게 일일이 Fonts Dictionary를 추가하는 것 보다 CaseItable를 사용하는 것이 더 낫습니다.

**CaseItable**

```swift
enum TextType: CaseIterable {
    case title
    case subtitle
    case sectionTitle
    case body
    case comment
}

var fonts = [TextType : UIFont]()

for type in TextType.allCases {
    switch type {
    case .title:
        fonts[type] = .preferredFont(forTextStyle: .headline)
    case .subtitle:
        fonts[type] = .preferredFont(forTextStyle: .subheadline)
    case .sectionTitle:
        fonts[type] = .preferredFont(forTextStyle: .title2)
    case .body:
        fonts[type] = .preferredFont(forTextStyle: .body)
    case .comment:
        fonts[type] = .preferredFont(forTextStyle: .footnote)
    }
}
```

이제 allCases를 사용하여 콜렉션(collection)의 모든 case에 순서대로 접근(access)할 수 있습니다. 그런 다음 case를 빠뜨릴 일 없이  Fonts Dictionary 을 만들 수 있습니다.

```swift
// 사용
let titleFont = fonts[.title] // titleFont의 타입은 UIFont? 입니다.
```

> ...  we also have a bit of a problem at the call site.
>
> Since we're using a dictionary to store our fonts, even though we're now using an exhaustive type for its keys, there's no way for the compiler to guarantee that we'll actually have a `UIFont` value for each key. 
>
> 딕셔너리를 사용하여 글꼴을 저장하고 있기 때문에 현재 키에 대해 완전한 유형을 사용하고 있지만 컴파일러가 실제로 각 키에 대한 UIFont 값을 가질 것이라고 보장 할 방법이 없습니다.

하지만 여기서 더 개선할 수 있는 부분이 있습니다. 위 코드는 모든 case에서 똑같이 fonts[type]을 반복적으로 수행하고 있습니다. 게다가  optional 입니다. 

<br>

**EnumMap의 아이디어를 빌려 직접 구현하여 옵셔널과 반복되는 코드를 해결할 수있습니다.**

**EnumMap 구현**

```swift
enum TextType {
    case title
    case subtitle
    case sectionTitle
    case body
    case comment
}

struct EnumMap<Enum: CaseIterable & Hashable, Value> {
    private let values: [Enum : Value]

    init(resolver: (Enum) -> Value) {
        var values = [Enum : Value]()

        for key in Enum.allCases {
            values[key] = resolver(key)
        }

        self.values = values
    }

    subscript(key: Enum) -> Value {
        // Here we have to force-unwrap, since there's no way
        // of telling the compiler that a value will always exist
        // for any given key. However, since it's kept private
        // it should be fine - and we can always add tests to
        // make sure things stay safe.
        return values[key]!
    }
}

let fonts = EnumMap<TextType, UIFont> { type in
    switch type {
    case .title:
        return .preferredFont(forTextStyle: .headline)
    case .subtitle:
        return .preferredFont(forTextStyle: .subheadline)
    case .sectionTitle:
        return .preferredFont(forTextStyle: .title2)
    case .body:
        return .preferredFont(forTextStyle: .body)
    case .comment:
        return .preferredFont(forTextStyle: .footnote)
    }
}

// 사용
let titleFont = fonts[.title]
let subtitleFont = fonts[.subtitle]
```

### 참고

1. [Enum iterations in Swift - Swift by Sundell](https://www.swiftbysundell.com/articles/enum-iterations-in-swift-42/)