# Item 11. equals를 재정의하려거든 hashCode도 재정의하라



`equals` (Swift에서는 `==`)를 재정의한 후, 각 언어의 컬렉션(Collection) 타입의 자료구조에 Car 타입의 인스턴스들을 추가해보면서 논리적으로 같은 객체로 판단되는지 살펴봅시다. 아래의 예제 코드는 Car 타입의 car1와 car2가 논리적으로 같은 객체로 판단되도록 하는 것이 목표입니다.

### 왜 Java에서는 equals와 hashCode는 같이 재정의해야 할까요?

1. **equals 재정의**<br>

   **결과**: 재정의한 equals를 바탕으로 Car 타입의 car1, car2 객체는 논리적으로 같은 객체로 판단됩니다.

```java
public class Car {
    private final String name;
    private final Int seater;

    public Car(String name, int seater) {
        this.name = name;
        this.seater = seater;
    }

    @Override
    public boolean equals(Object object) {
        if (this == object) return true;
        if (object == null || getClass() != object.getClass()) return false;
        Car car = (Car) object;
        return Objects.equals(name, car.name) && Objects.equals(seater, car.seater);
    }
}

public static void main(String[] args){
    Car car1 = new Car("TheSwiftists");
    Car car2 = new Car("TheSwiftists");
 
    System.out.println(car1.equals(car2)); // true 출력
}
```

2. **List에 담는 경우**<br>
   **결과**: List에 car1, car2 를 추가할 수 있습니다. 아무 문제가 없습니다.

   

   ```java
   public static void main(String[] args) {
       List<Car> cars = new ArrayList<>();
       Car car1 = new Car("TheSwiftists");
       Car car2 = new Car("TheSwiftists");
       cars.add(car1);
       cars.add(car2);
   
       System.out.println(cars.size()); // 2 출력
   }
   ```

3. **Set에 담는 경우(중복 값을 허용하지 않는 Collection에 담는 경우)**<br>

   **예상 결과**: 위에는 논리적으로 두 객체를 같은 객체로 판단하였기 때문에 car1과 car2를 HashSet에 추가하고 난 뒤의 HashSet의 size는 1일 것입니다.<br>
   **실제 결과**: HashSet의 size는 2가 출력됩니다.

   ```java
   public static void main(String[] args) {
       Set<Car> cars = new HashSet<>();
       Car car1 = new Car("TheSwiftists");
       Car car2 = new Car("TheSwiftists");
       cars.add(car1);
       cars.add(car2);
   
       System.out.println(cars.size()); // 2 출력
   }
   ```

* **정리** <br>
  `hashCode`를 `equals`와 함께 재정의하지 않으면 위의 예처럼 코드가 예상과 다르게 동작할 수 있습니다. 다시 말해,  **hash 값을 사용하는 Java의 Collection(HashSet, HashMap, HashTable)을 사용할 때 문제가 발생할 수 있습니다. 따라서 `equals`를 재정의할 때 코드의 안전성을 높이려면 `hashCode`를 같이 재정의해주는 것이 좋습니다.

