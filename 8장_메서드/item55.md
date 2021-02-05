# Item 55. 옵셔널 반환은 신중히 하라 

## 자바의 예외 처리, null 그리고 Optional

> 자바는 스위프트와 다르게 Null과 Optional 이 서로 연관된 개념이 아닌 분리된 개념입니다. 오히려 Null의 또 다른 대응책으로 Optional이 제안됩니다. 

자바 8 전에는 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택지가 두 가지 있었습니다.

* 예외를 던진다. 
* (반환 타입이 객체 참조라면) null을 반환한다.

두 방법 모두 허점이 있습니다. 
* 예외는 **진짜 예외적인 상황에서만 사용해야 하며(아이템 69)**, 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용이 만만치 않습니다.
* null을 반환할 수 있는 메서드를 호출할 때는, (null이 반환될 일이 절대 없다고 확신하지 않는 한) 별도의 null 처리 코드를 추가 합니다. null 처리를 무시하고 반환된 null 값을 어딘가에 저장해두면 언젠가 **NullPointerException**이 발생할 수 있습니다.    

자바 8 이후로는 또 하나의 선택지인 Optional\<T>이 생겼습니다. Optional\<T>는 null 이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있습니다. 옵셔널은 원소를 최대 1개 가질 수 있는 '불변' 컬렉션입니다. Optional\<T>가 Collection\<T>를 구현하지는 않았지만, 원칙적으로 그렇다는 것입니다.

* 보통은 T를 반환해야 하지만 특정 조건에서는 **아무것도 반환하지 않아야 할 때** T 대신 Optional\<T>를 반환하도록 선언하면 됩니다. 그러면 유효한 반환값이 없을 때는 빈 결과를 반환하는 메서드가 만들어집니다. 옵셔널을 반환하는 메서드는 **예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적습니다.** 

> 코드 55-1 컬렉션에서 최대값을 구한다(컬렉션이 비었으면 예외를 던진다).

```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty()) {
        throw  new IllegalArgumentException("빈 컬렉션");
    }

    E result = null;
    for(E e: c) {
        if (result == null || e.compareTo(result) > 0) {
            result = Objects.requireNonNull(e);
        }
    }
    
    return result;
}
```

> 코드 55-2 컬렉션에서 최댓값을 구해 Optional<E>로 반환한다. 

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty()) {
        return Optional.empty();
    }

    E result = null;
    for(E e: c) {
        if (result == null || e.compareTo(result) > 0) {
            result = Objects.requireNonNull(e);
        }
    }

    return Optional.of(result);
}
```
=> 빈 옵셔널은 Optional.empty()로 만들고, 값이 든 옵셔널은 Optional.of(value)로 생성했습니다. Optional.of(value)에 null을 넣으면 NullPointerException을 던지니 주의합시다. 

> 코드 55-3 스트림 버전

```java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
```

## 스위프트의 Optional

스위프트에서는 **Optional<T> 만이 `nil`을 취할 수 있습니다.** 
옵셔널로 Wrapping 되어있지 않은 타입은 `nil`을 취할 수 없습니다. 옵셔널 안에 값이 있는 경우 Optional(T)로 표현되지만, 옵셔널 안에 값이 없는 경우 `nil`로 표현됩니다.

```Swift
let nonOptionalValue: Int = 6
print(nonOptionalValue) // 6

var optionalValue: Int? = nil
print(optionalValue) // nil
optionalValue = 6
print(optionalValue) // Optional(6)
```

Swift 에서는 특정 조건에서 값을 반환할 수 없을 때 주로 Optional 타입을 사용해 nil을 반환합니다. **예외(Error)는 정말 예외인 경우에 사용합니다.**  
다음은 위의 자바 코드에 대해 대응되는 적절한 예시의 Swift 코드입니다. 

> 코드 55-2 를 Swift로 바꾼 코드 
```Swift
static func max<T: Comparable>(collections: [T]) -> T? {
    if collections.isEmpty { return nil }
    
    var result: T = collections.first!
    for element in collections {
        if element > result {
            result = element
        }
    }
    
    return result
} 

let maxValue = max(collections: [1, 2, 3])
print(maxValue) // Optional(3)

if let maxValue = max(collections: [1, 2, 3]) {
    print(maxValue) // 3
}

guard let maxValue = max(collections: [1, 2, 3]) else { return }
print(maxValue) // 3
```
=> `if let` 구문이나 `guard let` 구문을 통해 Optional값을 unwrap 합니다. `max` 메서드의 반환값이 nil 인 경우 maxValue 는 값이 할당되지 않고, 반환값이 nil 이 아닌 경우 maxValue 에 값이 할당되어 print 됨을 알 수 있습니다. maxValue 의 타입은 Int, 즉 non-optional 합니다. 

> 코드 55-3을 Swift로 바꾼 코드 (스트림 버전)

```swift
static func max<T: Comparable>(collections: [T]) -> T? {
    return collections.max()
}
```

## 옵셔널 활용

* 메서드가 옵셔널을 반환한다면 클라이언트는 값을 받지 못했을 때 취할 행동을 선택해야 합니다. 선택지로는 기본값 설정, 예외던지기, 값 바로 꺼내기 등 3가지 방법이 있습니다.  

### 옵셔널 활용 1 - 기본값을 정해둘 수 있습니다. 

> Java 

```java
String lastWordInLexicon = max(words).orElse("단어 없음...");
```

> Swift

```Swift
let maxValue: Int = Example.max(collections: [ ]) ?? 100
print(maxValue) // 100
```
=> 메소드 반환값이 nil 인 경우 `??` 을 이용해 디폴트 값으로 설정된 100이 반환됩니다. 

### 옵셔널 활용 2 - 원하는 예외를 던질 수 있습니다.

또는 상황에 맞는 예외를 던질 수 있습니다. 다음 코드에서 실제 예외가 아니라 예외 팩터리를 건넨것에 주목합시다. 이렇게 하면 예외가 실제로 발생하지 않는 한 예외 생성 비용은 들지 않습니다. 

> Java 

```java
Toy myToy = max(toys).orElseThrow(SometingException::new);
```

> Swift 

Swift 는 해당 기능이 없습니다. 따라서 똑같이 기능을 갖추려면 extension 을 이용해 기능 구현해야 합니다. 

```swift
extension Optional {
    func orThrow(_ errorExpression: @autoclosure () -> Error) throws -> Wrapped {
        switch self {
        case .some(let value):
            return value
        case .none:
            throw errorExpression()
        }
    }
}

