# Item71. 필요없는 검사 예외 사용은 피하라

# Java 

## 자바 Exception 계층 구조 

<img width="400" alt="자바 에러 구조" src="https://user-images.githubusercontent.com/38216027/113489978-5399e580-9502-11eb-9d98-0f8b591a3ca8.png">

* 자바의 에러는 크게 에러(Error), 검사 예외(Checked Exception), 비검사 예외(UnChecked Exception) 이 세가지로 나뉠 수 있습니다. (사실, 에러도 비검사 예외에 포함되지만 쉬운 설명을 위해 3가지로 분류하였습니다)
  * 에러: 에러(Error)란 컴퓨터 하드웨어의 오동작 또는 고장으로 인해 응용프로그램에 이상이 생겼거나 JVM 실행에 문제가 생겼을 경우 발생합니다. 따라서 개발자가 미리 예측하여 처리할 수 없기 때문에, **애플리케이션에서 오류에 대한 처리를 신경 쓰지 않아도 됩니다.**
  * 검사 예외: 검사 예외(Checked Exception)는 어플리케이션 수행 중에 일어날법한 예외를 검사하고 개발자가 대비하라는 목적으로 사용합니다. **검사 예외는 반드시 예외처리를 해야 합니다.** 따라서 try-catch로 에러를 처리하거나 throw를 이용해 위로 에러를 던져야 합니다(예외 처리를 하지 않으면 컴파일 에러가 발생합니다). 예외 처리를 해야한다는 뜻은 **개발자가 사전에 처리할 수 있는 에러를 뜻합니다.** 대표적인 예외로 `IOException`이 있습니다.
  * 비검사 예외: 비검사 예외(UnChecked Exception)는 Runtime Exception를 상속한 예외로 **프로그래머의 실수로 발생하는 예외들입니다. 비검사 예외는 검사 예외와 달리 예외 처리를 강제하지 않습니다.** 런타임 에러를 발생시키는 에러들로 처리할수 없기 때문에 예외 처리를 강제하지 않습니다. 다만, 비검사예외도 예외 처리를 할 수 있기 때문에 예외 처리만 한다면 런타임 에러는 발생하지 않습니다, 하지만 좋지 않은 방법입니다. 그로 인한 더 큰 sideEffect가 일어날 겁니다.(알고리즘 문제를 `IndexOutOfBoundsException`을 예외처리하여 성공시킨 코드를 한번쯤은 보셨을 겁니다). 대표적인 예외로 `IndexOutOfBoundsException`, `NullPointerException` 등이 있습니다. 

## 예외처리를 해야하는 검사 예외, 잘 쓰면 약 못 쓰면 독.

검사 예외를 제대로 활용하면 API와 프로그램 질을 높일 수 있습니다. 결과를 코드로 반환하거나 비검사 예외를 던지는 것과 달리, **검사 예외는 발생한 문제를 프로그래머가 처리**하여 안정성을 높이게끔 해줍니다. 

하지만 검사 예외를 과도하게 사용하면 오히려 불편한 API가 됩니다. 메서드가 검사 예외를 던질 수 있다고 선언됐다면, 이를 호출하는 코드에서는 catch 블록을 두어 **그 예외를 붙잡아 처리하거나 더 바깥으로 던져 문제를 전파**해야만 합니다.

더구나 검사 예외를 던지는 메서드는 **스트림 안에서 직접 사용할 수 없기 때문에** 자바 8부터는 부담이 더욱 커집니다.

따라서 API를 제대로 사용해도 발생할 수 있는 예외이거나, 프로그래머가 의미 있는 조치를 취할 수 있는 경우라면 이 정도 부담쯤은 받아들일 수 있을 것입니다. 

### 그러나 둘 중 어디에도 해당하지 않는다면 비검사 예외를 사용하는게 좋습니다. 

* 참조한 곳: https://pridiot.tistory.com/54, https://reference-m1.tistory.com/246

<br>

# Objective-C

