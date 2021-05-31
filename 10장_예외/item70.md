# Item 70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라 

이번 아이템에서는 자바, 오브젝티브 씨, 스위프트 각각의 예외 종류를 알아보고 각 언어에서 예외(또는 오류)를 어떻게 사용하는게 좋은지 알아봅니다. 

# Java 

자바는 문제 상활을 알리는 타입(Throwable)으로 검사 예외, 런타임 예외, 에러 이렇게 세 가지를 제공하는데, 언제 무엇을 사용해야 하는지 헷갈려 하는 프로그래머들이 종종 있습니다.
언제나 100% 명확한 건 아니지만 이럴 때 참고하면 좋은 멋진 지침들이 있으니 함께 살펴봅시다. 

우선 자바의 예외 종류에 대해 알아봅시다. 

## Java의 예외 및 에러 종류

<img width="400" alt="자바 에러 구조" src="https://user-images.githubusercontent.com/38216027/113489978-5399e580-9502-11eb-9d98-0f8b591a3ca8.png">

  * 검사 예외: 검사 예외(Checked Exception)는 어플리케이션 수행 중에 일어날법한 예외를 검사하고 개발자가 대비하라는 목적으로 사용합니다. **검사 예외는 반드시 예외처리를 해야 합니다.** 따라서 try-catch로 에러를 처리하거나 throw를 이용해 위로 에러를 던져야 합니다(예외 처리를 하지 않으면 컴파일 에러가 발생합니다). 예외 처리를 해야한다는 뜻은 **개발자가 사전에 처리할 수 있는 에러를 뜻합니다.** 대표적인 예외로 `IOException`이 있습니다.
  
  * 비검사 예외: 비검사 예외(UnChecked Exception)는 Runtime Exception를 상속한 예외로 **프로그래머의 실수로 발생하는 예외입니다. 비검사 예외는 검사 예외와 달리 예외 처리를 강제하지 않습니다.** 런타임 에러를 발생시키는 에러들로 처리할수 없기 때문에 예외 처리를 강제하지 않습니다. 다만, 비검사예외도 예외 처리를 할 수 있기 때문에 예외 처리만 한다면 런타임 에러는 발생하지 않습니다, 하지만 좋지 않은 방법입니다. 그로 인한 더 큰 side effect가 일어날 겁니다.(알고리즘 문제를 `IndexOutOfBoundsException`을 예외처리하여 성공시킨 코드를 한번쯤은 보셨을 겁니다). 대표적인 예외로 `IndexOutOfBoundsException`, `NullPointerException` 등이 있습니다. 
  
  * 에러: 에러(Error)는 보통 JVM이 자원 부족, 불변식 깨짐 등 더 이상 수행을 계속할 수 없는 상황을 나타낼 때 사용합니다. 따라서 개발자가 미리 예측하여 처리할 수 없기 때문에, **애플리케이션에서 오류에 대한 처리를 신경 쓰지 않아도 됩니다.**

## 호출하는 쪽에서 복구하리라 여기지는 상황이라면 검사 예외를 사용하라

검사 예외를 던지면 호출자가 그 예외를 catch로 잡아 처리하거나 더 바깥으로 전파하도록 **강제하게 됩니다.**
따라서 API 설계자는 호출하는 쪽에서 복구하리라 여기지는 상황이라 판단되면 API 사용자에게 검사 예외를 던져주어 그 상황에서 회복해내라고 요구하면 됩니다.

검사 예외는 일반적으로 복구할 수 있는 조건일 때 발생합니다. 따라서 호출자가 예외 상황에서 벗어나는 데 필요한 정보를 알려주는 메서드를 함께 제공하는 것이 중요합니다. 예컨대 쇼핑몰에서 물건을 구입하려는 데 카드 잔고가 부족하여 검사 예외가 발생했다고 해봅시다. 그렇다면 이 예외는 잔고가 얼마나 부족한지를 알려주는 접근자 메서드를 제공해야 합니다.

