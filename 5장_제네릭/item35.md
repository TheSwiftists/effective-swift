# ordinal 메서드 대신 인스턴스 필드를 사용하라 

대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다. 그리고 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal이라는 메서드를 제공한다.

유혹!!

## Java

코드 35-1 ordianal을 잘못 사용한 예 - 따라하지 말것! 
```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTET, OCTET, NONET, DECTET;
        
    public int numberOfMusicians() { return ordinal() + 1; }
}
```

## ordinal 를 대체하는 코드 

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

## ordinal를 잘 사용한 예

=> EnumSet, EnumMap 의 key로 사용하는 경우!!
<br>=> 이런 성격의 목적에만 사용하도록 하자.

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

```java
// Java program to illustrate working 
// of EnumMap and its functions.

import java.util.EnumMap;


public class EnumMapExample {
	public enum GFG {
		CODE, CONTRIBUTE, QUIZ, MCQ;
	}

	public static void main(String args[]) { 
		// Java EnumMap 
		// Creating EnumMap in java with key 
		// as enum type STATE
		EnumMap<GFG, String> gfgMap = new
					EnumMap<GFG, String>(GFG.class);

		// Java EnumMap Example 2:
		// Putting values inside EnumMap in Java
		// Inserting Enum keys different from 
		// their natural order
		gfgMap.put(GFG.CODE, "Start Coding with gfg");
		gfgMap.put(GFG.CONTRIBUTE, "Contribute for others");
		gfgMap.put(GFG.QUIZ, "Practice Quizes");
		gfgMap.put(GFG.MCQ, "Test Speed with Mcqs");
		
		// Printing size of EnumMap in java
		System.out.println("Size of EnumMap in java: " + 
									gfgMap.size());
	
		// Printing Java EnumMap 
		// Print EnumMap in natural order
		// of enum keys (order on which they are declared)
		System.out.println("EnumMap: " + gfgMap);
	
		// Retrieving value from EnumMap in java
		System.out.println("Key : " + GFG.CODE +" Value: "
								+ gfgMap.get(GFG.CODE));
	
		// Checking if EnumMap contains a particular key
		System.out.println("Does gfgMap has "+GFG.CONTRIBUTE+": "
							+ gfgMap.containsKey(GFG.CONTRIBUTE));
	
		// Checking if EnumMap contains a particular value
		System.out.println("Does gfgMap has :" + GFG.QUIZ + " : "
							+ gfgMap.containsValue("Practice Quizes"));
		System.out.println("Does gfgMap has :" + GFG.QUIZ + " : "
							+ gfgMap.containsValue(null));
	}
}
```


## Swift

잘못된 쓰임새

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


올바른 쓰임새 

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
    case doubleQuartet
    case nonet
    case dectet
    case tripleQuartet
    
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