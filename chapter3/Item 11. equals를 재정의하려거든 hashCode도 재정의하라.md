# Item 11. equals를 재정의하려거든 hashCode도 재정의하라



Car라는 클래스가 있습니다.

해당 예제는 car1와 car2가 논리적으로 같은 객체로 판단되도록 하는 것이 목표입니다.

 equals를 재정의한 후, 이 Car를 각 언어의 컬렉션(Collection) 타입의 자료구조에 넣어보면서 Car 타입의 인스턴스들이 논리적으로 같은 객채로 판단되는지 살펴봅시다. 

### 왜 Java에서는 equals와 hashCode는 같이 재정의해야 할까요?

1. **equals 재정의**

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

2. **List에 담는 경우**
   **결과**: List에 car1, car2 을 추가할 수 있습니다. 아무 문제가 없습니다.

   

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

3. **Set에 담는 경우(중복 값을 허용하지 않는 Collection에 담는 경우)**

   **예상 결과**: 위에는 논리적으로 두 객체를 같은 객체로 판단하였기 때문에 car1과 car2를 HashSet에 추가하고 난 뒤의 HashSet의 size는 1일 것입니다.
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
  hashCode를 equals와 함께 재정의하지 않으면 위의 예처럼 코드가 예상과 다르게 동작할 수 있습니다. 다시 말하자면,**  **hash 값을 사용하는 Java의 Collection(HashSet, HashMap, HashTable)을 사용할 때 문제가 발생할 수 있습니다. 따라서 equals를 재정의할 때 코드의 안전성을 높이려면 hashCode를 같이 재정의해주는 것이 좋습니다.**

4. **hashCode 재정의**

   3번의 결과는 왜 나오는 이유를 살펴봅시다.
   Java에서 Collection(HashMap, HashSet, HashTable)은 객체가 논리적으로 같은지 비교할 때 아래 그림과 같은 과정을 거칩니다.

   ![hash 값을 사용하는 객체의 비교 로직](https://woowacourse.github.io/javable/images/2020-07-29-equals-and-hashcode.png)

   

   hashCode 메서드의 리턴 값으로 먼저 동등객체인지 확인하고 equals 메서드의 리턴 값이 true여야 논리적으로 같은 객체라고 판단합니다. 그런데 Car 클래스에는 hashCode 메서드가 재정의 되어있지 않아 Object 클래스의 hashCode 메서드가 사용된 것입니다. Object 클래스의 hashCode 메서드는 객체의 고유한 주소 값을 int 값으로 변환하기 때문에 객체마다 다른 값을 리턴합니다. 두 개의 Car 객체는 equals로 비교도 하기 전에 서로 다른 hashCode 메서드의 리턴 값으로 인해 다른 객체로 판단된 것입니다.

   

   그렇다면 hashCode를 재정의해봅시다.

   ```java
   public class Car {
       private final String name;
   
       public Car(String name) {
           this.name = name;
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



### 참고

[Hashable 프로토콜](https://velog.io/@dev-lena/Hashable-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)

[Hashable - Apple Developer Documentation](https://developer.apple.com/documentation/swift/hashable)

[Equatable - Apple Developer Documentation](https://developer.apple.com/documentation/swift/equatable)

[equals와 hashCode는 왜 같이 재정의해야 할까?](https://woowacourse.github.io/javable/2020-07-29/equals-and-hashCode)