# item16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

이따금 인스턴스 필드들을 모아놓는 일 외에는 아무 목적도 없는 퇴보한 클래스를 작성하려 할 때가 있습니다. 

```SWIFT
class Point {
  var x: Int = 0
  var y: Int = 0
}
```

이런 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못합니다(아이템 15). 

`public` 클래스가 프로퍼티를 공개하면 이를 사용하는 클라이언트가 생겨나게 되므로 불변식을 보장할 수 없고, 외부에서 프로퍼티에 접근 할 때 사이드이펙트가 발생할 여지가 있습니다. 또한 객체간의 결합도가 높아지며, 객체의 오용을 야기할 수 있습니다. 

아래는 흔하게 만들 수 있는 캡슐화된 클래스입니다. 프로퍼티들을 `private` 접근제어자로 감싸고, getter와 setter를 두어 프로퍼티를 읽고 쓰게 만들었습니다.

```swift
class Point {
  private var x: Int
  private var y: Int
  
  init(x: Int, y: Int) {
    self.x = x
    self.y = y
  }
  
  public func x() { return x }
  public func y() { return y }
  
  public func setX(x: Int) { self.x = x }
  public func setY(y: Int) { self.y = y }
}
```

`public`인 클래스라면 반드시 이정도의 캡슐화는 해야 합니다. 이렇게 하면 메소드를 두어 내부의 속성들을 언제든지 바꿀 수 있는 유연성을 제공할 수 있게 됩니다.

만약 `let` 으로 선언된 불변 프로퍼티라면 직접 노출할 때의 단점이 조금은 줄어 들지만, 여전히 결코 좋은 생각은 아닙니다. 프로퍼티를 읽을 때 변경하는 등의 부수 작업을 실행할 수 없고, API를 변경하지 않는 이상 저장된 값을 바꿀 수 없습니다.

코드의 변경에 유연하게 대응하기 위해서는 캡슐화 하는 것이 중요합니다. 새로운 기능이 추가되면 객체는 새로운 책임을 갖거나, 원래 설계되지 않은 방식으로의 변경이 일어날 수 있습니다. 이런식으로 새로운 기능을 추가하게되면 추후의 유지보수에 악영향을 미치게 됩니다.

여러 면에서 이것은 API 설계로 귀결되는데, API를 명확하게 정의하면 코드를 캡슐화하고 불필요한 세부 구현 사항을 다른 객체와 공유하는 것을 막을 수 있습니다. 

<br>

### 캡슐화를 하지 않는다면

그럼 다른 객체의 세부 구현사항을 숨겨 캡슐화 하는 것이 왜 이리 중요할까요? 아래 예제를 통해 함께 알아봅시다.

```swift
class ProfileViewController: UIViewController, ProfileHeaderViewDelegate {
  lazy var headerView = ProfileHeaderView()
  
  override func viewDidLoad() {
    super.viewDidLoad()
    headerView.delegate = self
    view.addSubview(headerView)
  }
}
```

위의 코드는 매우 간단해 보입니다만 세부 구현 사항을 노출하게 됩니다. `headerView`가 `private`이 아니기 때문에 `ProfileViewController` 외부에서 `headerView`를 조작 할 가능성이 얼마든지 있어 버그를 발생할 위험을 증가시키게 됩니다.



예를 들어, 사용자가 프리미엄 구독을 한다고 가정하고 외부에서 `headerView`를 `PremiumHeaderView`로 지정한다고 해봅시다.

```swift
func userUnlockedPremiumSubsription() {
  profileViewController.headerView = PremiumHeaderView()
}
```

위의 함수를 실행하면 인스턴스를 교체하기 때문에 `profileViewController`와 `headerView` 간의 델리게이트 관계가 손실되는 문제가 발생합니다. 이는 멈춰버리는 등의 버그를 발생시킬 가능성이 높습니다. 

<br>

### 속성을 감추자

이러한 버그가 발생하지 않게끔 하려면 `headerView` 속성을 접근제어자를 통해 감추면 됩니다.

```swift
class ProfileViewController: UIViewController, ProfileHeaderViewDelegate {
  private lazy var headerView = ProfileHeaderView()
}
```

이제는 `profileViewController.headerView` 로 접근 할 수 없게 됩니다. 하지만 설계상 `headerView`를 `PremiumHeaderView`로 지정하는 함수가 반드시 필요한데, 이는 어떻게 할 수 있을까요?

`headerView` 자체를 노출하는 대신 다음과 같이 `ProfileViewController`가 사용자 모드를 지정할 수 있는 API를 만드는 것입니다.

```swift
extension ProfileViewController {
    enum Mode {
        case standard
        case premium
    }

    func enterMode(_ mode: Mode) {
        switch mode {
        case .standard:
            headerView.applyStandardAppearance()
        case .premium:
            headerView.applyPremiumAppearance()  
        }
    }
}
```

> `headerView`에 `Mode`를 노출하지 않는다는 것에 주목할 만 합니다. 대신 headerView 내부에 별도의 모드를 적용하는 메서드를 구현합니다.(UI 수준의 조정) 이렇게 하면 ProfileViewController와 HeaderView 사이에 결합도를 낮출 수 있습니다.



이제 사용자가 프리미엄 구독을 할 때 실행하는 코드는 다음과 같습니다.

```swift
func userDidUnlockPremiumSubscription() {
    profileViewController.enterMode(.premium)
}
```



이렇게 캡슐화 함으로써 `ProfileHeaderView`가 잘못된 방식으로 사용되는 위험을 줄일 수 있고, 명시적 API를 추가함으로 앱이 계속 발전함에 따라 수반되는 변경사항을 보다 쉽게 적용할 수 있게 되었습니다.

<br>

### 핵심정리

public 클래스는 절대 가변 프로퍼티를 직접 노출해서는 안됩니다. 불변 프로퍼티라면 노출해도 덜 위험하지만 완전히 안심할 수는 없습니다.



### 참고

https://www.swiftbysundell.com/articles/code-encapsulation-in-swift/
