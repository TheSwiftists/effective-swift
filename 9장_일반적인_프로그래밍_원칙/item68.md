# Item 68. 일반적으로 통용되는 명명 규칙을 따르라



>  철자 규칙은 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룹니다. 이 규칙들은 특별한 이유가 없는 한 반드시 따라야 합니다. 이 규칙을 어긴 API는 사용하기 어렵고, 유지보수하기 어렵습니다. 철자 규칙이나 문법 규칙을 어기면 다른 프로그래머들이 코드를 읽기 번거로울 뿐 아니라 다른 뜻으로 오해할 수도 있고 그로 인해 오류까지 발생할 수 있습니다.
> -출처: 이펙티브 자바 




   ## API Design Guidelines Summary

|       Category       |       Subcategory       |                          Guidelines                          |
| :------------------: | :---------------------: | :----------------------------------------------------------: |
|     Fundamentals     |                         | ● 사용 시점에서의 명확성(Clarity)이 가장 중요한 목표(goal)입니다. |
|                      |                         |     ● 명료성(Clarity)은 간결성(brevity)보다 중요합니다.      |
|                      |                         |     ● 모든 선언에 대한 문서 주석(코멘트)을 작성하십시오.     |
|        Naming        |   Promote Clear Usage   | ● 이름이 사용되는 코드를 읽을 때, 모호함을 방지하는 데 필요한 모든 단어를 포함합니다. |
|                      |                         | ● 불필요한 단어는 생략하십시오. 이름에 나오는 모든 단어는 사용 현장에서 중요한 정보를 전달해야합니다. |
|                      |                         | ● 타입 제약 조건 보다는 역할에 따라, 변수, 매개 변수 및 관련타입(associated types)을 지정하십시오. |
|                      |                         | ● 취약한 타입 정보를 보완하여, 파라미터 역할을 명확히 합니다. |
|                      | Strive for Fluent Usage |                                                              |
|                      |                         | ● 사용 현장을 문법적인 영어 구(phrase) 형식으로 만드는 방법 및 함수 이름을 선호하십시오. |
|                      |                         |   ● "make"를 사용하여 팩토리 메소드의 이름을 시작하십시오.   |
|                      |                         |      ● 부작용에 따라 함수 및 메소드 이름을 지으십시오.       |
|                      |                         | ● Bool메소드 및 프로퍼티의 사용은 그 사용이 nonmutating일 때 수신자에 대한 주장(assertions)으로 해석해야합니다. |
|                      |                         |   ● "무엇"인지를 설명하는 프로토콜은 명사로 읽어야 합니다.   |
|                      |                         | ● 기능을 설명하는 프로토콜은 "`able`,`ible`,`ing`" 접미사를 사용하여 명명해야 합니다. |
|                      |                         | ● 다른 타입, 프로퍼티, 변수 및 상수의 이름은 명사로 읽어야 합니다. |
|                      |  Use Terminology Well   | ● 보다 일반적인 단어가 의미를 전달하는 경우, 모호한 용어는 사용하지 마십시오. |
|                      |                         | ● Term of Art를 사용한다면, 확립된 의미에 충실하십시오. (정해진 의미를 고수하십시오.) |
|                      |                         |                     ● 약어를 피하십시오.                     |
|                      |                         | ● 선례를 받아들이십시오.기존 문화를 따른다는 대가를 치르더라도, 초보자를 위한 용어를 최적화하지 마십시오. |
|     Conventions      |   General Conventions   |    ● O(1)이 아닌 연산프로퍼티의 복잡성을 문서화하십시오.     |
|                      |                         |      ● free 함수보다 메소드와 프로퍼티를 선호하십시오.       |
|                      |                         | ● case Convention을 따르십시오. 타입과 프로토콜의 이름은 UpperCamelCase입니다. 나머지는 모두 lowerCamelCase입니다. |
|                      |                         | ● 메소드는 동일한 기본 의미를 공유하거나, 고유한 도메인에서 작동 할 때, 이름을 공유 할 수 있습니다. |
|                      |       Parameters        | ● 문서를 제공 할 때, 파라미터 이름을 선택하십시오. 파라미터 이름은, 함수 또는 메소드의 사용 시점에 나타나지 않지만 중요한 설명 역할을 합니다. |
|                      |                         | ● 일반적인 사용을 단순화 할때, 기본 매개변수를 활용하십시오. 공통적으로 사용되는 값이 하나인 매개변수는 기본값의 후보입니다. |
|                      |                         | ● 매개변수(Parameter)목록의 끝 부분에, 기본값이 있는 매개변수를 찾으십시오. 기본값이 없는 매개변수는 일반적으로 메소드의 의미에 더 중요하며, 메소드를 호출 할 때, 안정적인 초기 사용 패턴을 제공합니다. |
|                      |     Argument Labels     | ● 전달인자를 유용하게 구분 할 수 없는 경우, 모든 레이블을 생략하십시오. |
|                      |                         | ● 값 보존 형 변환을 수행하는 이니셜라이저에서 첫번째 인수 레이블을 생략하십시오. |
|                      |                         | ● 첫번째 인수가 전치사 구의 일부인 경우, 전달인자 레이블을 지정하십시오. 전달인자 레이블은 일반적으로 전치사에서 시작해야 합니다. |
|                      |                         | ● 그렇지 않고, 첫번째 전달인자가 문법적 구문의 일부를 구성하는경우 해당 레이블을 생략하고, 앞의 단어를 기본이름에 추가합니다. |
|                      |                         |         ● 다른 모든 전달인자에 레이블을 붙히십시오.          |
| Special Instructions |                         | ● tuple멤버에게 레이블을 지정하고, API에 표시되는 클로져 매개변수 이름을 지정합니다. |
|                      |                         | ● overload Set에서 모호함을 방지하기 위해, 제약없는 다형성(polymorphism)에 각별히 주의하십시오. |



   ## 📑 Fundamentals

   ● **사용 시점에서의 명확성(Clarity)이 가장 중요한 목표(goal)입니다.**
   (**Clarity at the point of use** is your most important goal.)

   ● **명료성(Clarity)은 간결성(brevity)보다 중요합니다.**
   (Clarity is more important than brevity. )

   Swift코드에서 발견되는 간결함은, 강력한 타입 시스템의 부작용(side-effect)이며 자연스럽게 boilerplate(보일러 플레이트)를 줄이는 기능입니다.

   ( ※ boilerplate(보일러 플레이트) : 최소한의 변경으로 재사용 할 수 있는 것 / 적은 수정만으로 여러 곳에 활용가능한 코드, 문구)

   ● **모든 선언에 대한 문서 주석(코멘트)을 작성하십시오.**
   (Write a documentation comment for every declaration.)



   ## 📑 Naming



   ### 🔖 Promote Clear Usage

   ● **이름이 사용되는 코드를 읽을 때, 모호함을 방지하는 데 필요한 모든 단어를 포함합니다.**
   (Include all the words needed to avoid ambiguity for a person reading code where the name is used.)

   [Example - Collection내의 지정 된 위치에서 요소를 제거하는 메소드]
   ``` swift
   // 👍🏻
   extension List {
     public mutating func remove(at position: Index) -> Element
   }
   employees.remove(at: x) // x를 사용하여 제거할 요소의 위치를 나타내는 것이 아닌 메소드가 x와 동일한 요소를 검색하여 제거한다는 것을 암시 할 수 있음 
   ```

   ``` swift
   // 👎🏻
   employees.remove(x) // unclear: are we removing x? 모호
   ```



   ● **불필요한 단어는 생략하십시오.** 이름에 나오는 모든 단어는 사용 현장에서 중요한 정보를 전달해야합니다. 
   (Omit needless words. Every word in a name should convey salient information at the use site.)

   의도를 명확하게 하거나, 의미를 명확하게 하기 위해, 더 많은 단어가 필요할 수 있지만, 독자가 이미 소유하고 있는 정보와 중복되는 단어는 생략해야합니다. 특히 타입 정보만 반복하는 단어는 생략해야 합니다.

   ``` swift
   // 👎🏻
   public mutating func removeElement(_ member: Element) -> Element?
   
   allViews.removeElement(cancelButton)
   ```
   Element라는 단어는 호출부분에서 핵심적인, 이 메서드에서 가장 중요한 건 remove하는 행위일 겁니다. 따라서 아래 코드가 더 낫다고 문서에서는 소개하고 있습니다.

   ``` swift
   // 👍🏻
   public mutating func remove(_ member: Element) -> Element?
   
   allViews.remove(cancelButton) // clearer
   
   ```
   Occasionally, repeating type information is necessary to avoid ambiguity, but in general it is better to use a word that describes a parameter’s role rather than its type. See the next item for details.

   간혹 `모호성`(ambiguity)을 피하기 위해 타입 정보를 반복해야 하는 경우가 있지만, 일반적으로 매개 변수의 타입보다는 매개변수의 역할을 설명하는 단어를 사용하는 것이 좋습니다. 



   ● **타입 제약 조건 보다는 역할에 따라, 변수, 매개 변수 및 관련타입(associated types)을 지정하십시오.**
   (Name variables, parameters, and associated types according to their roles, rather than their type constraints.)


   ``` swift
   // 👎🏻
   var string = "Hello" // 🙅🏻‍♀️
   protocol ViewController {
     associatedtype ViewType : View
   }
   class ProductionLine {
     func restock(from widgetFactory: WidgetFactory)
   }
   ```
   이러한 방식(변수 이름 string)으로는 타입 이름을 용도 변경해도 명확함(clarity)과 표현력(expressivity)이 최적화되지 않습니다. 엔티티의 역할을 나타내는 이름을 선택하려고 노력하십시오.

   ``` swift
   // 👍🏻
   var greeting = "Hello"
   protocol ViewController {
     associatedtype ContentView : View
   }
   class ProductionLine {
     func restock(from supplier: WidgetFactory)
   }
   ```
   만약 연관 타입이 프로토콜 제약 조건으로 단단히 바인딩 되어 있어서 프로토콜 이름이 역할인 경우 프로토콜 이름에 `Protocol`을 추가하여 충돌을 피하십시오


   ``` swift
   protocol Sequence {
     associatedtype Iterator : IteratorProtocol
   }
   protocol IteratorProtocol { ... }
   ```

  

 ● **취약한 타입 정보를 보완하여, 파라미터 역할을 명확히 합니다.**
   (Compensate for weak type information to clarify a parameter’s role.)

   특히, 매개변수 타입이 NSObject, Any, AnyObject 또는 Int 또는 String과 같은 기본 타입 인 경우,  사용시점에서 타입 정보와 컨텍스트가 의도를 완전히 전달하지 못 할 수도 있습니다. 이 예에서는 선언이 명확하지만 사용 현장이 모호합니다. 

   ``` swift
   // 👎🏻
   func add(_ observer: NSObject, for keyPath: String)
   
   grid.add(self, for: graphics) 
   ```

   명료함을 회복하려면, **각 약식 매개변수(weakly typed parameter)**앞에 그 역할을 설명하는 명사를 붙힙니다. ( precede each weakly typed parameter with a noun describing its role:)

   ``` swift
   // 👍🏻
   func addObserver(_ observer: NSObject, forKeyPath path: String)
   grid.addObserver(self, forKeyPath: graphics) // clear
   ```



   ### 🔖 Strive for Fluent Usage

   ● **사용 현장을 문법적인 영어 구(phrase) 형식으로 만드는 방법 및 함수 이름을 선호하십시오.** 
   (Prefer method and function names that make use sites form grammatical English phrases.)

   ``` swift
   // 👍🏻
   x.insert(y, at: z)          “x, insert y at z”
   x.subViews(havingColor: y)  “x's subviews having color y”
   x.capitalizingNouns()       “x, capitalizing nouns”
   
   // 👎🏻
   x.insert(y, position: z)
   x.subViews(color: y)
   x.nounCapitalize()
   ```

   호출의 의미에서 가장 중요하지 않을 때 첫번째 두번째 전달 인자 후에 유창성(fluency)이 저하(degrade)될 수 있습니다.

   ``` swift
   AudioUnit.instantiate(
     with: description, 
     options: [.inProcess], completionHandler: stopProgressBar)
   ```


   ● **"make"를 사용하여 팩토리 메소드의 이름을 시작하십시오.** e.g. `x.makeIterator()`
   (Begin names of factory methods with “make”, e.g. x.makeIterator().)

   ● **부작용에 따라 함수 및 메소드 이름을 지으십시오.** 
   (Name functions and methods according to their side-effects)

   ↳○ 부작용이 없는 것들은 명사구(noun phrases)로 읽어야 합니다. e.g. `x.distance(to: y)`, `i.successor()`.
    
   ↳○ 부작용이 있는 것들은 필수 동사구( imperative verb phrases)로 읽어야 합니다. e.g., `print(x)`, `x.sort()`, `x.append(y)`.
    
   ↳○ Mutating/nonmutating메소드 쌍 이름을 일관되게 지정합니다. mutating메소드는 종종 유사한 의미를 같는 nonmutating메소드를 가지지만, 인스턴스를 현재 위치에서 업데이트 하는 대신 새로운 값을 반환합니다. 
    
   ↳↳■ 연산이 동사에 의해 자연스럽게 설명되는 경우에는 mutating메소드에 동사의 명령을 사용하고, "ed"또는 "ing"접미사를 적용하여 해당 nommutating이름을 지정하십시오. 

