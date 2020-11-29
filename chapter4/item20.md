# item 20. 추상 클래스보다는 인터페이스를 우선하라

`item 20`의 주제는 `추상 클래스보다는 인터페이스를 우선하라` 입니다. 이 아이템에서 Java의 추상 클래스와 인터페이스가 각각 무엇이고 어떤 목적을 위해 사용하는 지 알아볼 예정입니다. 그리고 스위프트에서 같은 목적을 이루기 위해 어떤 것들이 마련되어 있고, 어떻게 사용할 수 있는지를 함께 알아봅니다.



## Java

### Abstract class

추상 클래스란 구체적이지 않은 클래스를 의미합니다. 하나 이상의 추상 메소드(abstract method)를 포함합니다. **추상 메소드는 선언만 있고 본체는 없는 함수이며, 선언부에 `abstract` 라는 키워드를 붙입니다.**추상 메소드가 포함되었다면 클래스도 추상 클래스이므로 클래스명 앞에도 `abstract` 키워드를 붙여야 합니다. 

그리고 추상메소드의 접근 지정자로 `private`은 사용할 수 없습니다. 자식 클래스에서 해당 메소드를 구현해야만 하기 때문입니다.

```java
public abstract class Animal {
	public String name;		 					//일반 멤버 변수
  
  public abstract void sing();		//추상 메소드
  public void move() {			 		  //일반 메소드
  	System.out.println("어슬렁 어슬렁");
  }
}
```

추상 클래스는 추상 메서드를 포함하고 객체화 할 수 없다는 점을 제외하면 일반 클래스와 다르지 않습니다. 그리고 생성자, 멤버변수와 일반 메서드도 가질 수 있습니다. 인스턴스를 생성할 수 없으므로 추상 클래스 자체로는 객체를 생성할 수 없지만, 새로운 클래스를 작성하는 데 있어 부모 클래스로서의 중요한 역할을 가집니다.

```java
public class Dog extends Animal {
 public void move() {
   System.out.println("타닥 타닥");
 } 
  
  public void sing() {
    System.out.println("멍멍");
  }
}

public class Cat extends Animal {
   public void sing() {
    System.out.println("냥냥");
  }
}
```

**추상 클래스는 다른 클래스들이 공통으로 가져야하는 메소드의 원형을 정의하고, 그것을 상속받은 경우 반드시 구현하도록 강요합니다.** 메소드 오버라이드와 유사해서 혼동하기 쉬우나 오버라이드는 안해도 상관없습니다. 위의 예에서도 `Cat`은 `move()`를 오버라이드 하지 않았습니다. 그럼 `Cat`의 `move()`를 호출하게 되는 경우, `Animal`의 `move()`가 호출됩니다. 하지만 `sing()`은 `Animal`을 상속받은 `Dog`과 `Cat` 모두 구현해야만 합니다. 그리고 만약 어떤 추상클래스를 상속받은 자식 클래스에서 추상 메소드를 구현하지 않았다면 자식 클래스도 추상클래스가 되어야 합니다.

<br>

### Interface

인터페이스는 추상 클래스보다 한단계 더 추상화된 클래스라고 볼 수 있습니다. 추상 클래스보다 추상화 정도가 높아서 구현부를 지닌 일반 메소드나 멤버 변수를 가질 수 없고 오직 추상 메서드와 상수만을 가질 수 있습니다.

```java
interface 인터페이스이름 {
  public static final 자료형 변수명 = 변수값; 
  public abstract 반환자료형 함수명(입력인자);
}

//생략된 버전
interface 인터페이스이름 {
  자료형 변수명 = 변수값;
  반환자료형 함수명();
}
```

인터페이스의 멤버 변수는 `public static final` 로만 지정이 가능하고 생략할 수 있습니다. `final`이므로 선언시 변수 값을 반드시 지정해 주어야 한다. 멤버는 `public abstract`가 기본형이고 생략 가능합니다.

인터페이스도 직접 객체를 생성할 수 없습니다.

```java
interface Talkable {
  int volume = 1;
  void talk();
}

public class Robot implements Talkable {
  public void talk() {
    System.out.println("#$%^&*");
  }
}

public class Parrot implements Talkable {
  public void talk() {
    System.out.println("안녕");   
  }
}
```

인터페이스를 구체화하기 위해선 `implements` 키워드를 사용합니다. 그리고 `talk()`의 접근지정자는 반드시 `public`이어야 하는데, 인터페이스에서 (생략되었다 할 지라도) `public abstract`로 정의되었기 때문입니다.

인터페이스는 구현하고자 하는 여러 클래스의 공통적인 부분(정적 변수와 public 함수)만 기술해놓은 기초 설계도와 같습니다. 인터페이스를 구현하는 클래스에서 추상 메소드의 몸체를 모두 정의하도록 강요합니다. 만약 인터페이스가 구현하는 어떤 클래스가 모든 추상 메소드를 구현하지 않는다면 그 클래스는 추상클래스가 되어야 합니다. 

<br>



### 추상 클래스와 인터페이스의 공통점

- 선언만 있고 구현 내용은 없다
- 인스턴스화 할 수 없다
- 추상 클래스를 상속받은 자식과 인터페이스를 구현한 객체만 인스턴스화 할 수 있다



### 추상 클래스와 인터페이스의 차이점

- 추상클래스: 단일 상속 
- 추상 클래스의 목적: 상속을 받아 기능을 확장시키는 것(부모의 유전자를 물려받음)
- 인터페이스: 다중 상속
- 인터페이스의 목적: 구현하는 모든 클래스에 대해 반드시 특정한 메서드가 존재하도록 강제하는 역할. 구현 객체가 같은 동작을 한다는 걸 보장하기 위함



