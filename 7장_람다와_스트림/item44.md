# 표준 함수형 인터페이스를 사용하라 

자바에는 함수형 인터페이스가 있고, 특히 매개변수로 함수 객체를 사용할 때 **타입**으로 사용됩니다.

```java
private <T> List<T> filter(List<T> list, Predicate<T> predicate) {
    List<T> filteredList = new ArrayList<>();
    for(T element: list) {
        if (predicate.test(element)) {
            filteredList.add(element);
        }
    }
    return filteredList;
}
```

* 해당 자바 코드를 스위프트로 바꾸면 아래와 같습니다. 

```swift
private func filter<T>(list: [T], isIncluded: (T) -> Bool) -> [T] {
    var filteredList = [T]()
    for element in list {
        if isIncluded(element) {
            filteredList.append(element)
        }
    }
    return filteredList
}
```

## 표준 함수형 인터페이스란?

그 중에서도 java.util.function 패키지에서 제공하는 함수형 인터페이스를 **표준 함수형 인터페이스**라고 합니다. **책에서는 필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라고 조언합니다.** 

> 표준 함수형 인터페이스를 왜 사용하는가? 

자바도 스위프트처럼 함수를 일급 객체로 다룹니다. 즉 함수 자체를 함수 인수(매개변수)로 전달 할 수 있습니다. 
그럼 함수를 파라미터 타입으로 표현해야 하는데 자바의 경우 함수형 인터페이스(ex. `Predicate`, `Supplier`)로 표현합니다. 근데 함수형 인터페이스 각각은 `(인수) -> 반환타입` 이 같아도 됩니다. 아래 예시처럼요.

```java
interface Predicate<T> {
    boolean test(T t);
}

interface PredicateA<T> {
    boolean test(T t);
}

interface PredicateB<T> {
    boolean test(T t);
}

interface PredicateC<T> {
    boolean test(T t);
}
```

이처럼 같은 `(인수)-> 반환타입` 이어도 여러 개의 함수형 인터페이스로 나타낼 수 있고, 이는 이미 똑같은 `(인수) -> 반환타입`인 함수형 인터페이스가 존재하여도 그때그때 직접 구현하는 것은 매우 비효울적이라 할 수 있습니다. 
따라서 자바에서는 표준 함수형 인터페이스라는 걸 만든 것이고 함수를 전달할 때 **직접 구현하지 말고 만들어둔 표준 함수형 인터페이스를 쓰라고 하는 것**입니다. 또 이렇게 표준 함수형 인터페이스를 사용하게 되면 직접 구현한 함수형 인터페이스와 달리 인터페이스 이름만 보고도 **어떤 `(인수) -> 반환타입` 인지 알 수 있는 장점**이 있습니다. 

## 자바의 대표적인 표준 함수형 인터페이스

| 인터페이스 | 함수 시그니처 | 예 |
|---------|------------|---|
|UnaryOperator\<T> | T apply(T t) | String::toLowerCase | 
|BinaryOperator\<T> | T apply(T t1, T t2) | BigInteger::add | 
|Prediacate\<T> | boolean test(T t) | Collection::isEmpty | 
|Supplier\<T> | T get() | Instant::now | 
|Consumer\<T> | void accept(T t) | System.out::println |

<br>

* 해당 자바의 표준 함수형 인터페이스의 함수 시그니처를 스위프트 버전으로 바꾸면 아래와 같습니다.  

| 인터페이스 | Swift 버전 함수 시그니처 | 
|---------|-----------------|
|UnaryOperator\<T> | (T) -> T |
|BinaryOperator\<T> | (T, T) -> T |  
|Prediacate\<T> | (T) -> Bool |
|Supplier\<T> | () -> T |
|Consumer\<T> | (T) -> Void |