let file = try loadFile(at: path).orThrow(MissingFileError())
```
=> [Swift by Sundell](https://www.swiftbysundell.com/tips/unwrapping-an-optional-or-throwing-an-error/)의 예시입니다. 

### 옵셔널 활용 3 - 항상 값이 채워져 있다고 가정합니다. 

> Java

옵셔널에 항상 값이 채워져 있다고 확신한다며 그냥 곧바로 값을 꺼내 사용하는 선택지도 있습니다. 다만 잘못 판단한 것이라면 `NoSuchElementException` 이 발생합니다. 

```java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
```

> Swift
Swift 에서는 `!`을 이용하면 강제적으로 Optional 에서 값을 바로 꺼내 사용할 수 있습니다(Forced Unwrapping). 대신 마찬가지로 값이 없는 경우엔 런타임 에러가 발생합니다. 

```Swift
let lastNobleGas: Element = Example.max(collections: Elements.NobleGases)!
// ! 을 사용해서 타입은 옵셔널이 아닙니다. 
```

## 옵셔널의 안전 밸브 역할의 메서드 

> Java

* isPresent 메서드 
  * 옵셔널이 채워져 있으면 true를, 비어 있으면 false를 반환합니다.

```java
static Optional<String> getOptionalName() {
    return Optional.of("Jason");
}

Optional<String> name = getOptionalName();
System.out.println("내 이름: " + (name.isPresent() ? name.get() : "이름 없음"));
```

> Swift 

* 옵셔널 바인딩 (`if let`, `guard let`) 
  * 값이 nil이 아닌 경우에만 unwrapping 되어 변수에 값이 할당됩니다. !(forced unwrapping) 보다 매우 안전합니다. 

```swift
if let maxValue = max(collections: [1, 2, 3]) {
    print(maxValue) // 3
}

guard let maxValue = max(collections: [1, 2, 3]) else { return }
print(maxValue) // 3
```

## 반환 값으로 옵셔널을 사용하면 안되는 경우 

반환 값으로 옵셔널을 사용한다고 해서 무조건 득이 되는 건 아닙니다. 
**컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안됩니다.** 빈 Optional<List\<T>> 를 반환하기 보다는 빈 List<T>를 반환하는 게 좋습니다(아이템 54).
<br>빈 컨테이너를 그대로 반환하면 클라이언트에 **옵셔널 처리 코드를 넣지 않아도 됩니다!** 

> BAD
```Swift
func generateCards() -> [Card]? // 외부에서 옵셔널 처리를 해야 하는 번거로움이 있습니다. 
```

> GOOD
```Swift
func generateCards() -> [Card] // 외부에서 옵셔널 처리를 하지 않아도 됩니다. 

```

## 반환값으로 옵셔널을 사용해야 하는 경우

결과가 없을 수 있을 때 사용하고 **Swift의 경우, 제일 우선되는 선택지**입니다. Java는 null과 Optional이 분리된 개념이라 두 가지 선택지를 모두 고려해야 하지만, Swift는 옵셔널과 nil이 연관된 개념이기 때문에 옵셔널을 먼저 고려합니다. 

## Java 맵, Swift 딕셔너리

> Java: Map

맵(Map)의 값으로 **옵셔널을 사용하면 절대 안됩니다.** 만약 그리 한다면 맵 안에 키가 없다는 사실을 나타내는 방법이 두 가지가 됩니다. 하나는 키 자체가 없는 경우이고, 다른 하나는 키는 있지만 그 키가 속이 빈 옵셔널(`Optional.empty()`)인 경우입니다. 쓸데없이 복잡성만 높여서 혼란과 오류 가능성을 키울 뿐입니다. 

```java
Map<String, Optional<Integer>> map = Map.of("key", Optional.empty());
System.out.println(map.get("notKey")); // null
System.out.println(map.get("key")); // Optional.empty
```
=> 자바는 위와 같이 키가 없는 경우 null을 반환합니다. 

> Swift: Dictionary

딕셔너리(Dictionary)는 키와 대응하는 값이 있든 없든, **특정 키로 대응하는 값을 반환할 때 옵셔널 타입으로 반환합니다.** 따라서 반환된 값은 키가 없으면 nil이 나오고, 키가 있으면 값이 옵셔널로 래핑되어 도출됩니다. 
따라서 딕셔너리의 값 타입도 옵셔널타입은 지양해야 합니다. 자바의 Map과 마찬가지로 맵 안에 키가 없다는 사실이 키 자체가 없는 경우와 키는 있지만 값이 nil인 경우 총 2가지가 되어 복잡성만 높이기 때문입니다. 

```swift
let three = dic["three"]
print(three) // nil 

let four = dic["four"]
print(four) // Optional(4)

if let four = four { // unwrapping
    print(four) // 4
}
```

=> **일반화해 이야기하면 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없습니다.**

## 프로퍼티로 옵셔널 타입을 두는 경우는 필수적인 프로퍼티가 아닌 선택적 프로퍼티인 경우에 둡니다. 