|   Mutating    |     Nonmutating      |
| :-----------: | :------------------: |
|  `x.sort()`   |   `z = x.sorted()`   |
| `x.append(y)` | `z = x.appending(y)` |

   ↳↳■  동사의 과거분사(대개 "ed"추가)를 사용하여 nommutating이름을 지정하는 것이 좋습니다. 

``` swift
     /// Reverses `self` in-place.
   mutating func reverse()

   /// Returns a reversed copy of `self`.
   func reversed() -> Self
   ...
   x.reverse()
   let y = x.reversed()
```

   ↳↳↳■ 동사에 직접적인 목적어가 있기 때문에, "ed"가 문법적이지 않을 경우, "ing"를 첨부하여 nonmutating이름을 지정하십시오.

 ``` swift
   /// Strips all the newlines from `self`
   mutating func stripNewlines()

   /// Returns a copy of `self` with all the newlines stripped.
   func strippingNewlines() -> String
   ...
   s.stripNewlines()
   let oneLine = t.strippingNewlines()
 ```

   ↳↳↳■ 연산이 자연스럽게 명사에 의해 설명되는 경우, 이 명사를 nonmutating메소드에 명사를 사용하고, "form"접두사를 사용하여 mutating이름을 지정하십시오.

|     Nonmutating      |       Mutating        |
| :------------------: | :-------------------: |
|   `x = y.union(z)`   |   `y.formUnion(z)`    |
| `j = c.successor(i)` | `c.formSuccessor(&i)` |



   ● **Bool메소드 및 프로퍼티의 사용은 그 사용이 nonmutating일 때 수신자에 대한 주장(assertions)으로 해석해야합니다.** e.g. `x.isEmpty`, `line1.intersects(line2)`.
   (Uses of Boolean methods and properties should read as assertions about the receiver when the use is nonmutating, e.g. x.isEmpty, line1.intersects(line2).)

   ● **"무엇"인지를 설명하는 프로토콜은 명사로 읽어야 합니다.** (e.g. `Collection`).
   (Protocols that describe what something is should read as nouns (e.g. Collection).)

   ● **기능을 설명하는 프로토콜은 "`able`,`ible`,`ing`" 접미사를 사용하여 명명해야 합니다.**
   (Protocols that describe a capability should be named using the suffixes able, ible, or ing (e.g. Equatable, ProgressReporting).)

   ● **다른 타입, 프로퍼티, 변수 및 상수의 이름은 명사로 읽어야 합니다.**
   (The names of other types, properties, variables, and constants should read as nouns.)

   


   ### 🔖 Use Terminology Well

   👉🏻 **Term of Art** : 명사(noun) - 특정 분야 또는 직업 내에서 정확하고 특수한 의미를 지닌 단어 또는 구문.

   ● **보다 일반적인 단어가 의미를 전달하는 경우, 모호한 용어는 사용하지 마십시오.**
   (Avoid obscure terms)

