# Item 18. 상속보다는 컴포지션을 사용하라

상속은 잘못 사용하면 오류를 내기 쉽습니다. 상속을 이용하게 되면 하위 클래스는 상위 클래스의 내부 구현에 의존하게 되기 때문입니다. 상위 클래스의 내부 구현에 의존할 경우, 자신의 다른 부분을 사용하는 self-use 여부에 의해 잘못 동작할 수 있습니다. 또한 상위 클래스의 내부 구현은 릴리즈 시마다 얼마든지 달라질 수 있기 때문에, 하위 클래스의 코드를 변경하지 않아도 상위 클래스의 내부 구현이 바뀌면 하위 클래스가 오동작할 수 있습니다.

만약 하위 클래스의 데이터가 특정 조건을 만족해야 정상 동작하는 경우, 상위 클래스에 이 데이터를 변경 혹은 추가할 수 있는 새로운 메서드가 추가된다면 하위 클래스의 정상 동작을 보장할 수 없게 됩니다.

이런 오동작을 막기 위해 메서드 재정의를 피하고 새로운 메서드만 추가하는 방식으로 상속하더라도, 운 나쁘게 다음 릴리즈에서 상위 클래스에 같은 시그니처의 메서드가 생긴다면 재정의한 꼴이 되며 리턴 타입만 다르다면 컴파일 에러가 납니다.

### 컴포지션과 포워딩

컴포지션을 이용하면 앞서 언급한 문제들을 모두 피할 수 있습니다. 컴포지션은 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하도록 만드는 설계 방식입니다.

그리고 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환하게 합니다. 이 방식을 포워딩(forwarding)이라 합니다.

다음은 컴포지션과 포워딩 방식으로 구현한 코드입니다.

```java
// 상속 대신 컴포지션을 사용하는 Wrapper 클래스 
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    // ...
}

// 재사용이 가능한 포워딩 클래스
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public boolean add(E e) { return s.add(e); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }

    // ...
}
```

이처럼 구현할 경우, `addAll()`메서드의 자기 사용 여부에 상관 없이 `addCount`가 정상 동작합니다. 그 이유는 전달 클래스 덕분에 `ForwardingSet`의 `addAll()`에서 `add()`를 호출하지 않고 `Set`의 오버라이드되지 않은 원본 `addAll()`을 호출하기 때문입니다.

또한 새로운 클래스는 기존 클래스의 내부 구현 방식에 의존하지 않게 되며, 기존 클래스에 새로운 메서드가 추가 되더라도 전혀 영향을 받지 않습니다. 그리고 인터페이스를 활용해 설계되었기 때문에 유연합니다.

### 스위프트에서의 컴포지션 예시

오버라이드로 인해 문제가 발생하는 예시는 아니지만, 스위프트에서 컴포지션을 사용한 예시를 설명하겠습니다.

```swift
import RxSwift

/// PublishRelay is a wrapper for `PublishSubject`.
///
/// Unlike `PublishSubject` it can't terminate with error or completed.
public final class PublishRelay<Element>: ObservableType {
    private let _subject: PublishSubject<Element>
    
    // Accepts `event` and emits it to subscribers
    public func accept(_ event: Element) {
        self._subject.onNext(event)
    }
    
    // ...
}
```

UI 바인딩의 경우 Error나 Completed로 인해 스트림이 끊기지 않을 필요가 있습니다. Relay는 Subject의 인스턴스를 갖고 있으면서 onNext 이벤트만 전달해 스트림이 끊기지 않도록 하는 wrapper입니다.

만약 여기서 컴포지션이 아니라 상속과 오버라이드를 사용하였다면, 여러 문제가 발생합니다. 먼저 PublishSubject의 메서드들이 이미 public이기 때문에 클라이언트에서의 onError, onCompletion 호출을 완전히 막을 수는 없게 됩니다. 이 메서드들은 사용자를 혼란스럽게 할 수 있을 뿐더러, 상위 클래스의 메서드를 호출하여 하위 클래스인 PublishRelay의 정상 동작을 해칠 수도 있습니다.

또 다른 문제로는 만약 (그럴 리는 없겠지만) PublishSubject에 onBefore와 같은 메서드가 추가된다면, 클라이언트 측에서 해당 메서드를 사용해 의도되지 않은 이벤트를 전달할 수 있습니다.

### 스위프트에서의 기능 확장

추가적으로, 스위프트에서는 객체의 기능 확장을 목적으로 상속 뿐만 아니라 extension을 사용하기도 합니다. extension을 사용해서 기존 타입에 연산 프로퍼티, 메서드, nested 타입 등을 추가할 수 있습니다. 이번 아이템에서 소개한 문제점들을 바탕으로 기능 확장에 extension을 사용하는 경우엔 어떤 장단점이 있을지 확인해 보겠습니다.

```swift
extension Array {
    func take(_ number: Int) -> ArraySlice<Element> {
        return self[0..<Swift.max(0, number)]
    }
}

[1, 2, 3, 4].take(2) // [1, 2]
```

가장 큰 차이점으로는, 오버라이드를 사용하지 않기 때문에 이로 인한 오동작은 나타나지 않습니다. 내부적으로 자기 사용을 하더라도, 오버라이드된 메서드가 아닌 원본 메서드가 호출됩니다.

반면 상속과 마찬가지로, 운 나쁘게 다음 릴리즈에서 원본 객체에 같은 시그니처의 메서드가 생긴다면 extension의 수정이 필요합니다. 상속과 달리 재정의되는 건 아니지만 리턴 타입을 포함한 시그니처가 겹친다면 중복 정의로 인한 컴파일 에러가 발생합니다.

만약 배열의 데이터가 특정 조건을 만족하도록 만들고 싶다면, extension은 원본 메서드에 대한 호출을 막는 방법은 아니기 때문에 앞서 언급한 컴포지션 등을 활용해야 합니다.

### 마무리

상속을 이용해 객체의 기능을 확장하려 한다면, 이는 상위 클래스의 내부 구현에 영향을 받는 방식이기 때문에 캡슐화를 해친다는 문제가 있으며 주의하여 사용해야 합니다. 상속을 대체할 수 있는 컴포지션과 포워딩에 대해 살펴보고 스위프트에서 컴포지션을 사용한 예시에 대해서도 설명하였습니다.

마지막으로, 상속을 이용한 기능 확장과 스위프트에서 자주 사용하는 extension을 이용한 기능 확장 두 가지를 저자가 설명한 내용 관점으로 비교해 보았습니다.