* Objective-C 에서는 에러가 크게 NSError와 NSException이 있습니다. 
  * NSError(Class Type): NSError는 치명적이지 않은, 개발자가 처리 가능한 에러입니다. NSError에 의해 캡처되도록 설계된 문제는 종종 사용자 오류 (또는 사용자에게 표시 될 수있는 오류)입니다. 
  (따라서 -presentError : 및 NSErrorRecoveryAttempting)에서 종종 복구 될 수 있습니다.
  일반적으로 예상되거나 예측 가능한 오류입니다(예 : 액세스 할 수없는 파일을 열려고 시도하는 경우,
  또는 호환되지 않는 문자열 인코딩 간 변환 시도).

  * NSException: NSException은 잠재적으로 치명적인 **프로그래머 오류**를 위해 설계되었습니다. 이러한 오류는 일부 작업을 수행하기위한 사전 조건을 올바르게 확인하지 않은 애플리케이션의 잠재적 인 결함을 나타 내기 위해 설계되었습니다 (예 : 경계를 벗어난 배열 인덱스에 액세스하려고 시도하거나 변경 불가능한 객체를 변경하려고 시도 함).


```objective-c
NSException *exception = [[NSException alloc] initWithName:NSRangeException reason:@"" userInfo:nil];
```
=> Objective-c 에서는 NSRangeException이 에러 이름(스트링 타입)으로서 존재합니다.

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
## Swift is made to be typeSafe language: 런타임 에러는 처리가 아닌 예방으로

* Swift의 에러는 크게 Error 와 Runtime Exception으로 나눌 수 있겠습니다. 
  * Error(enumType): 처리 가능한 에러들로 **무조건 에러 처리를 해야 합니다** 에러를 throw 하거나 catch 해서 에러를 잡아냅니다. 처리 가능한 에러 이므로 main 함수까지 계속 던지지만 않는다면 런타임에러는 발생하지 않습니다. 
  * Runtime Error: 런타임 예외 (trap, atal error, assertion failure, etc 등이라고도 함)는 프로그래머가 발생시키는 오류입니다. 
  -Ounchecked 빌드를 제외하고 Swift는 **일반적으로 불량 / 정의되지 않은 상태에서 계속 실행하는 대신 프로그램이 크래시가 발생할 것입니다.**
  -Ounchecked 빌드를 제외하고 Swift는 일반적으로 불량 / 정의되지 않은 상태에서 계속 실행하는 대신 프로그램이 충돌 할 것이라고 보장합니다.  
  따라서 개발자가 해결할 수 없는 에러들이라서 개발자는 사전에 옵셔널을 이용해 런타임 에러가 발생하지 않도록 잘 예방해야 합니다. 이런 에러의 종류로는 Forced Unwrapping(==implicit unwrapping), unowned 키워드 사용 남용, overflow되는 정수(숫자) 연산 및 변환, fatalError () s 및 precondition () s 및 assert () s 등으로 인해 발생할 수 있습니다.
  그리고 불행하게도 옵젝씨 예외도 있습니다. 

## 사전에 예방하기

## Optional 사용

```Swift
extension Collection {

    /// Returns the element at the specified index if it is within bounds, otherwise nil.
    subscript (safe index: Index) -> Element? {
        return indices.contains(index) ? self[index] : nil
    }
}
```

## try? 사용

```swift
func fetchData() -> Data? {
    if let data = try? fetchDataFromDisk() { return data }
    if let data = try? fetchDataFromServer() { return data }
    return nil
}
```

참고한곳 : https://stackoverflow.com/questions/25329186/safe-bounds-checked-array-lookup-in-swift-through-optional-bindings/30593673#30593673, https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html



## 클로저에서 throw 사용하기 

* 오직 sync closure, 즉 메소드 에서만 throw, rethrow를 사용할 수 있습니다. 

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
}
catch {
    print ("DONE")
}
```

## @escaping closure

### 매개변수에 Error 추가에서 위로 올림

```swift
completionHandler(data)
failureHandler(error)
```

### Result Type

스위프트 라이브러리에 정의된 Result Type을 사용할 수 있습니다.

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

결론: 스위프트가 짱이다. 옵셔널을 잘 사용하자.

참고한 곳: https://stevenpcurtis.medium.com/throw-inside-swifts-closures-e69fb378500e