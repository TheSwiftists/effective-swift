# item39. 명명패턴보다 애너테이션을 사용하라

> Swift의 Attributes에 대해 알아봅시다



## Java

### 명명패턴

명명패턴이란 테스트 메서드인 경우 이름을 `testXXX`로 만드는 것 처럼 정해진 이름 규칙을 지키는 것을 말합니다. 간단하게 지킬 수 있는 반면에 오타가 나기 쉽고, 오타가 난 경우 JUnit 3이 이 메서드를 무시하기 때문에 제대로 테스트 할 수 없습니다. 또한 메서드에만 `test-` 접두어를 붙여야 하는데 클래스 앞에 붙여놓고 테스트가 올바로 통과하길 바라는 개발자가 있을 수 있는데, 이를 막을 방법 또한 없습니다. 하지만 애너테이션을 사용한다면 이 모든 문제가 해결됩니다.

### 애너테이션

프로그램의 소스코드 안에 추가 정보를 제공할 수 있는 도구로, 미리 약속된 형식으로 표현할 수 있습니다. 사전적 정의로는 주석이라는 뜻이며 애너테이션 이전에는 주석을 사용해 소스코드에 대한 설명을 적었습니다. 

현재는 주석과 역할이 다르지만 주석처럼 달아 클래스에 특수한 의미를 부여하거나, 기능을 주입할 수 있습니다. 크게 문서화, 컴파일러 체크, 코드 분석을 위한 용도로 사용되고 본질적인 목적은 소스 코드에 메타데이터를 표현하는 것입니다.



<br>

## Swift

Java의 테스트 메서드에 `test-` 접두사가 붙어야 했던 것처럼 iOS 앱 개발시 유닛테스트 메소드에도 `test-` 접두사가 붙어야만 테스트 메소드로 인지합니다. (그 외에도 테스트 메서드는 인자와 리턴 값이 없어야 한다는 조건이 있습니다.) 하지만 Swift에는 Java처럼 테스트에서의 명명패턴을 대체할 수 있는 기능이 없습니다.(혹여 제가 잘못 알고있는 것이라면 피드백 부탁합니다.) 

그대신 Java의 애너테이션과 비슷하게 Swift에서 추가 정보를 나타내기 위한 방법으로는 Attributes가 있습니다. Attributes는 선언(declaration)시나 특정한 타입(type)에 대한 추가 정보를 제공합니다. `@` 심볼 다음에 지정하려는 Attributes의 이름과 이 Attributes가 적용될 인수를 작성하여 속성을 지정합니다.

>@`attribute name`
>
>@`attribute name`(`attribute arguments`)



주요한 몇가지 Attributes에 대해 알아봅니다.

<br>

### Declaration Attributes

> 선언부에만 적용이 가능한 attributes

#### available

Swift의 특정 버전이나 특정 플랫폼, 운영체제 버전과 관련된 선언의 수명주기를 표시하기 위해 사용됩니다. `available`은 항상 두 개 이상의 인수 목록을 받습니다. 그리고 해당 인수는 다음 플랫폼 또는 언어 이름 중 하나로 시작합니다.

> - iOS / iOSApplicationExtension
> - macOS / macOSApplicationExtension
> - macCatalyst / macCatalystApplicationExtension
> - watchOS / watchOSApplicationExtension
> - tvOS / tvOSApplicationExtension
> - swift

또한 별표(\*)를 사용해 모든 플랫폼 이름에서 선언이 가능함을 나타낼 수 있습니다. 하지만 플랫폼 이름과 달리 Swift는 버전 번호를 이용해 구체적인 가용성을 나타내기 위해서 별표(\*)는 사용되지 않습니다.

`available` 속성과 함께 사용이 가능한 몇가지 필드에 대해 알아봅시다.

- `unavailable` : 이 선언이 지정된 플랫폼에서 사용할 수 없음을 나타냅니다. swift 버전 가용성을 지정할 때는 사용 불가합니다.

- `introduced`: 이 선언이 도입된 특정 플랫폼이나 언어의 첫 번째 버전임을 나타냅니다. 

  ```swift
  //introduced: version number
  @available(iOS, introduced: 12.0)
  ```

- `deprecated`: 이 선언이 도입되면 더 이상 사용되지 않는 버전임을 나타냅니다.

  ```swift
  @available(*, deprecated)
  class MyClass { ... }
  
  // message 필드를 추가해 빌드 경고 메시지 표현 가능
  @available(*, deprecated, message: "use newMethod instead")
  func oldMethod() { ... }
  
  // 이름이 바뀐 경우라면 별도로 renamed 필드를 이용해 빌드 시 표현 가능
  @available(*, deprecated, renamed: "newMethod")
  func oldMethod() { ... }
  
  // 특정 플랫폼, 특정 버전에 deprecated 됨을 표현
  @available(iOS, deprecated: 9.0)
  
  // obsolete 필드를 이용해 완전히 사라질 버전을 표현
  @available(iOS, deprecated: 9.0, obsolete: 10.0)
  ```



