# Item46. 스트림에서는 부작용 없는 함수를 사용하라 

스트림은 처음 봐서는 이해하기 어려울 수 있습니다. 
원하는 작업을 스트림 파이프 라인으로 표현하는 것 조차 어려울지 모릅니다. 
성공하여 프로그램이 동작하더라도 장점이 무엇인지 쉽게 와 닿지 않을 수도 있습니다. 
**스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임**이기 때문입니다.
스트림이 제공하는 **표현력, 속도, (상황에 따라서는)병렬성**을 얻으려면 API는 말할 것도 없고 이 패러다임까지 함께 받아들여야 합니다. 

스트림 패러다임의 핵심은 계산을 **일련의 변환(transformation)** 으로 재구성하는 부분입니다.
이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 **순수 함수(pure function)** 여야 합니다.
순수 함수란 **오직 입력만이 결과에 영향을 주는 함수**를 말합니다.
다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않습니다.
이렇게 하려면 스트림 연산에 건네는 함수 객체는 **모두 부작용(side effect)가 없어야 하므로 순수 함수**이어야 합니다.

## Side effect가 있는 스트림 코드

```swift
func sideEffectStream(file: String) {
    var freq = [String: Int]()
    let words = file.split(separator: " ").map { String($0) }
    words.forEach { word in
        freq.merge(key: word.lowercased(), value: 1) { count, incr in count + incr }
    }
    print(freq)
}

extension Dictionary {
    mutating func merge(key: Key, value: Value, remappingHandler: (Value, Value) -> (Value)) {
        let oldValue = self[key]
        let newValue = (oldValue == nil) ? value : remappingHandler(oldValue!, value)

        self[key] = newValue
    }
}

sideEffectExample(file: "Hello I'm Jason. Why not? i'm jason.") // ["not?": 1, "hello": 1, "jason.": 2, "why": 1, "i\'m": 2]
```

위 코드는 외부 상태(freq)를 수정하며 side-effect를 발생시키는 스트림 코드입니다. `word -> { freq.merge(word.toLowerCase(), 1L, Long::sum); });` 구문을 보면 forEach 문으로 반복적으로 freq를 수정하는 것을 알 수 있습니다.
<br>forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것을 보니 나쁜 코드일 것 같은 냄새가 납니다. **forEach가 계산을 하는 코드는 보통 외부 값을 수정하는 side effect가 일어나는 코드**이기 때문입니다.

다음은 올바르게 작성한 스트림 코드를 보겠습니다. 

## Side effect가 없는 스트림 코드 

```swift
func nonSideEffectExample(file: String) {
    let words = file.split(separator: " ").map { String($0) }
    let freq: [String: Int] = [String: [String]](
        grouping: words, 
        by:{ $0.lowercased() }
    ).mapValues { values -> Int in values.count }

    print(freq)
}
```

```swift
@inlinable public init<S>(grouping values: S, by keyForValue: (S.Element) throws -> Key) rethrows where Value == [S.Element], S : Sequence
```

=> 앞서와 같은 일을 하지만, 이번엔 스트림 API를 제대로 사용했습니다. 그뿐만 아니라 짧고 명확합니다.

## forEach문은 보고할 때만 사용하고 계산할 때는 사용하지 마십시오 

자바 프로그래머(스위프트도 마찬가지)라면 for-each 반복문을 사용할 줄 알텐데, for-each 반복문은 forEach 종단 연산과 비슷하게 생겼습니다. 
하지만 forEach 종단 연산은 종단 연산 중 기능이 가장 적고 가장 **'덜' 스트림**답습니다. 
대놓고 **반복적이라서 병렬화할 수도 없습니다.** 
<br>**forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 마십시오.**
forEach로 계산한다는 것은 외부 상태를 수정한다는 뜻입니다. 
반복문을 사용하세요.
물론 가끔은 forEach문이 스트림 계산 결과를 기존 컬렉션에 추가하는 등의 다른 용도로는 쓰일 수 있습니다. 

## Collectors Method: toList, toMap, toSet

* 자바에서는 스트림을 사용하는데 수집기(Collector)를 사용할 수 있습니다. `java.util.stream.Collectors` 클래스는 메서드를 무려 39개나 가지고 있고, 타입 매개변수가 5개나 되는 것도 있습니다. 
<br>수집기는 총 세가지로, toList(), toMap(), toSet() 가 주인공입니다. 이 중에서 toList()와 toMap()을 알아보았습니다.

