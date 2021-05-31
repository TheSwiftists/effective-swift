# Item 57. 지역변수의 범위를 최소화하라




### 요점
> 지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아집니다.
> -출처: 이펙티브 자바 

9장 일반적인 프로그래밍 원칙이므로 이 문서에서는 책에 소개된 Swift에서도 알아두면 좋은 원칙들을 정리하고 책에 소개된 Java와 Swift의 다른 점을 소개합니다. 

### 지역변수의 범위를 줄이는 2가지 기법

#### 1. 지역변수의 범위를 줄이는 가장 강력한 기법은 역시 '가장 처음 쓰일 때 선언하기'입니다.

다음과 같은 이유로 이유없이 미리 선언부터 하지 않기를 권장합니다. 

*  코드가 어수선해져 가독성이 떨어집니다
*  변수를 실제로 사용하는 시점엔 타입과 초깃값이 기억나지 않을 수 있습니다.
*  지역 변수를 생각 없이 선언하다보면 변수가 쓰이는 범위보다 너무 앞서 선언하거나 다 쓴 뒤에도 여전히 살아있게 되기 쉽습니다. 이런 경우 의도하지 않은 사이드 이펙트를 야기할 수 있습니다.

**< 거의 모든 지역변수는 선언과 동시에 초기화해야 합니다. >**

초기화에 필요한 정보가 충분하지 않다면 충분해질 때까지 선언을 미뤄야 합니다.

단, try- catch 문은 예외입니다.

변수 초기화하는 표현식에서 검사 예외를 던질 가능성이 있다면 try 블록 안에서 초기화해야 합니다.

**< 반복문은 독특한 방식으로 변수 범위를 최소화해줍니다. >**

> 반복 변수 (loop variable)의 범위가 반복문의 몸체, 그리고 for 키워드와 몸체 사이의 괄호 안으로 제한됩니다. 따라서 반복 변수의 값을 반복문이 종료된 뒤에도 써야하는 상황이 아니라면 while 문보다 for 문을 쓰는 편이 낫습니다. 
> -출처: 이펙티브 자바 
- 이펙티브 자바

반복문은 독특한 방식으로 변수 범위를 최소화해줍니다는 내용은 Swift와 Java 모두 공통적으로 해당하는 말이지만 '반복 변수의 값을 반복문이 종료된 뒤에도 써야하는 상황이 아니라면 while 문보다 for 문을 쓰는 편이 낫습니다.' 이 말이 Swift 에도 해당하는지 확인해봅시다.

```swift
// For-In Loops
// Array
let names = ["Anna", "Alex", "Brian", "Jack"]
for name in names {
    print("Hello, \(name)!")
}
// Hello, Anna!
// Hello, Alex!
// Hello, Brian!
// Hello, Jack!

// Dictionary
let numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
for (animalName, legCount) in numberOfLegs {
    print("\(animalName)s have \(legCount) legs")
}
// cats have 4 legs
// ants have 6 legs
// spiders have 8 legs

// with Numeric ranges
for index in 1...5 {
    print("\(index) times 5 is \(index * 5)")
}
// 1 times 5 is 5
// 2 times 5 is 10
// 3 times 5 is 15
// 4 times 5 is 20
// 5 times 5 is 25
```
(위의 예는 Swift Language Guide 5.3에 있는 예제입니다. )

그렇다면 이제 While 문을 살펴봅시다.

```java
// 책에 나온 예시
Iterator<Element> i = c.iterator();
while (i.hasNext()) {
  doSomething(i.next());
}
...
Iterator<Element> i2 = c2.iterator();
while (i.hasNext()) {
  doSomething(i2.next()); // runtime error
}
```
아래는 위 Java 코드를 Swift로 변환한 예제입니다.
```swift
let numbers = [2, 3, 5, 7]
var numbersIterator = numbers.makeIterator()

while let num = numbersIterator.next() {
    print(num)
}
// Prints "2"
// Prints "3"
// Prints "5"
// Prints "7"

while let num2 = numbersIterator.next() {
    print(num) // compile error
}
```

Swift에서는 while문에 필요한 변수의 범위를 지정할 수 있습니다. 자바와 달리 while 조건문에 필요한 변수를 while문 범위 밖에 선언하지 않아도 됩니다. 즉, Java와 달리 Swift에서는 while과 for 키워드 모두 몸체 사이의 괄호 안으로 제한할 수 있습니다. 따라서 책에 나온 'while 문보다 for 문을 쓰는 편이 낫습니다.'는 Swift에서는 해당하지 않습니다.

#### 2. 메서드를 작게 유지하고 한 가지 기능에 집중하는 것이다.

한 메서드에서 여러 가지 기능을 처리한다면 그 중 한 기능과만 관련된 지역변수라는 다른 기능을 수행하는 코드에서 접근할 수 있을 것입니다. 해결책은 단순한데, 단순히 메서드를 기능별로 쪼개면 됩니다.

### 참고
1. 이펙티브 자바 (Effective Java), Effective Java 3/E ,조슈아 블로크
2. [Control Flow - the swift programming language swift 5.4](https://docs.swift.org/swift-book/LanguageGuide/ControlFlow.html)
3. [next() - apple developer documentation](https://developer.apple.com/documentation/swift/iteratorprotocol/1641682-next)