swift의 가용성을 나타내는 표현은 반드시 별개로 작성되어야 하고, 플랫폼에 관한 가용성 설정이 필요한 경우 `available` 속성을 아래와 같이 하나 더 지정합니다.

```swift
@available(swift 3.0.2)
@available(macOS 10.12, iOS 10.0, *)
class MyClass { ... }
```

<br>

#### dynamicCallable

`dynamicCallable` 속성을 이용하면 class, struct, enum, protocol을 여러 인수를 받는 함수처럼 동적으로 호출(dynamically callable)이 가능합니다. 다시 말해 객체지만 함수처럼 사용이 가능합니다.  `dynamicallyCall(withArguments:)`, `dynamicallyCallable(withKeywordArguments:)` 둘 중 하나 혹은 둘 다를 구현해야 합니다.

**dynamicallyCall(withArguments:)**

인자는 ExpressibleByArrayLiteral을 준수해야 합니다. 리턴타입은 모든 타입이 가능합니다.

```swift
@dynamicCallable
struct TelephoneExchange {
    func dynamicallyCall(withArguments phoneNumber: [Int]) {
        if phoneNumber == [4, 1, 1] {
            print("Get Swift help on forums.swift.org")
        }
    }
}

let dial = TelephoneExchange()
dial() // dial.dynamiccalyCall(withArguments: [])와 같은 의미
dial(4, 1, 1) // dial.dynamiccalyCall(withArguments: [4, 1, 1])와 같은 의미
dial(a: 8, 6, 7) // error. 'withKeywordArguments:' 메소드가 구현되지 않음 

// Call the underlying method directly.
dial.dynamicallyCall(withArguments: [4, 1, 1])
```



**dynamicallyCall(withKeywordArguments:)**

동적 메소드 호출에 레이블을 포함할 수 있습니다. 인자는 ExpressibleByDictionaryLiteral을 준수해야 합니다.
인자.Key는 ExpressibleByStringLiteral을 준수해야 합니다. 인자.Value는 모든 타입이 가능합니다.