## 프로그래밍 오류를 나타낼 때는 런타임 예외를 사용하라

비검사 throwable은 두 가지로, 바로 런타임 예외와 에러입니다. 둘 다 동작 측면에서는 다르지 않습니다. 이 둘은 프로그램에서 잡을 필요가 없거나 혹은 통상적으로는 **잡지 말아야 합니다.** 
**프로그램에서 비검사 예외나 에러를 던졌다는 것은 복구가 불가능하거나 더 실행해봐야 득보다는 실이 많다는 뜻이기 때문입니다.** 

이 두가지 비검사 throwable 중에서 **프로그래밍 오류를 나타낼 때는 런타임 예외를 사용해야 합니다.** 런타임 예외의 대부분은 전제조건을 만족하지 못했을 때 발생합니다. 전제조건 위배란 단순히 클라이언트가 해당 API의 명세에 기록된 제약을 지키지 못했다는 뜻입니다. 예컨대 배열의 인덱스는 0에서 '배열 크기 -1' 사이어야 합니다. `ArrayIndexOutOfBoundsException`이 발생했다는 건 이 전제조건이 지켜지지 않았다는 뜻입니다. 

## Error 클래스는 사용하지 말자

위에서 말씀드렸든 Error 는 보통 JVM이 자원 부족, 불변식 깨짐 등 더이상 수행을 계속할 수 없는 상황을 나타낼 때 사용합니다. 따라서 JVM 관련 상황에서만 사용되므로 Error 클래스를 상속해 하위 클래스를 만드는 일은 자제해야 합니다. **다시 말해 우리가 구현하는 비검사 throwable은 모두 런타임 예외(RuntimeException)의 하위 클래스여야 합니다.**

# Objective-C

## Objective-C의 예외 및 에러 종류 

Objective-C 에서는 에러가 크게 NSError와 NSException이 있습니다. 
  * NSError(Class Type): NSError는 치명적이지 않은, 개발자가 처리 가능한 에러입니다. NSError에 의해 캡처되도록 설계된 문제는 종종 사용자 오류 (또는 사용자에게 표시 될 수있는 오류)입니다. 
  (따라서 -presentError : 및 NSErrorRecoveryAttempting)에서 종종 복구 될 수 있습니다.
  일반적으로 예상되거나 예측 가능한 오류입니다(예 : 액세스 할 수없는 파일을 열려고 시도하는 경우,
  또는 호환되지 않는 문자열 인코딩 간 변환 시도).

  * NSException: NSException은 잠재적으로 치명적인 **프로그래머 오류**를 위해 설계되었습니다. 이러한 오류는 일부 작업을 수행하기위한 사전 조건을 올바르게 확인하지 않은 애플리케이션의 잠재적인 결함을 나타내기 위해 설계되었습니다 (잠재적인 결함 예: 경계를 벗어난 배열 인덱스에 액세스하려고 시도하거나 변경 불가능한 객체를 변경하려고 시도함).

```objective-c
NSException *exception = [[NSException alloc] initWithName:NSRangeException reason:@"" userInfo:nil];
```
=> **Objective-c 에서는 NSRangeException이 에러 이름(스트링 타입)으로서 존재합니다.**
<br>=> Swift에서는 런타임 예외가 이름, 즉 객체로서 있지 않습니다. 신기하죠?

### NSException도 자바의 비검사 예외와 마찬가지로 @try - @catch 문으로 해결할 수 있습니다. 하지만 프로그래머의 실수를 이렇게 해결하는 건 좋지 않은 방법입니다.  

```objective-c
@try {
    NSArray *array = [[NSArray alloc] initWithObjects:@"aaa", nil];
    NSString *result = [array objectAtIndex:5];
} @catch (NSException *exception) {
    NSLog(@"%@ is exception name", exception.name);
    NSLog(@"%@ is exception reason", exception.reason);
      
} @finally {
    NSLog(@"finally");
}
```

