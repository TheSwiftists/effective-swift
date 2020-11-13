# Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

### Java에서는 

본문에서는 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 인터페이스의 디폴트 메서드를 작성하기 어렵고 때문에 디폴트 메서드로 인해 기존 클라이언트를 망가뜨릴 수 있기 때문에 꼭 필요한 경우가 아니면 인터페이스를 구현하는 쪽으로 생각해 설계하라고 권고하고 있습니다. 

### 그렇다면 Swift에서는 어떨까요?

(아래 내용은 자바의 인터페이스와 디폴트 메서드 구현을 스위프트의 Protocol과 Protocol extension을 통한 Protocol 메서드 구현과 매칭된다고 생각하고 작성하였습니다.)

Delegation에 대한 Apple의 공식문서 ([Cocoa Core Competencies - Delegation](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html))에도 언급되었듯이 Cocoa Touch Framework에서 [Delegation](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html#ID276) 디자인 패턴을 많이 사용하고 있습니다. 그 중의 한 예로 앱을 런칭할 때도 UIAppDelegate를 통해서 앱을 동작시킵니다.

### UIAppDelegate

```swift
protocol UIApplicationDelegate
```

>  먼저 UIAppDelegate란, 앱의 공유 동작을 관리하는데 사용하는 메서드의 모음입니다. 앱 델리게이트(app delegate) 객체가 앱의 공유된 행동을 관리합니다. 앱 델리게이트는 사실상 앱의 루트 객체이며, UIApplication와 함께 몇몇 시스템과의 상호작용을 위해 동작합니다.
>
> 출처: UIApplicationDelegate - Apple developer documentation

- delegate 메서드
  1. `application:willFinishLaunchingWithOptions:` 애플리케이션이 최초 실행될 때 호출되는 메소드입니다.
  2. `application:didFinishLaunchingWithOptions:` 애플리케이션이 실행된 직후 유저의 화면에 보여지기 직전에 호출되는 메소드입니다.
  3. `applicationDidBecomeActive:` 애플리케이션이 Active 상태로 전환된 직후 호출되는 메소드입니다.
  4. `applicationWillResignActive:` 애플리케이션이 Inactive 상태로 전환되기 직전에 호출되는 메소드입니다.
  5. `applicationDidEnterBackground:` 애플리케이션이 백그라운드 상태로 전환된 직후 호출되는 메소드입니다.
  6. `applicationWillEnterForeground:` 애플리케이션이 Active 상태가 되기 직전에, 화면에 보여지기 직전의 시점에 호출되는 메소드입니다.
  7. `applicationWillTerminate:` 애플리케이션이 종료되기 직전에 호출되는 메소드입니다.

그렇다면 이 델리게이트는 언제 사용할까요?

<img src = "https://i.imgur.com/qxfW3Zy.png" width = "70%">

<center>이미지 출처: Apple Developer Documentation</center>

#### 앱의 실행과정

앱의 실행과정을 살펴보면

1. 유저가 명시적으로 또는 시스템이 암시적으로 앱이 실행됩니다.
2. Xcode에서 제공하는 `main()`가 `UIKit's UIApplicationMain(_:_:_:_:)`를 호출합니다.
3. `UIApplicationMain(_:_:_:_:)`는 `UIApplication` 객체를 생성하고 앱 델리게이트를 생성합니다.
   `UIApplicationMain(_:_:_:_:)`: 코코아 터치 프레임워크에서 앱의 라이프 사이클을 시작하는 함수
4. UIKit은 main storyboard 또는 nib 파일으로부터 앱의 기본 인터페이스 nib파일을 로드합니다.
5. UIKit은 앱 델리게이트와 뷰 컨트롤러의 추가적인 메서드를 호출하여 상태 반환을 수행합니다. (UIKit performs state restoration, which calls additional methods of your app delegate and view controllers.) 
6. UIKit이 앱 델리게이트의 `application(_:didFinishLaunchingWithOptions:)` 를 호출합니다.

그렇다면 UIAppDelegate는 어떻게 구현되어있을까요?

#### UIAppDelegate

앱 실행시 처음에 불리는 메서드인 `main()`은 `UIApplicationDelegate`의 extension으로 구현되어있습니다.

```swift
public protocol UIApplicationDelegate : NSObjectProtocol {

    @available(iOS 2.0, *)
    optional func applicationDidFinishLaunching(_ application: UIApplication)

    @available(iOS 6.0, *)
    optional func application(_ application: UIApplication, willFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool

    @available(iOS 3.0, *)
    optional func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool

    @available(iOS 2.0, *)
    optional func applicationDidBecomeActive(_ application: UIApplication)

    @available(iOS 2.0, *)
    optional func applicationWillResignActive(_ application: UIApplication)

    @available(iOS, introduced: 2.0, deprecated: 9.0)
    optional func application(_ application: UIApplication, handleOpen url: URL) -> Bool
// 나머지 구현부 생략
}

extension UIApplicationDelegate {

    public static func main() // main 메서드
}
```



이처럼 제 생각에는 Java와 다르게 Swift에서 extension을 사용해서 Protocol의 메서드를 구현하는 것을 지양하지 않는 것 같습니다. 오히려 Cocoa Touch Framework에서 사용할 만큼 선호하는 방식 같습니다. 다들 분들의 생각은 어떠신지 같이 이야기 나눠보면 좋을 것 같습니다. 



### 참고

1. [Delegation - The Swift Programming Language](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html#ID276)

2. [UIApplicationDelegate -  Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)

3. [About the App Launch Sequence -  Apple Developer Documentation](https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app/about_the_app_launch_sequence)

4. [[iOS] 앱의 생명주기(App Life Cycle)와 앱의 구조(App Structure)](https://jinshine.github.io/2018/05/28/iOS/%EC%95%B1%EC%9D%98%20%EC%83%9D%EB%AA%85%EC%A3%BC%EA%B8%B0(App%20Life%20Cycle)%EC%99%80%20%EC%95%B1%EC%9D%98%20%EA%B5%AC%EC%A1%B0(App%20Structure)/)

5. [Delegation - Cocoa Core Competencies in Documentation Archive](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html)