레이블이 없는 인자는 빈 문자열("")을 레이블로 가지게 됩니다. 그런데 만약 인자가 Dictionary 타입인 경우, 키의 중복을 허용하지 않으므로 값이 유실될 수 있습니다. 따라서 인자 타입으로 추천되는 타입은 [KeyValuePairs](https://developer.apple.com/documentation/swift/keyvaluepairs)입니다.

```swift
@dynamicCallable
struct Repeater {
    func dynamicallyCall(
      withKeywordArguments pairs: KeyValuePairs<String, Int>
    ) -> Int {
        return pairs.count
    }
}

let repeater = Repeater()
repeater() //repeater.dynamicallyCall(withKeywordArguments: [:])와 같은 의미
repeater(1, 2) //repeater.dynamicallyCall(withKeywordArguments: ["": 1, "": 2])와 같은 의미
repeater(a: 1, b: 2) //repeater.dynamicallyCall(withKeywordArguments: ["a": 1, "b": 2])와 같은 의미

```



두 메서드를 모두 구현하는 경우는 메서드 호출에 레이블이 포함된 경우 `dynamicallyCall(withKeywordArguments:)` 이 호출되고 그 외의 경우에는 `dynamicallyCall(withArguments:)` 이 호출됩니다. 

<br>

#### dynamicMemberLookup

class, struct, enum, protocol에 적용하면 런타임시 member를 이름별로 조회할 수 있습니다. `subscript(dynamicMemberLookup:)` 메소드를 구현해야 합니다. 리턴 타입은 정해져있지 않고 인자는 `ExpressibleByStringLiteral`을 준수해야 합니다. 내부에 지정된 레이블 이름으로 외부에서 호출이 가능합니다.

```swift
@dynamicMemberLookup
struct DynamicStruct {
    let dictionary = ["someDynamicMember": 325,
                      "someOtherMember": 787]
    subscript(dynamicMember member: String) -> Int {
        return dictionary[member] ?? 1054
    }
}
let s = DynamicStruct()

// Use dynamic member lookup.
let dynamic = s.someDynamicMember
print(dynamic) // Prints "325"

// Call the underlying subscript directly.
let equivalent = s[dynamicMember: "someDynamicMember"]
print(dynamic == equivalent)
// Prints "true"

```

위의 코드를 보면 `someDynamicMember`라는 프로퍼티를 가지고 있지 않음에도 정상적으로 컴파일이 되고, `someDynamicMemeber`라는 이름으로 `DynamicStruct`의 `dictionary` 프로퍼티를 정상적으로 참조해오고 있습니다.

<br>

#### main

class, struct, enum의 정의에 이 속성을 명시하면 프로그램 흐름의 top-level 진입점임을 나타냅니다. `main` 이 표기된 타입에는 반드시 `Void`를 반환하는 `main` 이라는 이름의 메서드가 존재해야 합니다.

```swift
@main
struct MyTopLevel {
  static func main() {
    // Top-level code goes here
  }
}
```

혹은 `main` 속성을 정의한 타입은 다음의 가상 프로토콜을 준수하는 것과 같은 요구사항을 준수해야만 합니다.

```swift
protocol ProvidesMain {
  static func main() throws
}
```

실행 파일을 만들기 위해 컴파일 한 swift 코드는 [Top-Level Code](https://docs.swift.org/swift-book/ReferenceManual/Declarations.html#ID352) 에서 논의 된 것처럼 최대 하나의 top-level 진입점을 포함할 수 있습니다.

<br>

### testable

테스트가 가능한 컴파일된 모듈의 `import` 부분에 이 속성을 추가하면 해당 범위에서 해당 모듈을 확장된 액세스로 활성화합니다. 접근제한자가 `internal`이나 `public`으로 설정된 클래스와 클래스의 멤버들을 `open`으로 사용이 가능하고, `internal`로 설정된 다른 entity들(struct, enum..)은 `public`처럼 사용이 가능해집니다.

<br>

## Type Attributes

> 타입에만 적용이 가능한 attributes

### escaping

메서드 또는 함수 선언시 매개변수의 타입에 적용이 가능하며, 지금 당장이 아닌 나중에 실행하기 위해 매개변수 값을 저장할 수 있음을 나타냅니다. 메서드 호출 수명보다 값의 수명이 더 오래 지속될 수 있음을 의미합니다. 이 속성이 선언된 매개변수의 함수는 프로퍼티나 메서드를 사용하기 위해 `self`의 명시적인 사용이 요구됩니다. 자세한 내용은 [Escaping Closures](https://docs.swift.org/swift-book/LanguageGuide/Closures.html#ID546)를 참고해주세요.

<br>

## Attribute related to Library Evolution Mode

먼저 [Library Evolution Mode](https://swift.org/blog/abi-stability-and-more/)가 무엇인지 설명드립니다. 

* Library Evolution Mode가 아닌 라이브러리
  * 라이브러리의 버전이 변할 때, 해당 라이브러리를 사용하는 모든 앱은 전부 re-compile 되어야 합니다. 이것은 몇 가지 장점이 있습니다: (re-compile 함으로써) 컴파일러는 앱이 사용하는 라이브러리가 정확히 어떤 버전인지 알게 되기 때문에, code size를 줄이거나 앱이 더 빠르게 동작하게 하는 이점을 취할 수 있습니다. 즉 컴파일러가 최적화됩니다.

  * 그러나 만약 프로젝트의 한 entity가 다른 프레임워크에서 선언되고 해당 프레임워크가 이전 버전의 라이브러리를 사용하는 경우라면, 이 entity는 새로운 버전의 라이브러리에 맞게 re-compile 되지 않습니다(그래서 앱이 오작동 할 수 있습니다). 따라서 이러한 Version 취약성 때문에 해당 entity를 사용하는 코드는 라이브러리 버전 업데이트에 보수적일수 밖에 없습니다. 

* Library Evolution Mode인 라이브러리
  * 라이브러리가 새로운 버전이 제공되어도, 라이브러리리를 사용하는 client(앱)들은 re-compile 하지 않아도 됩니다. 즉 클라이언트는 기존 버전의 라이브러리를 사용 가능하면서도 다음 버전의 라이브러리도 사용 가능합니다. 따라서 컴파일러가 덜 최적화되지만, 라이브러리를 버전업해도 위와 같은 Version 취약성으로 인한 문제는 해결됩니다.
  * Swift Standard Library는 Library Evolution Mode 인 라이브러리입니다.

### frozen

> 라이브러리가 Library Evolution Mode여도 Enum이나 Struct를 최적화하기 위한 attributes 

frozen는 라이브러리가 Library Evolution Mode인 경우에만 사용이 허용됩니다. frozen을 struct 또는 enumeration 선언에 적용하면 해당 타입에 대해 수행할 수 있는 변경의 종류를 제한할 수 있습니다. 따라서 다음 버전의 라이브러리에서는 `@frozen enum` 타입의 **여러 case** 와 `@frozen struct` 타입의 **stored property들을** 재정렬(reordering)하거나 삭제 및 추가할 수 없습니다. 이러한 변화들은 nonfrozen 인 타입에서는 허용되지만, frozen type에 대한 ABI 호환성을 깨뜨립니다.

> NOTE
<br> 컴파일러가 Library Evolution Mode에 있지 않으면, 모든 struct와 enum이 implicit하게 frozen으로 적용되므로, 이 attribute는 무시됩니다.

Library Evolution Mode에서 nonfrozen struct 및 enum의 멤버와 상호 작용하는 코드는 향후 버전의 라이브러리에서 해당 유형의 멤버 중 일부를 추가, 제거 또는 재정렬하더라도 (위의 설명처럼) 다시 컴파일하지 않고 계속 작업할 수있는 방식으로 컴파일됩니다. 컴파일러는 런타임시 정보 검색 및 간접 계층 추가와 같은 기술을 사용하여 이를 가능하게 합니다(따라서 컴파일러는 덜 최적화됩니다).

frozen을 쓴다는 것은 nonfrozen의 유연성(flexibility)을 포기하고 대신 성능 최적화를 택한다는 것입니다. 이후 버전의 라이브러리는 type을 제한적으로만 변경할 수 있지만, 컴파일러는 type의 멤버(enum's case or struct's stored property)와 상호 작용하는 코드에서 추가 최적화를 수행할 수 있습니다.

frozen type(enum or struct)과 frozen struct의 저장 프로퍼티 type, frozen enum의 associated value 들은 모두 public 이거나 @usableFromInline 로 표시되어야 합니다(이는 애초에 라이브러리 코드를 외부에서 사용하는 경우에 frozen이 필요한거라 필요한 조건입니다).

요약하자면, 
  * Library Evolution Mode가 아닌 라이브러리의 모든 enum 과 struct 는 default로 frozen 타입입니다. 타입이 모두 frozen인 이유는 버전업 될 때마다 client 코드를 re-compile 해야해서 다른 버전(이전 버전 or 다음 버전)을 고려할 필요가 없기 때문입니다. 따라서 해당 라이브러리를 사용하면 Evolution Mode 라이브러리를 사용할 때보다 client 코드가 성능 최적화될 수 있습니다.
  * Library Evoltion Mode인 라이브러리는 enum 과 struct 가 default로 nonfrozen 타입입니다. default로 nonfrozen인 이유는 client 코드가 기존 버전과 다음 버전의 라이브러리 모두 사용할 수 있게 제공해야 하기 때문입니다. 따라서 nonfrozen 타입을 사용하는 Evolution Mode를 client가 사용하면 성능 최적화보다 여러 버전의 라이브러리를 re-compile 없이 사용할 수 있다는 유연성을 가질 수 있습니다. 대신 Library Evoltion Mode인 라이브러리도 **변하지 않는 것이 확정된 특정 enum과 struct**에 frozen을 마크해서 부분 최적화를 얻을 수 있습니다.
  * Swift Standard Library로 예시를 들면 `Optional`은 frozen enum이고 `DecodingError`는 nonfrozen enum입니다. 

### unknown

> nonfrozen enum을 사용하는 경우 사용되는 attributes 

기존 enum이나 frozen enum의 경우, 모든 case를 열거하면 default 구문을 적을 필요가 없습니다.
하지만 nonfrozen enum 을 사용하는 경우 모든 case를 나열해도 미래 버전의 enum case를 고려해야 하기 때문에 `@unknown default` 구문을 적어 대응해야 합니다. 그렇지 않으면 `Switch covers known cases, but 'TestEnum' may have additional unknown values` 라는 yellow warning이 발생합니다. 

switch case에 이 속성을 적용하면 이 case는 컴파일될 시점의 열거형의 어떤 case와도 매치되지 않을 것임을 나타냅니다. 자세한 사용 방법은 [Swtiching Over Future Enumeration Cases](https://docs.swift.org/swift-book/ReferenceManual/Statements.html#ID602)를 참고해주세요.

```swift
// FrozenEnum Type is OtherLibrary's frozen type
func foo(e: OtherLibrary.FrozenEnum) {
    switch e {
        case .first:
            break
        case .two:
            break
        }
}

// NonFrozenEnum Type is OtherLibrary's nonfrozen type
func goo(e: OtherLibrary.NonFrozenEnum) {
    switch e {
        case .first:
            break
        case .two:
            break
        @unknown default:
            break
        }
}
```

<br>

### 참고

- [Swift Language Guide - Attributes](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html)
- [Java Annotation](https://asfirstalways.tistory.com/309)
- [Attributes - deprecated](https://seorenn.tistory.com/85)
- [Attributes - dynamicCallable](https://jcsoohwancho.github.io/2020-06-20-dynamicMemberLookup,-dynamicCallable/)
