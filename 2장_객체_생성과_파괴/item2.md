# Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

### 점층적 생성자 패턴

선택적인 매개변수가 많은 클래스의 경우, 점층적 생성자 패턴을 사용할 수 있습니다.

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

### 점층적 생성자 패턴의 단점

클라이언트 측에서 설정하기 원치 않는 매개변수까지 값을 지정해야 하고, 각 인자들의 의미가 무엇인지 알기 어렵습니다.

### 점층적 생성자 패턴 in 스위프트

```swift
import Foundation

public class NutritionFacts {

    private let servingSize: Int
    private let servings: Int
    private let calories: Int
    private let fat: Int
    private let sodium: Int
    private let carbohydrate: Int
    
    init(
        servingSize: Int,
        servings: Int,
        calories: Int = 0,
        fat: Int = 0,
        sodium: Int = 0,
        carbohydrate: Int = 0
    ) {
        self.servingSize = servingSize
        self.servings = servings
        self.calories = calories
        self.fat = fat
        self.sodium = sodium
        self.carbohydrate = carbohydrate
    }
}

let cocaCola = NutritionFacts(servingSize: 240, servings: 8, fat: 35)
```

자바에서는 default 파라미터가 없지만 스위프트에서는 지원해 줘서 설정을 원하는 매개변수에만 값을 지정할 수 있습니다.

또한 argument label을 통해 어떤 매개변수에 값을 전달하는 지도 알기 쉽습니다.

### 자바빈즈 패턴

자바빈즈 패턴은 매개변수가 없는 생성자로 객체를 만든 후, setter를 이용해 원하는 매개변수의 값을 설정하는 방식을 말합니다.

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

### 자바빈즈 패턴의 장점

점층적 생성자 패턴과 달리 어떤 매개변수에 값을 설정하는지 알기 쉬워져서 읽기 쉬운 코드가 되었으며, 클라이언트가 원하는 값만 설정할 수 있습니다.

### 자바빈즈 패턴의 단점

하지만 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전 객체를 사용하게 되는 것을 막을 수 없습니다. 또한 필드들을 final로 선언할 수가 없습니다.

### 자바빈즈 패턴 in 스위프트

```swift
public class NutritionFacts {
    
    private var servingSize: Int = -1
    private var servings: Int = -1
    private var calories: Int = 0
    private var fat: Int = 0
    private var sodium: Int = 0
    private var carbohydrate: Int = 0
    
    init() { }
    
    func set(servingSize: Int) {
        self.servingSize = servingSize
    }
    
    func set(servings: Int) {
        self.servings = servings
    }
    
    func set(calories: Int) {
        self.calories = calories
    }
    
    func set(fat: Int) {
        self.fat = fat
    }
    
    func set(sodium: Int) {
        self.sodium = sodium
    }
    
    func set(carbohydrate: Int) {
        self.carbohydrate = carbohydrate
    }
}

let cocaCola = NutritionFacts()
cocaCola.set(servingSize: 240)
cocaCola.set(servings: 8)
cocaCola.set(fat: 35)
```

### 빌더 패턴

빌더 패턴에서는 필수 매개변수만으로 이루어진 빌더 생성자를 호출해 빌더 객체를 얻고, 빌더 객체의 setter 메서드들로 선택 매개변수들을 설정합니다. 마지막으로 build를 호출해 필요한 객체를 얻습니다.

### 빌더 패턴의 장점

빌더 패턴은 생성되는 객체를 불변으로 만들 수 있고, 빌더의 setter 메서드들을 method chaining 형식으로 연쇄적으로 호출할 수 있다는 장점이 있습니다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8) 
		.calories(100).sodium(35).carbohydrate(27).build();
```

또한 빌더 패턴은 위 예시 코드처럼 쓰고 읽기 쉽습니다.

가변인수 매개변수를 여러 개 사용할 수도 있게 되며, 빌더 하나로 다양한 객체를 만들 수도 있어 유연합니다. 빌더에서 일련번호를 채우는 등 추가 기능을 구현할 수도 있습니다.

### 빌더 패턴의 단점

객체 생성 전에 빌더부터 만들어야 해서 생성 비용이 클 수 있습니다.

또한 코드가 비교적 장황해서 매개변수 4개 정도 이상일 때 적합합니다.

### 기본 빌더 패턴 in 스위프트

```swift
import Foundation

public class NutritionFacts {
    
    private let servingSize: Int
    private let servings: Int
    private let calories: Int
    private let fat: Int
    private let sodium: Int
    private let carbohydrate: Int
    
    public class Builder {
        
        let servingSize: Int
        let servings: Int
        
        var calories = 0
        var fat = 0
        var sodium = 0
        var carbohydrate = 0
        
        init(servingSize: Int, servings: Int) {
            self.servingSize = servingSize
            self.servings = servings
        }
        
        func calories(_ calories: Int) -> Self {
            self.calories = calories
            return self
        }
        
