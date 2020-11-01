# toString을 항상 재정의하라

## 목차
- toString
- 왜 재정의 해야하나?
- Swift에서의 toString, CustomStringConvertible
    - CustomStringConvertible
    - CustomDebugStringConvertible
    - String(describing:), String(reflecting:)
- 디버깅을 위한 로그 남기기

<br>

## 내용
### toString
Java에서의 toString은 Java의 Object 클래스에 정의되어 있는 메소드를 오버라이드해서 사용할수 있도록 되어있습니다. <br>

System.out.print문 내에서 숫자 타입의 변수나 값은 자동으로 스트링으로 바뀌는데, 이때 컴파일러는 그 클래스의 toString() 메소드를 이용합니다. <br>

toString을 오버라이딩하지 않으면 Object 클래스에 정의되어 있는 toString을 사용하는데, 기본적으로는 **클래스_이름@16진수로_표시한_해시코드**로 표현되어 나타내게 됩니다. <br>

```java
getClass().getName() + '@' + Integer.toHexString(hashCode())
```

<br>

### 왜 재정의 해야하나?
>toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.

<br>

디버깅시 로그 추적을 용이하기 위해 재정의하여 객체가 가진 주요 정보를 모두 반환함에 있습니다. 위의 목차에서 봤듯이 클래스 이름, 해시코드만으로는 디버깅에 필요한 정보를 얻기 힘들기 때문입니다.

<br>

### Swift에서의 toString, CustomStringConvertible

#### CustomStringConvertible
Java에서는 toString이 존재하듯이 Swift에는 비슷한 개념으로  CustomStringConvertible이 존재합니다. <br>

CustomStringconvertible은 프로토콜입니다. 해당 프로토콜을 채택하여 description을 재정의 할 경우 print문을 사용해 출력할때 재정의한 description의 형식이 반환됩니다. <br>

UIViewController에서는 CustomStringConvertible을 채택하지 못하는데 이미 UIViewcontroller가 NSObjectProtocol을 채택하고 있어 따로 채택할 필요없이 바로 description을 재정의하여 사용하면 됩니다. <br>

```switf=
class ViewController: UIViewController {

    override func viewDidLoad() {
        // code
    }
    
    override var description: String {
        // code
    } 

}

```
<br>

- descrption을 재정의 하지 않은 상태로 print문을 실행했을때
```<WebKitCoupanSyncTest.ViewController: 0x7ff769e06480>```

<br>

- description을 재정의 한 후 print문을 실행했을때
   ```swift= 
    class ViewController: UIVewController {
        override func viewDidLoad() {
            super.viewDidLoad()
            print(self)
        }
    
        override var description: String {
            return "여기는 ViewController 입니다."
        }
    }
    
    // 여기는 ViewController 입니다. 출력
    ```
    
#### CustomDebugStringConvertible

위에서 보았던 ```CustomStringConvertible```이외에 ```CustomDebugStringConvertible```이란 프로토콜도 존재합니다. <br>

```CustomStringConvertible```과 차이점은 재정의 해야하는 프로퍼티가```debugDescription``` 로 바뀐다는 것입니다. <br>

조금 더 깊숙히 들어가보면, 구현부에서 차이가 나는 것을 알 수 있습니다. <br>

#### String(describing:), String(reflecting:)
describing, reflecting 모두 어떤 타입이든 인자로 받아 String으로 변환해주는 String 이니셜라이저입니다. <br>

```CustomStringConvertible```, ```CustomDebugStringConvertible``` 프로토콜을 
1. **모두 채택하지 않고** describing, reflecting을 부를 경우 
: Swift stand libaray가 자동으로 지원해줍니다.
2. **둘 중 하나만 채택하고** describing, reflecting을 부를 경우
: 재정의 된 프로퍼티(description 혹은 debugDescription)로 반환 됩니다.
3. **모두 채택하고** describing, reflecting을 부를 경우
: 각각 재정의된 프로퍼티로 반환됩니다.

```Swift=
// Point
struct Point: CustomStringConvertible, CustomDebugStringConvertible {
    let x: Int, y: Int
    
    var debugDescription: String {
        return "(debug : \(x), \(y))"
    }
    
    var description: String {
        return "(description \(x), \(y))"
    }
}

// Print
let p = Point(x: 1, y: 2)
        
let describe = String(describing: p)
let reflect = String(reflecting: p)
        
print(describe) // (description 1, 2)
print(reflect)  // (debug : 1, 2)

```
<br>

### 디버깅을 위한 로그 남기기

#### print, debugPrint

하지만 저렇게 일일이 String 객체를 만들기에는 번거로워, 보통의 경우 debugPrint 사용을 권장하고 있습니다. <br>

> debugPrint(_:separator:terminator:)
Writes the textual representations of the given items most suitable for debugging into the standard output.

```Swift=
debugPrint(1...5)
// Prints "ClosedRange(1...5)"
```

print와 debugPrint또한 description, debugDescription을 기반으로 출력하는 것 같다고 보입니다. <br> 레퍼런스 체크를 진행하지 못했지만 위의 코드에 이어서 print, debugPrint를 사용해보면 동일한 결과가 나오는 것을 확인할 수 있었습니다.

```Swift=
print(p)      // (description 1, 2)
debugPrint(p) // (debug : 1, 2)
```



#### file, function, line
Swift에서는 단순히 객체정보나 값 뿐만 아니라 현재 파일명, 함수명, 라인번호까지 출력할 수 있습니다. <br>


```Swift=
// Example
struct Logger {
    public static func debug(_ msg: Any, file: String = #file, function: String = #function, line: Int = #line) {
        print("[\(dateFormatter.string(from: Date()))] [\(fileName)] [\(funcNmae)(\(line))] : \(msg)")
    }
}
```

이렇게 로그를 남기고 싶은곳에 msg만 인자로 주게되면 콘솔에 ```[시간] [파일명] [함수명(라인)] : msg```로 남게 되어 필요한 정보를 보기 편하게 사용할 수도 있습니다.

<br>

### 마무리

현재 제가 있는 곳에서는 다른 방식으로 로그를 남기지만 같이 일하는 동료와 협의한 포맷을 만들어 descrption을 재정의하고, 해당 프로토콜을 객체마다 전부 채택해줘야 하는 다소 높은 초기비용을 넘어서기만 한다면 좀 더 다양한 정보를 디버깅할때 볼 수 있을것 같습니다 :)  


### 참고한 레퍼런스
- [공식문서: Expression](https://docs.swift.org/swift-book/ReferenceManual/Expressions.html)
- [공식문서: debugPrint](https://developer.apple.com/documentation/swift/1539920-debugprint)
- [공식문서: CustomStringConvertible](https://developer.apple.com/documentation/swift/customstringconvertible)
- customStringConvertible : https://hiddenviewer.tistory.com/249
