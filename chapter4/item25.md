# Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라

### Java에서의 톱레벨 클래스 선언

Java에서는 하나의 소스 파일에 톱레벨 클래스를 여러 개 선언할 수 있습니다. 다만, 이는 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 하는 행위입니다. 한 클래스를 여러 가지로 정의할 수 있으며, 그 중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하느냐에 따라 달라지기 때문입니다. 컴파일러에 어떤 소스를 먼저 건네느냐에 따라 동작이 달라지는 문제가 생길 수 있습니다.

```swift
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}

// 코드 25-1 두 클래스가 한 파일(Utensil.java)에 정의되었다. - 따라 하지 말 것! 
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}

// 코드 25-2 두 클래스가 한 파일(Dessert.java)에 정의되었다. - 따라 하지 말 것!
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}

// 코드 25-3 톱레벨 클래스들을 정적 멤버 클래스로 바꿔본 모습 
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```



### Swift에서의 톱레벨 클래스 선언

Xcode에서 하나의 소스 파일에 톱레벨 클래스를 여러 개 선언할 수 있습니다. 하지만 Java와 달리 다른 파일에 있다고 하더라도 같은 이름의 톱레벨 클래스를 선언할 수 없습니다.  `Invalid redeclaration of 'className'` 라는 오류 메세지를 띄우며 클래스를 생성할 수 없도록 막아놓았기 때문에 같은 이름의 클래스를 생성할 수 없습니다.

<br><br>

**추가적으로, item25와 직접적으로 관련된 내용은 아니지만 두 언어에서 클래스를 어떻게 식별하는지에 대해 조사하였습니다. 아래 내용은 _알아두면 좋은 내용_ 으로 봐주시면 좋을 것 같습니다.**

### 개체를 구분하는 방식, 네임스페이스

* 네임스페이스(Namespace)

  >In [computing](https://en.wikipedia.org/wiki/Computing), a **namespace** is a set of signs (*names*) that are used to identify and refer to objects of various kinds. A namespace ensures that all of a given set of objects have unique names so that they can be easily [identified](https://en.wikipedia.org/wiki/Identifier).
  >
  >-Wikipedia

  > Namespace is a named region of program used to group variable, types and methods.
  > -iOS 9 Programming Fundamentals with Swift

  네임스페이스는 다양한 종류의 개체를 식별하고 참조하기 위해 사용되는 기호(이름)의 집합으로, 지정된 모든 개체 집합의 고유한 이름을 보장하므로 쉽게 식별할 수 있습니다. 즉, 개체를 구분할 수 있는 범위를 나타내는 말로 일반적으로 하나의 이름 공간에서는 하나의 이름이 단 하나의 개체만을 가리킵니다.

네임 스페이스는 다음과 같은 이점이 있습니다.

1. 이름 충돌을 방지합니다.
2. 캡슐화를 제공합니다.

< 예시 >

```swift
class Manny() {
  class Klass { }
}
```

- Manny 안에 Klass 를 효과적으로 숨길 수 있습니다.
  - Manny는 바로 네임스페이스입니다.
  - Manny 내부의 코드는 Klass 를 직접 볼 수 있지만, 외부에서는 직접 볼 수 없습니다. 이를 위해서 ‘dot notation’을 사용한다. e.g.) Manny.Klass
- 네임스페이스는 이처럼 공간을 구획하기 위한 편의를 제공합니다.

**Java**에서 패키지(package) 정의를 통해 네임스페이스를 관리하며 `import` 를 정의하여 패키지에 정의된 클래스를 불러옵니다. 그리고 컴파일시 클래스가 유일한 식별이 가능한지를 확인합니다.

Swift에 대한 내용을 설명하기 전에, Swift 이전에 사용하였던 언어인 **Objective-C**에 대해 간단히 설명하고 넘어가겠습니다.

**Objective-C**에서는 네임스페이스가 없어 다른 라이브러리, 프레임워크와 이름이 충돌되지 않기 위해 **UI**View, **CG**Rect, **CA**Layer 와 같이 Objective-C 클래스에 접두어를 붙여 고유한 이름을 사용하였습니다. 하지만 **Swift**는 모듈 단위의 네임스페이스를 사용하여 이름 충돌 문제를 해결하였고 때문에 대부분의 경우 모듈 접두사가 필요하지 않습니다.

#### Swift에서의 모듈과 프레임워크

- 최상위 레벨의 **네임스페이스**가 바로 **모듈**입니다. 만약 새로 앱을 만들 때 **MyApp** 이라는 이름을 붙이고 최상위 레벨에 `Manny` 클래스를 선언한 경우, 이 클래스의 정확한 이름은 `MyApp.Manny` 가 됩니다.
- **프레임워크** 또한 모듈이며, 하나의 **네임스페이스**라고 할 수 있습니다.
- 예를 들어 코코아의 `Foundation` 프레임워크 내의 `NSString` 클래스는 문자열 선언을 위한 최상위 레벨의 모듈입니다. 파운데이션 프레임워크를 `import`한 경우 `Foundation.NSString` 이 아닌 `NSString` 로 간단하게 클래스를 사용할 수 있습니다.

공식문서에 설명된 모듈(module)의 정의: 

> A *module* is a single unit of code distribution—a framework or application that is built and shipped as a single unit and that can be imported by another module with Swift’s `import` keyword.
> -the swift programming language swift 5.3

### 참고

1. [Access Control(Modules and Source Files) - the swift programming language swift 5.3](https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html) 
2. [Package Manager - the swift programming language swift 5.3](https://swift.org/package-manager/#conceptual-overview)
3. [Namespace - Wikipedia](https://en.wikipedia.org/wiki/Namespace)
4. 『iOS 9 Programming Fundamentals with Swift(스위프트로 하는 iOS 9 프로그래밍)』, Neuburg&Matt, O'Reilly Media(2015)
5. [The Power of Namespacing in Swift](https://www.vadimbulavin.com/the-power-of-namespacing-in-swift/)

### 인용

* 스위프트에는 네임스페이스가 암시되어있습니다.* 

>  Namespacing is implicit in swift, all classes (etc) are implicitly scoped by the module (Xcode target) they are in. no class prefixes needed.
> [*Chris Lattner*의 네임 스페이스에 대한 트윗](https://twitter.com/clattner_llvm/status/474730716941385729) 
> (Chris Lattner는 LLVM과 Clang 컴파일러, **Swift** 프로그래밍 언어의 주 작성자로 가장 잘 알려진 미국의 소프트웨어 엔지니어입니다.)

