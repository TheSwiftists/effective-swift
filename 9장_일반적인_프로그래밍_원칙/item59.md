# Item 59. 라이브러리를 익히고 사용하라



### 표준 라이브러리 사용의 5가지 이점

#### 1. 표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 여러분보다 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있습니다.

#### 2. 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 됩니다.

#### 3. 따로 노력하지 않아도 성능이 지속해서 개선됩니다.

#### 4. 기능이 점점 많아집니다. 라이브러리에 부족한 부분이 있다면 개발자 커뮤니티에서 이야기가 나오고 논의된 후 다음 릴리스에 해당 기능이 추가되곤 합니다.

#### 5. 개발자가 작성한 코드가 많은 사람에게 낯익은 코드가 됩니다. 자연스럽게 다른 개발자들이 더 읽기 좋고, 유지보수하기 좋고, 재활용하기 쉬운 코드가 됩니다. 

Swift에서 제공하는 표준 라이브러리는 어떤게 있을까요?
Swift Standard Library는 Swift 프로그램을 작성하기 위한 기본 기능 계층(base layer of functionality)을 정의합니다.

> Fundamental data types such as Int, Double, and String
> Common data structures such as Array, Dictionary, and Set
> Global functions such as print(_:separator:terminator:) and abs(_:)
> Protocols, such as Collection and Equatable, that describe common abstractions.
> Protocols, such as CustomDebugStringConvertible and CustomReflectable, that you use to customize operations that are available to all types.
> Protocols, such as OptionSet, that you use to provide implementations that would otherwise require boilerplate code.

### 이런 표준 라이브러리의 기능에도 직접 구현해서 쓰는 경우, 그 이유는?

가능성 있는 이유 중 하나는 아마도 라이브러리에 그런 기능이 있는지 모르기 때문일 것입니다. 메이저 릴리스마다 주목할 만한 수많은 기능이 라이브러리에 추가됩니다. 

Swift의 경우는 매년 발표되는 WWDC 영상과 release note를 확인하면 되겠죠? 🤓

아래는 WWDC What's New in Swift 영상 모음입니다. 
1. [What's New in Swift 2016](https://developer.apple.com/videos/play/wwdc2016/402/)
     : the first major release built with the open source community. Gain insight into the latest changes in Xcode including enhanced migration support to help move your code to Swift 3
2. [What's New in Swift 2017](https://developer.apple.com/videos/play/wwdc2017/402/)
     : new String and improved generics, see how Swift 4 maintains support for your existing Swift 3 code
3. [What's New in Swift 2018](https://developer.apple.com/videos/play/wwdc2018/401/)
     : improvements to build times, code size, and runtime performance
4. [What's New in Swift 2019](https://developer.apple.com/videos/play/wwdc2019/402/)
     : Swift is now the language of choice for a number of major frameworks across all of Apple's platforms, including SwiftUI, RealityKit and Create ML. Join us for a review of Swift 5.0 and an exploration of Swift 5.1, new in Xcode 11. 
5. [What's New in Swift 2020](https://developer.apple.com/videos/play/wwdc2020/10170/)
     : Discover the latest advancements in runtime performance, along with improvements to the developer experience that make your code faster to read, edit, and debug. Find out how to take advantage of new language features like multiple trailing closures. Learn about new libraries available in the SDK, and explore the growing number of APIs available as Swift Packages.


### 라이브러리가 필요한 기능을 충분히 제공하지 못할 경우
이런 경우에도 우선은 라이브러리를 사용하려고 시도해봅시다. 어떤 영역의 기능을 제공하는지 살펴보고 원하는 기능이 아니라고 판단되면 대안을 사용합시다. 이런 경우 대안은 고품질의 서드파티 라이브러리가 될 수 있습니다. 

EX) 대표적인 Third Party 라이브러리: 

1. [Alamofire](https://github.com/Alamofire/Alamofire)
   : Alamofire is an elegant and composable way to interface to HTTP network requests. It builds on top of Apple’s [URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system/) provided by the Foundation framework. At the core of the system are *URLSession* and *URLSessionTask* subclasses.

2. [RxSwift](https://github.com/ReactiveX/RxSwift)
   : Rx's intention is to enable easy composition of asynchronous operations and event/data streams. KVO observing, async operations and streams are all unified under abstraction of sequence. This is the reason why Rx is so simple, elegant and powerful.
   
3. [Kingfisher](https://github.com/onevcat/Kingfisher)
   : powerful, pure-Swift library for downloading and caching images from the web. It will download the image from url, send it to both memory cache and disk cache, and display it in an UIImageView, NSImageView, NSButton or UIButton. When you try to retrieve an image with the same URL later, the image will be retrieved from cache and shown immediately.
   
4. [Lottie](https://github.com/airbnb/lottie-ios)
   : ince creating animations by hand using UIView or CoreGraphics animations can prove to be quite challenging and time-consuming, Lottie provides us with a perfect tool to incorporate desinger's creations into our applications. Lottie loads and renders animations and vectors exported in the bodymovin JSON format.
   
5. [SnapKit](https://github.com/SnapKit/SnapKit)
   : SnapKit is an Auto Layout library that simplifies writing auto layout in code with a minimal amount of code needed without losing readability. It is type safe by design to help you avoid programming errors while coding your user interface.
   
6. [Realm](https://realm.io/)
   : Apple’s Core Data can probably fulfill most of your persistence needs, but it was always likely that something would come and do it better. Realm is one such alternative, providing a framework that is faster and easier to work with. 

### 핵심 정리

> 바퀴를 다시 발명하지 맙시다. 아주 특별한 나만의 기능이 아니라면 누군가 이미 라이브러리 형태로 구현해놓았을 가능성이 큽니다. 그런 라이브러리가 있다면, 쓰면 됩니다. 있는지 잘 모르겠다면 찾아보는 걸 권장합니다. 일반적으로 라이브러리의 코드는 여러분이 직접 작성한 것보다 품질이 좋고, 점차 개선될 가능성이 큽니다. 여러분의 실력을 폄하하는 게 아닙니다. 코드 품질에도 규모의 경제가 적용됩니다. 즉, 라이브러리 코드는 개발자 각자가 작성하는 것보다 주목을 훨씬 많이 받으므로 코드 품질도 그만큼 높아집니다.
-출처: 이펙티브 자바

### 출처
1. 이펙티브 자바, 
2. [Swift Standard Library](https://developer.apple.com/documentation/swift/swift_standard_library)
3. [Top 10 Most Useful iOS Libraries in 2020](https://infinum.com/the-capsized-eight/top-10-most-useful-iOS-libraries)