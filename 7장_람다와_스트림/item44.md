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
