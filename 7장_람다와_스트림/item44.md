# 표준 함수형 인터페이스를 사용하라 

자바에는 함수형 인터페이스가 있고, 특히 매개변수로 함수 객체를 사용할 때 **타입**으로 사용되기도 합니다.

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