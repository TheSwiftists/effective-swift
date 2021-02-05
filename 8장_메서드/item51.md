# Item 51. 메서드 시그니처를 신중히 설계하라

## API 설계 요령들

### 메서드 이름을 신중히 짓자.

[Swift API Design Guide](https://swift.org/documentation/api-design-guidelines/)는 모든 Swift Style Guide의 근간이 되는 가이드입니다. Swift API Design Guide를 참고하고 메서드 이름을 지읍시다. 다음은 가이드 중에 메서드, 매개변수 네이밍에 대한 일부 규칙입니다. 

* 이름으로 명확한 사용법을 제시하세요. 그 이름을 사용하는 부분의 코드를 읽는 사람에게 **혼란을 줄 수 있는 단어는 피하세요.**

> BAD 

```Swift
employees.remove(x) // 명확하지 않음: x값을 제거하는건가?
```

> GOOD

```swift
```swift
extension List {
	public mutating func remove(at position: Index) -> Element
}

employees.remove(at: x)
```
​=> 메소드 시그니처에서 at을 생략한다면 해당 메소드가 x과 같은 요소를 제거하는 건지, x위치에 있는 요소를 찾아서 제거한다는 건지 헷갈릴 수 있습니다.

* **쓸모없는 단어를 제거하세요.** 이름의 모든 단어는 사용자 관점에서 주요한 정보를 제공해야만 합니다.

> BAD

```Swift
public mutating func removeElement(_ member: Element) -> Element?

allViews.removeElement(cancelButton)
```
=> 더 많은 단어를 사용하면 의도가 명확해지거나 헷갈리지 않을 수 있지만 코드를 읽는 사람에게 **중복된 정보를 제공하는 경우**는 제거해야 합니다. 위의 코드에서 Element는 호출하는 지점에서는 의미가 없으니 다음과 같은 코드가 더 좋습니다.

> GOOD

```Swift
public mutating func remove(_ member: Element) -> Element?
	
allViews.remove(cancelButton) // 보다 명확함
```

* 매개변수 역할을 명확하게 넣어서 부족한 타입 정보를 보완하세요.

매개변수 타입이 `NSObject`, `Any`, `AnyObject` 이거나 `Int`나 
`String` 같은 기본 타입이면, 사용하는 지점에서 맥락상 타입 정보가 불명확할 수 있습니다.

> BAD

```Swift
func add(_ observer: NSObject, for keyPath: String)
grid.add(self, for: graphics) // 불분명함 
```

> GOOD
```Swift
func addObserver(_ observer: NSObject, forKeyPath path: String)
grid.addObserver(self, forKeyPath: graphics) // 명확함
```
=> ​ 명확성을 갖도록 **부족한 타입 정보마다 역할을 설명하는 명사를 붙여줍니다.**

* 문서화 할 수 있는 매개 변수 이름을 선택하세요. 함수나 메소드를 사용할 때 **매개변수가 감춰져있더라도** 여전히 중요한 역할을 합니다.
  * 다음과 같은 이름은 문서화해도 읽기 편하고, 문서로 만들어도 자연스럽습니다.

```Swift
/// Return an `Array` containing the elements of `self`
/// that satisfy `predicate`.
func filter(_ predicate: (Element) -> Bool) -> [Generator.Element]

/// Replace the given `subRange` of elements with `newElements`.
mutating func replaceRange(_ subRange: Range, with newElements: [E])
```

* 반면에 다음과 같은 표현은 문법도 안맞고, 불분명합니다. 

```Swift
/// Return an `Array` containing the elements of `self`
/// that satisfy `includedInResult`.
func filter(_ includedInResult: (Element) -> Bool) -> [Generator.Element]

/// Replace the range of elements indicated by `r` with
/// the contents of `with`.
mutating func replaceRange(_ r: Range, with: [E])
```

* 매개변수 기본값(defaulted parameter)을 지정해서 편리함을 더하세요. 매개 변수에 흔히 넘기는 값 자체를 기본값으로 활용하세요.

더 자세한 내용은 Swift API Design Guide를 참고 바랍니다. 

### 편의 메서드를 너무 많이 만들지 말자.

편의 메서드를 너무 많이 만들지 맙시다. 모든 메서드는 각각 자신의 소임을 다해야 합니다. 메서드가 너무 많은 클래스는 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수하기 어렵습니다. 인터페이스도 마찬가지입니다. 클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야 합니다. 아주 자주 쓰일 경우에만 별도의 약칭 메서드를 두기 바랍니다. **확신이 서지 않으면 만들지 맙시다.**

특히 API의 프로퍼티로 배열이 있어서 배열의 특정 값을 순회하며 찾는 경우, 해당 값을 반환하는 메소드를 만들지 말고 클로저(핸들러)를 이용해 API를 사용하는 클라이언트가 특정 조건을 클로저로 작성해 값을 찾도록 만드는 방법이 있습니다.  

```swift
protocol CardRepeatable {    
    func repeatCard(handler: (Card) -> (Void))
}

class Deck: CardRepeatable {
    private var cards = [Card]()

    //... 생략 
    func repeatCard(handler: (Card) -> (Void)) {
        cards.forEach { handler($0) }
    }
}

struct Card {    
    enum Suit: CaseIterable {
        case spade
        //... 생략 
    }
    
    enum Number: Int, CaseIterable {
        case ace = 1
        case two
        case three
        //... 생략 
    }
    //... 생략
}
```
=> `repeatCard` 메소드를 통해 클라이언트는 특정 조건을 클로저로 작성해 특정 값을 찾을 수 있습니다.

### 매개변수 목록은 짧게 유지하자.

매개변수 개수는 4개 이하가 좋습니다. 일단 4개가 넘어가면 매개변수를 전부 기억하기가 쉽지 않습니다. 우리가 만든 API에 이 제한을 넘는 메서드가 많다면 프로그래머들은 API 문서를 옆에 끼고 개발해야 할 것입니다.

**Swift는 인자 레이블이 있어서 매개변수끼리 구분이 되어 수고를 많이 덜 수 있지만**, 여전히 매개변수 수는 적은 쪽이 훨씬 낫습니다. 

**같은 타입의 매개변수 여러 개가 연달아 나오는 경우가 특히 해롭습니다.** 사용자가 매개변수 순서를 기억하기 어려울 뿐더러(Swift 경우에는 인자 레이블이 생략된 경우 똑같이 적용됩니다) 실수로 순서를 바꿔 입력해도 그대로 **컴파일되고 실행**됩니다. 단지 의도와 다르게 동작할 뿐입니다. 

## 과하게 긴 매개변수 목록을 짧게 줄여주는 기술 3가지

### 기술 1: 여러 메서드로 쪼갠다. 

여러 메서드로 쪼갭니다. 쪼개진 메서드 각각은 원래 매개변수 목록의 부분집합을 받습니다. 
잘못하면 메서드가 너무 많아질 수 있지만, 직교성(orthogonality)을 높여 오히려 메서드 수를 줄여주는 효과도 있습니다. 

예를 들어 리스트에서 주어진 원소의 인덱스를 찾아야 하는데, 전체 리스트가 아니라 지정된 범위의 부분리스트에서의 인덱스를 찾는다고 해봅시다. 이 기능을 하나의 메서드로 구현하려면 '부분 리스트의 시작', '부분 리스트의 끝', '찾을 원소' 까지 총 3개의 매개변수가 필요합니다. 그런데 리스트는 그 대 신 부분리스트를 반환하는 subList 메서드와 주어신 원소의 인덱스를 알려주는 indexOf 메서드를 별개로 제공합니다. subList가 반환한 부분리스트 역시 완벽한 List이므로 두 메서드를 조합하면 원하는 목적을 이룰 수 있습니다. 결과적으로 강함과 유연함이 절묘하게 균형을 이룬 API가 만들어진 것입니다.  

> Java

```java
List<Integer> nums = List.of(3, 3, 4, 4);
if (nums.size() < 3) return;

int firstIndex = nums.subList(0, 3).indexOf(3);
```

> Swift 

```swift
let nums = [3, 3, 4, 4]
guard nums.count >= 3 else { return }

let firstIndex = nums[0 ..< 3].firstIndex(of: 3)
```

### 기술 2: 매개변수 여러 개를 묶어주는 클래스를 만든다. 

매개변수 수를 줄여주는 기술 두 번째는 매개변수 여러 개를 묶어주는 도우미 클래스를 만드는 것입니다. 이런 도우미 클래스는 보통 메소드(동작)가 없으므로 Swift에서는 구조체(struct)로 두면 좋을 것 같습니다. 특히 **잇따른** 매개변수 몇 개를 독립된 하나의 개념으로 볼 수 있을 때 추천하는 기법입니다. 
예를 들어 카드게임을 클래스로 만든다고 해봅시다. 그러면 메서드를 호출할 때 카드의 숫자(rank)와 무늬(suit)를 뜻하는 두 매개변수를 **항상 같은 순서로 전달할 것**입니다. 따라서 이 둘을 묶는 도우미 클래스를 만들어 하나의 매개변수로 주고받으면 API는 물론 클래스 내부 구현도 깔끔해질 것입니다.

```swift
class Card {
    struct CardInfo {
        let rank: Rank
        let suit: Suit
    }
    
    let cardInfo: CardInfo
    
    init(cardInfo: CardInfo) {
        self.cardInfo = cardInfo
    }
    
    //...
}

let card = Card(cardInfo: Card.CardInfo(rank: .one, suit: .spade))
```

* 또 Swift 에서는 매개변수를 여러 개 묶을 수 있는 튜플(Tuple)을 이용할 수 있습니다.

```Swift
class Card {
    let cardInfo: (rank: Rank, suit: Suit)
    
    init(cardInfo: (rank: Rank, suit: Suit)) {
        self.cardInfo = cardInfo
    }
    
    //...
}

let card = Card(cardInfo: (rank: .one, suit: .spade))
```

### 기술 3: 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용한다.

세 번째는 앞서의 두 기법을 혼합한 것으로, 객체 생성에 사용한 빌더 패턴(아이템 2)을 메서드 호출에 응용한다고 보면 됩니다. 이 기법은 매개변수가 많을 때, 특히 **그중 일부는 생략해도 괜찮을 때** 도움이 됩니다. 먼저 모든 매개변수를 하나로 추상화한 객체를 정의하고, 클라이언트에서 이 객체의 세터(setter) 메서드를 호출해 필요한 값을 설정하게 하는 것입니다. 이때 각 세터 메서드는 매개변수 하나 혹은 서로 연관된 몇 개만 설정하게 하는 것입니다. 클라이언트는 먼저 필요한 매개변수를 다 설정한 다음, execute 메서드를 호출해 앞서 설정한 매개변수들의 유효성을 검사합니다. 마지막으로, 설정이 완료된 객체를 넘겨 원하는 계산을 수행합니다. 

> Java

```java
class NutritionFacts {
    private int servingSize;
    private int servings;
    private int calories;
    private int fat;
    private int sodium;

    public NutritionFacts servingSize(int servingSize) {
        this.servingSize = servingSize;
        return this;
    }

    public NutritionFacts servings(int servings) {
        this.servings = servings;
        return this;
    }

    public NutritionFacts calories(int calories) {
        this.calories = calories;
        return this;
    }

    public NutritionFacts fat(int fat) {
        this.fat = fat;
        return this;
    }

    public NutritionFacts sodium(int sodium) {
        this.sodium = sodium;
        return this;
    }

    public boolean execute() {
        return servingSize > 0 && servings > 0 && calories > 0;
    }
}

NutritionFacts nutritionFacts = new NutritionFacts().servingSize(1).servings(2).calories(3);
if(!nutritionFacts.execute()) { return; }

add(nutritionFacts);
```

> Swift

```swift
class NutritionFacts {
    private let servingSize: Int
    private let servings: Int
    private let calories: Int
    private var fat: Int?
    private var sodium: Int?
    
    init?(
        servingSize: Int,
        servings: Int,
        calories: Int,
        fat: Int? = nil,
        sodium: Int? = nil
    ) {
        guard servingSize > 0, servings > 0 , calories > 0 else {
            return nil
        }
        
        self.servingSize = servingSize
        self.servings = servings
        self.calories = calories
        self.fat = fat
        self.sodium = sodium
    }
}

guard let nutritionFacts = NutritionFacts(
                servingSize: 1, servings: 2, calories: 3) else { return }
        
add(nutritionFacts: nutritionFacts)
```

=> Swift 는 execute() 메소드를 따로 구현할 필요없이 `init?` 사용하면 됩니다. 

## 매개변수의 타입으로는 클래스보다는 프로토콜이 더 낫다. 

매개변수로 적합한 프로토콜이 있다면 (이를 구현한 클래스가 아닌) 그 프로토콜을 직접 사용합시다.
(프로토콜을 채택한)특정 클래스 대신 프로토콜 타입을 매개변수로 사용하면, 해당 프로토콜을 채택한 다른 어떤 클래스도 인수로 건넬 수 있기 때문입니다. **프로토콜 대신 클래스를 사용하면 클라이언트에게 특정 구현체만 사용하도록 제한하는 꼴이며, 혹시라도 입력 데이터가 다른 형태로 존재한다면 명시한 특정 구현체의 객체로 옮겨 담느라 비싼 복사 비용을 치러야 합니다.** 

```swift
protocol SomeProtocol {
    func describe()
}

func someFunction(someProtocol: SomeProtocol) {
    someProtocol.describe()
}
```
=> 이처럼 클래스보다는 프로토콜 타입으로 매개변수의 타입을 지정하십시오.

## boolean보다는 원소 2개짜리 열거 타입이 낫다(메서드 이름상 boolean을 받아야 의미가 더 명확할 때는 예외입니다).

열거 타입을 사용하면 코드를 읽고 쓰기가 쉬워집니다. 나중에 선택지를 추가하기도 쉽습니다. 
다음은 화씨온도(fahrenheit)와 섭씨온도(celsius)를 원소로 정의한 열거 타입입니다. 

```swift
enum TemperatureScale {
    case fahrenheit
    case celsius
}
```

온도계 클래스의 생성자가 이 열거 타입을 입력받아 적합한 온도계 인스턴스를 생성해준다고 해봅시다. 
`Thermomter(isFahrenheit: true)` 보다 `Thermometer(scale: .fahrenheit)`가 하는 일을 훨씬 명확히 알려줍니다. 나중에 캘빈온도도 지원해야 한다면, Thermometer에 새로운 생성자를 추가할 필요 없이 TemperatureScale 열거 타입에 캘빈온도(KELVIN)를 추가하면 됩니다. **또한, 온도 단위에 대한 의존성을 개별 열거 타입 상수의 메서드 안으로 리팩터링해 넣을 수도 있습니다(즉 상위 객체는 상위 객체대로, 하위 객체는 하위객체대로 SRP에 맞게 코드 작성할 수 있습니다)** 예컨대 double 값을 받아 섭씨온도로 변환해주는 메서드를 열거 타입 상수 각각에 정의해둘 수 있습니다. 