참고한 곳: https://stackoverflow.com/questions/11100951/nsexception-and-nserror-custom-exception-error, https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html#//apple_ref/doc/uid/TP30001163

# Swift

## Swift의 에러 및 예외 종류 

Swift의 에러 및 예외는 크게 Error 와 Runtime Exception으로 나눌 수 있겠습니다.
사실 런타임 예외는 이름으로서, 즉 객체로서 존재하지도 않지만 Runtime Exception이라고 칭하겠습니다. 

* Error(enum type): **처리 가능한 에러들로 무조건 에러 처리를 해야 합니다(처리 안하면 컴파일 에러 발생).** 호출자는 에러를 throw하거나 catch해서 에러를 잡아냅니다. 처리 가능한 에러이므로 main 함수까지 계속 던지지만 않는다면 런타임 에러는 발생하지 않습니다. 또 자바와 다르게 메소드 선언에 어떤 에러인지 명시하지 않고 throws 만 붙이면 됩니다. 
* Runtime Error: 런타임 예외 (trap, atal error, assertion failure, etc 등이라고도 함)는 프로그래머가 발생시키는 오류입니다. -Ounchecked 빌드를 제외하고 Swift는 일반적으로 불량 / 정의되지 않은 상태에서 계속 실행하는 대신 프로그램이 크래시가 발생할 것입니다. -Ounchecked 빌드를 제외하고 Swift는 일반적으로 불량 / 정의되지 않은 상태에서 계속 실행하는 대신 프로그램이 충돌 할 것이라고 보장합니다.
따라서 개발자가 해결할 수 없는 에러들이라서 개발자는 사전에 옵셔널을 이용해 런타임 에러가 발생하지 않도록 잘 예방해야 합니다. 이런 에러의 종류로는 Forced Unwrapping(==implicit unwrapping), unowned 키워드 사용 남용, overflow되는 정수(숫자) 연산 및 변환, `fatalError()` 및 `precondition()` 및 `assert()` 등으로 인해 발생할 수 있습니다. 그리고 불행하게도 Objective-C 예외도 있습니다.