사실, 스위프트에는 인수로 함수 객체를 `(인수) -> (반환타입)` 형태로 나타내기 때문에 따로 표준 함수형 인터페이스가 필요없고, 따라서 Predicate니 Supplier니 용어를 알 필요가 자바보다는 적다고 할 수 있습니다. 
<br>하지만 그럼에도 Swift 코드에서도 인수 이름으로 `Predicate` 가 많이 보여 해당 부분은 알면 좋겠다고 판단하여 `Predicate` 에 대해 조사하였습니다. `Predicate`도 `Object` 나 `atomic` 처럼 프로그래밍 세계에서의 고유명사라고 생각하시면 될 것 같습니다.

## Predicate의 사전적 의미 

* 술어, 술부, 빈사, 단언하다, 서술하다
* 단정하다

예시) 

predicate the eternity of human life
=> 인간의 생명은 영원하다고 단언하다

Most religions predicate life after death.
=> 대개의 종교는 내세가 있다고 단언한다.


: 프로그래밍적으로 해석하자면,,, 난 이 내용의 결과가 `true` 라고 확신해,, 이 결과는 `false`라고 단언해,,  

## predicate 가 파라미터로 포함된 Swift 메소드들

> 다음 메소드들은 Array의 메소드들로 predicate가 파라미터로 포함된 메소드들입니다.

* `@inlinable public func first(where predicate: (Element) throws -> Bool) rethrows -> Element?`

*  `@inlinable public func drop(while predicate: (Element) throws -> Bool) rethrows -> ArraySlice<Element>`

* `@inlinable public func prefix(while predicate: (Element) throws -> Bool) rethrows -> ArraySlice<Element>`

* `@inlinable public func firstIndex(where predicate: (Element) throws -> Bool) rethrows -> Int?`

* `@inlinable public func last(where predicate: (Element) throws -> Bool) rethrows -> Element?`

* `@inlinable public func lastIndex(where predicate: (Element) throws -> Bool) rethrows -> Int?`

등등 predicate가 파라미터로 있는 메소드들이 있습니다.

## NSPredicate 

* `NSArray`, `NSMutableArray`, `NSMutableSet`, `NSOrderedSet` 등등에서 filter 혹은 filtered 메소드에서 파라미터로 사용됩니다. 

```swift
let array = NSArray(arrayLiteral: "A", "B", "", "C", "", "D")
let predicate = NSPredicate(format: "SELF != ''")

let filteredArray = NSArray(array: array.filtered(using: predicate))
print("Input array: \(array)") 
print("Filterd array: \(filteredArray)")

// 실행 결과 
// Input array: (
//    A,
//    B,
//    "",
//    C,
//    "",
//    D
// )
// Filterd array: (
//     A,
//     B,
//     C,
//     D
// )
```

* 정규표현식을 사용할 때에도 NSPredicate 가 사용될 수 있습니다. 

```swift
func validateEmail(withString userEmail: String) -> Bool {
    let emailRegEx = "[A-Z0-9a-z._%+-]+@[A-Za-z]+\\.[A-Za-z]{2,}"

    let emailTest = NSPredicate(format:"SELF MATCHES %@", emailRegEx)
    return emailTest.evaluate(with: userEmail)
}

print(validateEmail(withString: "kimdo2297@gmail.com"))
print(validateEmail(withString: "kim!!2297@gmail.com"))
print(validateEmail(withString: "kimdo2297@gmail.c"))
print(validateEmail(withString: "kimdo2297@g1230.com"))

// 실행 결과
// true
// false 
// false 
// false 
```

### 참고 

* [자바 표준 함수형 인터페이스](https://johngrib.github.io/wiki/java-functional-interface/)
* [RegEx with NSPrediacate](https://stackoverflow.com/questions/16852875/filter-nsarray-using-nspredicate)
* [NSArray using NSPredicate](https://stackoverflow.com/questions/16852875/filter-nsarray-using-nspredicate)
* [Predicates in Swift](https://www.swiftbysundell.com/articles/predicates-in-swift/)