# Item 2. 생성자에 매개변수가 많다면 빌더를 고려하라

### 점층적 생성자 패턴

선택적인 매개변수가 많은 클래스의 경우, 점층적 생성자 패턴을 사용할 수 있다.

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

### 점층적 생성자 패턴의 단점

클라이언트 측에서 설정하기 원치 않는 매개변수까지 값을 지정해야 하고, 각 인자들의 의미가 무엇인지 알기 어렵다.

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

자바에서는 default 파라미터가 없지만 스위프트에서는 지원해 줘서 설정을 원하는 매개변수에만 값을 지정할 수 있다.

또한 argument label을 통해 어떤 매개변수에 값을 전달하는 지도 알기 쉽다.

### 자바빈즈 패턴

매개변수가 없는 생성자로 객체를 만든 후, setter를 이용해 원하는 매개변수의 값을 설정하는 방식

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

### 자바빈즈 패턴의 장점

점층적 생성자 패턴과 달리 어떤 매개변수에 값을 설정하는지 알기 쉬워져서 읽기 쉬운 코드가 되었으며, 클라이언트가 원하는 값만 설정할 수 있다.

### 자바빈즈 패턴의 단점

하지만 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전 객체를 사용하게 되는 것을 막을 수 없다. 또한 필드들을 final로 선언할 수 없다.

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

필수 매개변수만으로 이루어진 빌더 생성자를 호출해 빌더 객체를 얻고, 빌더 객체의 setter 메서드들로 선택 매개변수들을 설정한다. 마지막으로 build를 호출해 필요한 객체를 얻는다.

### 빌더 패턴의 장점

생성되는 객체를 불변으로 만들 수 있고, 빌더의 setter 메서드들을 method chaining 형식으로 연쇄적으로 호출할 수 있다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8) 
		.calories(100).sodium(35).carbohydrate(27).build();
```

쓰고 읽기 쉽다. 유연하다.

### 빌더 패턴의 단점

객체 생성 전에 빌더부터 만들어야 해서 생성 비용이 클 수 있다.

코드가 장황해서 매개변수 4개 정도 이상일 때 적합하다.

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

코드 추가 예정..

### 빌더 패턴 활용법

선택적 프로퍼티가 많은 URL Request 객체에 사용할 수 있다.

[Swift builder design pattern - The.Swift.Dev.](https://theswiftdev.com/swift-builder-design-pattern/)
