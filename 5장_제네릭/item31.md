# Item 31. 한정적 와일드카드를 사용해 API 유연성을 높이라

### 한정적 와일드카드 사용

```java
public class Stack<E> {
    // ...
    public void pushAll(Iterable<E> src) {
        for (E e : src)
            push(e);
    }
}
```

이렇게 작성할 경우, 매개변수화 타입은 불공변이기 때문에 `Stack<Number>`와 같이 선언할 경우 `Number`의 하위 타입인 `Integer`도 입력할 수 없다.

```java
public void pushAll(Iterable<? extends E> src) {
    // ...
}
```

위와 같이 와일드카드 연산자를 사용하여 Iterable 인터페이스를 구현한 E의 하위 타입도 입력할 수 있게 만들 수 있다.

### 스위프트

스위프트는 와일드카드 연산자가 없는데다가, 연관 타입을 가진 프로토콜은 Sequence<Int> 형태로도 쓸 수 없음

```swift
public protocol Sequence {
    associatedtype Element ...
    // ...
}

func f(sequence: Sequence<Int>) {} // 불가능
func g<S: Sequence>() -> S where S.Element == Int { ... } // 복잡
```

아래 형태는 복잡하며, 내부 구현인 Element 등을 외부로 노출하고 있다.

일반화하여 사용할 수 있도록 하려면 Type Erasing을 사용해야 한다.

### Type Erasure

조금 더 일반적인 타입을 만들기 위해 특정 타입을 지우는 것을 말함.

내부 구현 타입을 래핑하여 외부로부터 숨긴다.

구체적인 타입이 코드 베이스에 퍼지는 것을 막는 효과

코드는 draft...

```swift
// Sequence<Int>
class MAnySequence<Element>: Sequence {
    class Iterator: IteratorProtocol {
        func next() -> Element? {
            fatalError("Must override next()")
        }
    }
    func makeIterator() -> Iterator { // type-erased public API
        fatalError("Must override makeIterator()")
    }
    static func make<S: Sequence>(_ seq: S) -> MAnySequence<Element> where S.Element == Element {
        return MAnySequenceImpl<S>(seq)
    }
}

private class MAnySequenceImpl<S: Sequence>: MAnySequence<S.Element> {
    class IteratorImpl: Iterator {
        var wrapped: S.Iterator
        
        init(_ wrapped: S.Iterator) {
            self.wrapped = wrapped
        }
        override func next() -> S.Element? {
            return wrapped.next()
        }
    }
    
    var sequence: S
    
    init(_ sequence: S) {
        self.sequence = sequence
    }
    override func makeIterator() -> IteratorImpl {
        return IteratorImpl(sequence.makeIterator())
    }
}

MAnySequence.make([1, 2, 3, 4, 5]).forEach { print($0) }
MAnySequence.make([1 ..< 4]).forEach { print($0) }
```

연관 타입을 가진 프로토콜을 구체 타입으로 사용할 수 있도록 만들어준다.

스위프트의 AnyCollection, AnyHashable 등의 erased type을 제공하고 있다. AnySequence 또한 Sequence를 래핑하여 타입을 모른 채로 iterate를 가능하도록 해 준다.

### Type Erasure가 실제 사용되는 곳

```swift
public protocol ObserverType {
    associatedtype Element
    // ...
}

// A type-erased ObserverType.
public struct AnyObserver<Element> : ObserverType { 
    // ...
    public init<Observer: ObserverType>(_ observer: Observer) where Observer.Element == Element {
        self.observer = observer.on
    }
}
```

```swift
private func f1(_ arg: ObserverType<Int>) {} // Error
private func f2(_ arg: AnyObserver<Int>) {} // Legal
```

### 결론

자바에서 불공변인 것 때문에 하위 타입을 넣을 수 없는 문제점이 있으니, 한정적 와일드카드를 사용하여 하위 타입도 입력을 가능하게 만든다.

스위프트 제네릭은 아직 제약이 많아서 와일드카드를 사용하지 못할 뿐더러 제네릭 프로토콜의 경우 내부 구현 타입까지 명시해야 함. 일반적인 타입으로 사용하도록 만들기 위한 타입 이레이징 기법이 존재한다.

하지만 스위프트의 제네릭 또한 자바처럼 불공변이며, 와일드카드가 없기 때문에 하위 타입을 넣을 수 없다.
