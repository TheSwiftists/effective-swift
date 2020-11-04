# try-finally보다는 try-with-resources를 사용해라

## 목차
- try-finally
- try-with-resources
- 명시적으로 자원을 회수해야하는 상황
    - NotificationCenter
    - FileHandle
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

책에도 나와 있듯이 finally 구문은 자원을 회수(닫는)하는 용도로 주로 쓰이지만, Swift에서는 더이상 필요하지 않은 인스턴스를 자동으로 할당 해제하여 리소스를 확보하는 **ARC**라는 메모리 관리 기법이 존재합니다. <br> 순환 참조의 문제만 조심한다면 자원을 명시적으로 자원을 회수하지 않아도 됩니다.

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

### 명시적으로 자원을 회수해야하는 상황
기본적으로 ARC가 자동으로 할당 해제를 해주지만 명시적으로 자원을 회수해야하는 상황이 종종 있습니다. NotificationCenter 제거, FileHandler의 fileDescriptor 할당해제, dbConnection close(Sqlite)를 예로 들 수 있습니다.

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

#### FileHandler
File handle 객체를 사용해서 파일, 소켓, 파이프 및 디바이스와 관련된 데이터에 엑세스 할 수 있습니다. <br>
파일의 경우에는 읽기, 쓰기, 검색이 가능합니다.
<br>

이니셜라이저를 사용해 소켓에 쓸수있거나 읽을 수 있는 file handle을 반환하게되는데 해당 객체가 file descriptor를 소유하게 되어 닫아줘야한다고 쓰여져 있습니다. <br>

하지만 FileHandle 객체를 사용하나 명시적으로 닫아주지 않아도 되는 경우도 있습니다. <br>
밑의 예시 코드에서 2번째 케이스로 사용된 `init(fileDescriptor fd: Int32, 
closeOnDealloc closeopt: Bool)` 이니셜라이저에서 두번째 인자를 true로 전달해주게되면 자동으로 file descriptor를 닫아준다고 합니다.


```Swift
// case 1
let file = FileHandle(forReadingAtPath: filepath)

    if file == nil {
        print("File open failed")
    } else {
        file?.closeFile()
}

// case 2
let file = FileHandle(fileDescriptor: 'fileDescriptor', closeOnDealloc: true)
}
```

<br>

#### DBConnection
SQLite를 예로 들어보겠습니다. SQLite를 사용하기위해서 connection을 생성하고 이후 사용하지 않을때는 connection을 닫아줘야합니다. <br>
sqlite3_open()를 사용하게 되면 SQLite db handle은 두번째 인자에 sqlite3 객체의 인스턴스에 대한 포인터를 반환합니다. <br>
문서에서는 sqlite3_close()를 사용해 더 이상 연결이 필요하지 않을때 해제하라고 작성되어있습니다.

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

<br> 

### 참고 문서
- [FileHandler 애플공식문서](https://developer.apple.com/documentation/foundation/filehandle)
- [SQLite 공식문서](https://sqlite.org/c3ref/open.html)
- [FileHandle 예제](https://www.techotopia.com/index.php/Working_with_Files_in_Swift_on_iOS_8)