4. **hashCode 재정의**<br>

   *3번의 결과는 왜 나오는 이유*를 살펴봅시다.<br>
   Java에서 Collection(HashMap, HashSet, HashTable)은 객체가 논리적으로 같은지 비교할 때 아래 그림과 같은 과정을 거칩니다.

   ![hash 값을 사용하는 객체의 비교 로직](https://woowacourse.github.io/javable/images/2020-07-29-equals-and-hashcode.png)

   <p align="center"> 이미지 출처: javable</p>

   `hashCode` 메서드의 리턴 값으로 먼저 동등객체인지 확인하고 `equals` 메서드의 리턴값이 `true`여야 논리적으로 같은 객체라고 판단합니다. 그런데 `Car` 클래스에는 `hashCode` 메서드가 재정의 되어있지 않아 `Object` 클래스의 `hashCode` 메서드가 사용된 것입니다. `Object` 클래스의 `hashCode` 메서드는 객체의 고유한 주소 값을 `int` 값으로 변환하기 때문에 객체마다 다른 값을 리턴합니다. 두 개의 `Car` 객체는 `equals`로 비교도 하기 전에 서로 다른 `hashCode` 메서드의 리턴 값으로 인해 다른 객체로 판단된 것입니다.

   

   그렇다면 `hashCode`를 재정의해봅시다.

   ```java
   public class Car {
       private final String name;
       private final int seater; 
   
       public Car(String name, int seater) {
           this.name = name;
           this.seater = seater
       }
   
       // intellij Generate 기능 사용
       @Override
       public boolean equals(Object object) {
           if (this == object) return true;
           if (object == null || getClass() != object.getClass()) return false;
           Car car = (Car) object;
           return Objects.equals(name, car.name) && Objects.equals(seater, car.seater);
       }
   
       @Override
       public int hashCode() {
           return Objects.hash(name, seater);
       }
   }
   ```

### Swift 에서는 이 문제를 어떻게 다루고 있을까요?
1. **Equatable 프로토콜 채택과 `==` 재정의**<br>
   

**결과**: Java에서의 결과와 동일하게 재정의한 `equals`를 바탕으로 `Car` 타입의 car1, car2 객체는 논리적으로 같은 객체로 판단됩니다.

   ~~~ swift
   import Foundation
   
   class Car {
       private let name: String
       private let seater: Int
       
       init(name: String, seater: Int) {
           self.name = name
           self.seater = seater
       }
   }
   
   extension Car: Equatable {
       static func == (lhs: Car, rhs: Car) -> Bool {
           return lhs.name == rhs.name && lhs.seater == rhs.seater
       }
   }
   
   let car1 = Car(name: "TheSwiftists", seater: 2)
   let car2 = Car(name: "TheSwiftists", seater: 2)
   print(car1 == car2) // true 출력
   ~~~



2. **Array에 담는 경우**<br>
   **결과**: Java에서의 결과와 동일하게 Array에 car1, car2 을 추가할 수 있습니다. 아무 문제가 없습니다.

   ~~~ swift
   import Foundation
   
   class Car {
       private let name: String
       private let seater: Int
       
       init(name: String, seater: Int) {
           self.name = name
           self.seater = seater
       }
   }
   
   extension Car: Equatable {
       static func == (lhs: Car, rhs: Car) -> Bool {
           return lhs.name == rhs.name && lhs.seater == rhs.seater
       }
   }
   
   let car1 = Car(name: "TheSwiftists", seater: 2)
   let car2 = Car(name: "TheSwiftists", seater: 2)
   var cars = [Car]()
   
   cars.append(car1)
   cars.append(car2)
   
   print(cars.count) // 2 출력
   ~~~

3. **Set에 담는 경우(중복 값을 허용하지 않는 Collection에 담는 경우)**<br>

   **결과**: Swift에서는 `Car` 클래스가 Hashable 프로토콜을 준수하고 있지 않다는 에러를 표시하며 Set 생성 자체가 되지 않습니다. 
<img src="https://user-images.githubusercontent.com/52783516/97587165-6e8d9880-1a3e-11eb-9c52-9cf93dce182f.png" alt="image" style="zoom: 50%;" />
   
   ~~~swift
   import Foundation
   
   class Car {
       private let name: String
       private let seater: Int
       
       init(name: String, seater: Int) {
           self.name = name
           self.seater = seater
       }
   }
   
   extension Car: Equatable {
       static func == (lhs: Car, rhs: Car) -> Bool {
           return lhs.name == rhs.name && lhs.seater == rhs.seater
       }
   }
   
   let car1 = Car(name: "TheSwiftists", seater: 2)
   let car2 = Car(name: "TheSwiftists", seater: 2)
   
   var carsSet = Set<Car>() // 에러 발생
   ~~~
4. **Hashable 프로토콜 채택과 `hash(into:)` 메서드 구현**
   **결과**: car1와 car2을 논리적으로 같은 객체로 판단하기 때문에 Set에 car1을 추가한 다음 car2을 추가할 수 없습니다.(insert 결과: inserted false) <br>
따라서 cars의 count는 1이 출력이 됩니다. 
   
   ~~~swift
   extension Car: Hashable {
       func hash(into hasher: inout Hasher) {
           hasher.combine(name)
           hasher.combine(seater)
       }
   }
   
   var cars = Set<Car>()
   cars.insert(car1)
   cars.insert(car2) // failed (inserted false)
   print(cars.count) // 1 출력
   ~~~

### Hashable 프로토콜

- **Hashable**
  - 정수 해시 값을 제공하고 `Dictionary`의 key 또는 `Set`의 value가 될 수 있는 타입입니다. 
  - Hashable 프로토콜은 Equatable 프로토콜을 상속했기 때문에 Equatable 프로토콜을 준수해야합니다.
    * `static func == (lhs: Car, rhs: Car) -> Bool` 메서드 구현
  - 구조체(Structure)의 저장 프로퍼티는 모두 Hashable을 준수해야 합니다.
  - 열거형(Enumeration)의 모든 연관값(associated values)은 모두 Hashable을 준수해야 합니다.
  - String, Int, Bool 등의 기본 데이터 타입들은 모두 Hashable 프로토콜을 준수하고 있습니다. 하지만 커스텀 타입을 Dictionary의 key 타입으로 지정하거나 Set의 value 타입으로 지정한 경우 해당 타입이 Hashable 프로토콜을 준수하게 만들어야 합니다.

* **Hashable 프로토콜을 준수하려면**
  : 관련 모든 클래스에서 Hashable을 구현해야합니다. 

  - `hash(into:)` 필수 구현

    ~~~swift
    func hash(into hasher: inout Hasher)
    ~~~

    [Apple Developer Document](https://developer.apple.com/documentation/swift/hashable/2995575-hash)에서는 필수 구성요소들에게 주어진 hasher를 공급함으로써 value의 필수 구성요소들을 해쉬하라고 명시하고 있습니다.(Hashes the essential components of this value by feeding them into the given `hasher`.)
    Hashable 프로토콜을 채택 후 해쉬값을 제공하기 위한 인스턴스 메소드인 `hash(into:)`  메서드를 만들어 관련 클래스의 모든 프로퍼티를 Hashable 하도록 만들어야합니다. 즉, `hash(into:)` 메서드에서 관련 클래스의 프로퍼티를 모두 `combine` 해줘야 합니다.

    <br>

    **Hasher?**<br>
    구조체로, **해당 인스턴스의 구성요소를 결합할 때 사용합니다.**(The hasher to use when `combining` the components of this instance.)<br>**combine?**<br>
    
    ```swift
    mutating func combine<H>(_ value: H) where H : Hashable
    ```
    
    제네릭 인스턴스 메소드로 **Hasher 구조체에서 value를 추가하는 메소드** 입니다.<br>
        **해셔에 주어진 값을 추가하여 그 필수적인 부분을 해셔 상태로 혼합합니다.**(Adds the given value to this hasher, mixing its essential parts into the hasher state.)

### 참고

1. [Dictionary - Apple Developer Document](https://developer.apple.com/documentation/swift/dictionary) 

2. [Hashable - Apple Developer Document](https://developer.apple.com/documentation/swift/hashable)

3. [hash(into:) - Apple Developer Document](https://developer.apple.com/documentation/swift/dictionary/2995338-hash)

4. [hasher - Apple Developer Document](https://developer.apple.com/documentation/swift/hasher)

5. [combine(_:) - Apple Developer Document](https://developer.apple.com/documentation/swift/hasher/2995578-combine)

6. [Equatable - Apple Developer Documentation](https://developer.apple.com/documentation/swift/equatable)

7. [Hashable 프로토콜](https://velog.io/@dev-lena/Hashable-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)
8. [equals와 hashCode는 왜 같이 재정의해야 할까?](https://woowacourse.github.io/javable/2020-07-29/equals-and-hashCode)

### 이미지 출처

1. hashCode 재정의 이미지: [equals와 hashCode는 왜 같이 재정의해야 할까?](https://woowacourse.github.io/javable/2020-07-29/equals-and-hashCode)