* 스위프트는 따로 Collectors Method를 제공하고 있지 않습니다. 따라서 자바 스트림 코드를 그대로 변환만 해보았습니다. 

### toList

* 스트림을 컬렉션 List로 변환해주는 메서드입니다. 

> Java
```java
List<String> topTen = freq.keySet().stream()
                .sorted(comparing(freq::get).reversed())
                .limit(10)
                .collect(Collectors.toList());
```

> Swift
```swift
let topTen: [String] = freq.keys
    .sorted { (lhs, rhs) -> Bool in freq[lhs]! > freq[rhs]! }
    .enumerated()
    .filter { (index, _ ) in  return index >= 0 && index < 10 }
    .map { $0.element }
```

### toMap

* 스트림을 컬렉션 Map으로 변환해주는 메서드입니다.

> Java
```java
private static final Map<String, Operation> stringToEnum
                = Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

> Swift
```Swift
enum Operation: CaseIterable {
    case plus
    case minus
    case times
    case divide

    static func stringToEnum() -> [String: Operation] {
        return [String: [Operation]](
            grouping: allCases, 
            by: { operation in "\(operation)" }
        ).mapValues { value in value.first! }
    } // 일단 이렇게 변환하는게 저는 최선입니다. 
}
```

## Collectors Method: groupingBy

groupingBy 메서드는 Collectors의 또 다른 메서드로, 입력으로 분류함수(classifier)를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환합니다. 그리고 **이 카테고리가 해당  원소의 맵 키로 쓰입니다.**  
groupingBy는 다중정의된 메서드로 총 3가지 메서드가 있습니다. 

* classifier만 사용하는 메서드 

groupingBy 메서드는 가장 간단한 것으로서 분류함수 classfier만 인수로 받고 Map을 반환합니다. 반환된 맵에 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 List 입니다.

> Java

```java
Map<String, List<String>> map = words.collect(groupingBy(word -> alphabetize(word)))
```

* classifier와 downstream을 사용하는 메서드 

* groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림(downstream) 수집기도 명시해야 합니다. 아래와 같이 다운스트림 수집기로 counting()을 건네는 방법도 있습니다. 이렇게 하면 각 카테고리(키)를 (원소를 담은 컬렉션이 아닌) 해당 카테고리에 속하는 원소의 개수(값)와 매핑한 맵을 얻는다.

```java
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

* classfier와 downstream, mapFactory를 모두 사용하는 메서드가 있습니다.

> Swift 

자바의 groupingBy에 대응되는 것은 딕셔너리의 생성자 `init(grouping:by:)` 라고 할 수 있겠습니다. 

* Declaration

`init<S>(grouping values: S, by keyForValue: (S.Element) throws -> Key) rethrows where Value == [S.Element], S : Sequence`

=> 선언에서 알수 있듯이 해당 생성자를 사용하면 해당 디셕너리의 value 타입은 타입 파라미터 S의 배열임을 알 수 있습니다. 즉 타입은 `[S: [S]]` 입니다. 

* 위 자바 코드에 대응되는 스위프트 코드입니다. 

```swift
let map: [String: [String]] = [String: [String]].init(grouping: words, by: { (word) in alphabetize(word) })
```

## Swift 의 enumerated, zip 

또 스위프트의 스트림을 사용할 때 유용한 메소드로 `enumerated` 와 `zip` 이 있습니다. 

* enumerated

쌍의 시퀀스 (n, x)를 반환합니다. 여기서 n은 0에서 시작하는 연속 정수 즉 index를 나타내고, x는 시퀀스의 요소(value)를 나타냅니다.

```swift
"Swift"
    .enumerated()
    .forEach { n, x in print(n, x) }
// Prints "0: 'S'"
// Prints "1: 'w'"
// Prints "2: 'i'"
// Prints "3: 'f'"
// Prints "4: 't'"
```

* zip

두 타입을 합쳐 tuple로 만들어 주는 기능을 합니다. 

```swift
let names: Set = ["Sofia", "Camilla", "Martina", "Mateo", "Nicolás"]
zip(names.indices, names)
            .filter { (indice, name) -> Bool in return name.count <= 5}
            .forEach { (indice, name) in print(names[indice]) }
// Prints "Sofia"
// Prints "Mateo"
```