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

Java의 애너테이션과 비슷하게 Swift에서 추가 정보를 나타내기 위한 방법으로는 Attributes가 있습니다. Attributes는 선언(declaration)시나 특정한 타입(type)에 대한 추가 정보를 제공합니다. `@` 심볼 다음에 지정하려는 Attributes의 이름과 이 Attributes가 적용될 인수를 작성하여 속성을 지정합니다.

>@`attribute name`
>
>@`attribute name`(`attribute arguments`)



주요한 몇가지 Attributes에 대해 알아봅니다.

### Declaration Attributes

#### available

Swift의 특정 버전이나 특정 플랫폼, 운영체제 버전과 관련된 선언의 수명주기를 표시하기 위해 사용됩니다. `available`은 항상 두 개 이상의 인수 목록을 받습니다. 그리고 해당 인수는 다음 플랫폼 또는 언어 이름 중 하나로 시작합니다.

> - iOS / iOSApplicationExtension
> - macOS / macOSApplicationExtension
> - macCatalyst / macCatalystApplicationExtension
> - watchOS / watchOSApplicationExtension
> - tvOS / tvOSApplicationExtension
> - swift

또한 별표(\*)를 사용해 모든 플랫폼 이름에서 선언이 가능함을 나타낼 수 있습니다. 하지만 swift의 버전 번호를 이용해 구체적인 가용성을 나타내기 위해서 별표(\*)는 사용되지 않습니다.

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

### 참고

- [Swift Language Guide - Attributes](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html)
- [Java Annotation](https://asfirstalways.tistory.com/309)
- [Attributes - deprecated](https://seorenn.tistory.com/85)