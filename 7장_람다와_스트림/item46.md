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

## side effect가 있는 스트림 코드

#### Java
```java
void sideEffectStream(String file) {
    Map<String, Long> freq = new HashMap<>();
    try(Stream<String> words = new Scanner(file).tokens()) {
        words.forEach(word -> { freq.merge(word.toLowerCase(), 1L, Long::sum); });
    }
    System.out.println(freq); 
}

public static void main(String[] args) {
    Item46 item = new Item46();
    item.sideEffectStream("""
            Hello I'm Jason.
            Why not? i'm jason.
            """);
    // {why=1, hello=1, jason.=2, i'm=2, not?=1}
}
```

#### Swift
```swift
func sideEffectStream(file: String) {
    var freq = [String: Int]()
    let words = file.split(separator: " ").map { String($0) }
    words.forEach { word in
        freq.merge(key: word.uppercased(), value: 1) { count, incr in count +  }
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
```

위 코드는 외부 상태(freq)를 수정하며 side-effect를 발생시키는 스트림 코드입니다( `word -> { freq.merge(word.toLowerCase(), 1L, Long::sum); });` 구문을 보면 forEach 문으로 반복적으로 freq를 수정하는 것을 알 수 있습니다 ).
<br>forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것을 보니 나쁜 코드일 것 같은 냄새가 납니다.

## side effect가 없는 스트림 코드 

#### Java
```java
void nonSideEffectStream(String file) {
    Map<String, Long> freq;
    try(Stream<String> words = new Scanner(file).tokens()) {
        freq = words.collect(groupingBy(String::toLowerCase, counting()));
    }
    System.out.println(freq);
}
```

```java
<R, A> R collect(Collector<? super T, A, R> collector);
```

```java
Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                          Collector<? super T, A, D> downstream) {
    return groupingBy(classifier, HashMap::new, downstream);
}
```

```java
public static <T> Collector<T, ?, Long> counting() {
    return summingLong(e -> 1L);
}

summingLong(ToLongFunction<? super T> mapper) {
    return new CollectorImpl<>(
            () -> new long[1],
            (a, t) -> { a[0] += mapper.applyAsLong(t); },
            (a, b) -> { a[0] += b[0]; return a; },
            a -> a[0], CH_NOID);
}
```

#### Swift
```swift
func nonSideEffectExample(file: String) {
    let words = file.split(separator: " ").map { String($0) }
    let freq: [String: Int] = [String: [String]](grouping: words, by:{ $0.uppercased() })
        .mapValues { values -> Int in values.count }
        
    print(freq)
}
```

```swift
@inlinable public init<S>(grouping values: S, by keyForValue: (S.Element) throws -> Key) rethrows where Value == [S.Element], S : Sequence
```

=> 앞서와 같은 일을 하지만, 이번엔 스트림 API를 제대로 사용했습니다. 그뿐만 아니라 짧고 명확합니다. 

## forEach문은 보고할 때만 사용하고 계산할 때는 사용하지 마십시오 

자바 프로그래머(스위프트도 마찬가지)라면 for-each 반복문을 사용할 줄 알 텐데, for-each 반복문은 forEach 종단 연산과 비슷하게 생겼습니다. 하지만 forEach 연산은 종단 연산 중 기능이 가장 적고 가장 **'덜' 스트림**답습니다. 대놓고 **반복적이라서 병렬화할 수도 없습니다.** 
<br>**forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 마십시오. 반복문을 사용하십시오.**

## Collectors Method 

### toList

* 스트림을 컬렉션 List로 변환해주는 메서드입니다. 

```java
List<String> topTen = freq.keySet().stream()
                .sorted(comparing(freq::get).reversed())
                .limit(10)
                .collect(Collectors.toList());
```

```swift
let topTen = freq.keys
                .sorted { (lhs, rhs) -> Bool in freq[lhs]! > freq[rhs]! }[0 ... 10]
                     
```

### toMap

* 스트림을 컬렉션 Map으로 변환해주는 메서드입니다.

#### Java
```java
private static final Map<String, Operation> stringToEnum
                = Stream.of(values()).collect(toMap(Object::toString, e -> e));
```

#### Swift
```Swift
enum Operation: CaseIterable {
    case plus
    case minus
    case times
    case divide
    
static func stringToEnum() -> [String: Operation] {
    return [String: [Operation]](grouping: allCases, by: { operation in "\(operation)" })
            .mapValues { value in value.first! }
}
```


### groupingBy

groupingBy는 다중정의된 메서드로 총 3가지 메서드가 있습니다. 

* classifier만 사용하는 메서드 

```java
Map<String, List<String> map = words.collect(groupingBy(word -> alphabetize(word)))
```

* classifier와 downstream을 사용하는 메서드 

```java
Map<String, Long> freq;
try(Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
```

* classfire와 downstream, mapFactory를 모두 사용하는 메서드가 있습니다.  
