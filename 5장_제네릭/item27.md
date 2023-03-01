# item27. 비검사 경고를 제거하라



### Java에서는

모든 비검사 경고는 런타임에 *ClassCastException*을 일으킬 수 있는 잠재적 가능성을 뜻하기 때문에 최대한 제거하라고 권고합니다. 혹시 경고를 없앨 방법을 찾지 못했다면 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 *@SuppressWarnings("unchecked")* 애너테이션(항상 가능한 한 좁은 범위에 적용)으로 경고를 숨긴 다음 경고를 숨기기로 한 근거를 주석으로 남기는 것이 좋습니다.

**Java에서 비검사 경고란**

여기서 _비검사 경고_는 프로그래머에게 캐스트(cast)가 다른 곳에서 예외(Exception)를 발생시킬 수 있음을 알려주는 컴파일러 경고이며,  @SuppressWarnings("unchecked")로 경고를 숨기는 것은 프로그래머가 컴파일러에게 예기치 않은 예외를 발생시키지 않는다고 알리는 의미입니다. 이 경고를 무시하고 실행할 경우 런타임에서 ClassCastException이 발생할 수 있습니다.

< 비검사 경고 예시 >

- 예시1: `warning: [unchecked]`

```java
Set<Lark> exaltation = new HashSet();

Venery.java:9: warning: [unchecked] unchecked conversion
        Set<Lark> exaltation = new HashSet();
                               ^
  required: Set<Lark>
  found:    HashSet
```

* 예시2: 이미지

![Unchecked cast warning](https://i.stack.imgur.com/zNKeg.png)

**Java에서 *ClassCastException*이란?**

ClassCastException은 Java에서 발생하는 런타임 에러로, 클래스를 한 타입에서 다른 타입으로 부적절하게 캐스트하려고 할 때 발생합니다.



그렇다면 Swift에서 어떻게 대응하고 있을까요?

### Swift의 타입캐스팅

Swift에는 타입캐스트 연산자(Type Cast Operator) `as?` 와 `as!`를 사용하여 부모 클래스를 자식클래스 타입으로 다운캐스팅할 수 있습니다. Swift에서 타입 캐스팅을 할 때 런타임 에러가 발생하는 대표적인 경우는 `as!` 연산자를 사용하여 다운캐스팅을 시도했을 때 입니다. `as!` 연산자의 경우 다운캐스팅이 무조건 성공할 것이라고 확신하는 경우에 사용하며 다운캐스팅이 성공할 경우 옵셔널이 아닌 인스턴스가 반환되고 **실패할 경우 런타임 오류가 발생**합니다.

예시와 함께 살펴보면 이렇습니다.

```swift
class Coffee {
  let name: String
  let shot: Int
  
  var description: String {
    return "\(shot) shot(s) \(name)"
  }
  
  init(shot: Int) {
    self.shot = shot
    self.name = "coffee"
  }
}

class Latte: Coffee {
  var flavor: String
  
  override var description: String {
    return "\(shot) shot(s) \(flavor) latte"
  }
  
  func addMilk() { }
  
    init(flavor: String, shot: Int) {
    self.flavor = flavor
    super.init(shot: shot)
  }
}

let coffee: Coffee = Coffee(shot: 1)
let myCoffee: Latte = Latte(flavor: "vanilla", shot: 3)

// Success
let newCoffee: Coffee = myCoffee as! Coffee
//경고 메세지: Forced cast from 'Latte' to 'Coffee' always succeeds; did you mean to use 'as'?

// 런타임 오류 발생. 강제 다운캐스팅 실패.
let newLatte: Latte = coffee as! Latte
```

이렇듯 다운캐스팅에 실패할 가능성이 있다면 조건부 연산자인 `as?`를 사용해야 합니다. 조건부 연산자 as?를 사용하면 다운캐스팅에 성공할 경우 옵셔널 타입으로 인스턴스를 반환하며, 실패할 경우 nil을 반환합니다.

```swift
guard let newLatte: Latte = coffee as? Latte else { return }
```



### Java vs Swift

Java는 제네릭을 사용할 때 경고를 통해 타입 불안정성을 알려주지만, Swift는 경고가 아닌 컴파일 오류를 통해 타입 불안전성을 더 확실히 알려줍니다. 덧붙여 Java의 비검사 경고는 잠재적으로 런타임 에러 가능성이 있는 코드를 경고하지만, 이와 달리 Swift의 타입 캐스팅 관련 경고들은 직접적으로 런타임 오류 가능성이 나는 부분은 아니지만 교정해야 하는 부분(ex.`as를 써도 되는 부분에 as? 를 쓴 경우`, `타입이 서로 상관이 없어 타입 캐스팅이 항상 실패하는 경우`)을 알려주는 역할을 합니다. Swift의 타입 캐스팅 관련 경고(warning)들은 런타임 오류 가능성를 내포하지 않습니다. 


### 참고

1. [Java ClassCastException - Oracle Docs](https://docs.oracle.com/javase/9/docs/api/java/lang/ClassCastException.html)
2. [Type Casting - The Swift Programming Language](https://docs.swift.org/swift-book/LanguageGuide/TypeCasting.html)
3. 야곰, 『스위프트 프로그래밍 3판』, 한빛미디어(2019)
4. [Effective Java Generics](https://www.informit.com/articles/article.aspx?p=2861454&seqNum=2)
5. [What is SuppressWarnings (“unchecked”) in Java?](https://stackoverflow.com/a/48366669)
