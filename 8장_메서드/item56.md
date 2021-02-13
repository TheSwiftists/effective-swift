# item56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

> Markup Formatting References in Xcode

로버트 C. 마틴은 그의 저서 *클린 코드* 에서 `주석은 나쁜 코드를 보완하지 못한다` 고 했습니다. 허나 어떤 주석들은 개발자의 이해도와 사용성을 높이기에 아주 유용합니다. 이번 아이템에서는 Xcode 내에서 Quick Help 기능을 제공할 수 있는 주석을 작성하는 방법에 대해 알아봅니다.



### Xcode Quick Help란?

Markup을 이용해 Swift 코드의 모든 심볼(클래스, 메서드, enum..)에 대해 정형화된 Quick Help 주석을 작성하면 Option키를 클릭했을 때 작성된 주석을 Pop-up 형식으로 확인할 수 있습니다. 또한 입력 포인터가 심볼에 위치해 있을 때 Xcode 오른쪽에 있는 인스펙터 탭에 Quick Help 부분에서 작성된 주석을 확인할 수 있습니다.

<img width=80% alt="MFR_code_quick_help_2x" src="https://user-images.githubusercontent.com/40784518/107853021-1c3e7f00-6e57-11eb-9711-3e7c2152d1e5.png">

<br>

### 기본 Markup

기본적인 Markup 문법을 이용해 작성할 수 있습니다. 각각에 대해 조금 더 자세히 살펴봅시다.

- 문단은 빈 줄로 구분됩니다.
- 순서가 없는 목록은 불렛 문자(`-`, `+`, `*`)로 작성합니다.
- 순서가 있는 목록은 `1.`이나 `1)` 처럼 작성합니다.
- `#` 로 헤더를 나타냅니다. 혹은 바로 다음 줄에 `=`나 `-`를 표시해 헤더를 나타낼 수 있습니다.
- 링크와 이미지 또한 모두 작동하며, 웹 기반 이미지가 Xcode에 직접 표시됩니다.
- `----`, `***` 를 이용해 긴 수평선을 표시할 수 있습니다.
- 슬래시 3개를 사용하여 한줄짜리 Formatting Reference를 작성할 수 있습니다

```swift
/**
    # 목록

		*이탤릭체*와 **볼드체** 혹은 `코드`를 한 줄에 적용할 수 있습니다.

    ## 순서가 없는 목록

    - 목록을 작성할 수 있지만
    - 각각의 문단은 연관되어있지 않습니다.
    - 그래서 서브목록을 포맷팅하기엔
      - 썩 좋은 방법은 아닙니다.

    ## 순서가 있는 목록

    1. 순서가 있는 목록은
    2. 자동으로 순서대로 정렬됩니다.
    3. 일반적인 숫자로 표기 가능합니다.
*/
```

<img width = 60% src ="https://user-images.githubusercontent.com/40784518/107854046-b2c16f00-6e5c-11eb-8d21-3283784da439.png"/>

<br>

### Parameters & Return Values

- Parameters: `Parameter <param name>:` 으로 시작하며 매개변수에 대한 설명을 작성합니다.
- Return values: `Returns:` 로 시작하며 리턴 값에 대한 정보를 작성합니다.
- Thrown errors: `Throws:` 로 시작하며 발생할 수 있는 에러에 대한 설명을 작성합니다. Swift는 에러 타입이 `Error`인지만을 확인하므로 오류를 적절하게 문서화하는 것이 중요합니다.

```swift
/**
 받는 사람을 위한 맞춤 인사말을 생성

 - Parameter recipient: 환영 받을 사람

 - Throws: 만약 `recipient`이 "Derek"이면 
					`MyError.invalidRecipient` 발생

 - Returns: `recipient`를 향한 새로운 문자열 반환
 */
func greeting(to recipient: String) throws -> String {
    guard recipient != "Derek" else {
        throw MyError.invalidRecipient
    }

    return "Greetings, \(recipient)!"
}
```



매개변수가 많은 경우 아래처럼 줄바꿈해 작성할 수 있습니다.

```swift
/// 주어진 요소로부터 벡터의 크기를 3차원으로 반환
///
/// - Parameters:
///     - x: vector의 *x*값
///     - y: vector의 *y*값
///     - z: vector의 *z*값
func magnitude3D(x: Double, y: Double, z: Double) -> Double {
    return sqrt(pow(x, 2) + pow(y, 2) + pow(z, 2))
}
```



<br>

### Code blocks

코드 블럭을 임베드해 함수의 적절한 사용법이나 구현 세부사항을 보여줄 수 있습니다. 최소한 4개의 공백으로 코드 블럭을 삽입할 수 있습니다.  또는 세개의 백틱(```` `) 또는 틸드(`~~~`)로 코드 블럭 앞 뒤를 묶어 구분할 수 있습니다.

```swift
/**
    `Shape`의 인스턴스 영역

    생성된 인스턴스 모양에 따라 계산이 달라진다.
		삼각형의 경우 `area`는 다음과 같다

        let height = triangle.calculateHeight()
        let area = triangle.base * height / 2
*/
var area: CGFloat { get }
```



<img width=60% src="https://user-images.githubusercontent.com/40784518/107854398-d5ed1e00-6e5e-11eb-9380-c11f9646b926.png"/>



<br>

### 추가적인 필드들

`Parameters`, `Throws`, `Returns`외에도 미리 정의된 키워드들이 있는데, 이를 이용하면 섹션이 생성돼 추가적인 정보를 작성할 수 있습니다. 

- 알고리즘/안정성 정보: `Precondition`, `Postcondition`, `Requires`, `Invariant`, `Complexity`, `Important`, `Warning`
- 메타데이터: `Author`, `Authors`, `Copyright`, `Date`, `SeeAlso`, `Since`, `Version`
- 일반적 주석 & 권고사항: `Attention`, `Bug`, `Experiment`, `Note`, `Remark`, `ToDo`



<br>

### 작성시 유의사항

- 메서드의 주석은 `어떻게 동작하는지`가 아니라 `무엇을 하는지`에 초점을 맞춰서 작성합니다.
- 상속 할 가능성이 있는 클래스에는 되도록 상세하게 작성해 상속 받은 쪽에서 해당 메서드를 올바르게 재정의(override)할 수 있도록 합니다.
- 제네릭 타입이나 제네릭 메서드인 경우 모든 타입 매개변수에 주석을 작성해야 합니다.
- 열거형을 문서화 할 때는 모든 상수들에도 주석을 달아야 합니다. 설명이 짧다면 주석 전체를 한 문장으로 써도 좋습니다.
- 여러 클래스가 상호작용하는 복잡한 구조라면 전체 아키텍처를 설명하는 별도의 설명이 필요할 때가 있습니다. 관련 설명 문서가 있다면 관련 클래스의 주석에 `SeeAlso`등의 필드로 문서 링크를 달 수도 있습니다.



<br>

### 참고

- https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_markup_formatting_ref/SymbolDocumentation.html
- https://nshipster.com/swift-documentation/

