# Item35. ordinal 메서드 대신 인스턴스 필드를 사용하라 

자바의 경우, 대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응됩니다. 그리고 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공합니다.

## Java

###  ordinal를 잘못 사용한 경우

코드 35-1 ordianal을 잘못 사용한 예 - 따라하지 말것! 
```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;
        
    public int numberOfMusicians() { return ordinal() + 1; }
}
```

=> 동작은 하지만 유지보수하기가 끔찍한 코드입니다. 상수 선언 순서를 바꾸는 순간 `numberOfMusicians`가 오동작하며 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없습니다. 또한 값을 중간에 비워둘 수도 없습니다. 

### ordinal를 대체하는 해결책 

해결책은 간단합니다. **열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지 말고, 인스턴스 필드에 저장하면 됩니다.**

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBlE_QUARTET(8), 
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

### ordinal를 잘 사용한 예

EnumSet, EnumMap 의 key로 사용하는 경우입니다. 이런 성격의 목적에만 ordinal()를 사용하도록 합니다.

```java
import java.util.*;  

enum days {  
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY  
}  

public class EnumSetExample {  
    public static void main(String[] args) {  
        Set<days> set = EnumSet.of(days.TUESDAY, days.WEDNESDAY);  
    // Traversing elements  
        Iterator<days> iter = set.iterator();  
        while (iter.hasNext())  
            System.out.println(iter.next());  
    }  
}  
```

## Swift

* Swift의 rawValue는 값을 채택해야만 얻을 수 있습니다. (ex. Int, String) 

### 잘못된 쓰임새: Int - rawValue 사용하기 

* 자바 Enum의 ordinal()과 마찬가지로 스위프트에서는 Enum의 rawValue가 있습니다.
Int로 rawValue를 갖고 값을 명시하지 않으면 자바의 ordianl()과 같은 효과를 볼 수 있습니다.

```Swift
enum Ensemble: Int {
    case solo
    case duet
    case trio
    case quartet
    case quintet
    case sextet
    case septet
    case octet
    case nonet
    case dectet
        
    var numberOfMuscians: Int {
        return rawValue + 1
    }
}
```
=> 자바의 ordianl()과 마찬가지로, 동작은 하지만 유지보수하기가 끔찍한 코드입니다. 상수 선언 순서를 바꾸는 순간 `numberOfMusicians`가 오동작하며 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없습니다. 또한 값을 중간에 비워둘 수도 없습니다. 

> rawValue에 명시적으로 값 대입하여 사용하기 

* 명시적으로 rawValue에 값을 대입하면 위와 같은 상황이 해결될 것처럼 보입니다. 상수 선언 순서에도 의존되지 않고, 중간에 값을 비워도 되죠. 
그러나 값이 중복이 될때는 rawValue의 명시적 사용도 해결책이 될 순 없습니다. 
왜냐하면 rawValue는 unique 해야 되기 때문에 OCTET(= 8), DOUBLE_QUARTET(=8) 처럼 값이 중복이 되는 경우에는 rawValue를 사용할 수 없기 때문입니다.
그리고 numberOfMusicians 값을 rawValue의 값으로 할당해도 되는 대표 값인지 근거가 없기 때문에 사용하는 것은 부적절해 보입니다.

```Swift
enum Ensemble: Int, CaseIterable {
    case solo = 1
    case duet = 2
    case trio = 3
    case quartet = 4
    case quintet = 5
    case sextet = 6
    case septet = 7
    case octet = 8
    case doubleQuartet = 8 // 컴파일 에러! => Raw value for enum case is not unique
    case nonet = 9
    case dectet = 10
    case tripleQuartet = 12
    
    var numberOfMusicians: Int {
        return rawValue
    }
}
```

### rawValue를 대체하는 해결책: 연산프로퍼티(or 메서드) + default 구문 없는 switch 사용하기 

* Swift 에는 저장 프로퍼티를 둘 수 없고, 오직 연산 프로퍼티나 메서드만이 가능합니다. 따라서 rawValue의 대안책으로는 연산프로퍼티(or 메소드) + switch 방법입니다. 
* Swift의 switch는 **exhausive** 해서 default를 쓰지 않는한 값을 추가할 때 switch 컴파일 오류를 통해 case를 추가해야 함을 알 수 있습니다. 

```Swift
enum Ensemble: Int {
    case solo, duet, trio, quartet, quintet, sextet, septet, octet, doubleQuartet, nonet, dectet, tripleQuartet
    
    var numberOfMuscians: Int {
        switch self {
        case .solo:
            return 1
        case .duet:
            return 2
        case .trio:
            return 3
        case .quartet:
            return 4
        case .quintet:
            return 5
        case .sextet:
            return 6
        case .septet:
            return 7
        case .octet:
            return 8
        case .doubleQuartet:
            return 8
        case .nonet:
            return 9
        case .dectet:
            return 10
        case .tripleQuartet:
            return 12
        }
    }
}
```

## 번외: enum 상수들의 총 개수가 필요할때 CaseIterable을 채택해라 

* 과거 c언어에서는 enum의 모든 case의 개수를 구하기 위해 다음과 같이 코드를 작성하곤 했습니다. 

```c
enum MyType {
  Type1, // 0
  Type2, // 1  
  Type3,  // 2 
  NumberOfTypes // 3 == all count
}
```
=> 마지막 NumberOfTypes가 3이 되어 저절로 모든 case의 총 개수로 이용될 수 있습니다. 하지만 이 방법은 팀원들끼리 서로 알려줘야 하고, 이후 코드를 잘 못 건드릴 경우 총 개수로서 작동하지 않을 수 있는 위험이 있습니다. 

* 하지만 Swift 에는 CaseIterable을 채택만 하면 위와 같은 처리를 하지 않아도 안전하게 모든 case의 총 개수를 구할 수 있습니다.

```Swift 
enum MyType: CaseIterable {
    case type1
    case type2
    case type3
    
    static var count: Int {
        return allCases.count
    }
}
```

## References

https://docs.swift.org/swift-book/LanguageGuide/Enumerations.html
