# 표준 함수형 인터페이스를 사용하라 

자바에는 함수형 인터페이스가 있고, 특히 매개변수로 함수 객체를 사용할 때 **타입**으로 사용되기도 합니다.

```java
private <T> List<T> filter(List<T> list, Predicate<T> predicate) {
    List<T> filteredList = new ArrayList<>();
    for(T type: list) {
        if (predicate.test(type)) {
            filteredList.add(type);
        }
    }
    return filteredList;
}


 ```

## 대표적인 표준 함수형 인터페이스

| 인터페이스 | 함수 시그니처 | 예 |
|---------|------------|---|
|UnaryOperator\<T> | T apply(T t) | String::toLowerCase | 
|BinaryOperator\<T> | T apply(T t1, T t2) | BigInteger::add | 
|Prediacate\<T> | boolean test(T t) | Collection::isEmpty | 
|Supplier\<T> | T get() | Instant::now | 
|Consumer\<T> | void accept(T t) | System.out::println | 