### 추상클래스와 인터페이스의 장점

- 개발 시간을 단축할 수 있다
  - 개발자들이 각각의 부분을 완성할 때까지 기다리지 않고 정의된 규약을 따라 각각 작업할 수 있음

- 표준화가 가능하다
  - 클래스의 기본 틀을 제공해 개발자들에게 정형화된 개발을 강요할 수 있음

- 서로 관계 없는 클래스들에게 관계를 맺어줄 수 있음
  - 클래스들의 과도한 상속을 줄여 코드의 종속성을 줄이고 유지보수를 용이하게 함

- 독립적인 프로그래밍이 가능함
  - 클래스간의 직접적인 관계가 아닌 인터페이스로 연결된 간접적인 관계를 맺어주면, 한 클래스의 변경이 다른 클래스에 영향을 미치지 않는 독립적인 프로그래밍이 가능해짐

<br>

추상 클래스와 인터페이스를 통해 얻을 수 있는 장점을 위해 Swift에서는 어떤 걸 이용할 수 있을지 알아보겠습니다.

## Swift

### Implementing abstract class 

스위프트에서는 추상클래스를 지원하지 않습니다. 추상 클래스에 대한 대안을 찾아보기위해 먼저 추상 클래스 패턴을 모방해보겠습니다.

```swift
class Animal {
  func sound() { 
  	print("새근새근")
  }
}

class Cat: Animal {
  override func sound() {
    print("냥")
  }
}

class Dog: Animal {
  override func sound() {
    print("멍")
  }
}
```

자바에서 추상클래스를 상속 받아 사용하는 것처럼 스위프트에서도 부모 클래스를 자식이 상속받아 사용할 수 있습니다. 하지만 여기의 `Animal` 은 추상적이지 않습니다. 

```swift
class Animal {
    func sound() {
        fatalError("Subclasses need to implement the `sound()` method.")
    }
}
class Cat: Animal {
  override func sound() {
    print("냥")
  }
}

class Dog: Animal {
  
}

let cat = Cat()
cat.sound()

let dog = Dog()
dog.sound()			// fatalError
```

만약 `Animal`의 `sound()`를 호출하게 되는 경우 런타임에러가 발생하도록 구현해서, 상속받는 자식 객체들은 반드시 `sound`를 override 해야 한다고 가정해봅시다. 하지만 상속을 이용하면 자식 객체가 특정 메소드를 반드시 구현해야 하는 것을 강요할 수 없습니다. 그래서 `Dog`이  `sound()`를 override 하지 않는다면 `dog.sound()`는 부모의 `sound()`를 호출하게 되어 런타임시점에 에러를 발생하게 됩니다. 이러한 누락이 컴파일 타임에 감지되지 않으면 개발자에게 무척 피곤한 일이 됩니다.

앞서 작성한 이러한 패턴은 추상 클래스를 그저 모방하는 것입니다. 이러한 방식 대신 Protocol이 무엇이고 어떻게 사용하는 지에 대해 알아봅시다.



### Protocol

프로토콜은 특정 역할을 하기 위한 메서드, 프로퍼티 등의 청사진으로, 구조체, 클래스, 열거형은 프로토콜을 채택해 요구사항을 실제로 구현할 수 있습니다. 프로토콜은 정의를 하고 제시 할 뿐 스스로 기능을 구현하지는 않습니다. 하나의 타입으로 사용됩니다. 

```swift
protocol 프로토콜이름 {
  //프로토콜 정의
}
```

프로토콜에서는 프로퍼티가 저장 프로퍼티인지 연산 프로퍼티인지 명시하지 않고, 이름과 타입, gettable/settable 여부만 명시합니다. 또한 프로퍼티는 항상 var로 선언되어야 합니다.

```swift
protocol Women {
  var height: Double { get set }
  var name: String { get }
  
  static func eat()
  func talk(to women: Women)
  mutating func run()
}
```

프로토콜에서는 인스턴스 메서드와 타입 메서드를 정의할 수 있습니다. 하지만 메서드 파라미터의 기본 값은 프로토콜 안에서 사용할 수 없습니다. 선언부만 작성하고 구현부는 작성하지 않기 때문입니다. `mutating` 키워드를 사용하면 인스턴스에서 변경 가능하다는 것을 나타냅니다.

프로토콜이 올바르게 구현되지 않았다면 컴파일러가 컴파일 시점에 알려주는 장점이 있습니다.

```swift
struct Delma: Women {
  var heartRate = 100
  var roundingHeight: Double = 0.0
  var height: Double {
    get {
      return roundingHeight
    }
    set {
      roundingHeight = 160.0
    }
  }
  var name: String = "delma"
  
  static func eat() {
    print("냠냠")
  }
  
  func talk(to women: Women) {
    print("\(women) 안녕")
  }
  
  mutating func run() {
    heartRate += 10
  }
}
```





<br>

### 참고

[자바의 인터페이스](https://studymake.tistory.com/426?category=646127)

[자바의 추상클래스](https://studymake.tistory.com/423)

[추상 클래스와 인터페이스](https://velog.io/@lshjh4848/추상클래스와-인터페이스-bok2k2qjrg)

[추상 클래스와 인터페이스 차이](https://cbw1030.tistory.com/47)

[프로토콜](https://medium.com/@jgj455/오늘의-swift-상식-protocol-f18c82571dad)

[추상클래스 In Swift](https://cocoacasts.com/how-to-create-an-abstract-class-in-swift)

