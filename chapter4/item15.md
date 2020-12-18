# 클래스와 멤버의 접근 권한을 최소화하라.

## 목차
- 읽기 전에!
- 캡슐화(은닉화)
    - 장점
- 모든 클래스와 멤버의 접근성을 가능한 좁혀라
- public 클래스의 인스턴스 필드는 되도록 public이 아니여야 한다
- public static final

<br>

### 읽기 전에!
이번 아이템의 내용 중 package-orivate에 대한 내용들이 있어 자바의 패키지와 대치되는 스위프트의 프레임워크와 open, public을 묶어 설명하려 하였습니다.

하지만 비슷하게 대치되는 개념이 아니라고 생각하고, 커스텀 프레임워크 또한 일반적으로 자주 쓰이지 않는다고 생각하여 패키지에 대한 내용을 클래스로 바꾸어 fileprivate과 연관되어 작성하도록 하겠습니다.

또한 protected는 대치되는 개념이 없다고 생각해 생략하도록 하겠습니다.

<br>

### 캡슐화 (은닉화)
> 정보 은닉, 혹은 캡슐화라고 하는 이 개념은 소프트웨어 설계의 근간이 되는 원리이다.

캡슐화는 객체의 속성(프로퍼티)과 행위(메서드)를 하나로 묶고 실제 구현 내용 일부를 외부에 감추어 은닉하는것이라고 정의되고 있습니다.

외부에 감추는 방법으로는 접근 제한자를 사용하여 프로퍼티나 메서드에 대한 접근을 제한하도록 설정합니다.

#### 장점
책에서는 장점을 다섯가지로 서술하고 있습니다.
1. 시스템 개발 속도를 높인다.
2. 시스템 관리 비용을 낮춘다.
3. 캡슐화 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다.
4. 소프트웨어 재사용성을 높인다.
5. 큰 시스템을 제작하는 난이도를 낮춰준다.

공통적으로 시스템을 구성하는 컴포넌트를 독립시켜 서로에게 영향을 주지 않음으로써 위에 서술한 장점이 나타나게 된것으로 생각합니다.

<br>

### 모든 클래스와 멤버의 접근성을 가능한 좁혀라
자바에서는 정보 은닉을 위한 다양한 장치를 제공하는데 클래스, 인터페이스, 멤버의 접근성은 해당 요소가 선언된 위치와 접근 제한자로 정해집니다.

정보 은닉의 핵심은 이 **접근 제한자를 제대로 활용하는 것**이 핵심이라고 이야기하고 있습니다.

자바에서 제공하는 접근 제한자는 네 가지가 있습니다. 
접근 범위가 좁은 것부터 순서대로 나열했습니다.
- private
- package-private 
: package는 폴더의 개념입니다. 같은 package 내부에서만 접근이 가능하게 됩니다. 
- protected : package-private의 접근 범위를 포함하고 추가로 하위 클래스에서도 접근이 가능하게 됩니다.
- public

스위프트에서는 다섯가지 접근 제한자를 제공하고 있습니다. 
(자바와 마찬가지로 밑으로 명시된 접근 제한자일수록 접근 범위가 넓어집니다.)
- private : 엔티티(entites)의 사용을 해당 엔티티를 둘러싼 선언과 동일 파일내의 extension으로만 제한합니다. 
- fileprivate : 엔티티의 사용을 자신이 정의된 소스 파일로만 제한합니다.
- internal : 엔티티가 정의된 모듈의 어떤 소스 파일에서도 사용할 수 있지만, 외부 모듈에서는 사용하지 못하도록 합니다.
- public : 엔티티가 정의된 모듈의 어떤 소스 파일에서도 사용가능하고 외부 모듈에서도 사용이 가능합니다.
- open : `public`과 동일하지만 추가로 모듈 외부에서 하위 클래스를 만들고 '재정의'가 가능합니다. <br>
`open`을 명시했다는 의미는 클래스를 상위 클래스로 사용하는 다른 모듈의 코드에 대한 영향을 이미 고려했으며, 그에 따라 클래스 코드를 설계했음을 포함하게 됩니다. 


공개 API 이외의 프로퍼티나 메소드들은 private으로 만드는 것을 권장하고 있습니다.
그런 다음 같은 소스파일내에 다른 클래스(혹은 구조체 등)가 접근해야 하는 멤버에 한해서 fileprivate으로 접근 범위를 확장시켜줍니다.

```swift
// 같은 소스파일 내부
final class ImageViewController: UIViewController {

    fileprivate var imageView: UIImageView!

}

struct ImageProvider {

    let newImage: UIImage

    func updateImage(in viewController: ImageViewController) {
        // 외부에서는 접근하지 못함.
        viewController.imageView.image = newImage
    }
}

```

<br>

### public 클래스의 인스턴스 필드는 되도록 public이 아니여야 한다
> public 가변 필드를 갖는 클래스는 일반적으로 thread safety 하지 않다.
제 생각으로는 `public`이어서가 아니라 '가변' 필드기 때문에 thread unsafety한게 크다고 생각합니다.

물론 `private`으로 선언해주면 직접 접근이 불가능하여 접근에 있어서 까다로워 지는 효과는 있겠지만 완전하게 thread safety하게는 만들어주지 못합니다.
Swift에서 GCD를 이용해 처리를 해주거나 데이터 사본을 사용해서 작업을 해주는게 thread 접근 측면에 있어서 좋다고 생각합니다.

현재 item에서는 접근제한자에 대한 주제를 다루고 있기 때문에 좀더 관심이 있으신 분은 [해당 글](https://uynguyen.github.io/2018/06/05/Working-In-Thread-Safe-on-iOS/)을 읽어보시는것도 좋겠네요.

여기에서는 가변 객체를 참조하는 필드, final이 아닌 필드를 public으로 선언하지 말라고 권장합니다.

다만 예외를 하나 두는데 해당 클래스에 꼭 필요한 구성요소로써의 상수라면 `public static final` 필드로 공개해도 좋다고 합니다.
또한, 기본 타입 값이나 불변 객체를 참조해야 합니다.

스위프트에서는 `static let`으로 상수를 선언해 주면 될 것 같습니다. 자바와 마찬가지로 가변 객체를 참조해서 불이익을 받지 않도록 해야겠습니다.

참조된 객체 자체는 수정이 가능하니까요.

<br>

### public static final
public static final 배열 필드를 두거나 해당 필드에 접근할 수 있는 메소드를 둔다면 수정이 가능하기 때문에 두 가지 해결책을 제시하고 있습니다.

1. public을 private으로 선언하고 접근 가능한 필드에는 불변리스트를 추가하는 것
2. public을 private으로 선언하고 복사본을 반환하는 메서드를 만드는 것

스위프트에서는 굳이 1, 2번의 방법처럼 프로퍼티나 메서드를 추가하기 보다는
```swift 
public static let values = [ ... ]
```

이렇게 접근이 가능하더라도 `let`으로 선언해버려 수정이 불가능 하도록 만들어주면 될 것 같습니다.

<br>

### 참고한 곳
- https://ko.wikipedia.org/wiki/%EC%BA%A1%EC%8A%90%ED%99%94
- https://docs.swift.org/swift-book/LanguageGuide/AccessControl.html#//apple_ref/doc/uid/TP40014097-CH41-ID3
https://zeddios.tistory.com/383
- https://www.avanderlee.com/swift/fileprivate-private-differences-explained/
- http://xho95.github.io/swift/programming/language/grammar/2017/02/28/The-Swift-Programming-Language.html
