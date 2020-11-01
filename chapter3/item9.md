# try-finally보다는 try-with-resources를 사용해라

## 목차
- try-finally
- try-with-resources
- 자원을 회수해야하는 상황
    - NotificationCenter
    - DBConnection
- 마무리

<br>

### try-finally
```java
// java의 try-finally 문
try {
    //run code
} catch () {
    // exception handling
} finally {
    // After try or catch is terminated
}
```

try-catch-finally 구문은 Swift의 ```do-catch``` 구문과 마찬가지로 try문에서 발생한 오류를 catch문에서 잡기 위해 사용합니다.

```swift
// swift의 do-try-catch 문
let fileManager = FileManager()

do {
    let contentsDIR = try fileManager.contentsOfDirectory(atPath: "")
} catch {
    print(error)
    // exception handling
}
```

책에도 나와 있듯이 finally 구문은 자원을 회수(닫는)하는 용도로 주로 쓰이지만, Swift에서는 더이상 필요하지 않은 인스턴스를 자동으로 할당 해제하여 리소스를 확보하는 **ARC**라는 메모리 관리 기법이 존재합니다. 순환 참조의 문제만 조심한다면 자원을 명시적으로 자원을 회수하지 않아도 됩니다.

하지만 Swift에서도 명시적으로 닫아주는 작업도 필요할때가 있는데 이는 하단 목차에서 다루도록 하겠습니다.
<br>

### try-with-resources

```java
try(run code) {
    
} catch {

}
```
위의 try-finally와 비교해서 코드가 더 간결해지고, 오류 로그에서도 try 구문 내에서 발생된 예외가 기록되어 추적하기 쉬워진다는 장점들을 말하고 있습니다.

try문에서 인스턴스를 생성할때, 해당 자원이 `AutoCloseable`을 구현하였으면 자동으로 자원을 회수해줍니다. Swift에서는 위와 마찬가지로 대체되는 것이 있으며 맵핑되는것이 없어 넘어가도록 하겠습니다.

<br>

### 자원을 회수해야하는 상황
기본적으로 ARC가 자동으로 할당 해제를 해주지만 명시적으로 자원을 회수해야하는 상황이 종종 있습니다. NotificationCenter 제거, dbConnection close (Sqlite)를 예로 들 수 있습니다.

보통 `viewDidLoad()`에서 할당을 해주고, 클래스 인스턴스가 해제되기 직전에 불리는 `deinit()`에서 할당을 해제해주는 코드를 작성합니다.

#### NotificationCenter
공식문서에서는 iOS9.0 이나  macOS 10.11 이후 버전에서 앱을 제공한다면 따로 제거해주지 않아도 된다고 나와있지만, 자동으로 제거해주지 않고 명시적으로도 제거 할 수 있습니다.

```Swift
class ViewController: UIViewController {
    // 할당
    viewDidLoad() {
        NotificationCenter.default.addObserver(self, 
                                               selector: #selector(testFunc), 
                                               name: NSNotification.Name(rawValue: "testButton"),
                                               object: nil)
    }
    
    // 해제
    deinit() {
        NotificationCenter.default.removeObserver(self,
                                                  name: NSNotification.Name(rawValue: "testButton"),
                                                  object: nil) 
    }
}

```
<br>

#### DBConnection
SQLite를 예로 들어보겠습니다. SQLite를 사용하기위해서 connection을 생성하고 이후 사용하지 않을때는 connection을 닫아줘야합니다.

```Swift
class ViewController: UIViewController {
    let dbConnection: OpaquePointer?

    viewDidLoad() {
        self.dbConnection = openDatabase()
    }
    
    // connection 해제
    deinit() {
         sqlite3_close(dbPointer)
    }

    // SQLite 연결
    func openDatabase() -> OpaquePointer? {
        var db: OpaquePointer?
        guard let part1DbPath = part1DbPath else {
            print("part1DbPath is nil.")
            return nil
        }
        if sqlite3_open(part1DbPath, &db) == SQLITE_OK {
            print("Successfully opened connection to database at \(part1DbPath)")
            return db
        } else {
            print("Unable to open database.")
            return nil
        }
    }
}

```

<br>

### 마무리

이번 아이템은 Swift와 맵핑되는 부분이 많이 없었지만 Swift에서 명시적으로 자원을 회수해줘야하는 상황이 있을때, 자원관리에 대해서 한번 더 생각해볼수 있는 아이템이었습니다.