        func fat(_ fat: Int) -> Self {
            self.fat = fat
            return self
        }
        
        func sodium(_ sodium: Int) -> Self {
            self.sodium = sodium
            return self
        }
        
        func carbohydrate(_ carbohydrate: Int) -> Self {
            self.carbohydrate = carbohydrate
            return self
        }
        
        func build() -> NutritionFacts {
            return NutritionFacts(from: self)
        }
    }
    
    init(from builder: Builder) {
        servingSize = builder.servingSize
        servings = builder.servings
        calories = builder.calories
        fat = builder.fat
        sodium = builder.sodium
        carbohydrate = builder.carbohydrate
    }
}

let cocaCola = NutritionFacts.Builder(servingSize: 240, servings: 8)
    .calories(100)
    .sodium(35)
    .build()
```

### 계층적으로 설계된 클래스에서의 빌더 패턴 in 스위프트

계층적으로 설계된 클래스에서의 빌더 패턴을 스위프트로 구현해 보았습니다.

먼저 `Pizza` 추상 클래스의 경우 내부에 `Topping` enum이 있어야 해서 (프로토콜이 아닌) 클래스로 구현하였습니다.

`Pizza`의 abstract builder의 경우 `T`가 자기 자신을 확장한 타입이어야 하는데, 이를 표현하기 위해 프로토콜과 associated type을 이용하였고, associated type이 자기 자신 프로토콜을 따르도록 제한하였습니다.

> 이렇게 한 이유는 스위프트 클래스는 재귀적 타입 한정을 이용한 제네릭을 지원하지 않았기 때문입니다. 더 좋은 방법이 있다면 제안 부탁드립니다.

마지막으로, 클래스 내부에 프로토콜이 존재할 수 없기 때문에 Pizza의 abstract builder는 클래스 외부에 선언하였습니다.

```swift
protocol DefaultBuilder: class {

    associatedtype BuilderType: DefaultBuilder
    
    var toppings: Set<Pizza.Topping> { get set }
    
    // swift는 covariant return typing을 지원하지 않는 것 같아서 여기서 정의하지 않고, 
    // 각 구체 타입에 구현하였습니다.
    // 더 좋은 방법이 있다면 제안 부탁드립니다.
    //func build() -> Pizza
}

extension DefaultBuilder {
    
    func addTopping(_ topping: Pizza.Topping) -> Self { // 구체 하위 빌더 반환(책에서 T에 해당)
        toppings.insert(topping)
        return self
    }
}

class Pizza {
    
    enum Topping {
        case ham, mushroom, onion, pepper, sausage
    }
    
    let toppings: Set<Topping>
    
    init<Builder: DefaultBuilder>(with builder: Builder) {
        // Set은 value type이기 때문에 clone하지 않아도 복사본이 할당됩니다.
				toppings = builder.toppings
    }
}
```

다음은 Pizza를 상속한 두 구체 타입들입니다.

```swift
class NewYorkPizza: Pizza {
    
    enum Size {
        case small, medium, large
    }
    
    let size: Size
    
    class Builder: DefaultBuilder {
        
        typealias BuilderType = Builder
        
        var toppings = Set<Pizza.Topping>()
        
        let size: Size
        
        init(size: Size) {
            self.size = size
        }
        
        func build() -> NewYorkPizza {
            return NewYorkPizza(builder: self)
        }
    }
    
    init(builder: Builder) {
        size = builder.size
        super.init(with: builder)
    }
}

class Calzone: Pizza {
    
    let sauceInside: Bool
    
    class Builder: DefaultBuilder {
        
        typealias BuilderType = Builder
        
        var toppings = Set<Pizza.Topping>()
        
        var sauceInside = false
        
        func addSauce() -> Self { // 변수명과 겹쳐서 addSauce로 변경
            sauceInside = true
            return self
        }
        
        func build() -> Calzone {
            return Calzone(builder: self)
        }
    }
    
    init(builder: Builder) {
        sauceInside = builder.sauceInside
        super.init(with: builder)
    }
}
```

다음은 위 계층적 빌더들을 사용하는 클라이언트 코드의 예시입니다. `build()`에서 반환하는 타입을 명확히 하기 위해 변수의 타입을 명시하였습니다.

```swift
let newYorkpizza: NewYorkPizza = NewYorkPizza.Builder(size: .large)
    .addTopping(.ham).addTopping(.mushroom).build()
let calzone: Calzone = Calzone.Builder()
    .addSauce().addTopping(.sausage).build()
```

### 핵심 정리

생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫습니다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇습니다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전합니다.

### 추가: 빌더 패턴 활용법

선택적 프로퍼티가 많은 URLComponent에도 빌더 패턴을 활용할 수 있습니다.

[Swift builder design pattern - The.Swift.Dev.](https://theswiftdev.com/swift-builder-design-pattern/)