"skin(피부)"이 당신의 목적에 부합한다면, "epidermis(표피)"라고 말하지 마십시오.



   ● **Term of Art를 사용한다면, 확립된 의미에 충실하십시오.** (정해진 의미를 고수하십시오.)
   (Stick to the established meaning)

   보다 일반적인 단어보다, 전문 용어를 사용하는 이유는 그렇지 않으면 모호하거나 불명확 할 수 있는 것을 정확하게 표현한다는 것입니다. 따라서, API는 수용된 의미에 따라 엄격하게 용어를 사용해야 합니다.

     ○ **전문가를 놀라게 하지 마십시오** : 이미 그 용어에 익숙한 사람은 우리가 새로운 의미를 발명한 것처럼 보이면 놀라게 될 것이고, 분노를 느낄 것입니다.
    
     ○ **초보자를 혼동하게 하지 마십시오** : 용어를 배우려는 사람은 누구나 웹검색을 하여 그 전통적인 의미를 발견하기 쉽습니다.

   

   ● **약어를 피하십시오.** 
   (Avoid abbreviations.)

   약어, 특히 비표준 약어는 약어가 아닌 형태로 올바르게 번역하는 데 따라 이해도가 달라지기 때문에 사실상 '기술 용어'입니다 (Abbreviations, especially non-standard ones, are effectively 'terms-of-art', because understanding depends on correctly translating them into their non-abbreviated forms.)

   즉, 축약된 표현을 정확하게 축약되지 않은 표현으로 잘 바꾸냐 못바꾸냐에 따라 이해가 달라질 수 있다는 것 입니다.

 

   ● **선례를 받아들이십시오.** 기존 문화를 따른다는 대가를 치르더라도, 초보자를 위한 용어를 최적화하지 마십시오. 
   (Embrace precedent.)

   `List`,와 같은 단순화된 용어를 사용하는 것 보다, 연속적인 데이터 구조 `Array` 의 이름을 지정하는 것이 좋습니다. 초보자가 `List` 의 의미를 더 쉽게 이해 할 수도 있습니다. 배열은 현대 컴퓨팅의 기본 요소입니다. 따라서 모든 프로그래머느 배열이 무엇인지 곧 알게 될 것입니다. 대부분의 프로그래머가 익숙한 용어를 사용하면, 웹 검색 및 질문에 대한 보상이 제공됩니다. 

   수학과 같은 특정 프로그래밍 도메인 내에서 `sin(x)` 와 같이 널리 선행된 용어는 as`verticalPositionOnUnitCircleAtOriginOfEndOfRadiusWithAngle(x)`.와 같은 설명문구보다 바람직합니다.

   이 경우, 선례가 약어를 피하기 위한 지침을 능가한다는 것에 주목하십시오. 완전한 단어가 `sine`,일지라도 `sin(x)` 는 수십년 동안 프로그래머와 수세기 동안 수학자 사이에서 공통적으로 사용되어 왔습니다. 

​      

   아래 내용은 추가적으로 작성하도록 하겠습니다

   ## 📑 Conventions
   ### 🔖 General Conventions
   ### 🔖 Parameters
   ### 🔖 Argument Labels

   ## 📑 Special Instructions

   

   

   


   ## 참고
   1. [API Design Guidelines - swift.org](https://swift.org/documentation/api-design-guidelines/)
   2. [Swift ) API Design Guidelines](https://zeddios.tistory.com/401)
