# item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

## 목차
- 상속을 위한 문서화
    - 부작용
    - Java
    - Swift
- 상속을 위한 설계
    - 어떤 메서드를 재정의 할 수 있게 해야하나?
    - 제약사항
- 상속용 클래스가 아닌 일반적인 클래스에 대한 경우
- 참고한 곳

<br>

### 상속을 위한 문서화
클래스를 안전하게 상속할 수 있도록 문서화를 해놓는 것을 권장하고 있습니다. 특히, 재정의 가능 메서드를 호출할 수 있는 모든 상황을 문서로 남겨야합니다.
- 문서화를 할때 고려해야 할 부분들 입니다.
    1. 상속용 클래스(상위 클래스)는 재정의 할 수 있게 만든 메서드들을 내부적으로 어떻게 사용하는지 문서로 남겨야 합니다.
    2. 공개(public, open) 메서드에서 재정의 가능한 메서드를 호출 할 때, API에 명시하여야 합니다.
    3. 어떤 순서로 호출하는지, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 합니다.
    
#### 부작용
문서화할 때 '어떻게' 동작하는지도 설명해야 하는 것이 부작용입니다. 좋은 API 문서는 API가 '어떻게' 동작하는지 아닌 '무엇'을 하는지만을 설명해야 하지만, 상속으로 인해 캡슐화를 해치게 되어 내부 구현 방식을 설명해야하기 때문입니다.

<br>

#### Java
자바의 경우 API 문서에서는 Implementation Requirements로 시작하는 절이 있는데, 해당 절은 @implSpec 태그를 붙여주면 자바독 도구가 생성해줍니다.
> Javadoc : Java 소스를 문서화 하는 방법, html로 열 수 있습니다.

#### Swift
스위프트의 경우 JavaDoc처럼 따로 문서를 만들 수는 없지만, Xcode에서 다양한 주석 및 마크다운을 이용해 메서드 상단에 명시함으로써 호출하는 곳에서 해당 메서드의 설명을 팝업으로 볼 수 있게 되어있습니다.

<br>

### 상속을 위한 설계
#### 어떤 메서드를 재정의 할 수 있게 해야하나?
1. 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 재정의 가능한 메서드를 제공해야 할 수도 있습니다.
> hook(hooking) : 함수 호출, 중간에서 가로챈다고 표현한다.

<br>

예제에서는 Java의 `removeRange` 메서드를 예로 들고있습니다.
해당 메서드를 protected로 제공한 이유는 `clear` 메서드가 아래에 나와있는 `removeRange` 메서드를 부르기 때문입니다.
        
```Java
protected void removeRange(int fromIndex, int toIndex) {
    ListIterator<E> it = listIterator(fromIndex);
    for (int i=0, n=toIndex-fromIndex; i<n; i++) {
        it.next();
        it.remove();
    }
}
```
`removeRange`메서드 내부의 `it.remove()`가 O(n)이 걸리게 되면 `removeRange`는 O(n^2)이 걸리게 됩니다.
따라서 해당 메서드를 호출하는 `clear` 메서드를 고성능으로 만들기 쉽게 하기 위해 `removeRange` 메서드를 재정의 할수있도록 제공하는 것입니다. 
       
2. 상속용 클래스를 설계할때 어떤 메서드를 재정의하도록 노출해야 할지는 직접 하위 클래스를 만들어보면서 테스트(검증) 해보는것이 '유일'한 방법입니다.
    - 테스트를 지속해보면서 private하게 만들어야하는 메서드와 재정의 가능하게 해야하는 메서드를 나눌 수 있게 됩니다.

#### 제약사항
상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안됩니다.

```Swift
class Super {
    init() {
        overrideMe()
    }
            
    public func overrideMe() { }
}
        
class Sub: Super {
    private var subProperty: String!
            
    init(subProperty: String?) {
        self.subProperty = subProperty
    }
            
    override func overrideMe() {
        print(subProperty.description)
    }
}

let sub = Sub(subProperty: nil) // 런타임 에러 발생! Thread 1: Fatal error: Unexpectedly found nil while implicitly unwrapping an Optional value
sub.overrideMe()
```

- 요지는 Java에서는 상위 클래스의 생성자가 먼저 호출되어 하위 클래스의 overrideMe 메서드를 부르게 되는데 이때 subProperty가 초기화되어있지 않은 상태에서 부르게 되므로 `NullPointerException`을 던지게 된다는 것입니다.
- Swift에서는 상위 클래스 생성자를 호출하기 위해서는 하위 클래스의 stored property에 모든 값이 초기화 된 후 호출할 수 있기 때문에 예시와 같은 오류가 발생하기 어렵지만, stored property가 optional일 경우에는 위와 같은 상황이 발생할 여지가 있습니다.
- Swift에서 위와 같은 초기화 흐름이 발생하는 이유는 초기화를 2단계에 나누어서 하며, 2단계 초기화시 컴파일러에서 4가지의 안전 검사를 수행하기 때문입니다. <br> Swift가 이러한 초기화 방식을 둔 이유 중 하나는 초기화되기 전에 속성 값에 접근하는 것을 막아 주고, 예상치 못하게 속성 값이 또 다른 생성자에 의해 엉뚱한 값으로 설정되어 버리는 것도 막아주기 위함입니다. <br>
양이 방대하여 해당 문서에 따로 정리하지 않으나, Swift 공식문서 중 **Initialization**항목을 참고해보시면 도움이 될 것 같습니다.
- 다만 개인적인 생각으로는 재정의 가능한 함수를 생성자에서 호출시킨다는 것은 예측가능하기 어려운 흐름이 발생할 수 도 있기 때문에 지양하는 방향이 올바르다고 생각합니다.

<br>

### 상속용 클래스가 아닌 일반적인 클래스에 대한 경우
클래스가 final로 선언되지도 않았고, 상속용으로 설계되거나 문서화도 해놓지 않았을 경우입니다.
이러한 클래스는 수정이 생길때마다 하위 클래스가 오동작 할 수 있는 가능성이 있기 때문에 해결할 수 있는 몇가지 방법들이 있습니다.

1. final 클래스로 만들어 상속을 금지 시킨다.
2. 모든 생성자를 private으로 선언하고 정적팩터리 메서드를 제공합니다. (아이템 1, 17 참조)

#### 그래도 상속을 해야하는 경우
1. 내부에서 재정의 가능 메서드를 호출하지 않게 만들고 이를 문서화 합니다.
2. 재정의 가능 메서드의 내부 로직을 private한 '도우미 메서드'로 옮기고, 이 '도우미 메서드'를 호출하도록 수정합니다.
이렇게 하면 재정의 가능 메서드를 호출하는 다른 코드들은 '도우미 메서드'만을 호출하도록 합니다.

<br>

### 참고한 곳
- https://docs.swift.org/swift-book/LanguageGuide/Initialization.html#ID216
- http://xho95.github.io/xcode/swift/grammar/initialization/2016/01/23/Initialization.html
