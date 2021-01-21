# item43. 람다보다는 메서드 참조를 사용하라 

람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 간결함입니다.
자바에는 함수 객체를 심지어 람다보다도 더 간결하게 만드는 방법이 있으니, 바로 **메서드 참조(mthod reference)** 입니다.

## Java 

### 람다 

다음 코드는 임의의 키와 Integer 값의 매핑을 관리하는 프로그램의 일부입니다. 이때 값이 키의 인스턴스 개수로 해석된다면, 이 프로그램은 멀티셋(multiset)을 구현한 게 됩니다. 이 코드는 키가 맵안에 없다면 키와 숫자 1을 매핑하고, 이미 있다면 있다면 기존 매핑 값을 증가시킵니다. 

> Note. 멀티셋(multiset)이란?
<br><br>[multiset](https://stackoverflow.com/a/2858291/11612129)은 본질적으로 map으로서 key를 가지고 있고 key에 대한 개수를 값으로 가진 자료구조입니다.
<br> 예를 들어, 상품별로 재고의 개수나 팔린 상품의 개수를 저장하고 사용할 때 유용합니다.

```java
map.merge(key, 1, (count, incr) -> count + incr);

// default merge method of Map interface
default V merge(K key, V value,
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
               remappingFunction.apply(oldValue, value);
    if (newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```
위 코드의 Map의 merge 메서드는 키, 값, 함수를 인수로 받으며, 주어진 키가 맵 안에 없다면 주어진 {키, 값} 쌍을 그대로 저장합니다. 여기서 count는 해당 key의 원래 값을 나타내고 incr는 매개변수 값으로 넣은 1을 나타냅니다. 
<br>따라서 해당 코드는 key에 기존 값이 없으면 값에 1을 할당하고, key에 기존 값이 있으면 기존 값에 1을 더한 값이 key의 값이 되는 코드입니다.
깔끔해 보이는 코드지만 매개변수인 count와 incr은 크게 하는 일 없이 공간을 꽤 차지합니다. 사실 이 람다는 두 인수의 합을 단순히 반환할 뿐입니다. 

### 메서드 참조
```java 
map.merge(key, 1, Integer::sum);

// sum method of Interger class 
public static int sum(int a, int b) {
    return a + b;
}
```
자바 8이 되면서 Integer 클래스(와 모든 기본 타입의 박싱타입)는 위의 람다와 기능이 같은 정적 메서드 sum을 제공하기 시작했습니다. 따라서 람다 대신 이 메서드의 참조를 전달하면 똑같은 결과를 보기 좋게 얻을 수 있습니다.

<br>

## 람다가 메서드 참조보다 나은 경우 

1. 람다로 작성하면 매개변수(ex. count, incr)를 꼭 작성해야 하지만 메서드 참조는 매개변수를 작성하지 않습니다. 따라서 매개변수 수가 늘어날수록 메서드 참조로 제거할 수 있는 코드 양도 늘어납니다.
하지만 어떤 람다에서는 매개변수의 이름 자체가 프로그래머에게 좋은 가이드가 되기도 합니다. **이런 람다의 길이는 더 길지만 매개변수가 있어서 메서드 참조보다 읽기 쉽고 유지보수도 쉬울 수 있습니다.**

2. 람다가 메서드 참조보다 간결할 때가 있습니다. **주로 메서드와 람다가 같은 클래스에 있을 때 그렇습니다.** 예를 들어 다음 코드가 `GoshThisClassNameIsHumongous` 클래스 안에 있다고 해봅시다.  

```java
service.execute(GoshThisClassNameIsHumongous::acton);
```

이를 람다로 대체하면 다음처럼 됩니다. 

```java
service.execute(() -> acton());
```

### 메서드 참조 유형 

* 메서드 참조 유형은 다섯 가지가 있습니다. 제일 흔히 사용하는 것은 정적 메서드 참조입니다.

| 메서드 참조 유형 | 예 | 같은 기능을 하는 람다 |
|--------------|---|------------------|
| 정적 | Integer::parseInt | str -> Integer.parseInt(str) |
| 한정적(인스턴스) | Instant.now()::isAfter | Instant then = Instant.now();<br> t -> then.isAfter(t); |
| 비한정적(인스턴스) | String::toLowerCase | str -> str.toLowerCase() | 
| 클래스 생성자| TreeMap<K,V>::new | () -> new TreeMap<K,V>() |
| 배열 생성자| int[]::new | len -> new int[len] |

#### 인스턴스 메서드를 참조하는 유형이 두 가지 있습니다. 그중 하나는 수신 객체(receiving object; 참조 대상 인스턴스)를 특정하는 한정적(bound) 인스턴스 메서드 참조이고, 다른 하나는 수신 객체를 특정하지 않는 비한정적(unbound) 인스턴스 메서드 참조입니다. 

* 한정적 참조는 근본적으로 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같습니다. 

```java
Predicate<Instant> predicate;

// lambda
Instant then = Instant.now();
predicate = (t) -> then.isAfter(t);

// method reference
predicate = then::isAfter;

// anonymous class     
predicate = new Predicate<Instant>() {
    @Override
    public boolean test(Instant instant) {
        return then.isAfter(instant);
    }
};

Instant other = Instant.now();
predicate.test(other); // true

// isAfter method of Instant class 
public boolean isAfter(Instant otherInstant) {
    return compareTo(otherInstant) > 0;
}
```
=> 인수는 `then` 으로 한정되어 있고, 인수 `then`은  매개변수로 따로 추가되어 있지 않습니다. isAfter는 다른 Instant와 시간을 비교하는 메서드로 위의 경우 결과적으로 `then.isAfter(other)`가 호출됨을 알 수 있습니다.
<br>=> 메서드 참조가 수신 객체 then을 **캡처(capture)** 하고 있음을 알 수 있습니다. 자바는 스위프트와 달리 캡처한 객체(or primitive type)는 주소값(or 값)이 바뀌어선 안됩니다(`java: local variables referenced from an inner class must be final or effectively final`).

* 비한정적 참조에서는 함수 객체를 적용하는 시점에 수신 객체를 알려줍니다. 이를 위해 수신 객체 전달용 매개변수가 매개변수 목록의 첫번째로 추가되며, 그 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 뒤따릅니다.
* 비한정적 참조는 주로 스트림 파이프라인에서의 매핑과 필터 함수에 쓰입니다(아이템 45).

```java
UnaryOperator<String> unaryOperator;

// lambda
unaryOperator = (str) -> str.toUpperCase();

// method reference
unaryOperator = String::toUpperCase;

// anonymous class
unaryOperator = new UnaryOperator<String>() {
    @Override
    public String apply(String str) {
        return str.toUpperCase();
    }
};
unaryOperator.apply("receiving Objects");
```
=> 인수는 한정되어 있지 않고, 수신 객체가 매개변수 목록의 첫번째로 추가되어 있음을 알 수 있습니다. 
여기서 "receiving Objects" 가 수신 객체임을 알 수 있습니다. 위의 경우 `"receiving Objects".toUpperCase()` 가 호출됨을 알 수 있습니다. 

### 메서드 참조로는 가능하지만, 람다로는 불가능한 것

* 제네릭 함수 타입은 아래와 같이 메서드 참조 타입으로는 표현 가능하지만, 람다로는 불가능 합니다.
애초에 제네릭 람다식이라는 문법이 존재하지 않기 때문입니다. 

```java
@FunctionalInterface
public interface G extends G2 {
    @Override
    <T> T m(T a);

    static void main(String[] args) {
        G g = Temp::m;
        g.m("method reference is available");

        G g2 = (t) -> t; // compile error: Target method is generic
    }
}

interface G2 {
    <T> T m(T a);
}

class Temp {
    static <T> T m(T t) {
        return t;
    }
}
```

## Swift

### 람다

* 위 자바 코드를 스위프트로 바꾸면 아래와 같습니다.

```swift
struct MultiSet {
    private var dictionary = [String: Int]()
    
    subscript(index: String) -> Int? {
        get {
            return dictionary[index]
        }
        
        set(newValue) {
            dictionary[index] = newValue
        }
    }
    
    mutating func merge(key: String, value: Int, remappingHandler: (Int, Int) -> (Int)) {
        let oldValue = self[key]
        let newValue = (oldValue == nil) ? value : remappingHandler(oldValue!, value)
        
        self[key] = newValue
    }
}

var multiSet = MultiSet()
        multiSet.merge(key: "some-cloth", value: 1) { (count, value) in count + value }
```

### 메서드 참조

* 위 자바 코드를 스위프트로 바꾸면 아래와 같습니다.

```swift
extension Int {
    static func sum(lhs: Int, rhs: Int) -> Int {
        return lhs + rhs
    }
}

multiSet.merge(key: "some-cloth", value: 1, remappingHandler: Int.sum(lhs:rhs:))
```

### 스위프트는 제네릭 클로저가 존재할까? 

```swift
let closure: <T> (T) -> T = { return $0 } // => 존재하지 않습니다. 컴파일 에러 발생.

func m<T>(t: T) -> T {
    return t
}
```
* 스위프트도 자바처럼 제네릭 클로저라는 문법은 존재하지 않습니다. Swift는 함수 선언이나 타입 선언은 generic 하게 사용할 수 있지만, 변수나 expression 은 generic하게 사용할 수 없습니다. 

=> 일부 언어에서는 `forall a. a -> a`와 같이 `universal quantifier`가 있는 타입을 가질 수 있습니다. 이런 경우 제네릭 클로저가 가능합니다. **그러나 Swift에서 타입은 `universal quantifier`를 가질 수 없습니다.** 따라서 expression과 values는 그 자체가 generic 일수 없습니다. 함수 선언 및 타입 선언은 제네릭일 수 있지만, 이러한 제네릭 함수 또는 이러한 제네릭 타입의 인스턴스를 사용하면 일부 타입이 타입 매개변수로 선택되므로, **그 이후에 우리가 얻는 값은 더 이상 generic 하지 않습니다.**  

> 핵심 정리 
<br> 메서드 참조는 람다의 간단명료한 대안이 될 수 있습니다. **메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용합시다.**

## 참고자료 

* [multiset vs multimap](https://stackoverflow.com/a/2858291/11612129)
* [functon type](https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.9)
* [difference-between-method-reference-bound-receiver-and-unbound-receiver](https://stackoverflow.com/questions/35914775/java-8-difference-between-method-reference-bound-receiver-and-unbound-receiver)
* [is-possible-generic-closure-in-swift](https://stackoverflow.com/questions/25545220/is-it-possible-to-create-a-generic-closure-in-swift)