> NOTE: Ounchecked 빌드란? (by [Lena](https://github.com/TheSwiftists/effective-swift/pull/127/files#r638890293))
* Ounchecked : 안전성(safety)를 성능으로 교환할 의향이 있는 특정 라이브러리, 앱을 위한 특수한 최적화 모드입니다. 컴파일러는 모든 overflow검사와 일부 implicit(암시적) 타입 검사를 제거합니다. 하지만 이는 감지되지 않은 메모리 문제와 정수 overflow를 초래할 수 있기 때문에 일반적으로 사용되지 않습니다. 정수 overflow 및 타입 캐스트와 관련하여 코드가 안전하다는 것을 신중하게 검토 경우에만 이 Ounchecked를 사용합니다.

* 하지만 이제 Ounchecked 빌드는 존재하지 않습니다. 
  * ```Updates: As you can see there is no longer the -Ounchecked mode in Xcode10, instead a new option introduced Optimize for Size. The main difference between the O mode and Osize mode is “When compiling with -O the compiler tries to transform the code so that it executes with maximum performance. However, this improvement in runtime performance can sometimes come with a tradeoff of increased code size. With the new -Osize optimization mode the user has the choice to compile for minimal code size rather than for maximum speed” (swift.org)```

## 처리 가능한 예외 상황이라 여겨지면 Error(enum type)을 사용하세요

위에서 말씀드린 것처럼 **처리 가능한 에러들로 무조건 에러 처리를 해야 합니다(처리 안하면 컴파일 에러 발생).** 따라서 API 설계자는 처리 가능한 에러라면 Error를 사용해 사용자가 에러를 처리하도록 해야 합니다. 

## 처리 불가능한 프로그래밍 오류(자바에서의 미확인 예외)라면 사용할 예외 및 예외가 없습니다. 옵셔널로 예외 상황을 예방하세요.

위에서 언급했듯 스위프트의 경우 런타임 예외 상황을 처리할 수 있는 객체, 즉 예외 및 에러가 존재하지 않습니다. 따라서 nil 런타임 에러나 array index 관련 런타임 에러를 처리할 수 없고 크래시가 발생할 수 밖에 없습니다. 따라서 옵셔널을 활용해 런타임 에러, 즉 크래시가 발생하지 않도록 예방하십시오.

* 옵브젝티브 씨와 비교하자면 옵젝씨의 경우에는 `NSRangeException` 라는 것이 있는데 스위프트에는 대응하는 객체가 없습니다. 런타임 에러가 발생하면 아래처럼 설명으로만 알려줄 뿐입니다. 

<img width="1212" alt="스크린샷 2021-05-27 오후 7 02 22" src="https://user-images.githubusercontent.com/38216027/119807501-27a54a00-bf1e-11eb-89ca-dbd4857ea1a5.png">

* 사실 자바도 **위의 설명처럼 런타임 예외인 경우 잡으면 안된다고 나와 있습니다.** 스위프트는 이렇게 개발자가 런타임 예외를 잡을 수도 있는 기회를 주지 않기 위해 런타임 예외가 객체로 존재하지 않게 했다고 추측됩니다. 

### Optional 사용하기

```Swift
extension Collection {
    /// Returns the element at the specified index if it is within bounds, otherwise nil.
    subscript (safe index: Index) -> Element? {
        return indices.contains(index) ? self[index] : nil
    }
}
```

=> 이처럼 Optional를 잘 활용하면 런타임 에러 상황을 예방할 수 있습니다.

참고한곳 : https://stackoverflow.com/questions/25329186/safe-bounds-checked-array-lookup-in-swift-through-optional-bindings/30593673#30593673, https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html

## 번외: 비동기 상황에서의 에러 던지기

### 클로저에서 throw 사용하기 

* 에러를 던지는 throw, rethrow 키워드는 sync closure 에서만 사용 가능합니다. async closure 는 @escaping 키워드를 이용해 매개변수로 Error를 받아 상위로 에러를 던질 수 있습니다.

```swift
enum BadLuckError: Error {
    case unlucky
}

func incNum(num: Int, completion: (Int) throws -> Void) rethrows {
    // "Network call"
    try completion(num + 1)
}

func callingFunction(num: Int, closure: (Int, (Int) throws -> Void) throws -> Void ) throws {
    try closure(num, { result in
        print (result)
        if result == 13 {
             throw BadLuckError.unlucky
        }
    })
}

do {
    try callingFunction(num: 12, closure: incNum)
} catch {
    print ("DONE")
}
```

### @escaping closure

* 매개변수에 Error 추가해서 위로 올릴 수 있습니다. 

```swift
completionHandler(data)
failureHandler(error)
```

### Result Type

* 스위프트 라이브러리에 정의된 Result Type을 사용할 수 있습니다.

```swift
enum Result<Success, Failure> where Failure: Error {
  case success(Success)
  case failure(Failure)
}
```

```swift
struct Tutorial {
  let title: String
  let author: String
}
enum TutorialError: Error {
  case rejected
}
func feedback(for tutorial: Tutorial) -> Result<String, 
                                                TutorialError> {
  Bool.random() ? .success("published") : .failure(.rejected)
}
func edit(_ tutorial: Tutorial) {
    let result = feedback(for: tutorial)
    switch result {
    case let .success(data):
        print("\(tutorial.title) by \(tutorial.author) was \(data) on the website.")
        
    case let .failure(error):
        print("\(tutorial.title) by \(tutorial.author) was \(error).")
    }
}
let tutorial = Tutorial(title: "What’s new in Swift 5.1", author: "Cosmin Pupăză")
edit(tutorial)
``` 
